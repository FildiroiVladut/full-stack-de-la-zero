# JS-ul pe care îl vezi în cod real

Fișierul ăsta adună tiparele care apar în aproape orice fișier dintr-un proiect modern, plus trei subiecte la care greșesc chiar și dezvoltatori cu experiență: banii, datele calendaristice și formatarea. Le vei întâlni direct în P2 (facturi) și P3 (rezervări).

## Destructuring în parametri, cu valori implicite

**Ce e.** În loc să primești un obiect și să-i citești câmpurile în corpul funcției, le extragi direct în lista de parametri. Poți pune și valori implicite.

**De ce contează.** Așa arată fiecare componentă React (props) și fiecare funcție care primește opțiuni.

**Exemplu minim**
```js
function formatBook({ title, author = "Unknown author", year }) {
  return `${title} — ${author}${year ? ` (${year})` : ""}`;
}
console.log(formatBook({ title: "Dune", author: "Frank Herbert", year: 1965 }));
console.log(formatBook({ title: "Anonymous notes" }));
```

**Cum îl vezi în cod real.**
```tsx
function BookCard({ book, onAdd, compact = false }: BookCardProps) { /* ... */ }
```

**Capcana.** Valoarea implicită se aplică doar pentru `undefined`, nu și pentru `null`. Dacă API-ul trimite `"author": null`, rămâi cu `null`.

## Spread pentru update imutabil

**Ce e.** Imutabil înseamnă că nu modifici obiectul existent, ci creezi unul nou cu schimbarea aplicată. La obiecte imbricate copiezi fiecare nivel pe care îl schimbi.

**De ce contează.** React detectează schimbările comparând referințele: dacă e același obiect, consideră că nu s-a schimbat nimic. Același tipar apare în reducer-e (`useReducer`, Redux) și în orice cod care vrea să evite efectele secundare.

**Exemplu minim**
```js
const invoice = {
  id: 7,
  client: { name: "Acme", city: "Cluj" },
  items: [{ id: "a", qty: 1 }, { id: "b", qty: 2 }],
};

const updated = {
  ...invoice,
  client: { ...invoice.client, city: "Iasi" },      // change a nested field
  items: invoice.items.map((it) =>                   // change one item in an array
    it.id === "b" ? { ...it, qty: 3 } : it,
  ),
};
// invoice is untouched; unchanged parts (items[0]) are shared, not copied
```

**Cum îl vezi în cod real.** Adăugare: `[...items, newItem]`. Ștergere: `items.filter((it) => it.id !== id)`. Modificare: `map` ca mai sus. Le vei vedea zilnic în `setState`.

**Capcana.** `const copy = { ...invoice }; copy.client.city = "Iasi"` modifică și originalul, pentru că spread-ul copiază doar primul nivel. Când chiar ai nevoie de o copie completă, independentă, folosești `structuredClone`.

## `??` vs `||`, și optional chaining în lanț

**Ce e.** Pe `??` și `?.` le știi deja. Ce trebuie reținut e diferența dintre `??` și `||`: `||` alege dreapta pentru **orice** valoare falsy, inclusiv `0` și `""`, pe când `??` o alege doar pentru `null` și `undefined`.

**Exemplu minim**
```js
const qty = 0;
console.log(qty || 1); // 1 — wrong: 0 was a real quantity
console.log(qty ?? 1); // 0 — correct

const city = invoice.client?.address?.city ?? "—";
const firstAuthor = book.author_name?.[0] ?? "Unknown";
```

**Capcana.** Codul mai vechi, și uneori AI-ul, folosește `||` pentru valori implicite. La cantități, prețuri, discount-uri și indexuri, asta e bug. La review, fiecare `|| valoare` merită o privire.

## Object.entries, fromEntries, groupBy

**Ce e.** Trei funcții care te lasă să tratezi obiectele ca liste:
- `Object.entries(obj)` → array de perechi `[key, value]`;
- `Object.fromEntries(pairs)` → invers;
- `Object.groupBy(arr, fn)` → grupează elementele unui array după o cheie.

**Exemplu minim** (P2: totaluri pe status)
```js
const invoices = [
  { id: 1, status: "paid", totalCents: 120000 },
  { id: 2, status: "sent", totalCents: 45050 },
  { id: 3, status: "paid", totalCents: 30000 },
];
const byStatus = Object.groupBy(invoices, (inv) => inv.status);
const totals = Object.fromEntries(
  Object.entries(byStatus).map(([status, list]) => [
    status,
    list.reduce((sum, inv) => sum + inv.totalCents, 0),
  ]),
);
console.log(totals); // { paid: 150000, sent: 45050 }
```

**Capcana.** `Object.groupBy` e relativ nou. Pe Node LTS și în browserele actuale merge, dar în cod mai vechi vei vedea aceeași grupare făcută cu `reduce`. Nu te mira de asta.

## Metodele de array care nu modifică originalul

**Ce e.** Variante imutabile pentru metodele care modificau pe loc: `toSorted`, `toReversed`, `toSpliced`, `with(index, value)`. Plus `at(-1)`, care ia ultimul element fără `arr[arr.length - 1]`.

**Exemplu minim**
```js
const prices = [5, 1, 3];
const sorted = prices.toSorted((a, b) => a - b); // [1, 3, 5]; prices is still [5, 1, 3]
const last = sorted.at(-1);                       // 5
const fixed = prices.with(0, 4);                  // [4, 1, 3]
```

**Cum îl vezi în cod real.** `[...arr].sort()` și `toSorted()` fac același lucru. Primul e mai vechi și mai răspândit, al doilea e mai clar. Le vei vedea pe amândouă.

## Banii se țin în întregi

**Ce e.** Numerele din JavaScript sunt în virgulă mobilă (IEEE 754, același `double` din C), iar unele zecimale nu se pot reprezenta exact în binar. De aceea banii se țin **în cea mai mică unitate, ca întregi**: bani, nu lei; cenți, nu euro.

**De ce contează.** P2 Invoicer face exact asta. La facturi, o diferență de un ban între total și suma liniilor e un bug pe care contabilul îl găsește imediat.

**Exemplu minim**
```js
console.log(0.1 + 0.2);           // 0.30000000000000004
console.log(Math.round(1.005 * 100)); // 100, not 101 — 1.005 * 100 is 100.49999999999999

// store and compute in cents (bani)
const items = [
  { unitPriceCents: 1240, quantity: 2.5 }, // 12.40 lei × 2.5 kg
  { unitPriceCents: 4590, quantity: 1 },
];
const subtotal = items.reduce(
  (sum, it) => sum + Math.round(it.unitPriceCents * it.quantity), 0); // round per line
const vat = Math.round(subtotal * 0.21);
console.log(subtotal, vat, subtotal + vat); // 7690 1615 9305
```

**Cum îl vezi în cod real.** Coloane ca `total_cents integer` în baza de date, conversia la lei doar la afișare și totalurile calculate **pe server**, niciodată acceptate gata calculate de la client.

**Capcana.** Dacă transformi din lei cu zecimale în bani cu `Math.round(lei * 100)` *după* ce ai făcut calcule cu zecimale, eroarea e deja acolo. Convertește o singură dată, la intrare, și apoi lucrează doar cu întregi. Regula de rotunjire (pe linie sau pe total) e o decizie de business, așa că o scrii explicit.

<details>
<summary>Întrebare de interviu: Cum ai stoca prețuri într-o aplicație?</summary>

Ca întregi în cea mai mică unitate monetară, adică bani sau cenți, plus moneda separat. Numerele în virgulă mobilă nu reprezintă exact multe zecimale, iar erorile se acumulează la adunări și înmulțiri. Fac toate calculele în întregi, rotunjesc explicit acolo unde cere regula de business, de exemplu pe fiecare linie de factură, și formatez în lei doar la afișare. În Postgres pot folosi și tipul `numeric`, care e exact.
</details>

## Date, ore și fusuri orare

**Ce e.** Un `Date` din JavaScript e intern doar un număr: milisecunde de la 1 ianuarie 1970, **UTC**. Fusul orar intervine doar când îl citești sau îl afișezi. Formatul standard de transport e string-ul ISO 8601 cu `Z` la final (`"2026-11-03T18:00:00.000Z"`), unde `Z` înseamnă UTC.

**De ce contează.** P3 Booking e o aplicație de rezervări: un teren rezervat „la 20:00” trebuie să însemne aceeași oră pentru server (care pe Render rulează în UTC), pentru baza de date și pentru utilizatorul din București.

**Exemplu minim**
```js
const slot = new Date("2026-11-03T18:00:00Z");
console.log(slot.toISOString()); // "2026-11-03T18:00:00.000Z" — what you store and send
console.log(
  new Intl.DateTimeFormat("ro-RO", {
    dateStyle: "full", timeStyle: "short", timeZone: "Europe/Bucharest",
  }).format(slot),
); // "marți, 3 noiembrie 2026 la 20:00" — what you show
```

**Cum îl vezi în cod real.** Regula de aur: **stochezi și transmiți în UTC (ISO), afișezi în fusul utilizatorului.** În Postgres folosești coloane `timestamptz`. Pentru calcule pe date (plus 7 zile, începutul săptămânii) proiectele folosesc de obicei o bibliotecă precum `date-fns`. Noul API standard pentru date, `Temporal`, e în curs de adoptare și îl vei întâlni tot mai des.

**Capcana.** Două, amândouă reale:
- `new Date("2026-11-03")` (doar data) e interpretat ca **UTC**, dar `new Date("2026-11-03T10:00")` (fără `Z`) e interpretat în **fusul local** al mașinii. Același cod dă rezultate diferite pe laptopul tău și pe server.
- Ora de vară. În România, pe 29 martie 2026, ceasul sare de la 03:00 direct la 04:00, deci ora 03:30 nu există în ziua aceea. Un sistem de rezervări care face calcule cu „ora locală + 1” va greși exact atunci.

<details>
<summary>Întrebare de interviu: Cum lucrezi cu date și fusuri orare într-o aplicație web?</summary>

Stochez momentele în timp în UTC (în Postgres ca `timestamptz`) și le transmit între frontend și backend ca string-uri ISO 8601 cu `Z`. Fusul orar îl aplic doar la afișare, în frontend, cu `Intl.DateTimeFormat` sau cu o bibliotecă. Evit parsarea string-urilor fără fus orar, pentru că depinde de mașina pe care rulează codul, și sunt atent la trecerea la ora de vară când fac calcule pe ore locale.
</details>

## Intl.NumberFormat pentru monede

**Ce e.** API-ul standard care formatează numere după regulile unei limbi: separatorul de mii, cel zecimal, simbolul monedei și poziția lui.

**Exemplu minim**
```js
const ron = new Intl.NumberFormat("ro-RO", { style: "currency", currency: "RON" });
const eur = new Intl.NumberFormat("en-GB", { style: "currency", currency: "EUR" });
console.log(ron.format(9305 / 100)); // "93,05 RON"
console.log(eur.format(1234.5));      // "€1,234.50"
```

**Cum îl vezi în cod real.** O funcție `formatMoney(cents, currency)` definită o singură dată, într-un fișier de utilitare, și folosită peste tot în UI.

**Capcana.** Formatarea făcută de mână cu `toFixed(2) + " lei"`, cum ai făcut la bonul fiscal din ziua 1, e în regulă la un exercițiu, dar nu respectă separatorul de mii și nu schimbă formatul pentru alți clienți. Iar `toFixed` întoarce string, deci după el nu mai faci calcule.

## Mai departe

- [MDN — Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)
- [MDN — Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [MDN — Object.groupBy()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy)
