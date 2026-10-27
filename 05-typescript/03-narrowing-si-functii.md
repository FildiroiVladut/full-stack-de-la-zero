# Narrowing și funcții

Cum lucrează TypeScript cu union-urile în cod real: verifici tipul, iar compilatorul ține minte rezultatul verificării. Plus tipurile de funcții, `void` și `never`, și diferența dintre `any` și `unknown`, una dintre cele mai frecvente întrebări de interviu pe TypeScript.

## Narrowing cu `typeof` și verificări simple

**Ce e.** Când o valoare are un tip union (`number | string`), TypeScript te lasă să folosești doar ce au în comun toate variantele. După un `if` care verifică tipul, compilatorul **restrânge** (_narrowing_) tipul în interiorul ramurii.

**Exemplu minim**
```ts
function formatPrice(value: number | string): string {
  if (typeof value === "number") {
    return value.toFixed(2);  // here value is number
  }
  return value.trim();        // here value is string
}
```

**Cum îl vezi în cod real.** Merge cu orice verificare pe care o înțelege TypeScript: `typeof`, `=== null`, `Array.isArray`, `instanceof`, truthiness (`if (user)`), `in`.

**Capcana.** `typeof null === "object"`. Un `if (typeof x === "object")` nu exclude `null`, așa că adaugi `&& x !== null`.

## Narrowing cu `in`

**Ce e.** `"camp" in obiect` verifică dacă obiectul are acel câmp, iar TypeScript restrânge tipul după rezultat.

**Exemplu minim**
```ts
type Court = { kind: "court"; surface: "clay" | "hard" };
type Room = { kind: "room"; seats: number };

function describe(resource: Court | Room) {
  if ("seats" in resource) return `Room for ${resource.seats}`;
  return `Court (${resource.surface})`;
}
```

Merge, dar când controlezi tu forma datelor există o variantă mai bună: un câmp comun care spune explicit ce variantă e. Vezi fișa următoare.

## Discriminated unions: modelezi stări

**Ce e.** Un union de obiecte care au toate un câmp comun cu valori literale diferite (de obicei `status`, `kind` sau `type`). Când verifici câmpul ăla, TypeScript știe exact ce alte câmpuri există.

**De ce contează.** E cel mai util tipar TypeScript pentru logica de business. Face ca stările imposibile să fie imposibil de scris, de exemplu o factură „draft” care are dată de plată.

**Exemplu minim** (P2 Invoicer)
```ts
type InvoiceState =
  | { status: "draft" }
  | { status: "sent"; sentAt: string; dueDate: string }
  | { status: "paid"; sentAt: string; paidAt: string }
  | { status: "cancelled"; reason: string };

function describe(inv: InvoiceState): string {
  switch (inv.status) {
    case "draft":     return "Not sent yet";
    case "sent":      return `Due on ${inv.dueDate}`;  // dueDate exists only here
    case "paid":      return `Paid on ${inv.paidAt}`;
    case "cancelled": return `Cancelled: ${inv.reason}`;
    default: {
      const unreachable: never = inv; // compile error if you forget a case
      return unreachable;
    }
  }
}
```

**Cum îl vezi în cod real.** Statusuri de comenzi și facturi, stări de request (`{ status: "loading" } | { status: "success"; data } | { status: "error"; error }`), acțiuni în reducer-e, evenimente.

**Capcana.** Varianta „plată” pe care o generează des AI-ul: un singur tip cu toate câmpurile opționale (`{ status: string; paidAt?: string; dueDate?: string; reason?: string }`). Compilează, dar permite orice combinație fără sens și te obligă la `?.` peste tot. Când vezi multe câmpuri opționale care depind de un status, propune un discriminated union.

<details>
<summary>Întrebare de interviu: Ce e un discriminated union și la ce folosește?</summary>

E un union de tipuri obiect care au un câmp comun, discriminantul, cu valori literale diferite, de exemplu `status: "draft" | "sent" | "paid"`. Când verific discriminantul într-un `if` sau `switch`, TypeScript restrânge automat tipul și știe ce câmpuri are fiecare variantă. Îl folosesc ca să modelez stări, ca stările imposibile să nu poată fi construite. Cu un `default` care atribuie valoarea unei variabile `never`, compilatorul mă anunță dacă uit să tratez o variantă nouă.
</details>

## Tipuri de funcții

**Ce e.** Parametrii se tipează întotdeauna. Tipul de return e dedus, dar e bine să-l scrii la funcțiile exportate: devine un contract, iar o greșeală din interior apare ca eroare exact acolo. O funcție poate fi și ea un tip, ceea ce e util pentru callback-uri și props.

**Exemplu minim**
```ts
type PriceCalculator = (subtotalCents: number) => number;

const withVat: PriceCalculator = (cents) => Math.round(cents * 1.21); // cents: number, inferred

type BookCardProps = {
  title: string;
  onAdd: (key: string) => void;   // a callback prop
};
```

**Cum îl vezi în cod real.** Props de tip callback în React (`onClick`, `onChange`, `onSubmit`), metode în services din Nest cu return type explicit (`async findOne(id: number): Promise<Invoice>`).

## `void` și `never`

**Ce e.**
- `void` înseamnă că funcția nu întoarce nimic util (ca în C).
- `never` înseamnă că funcția **nu se termină niciodată normal**: fie aruncă mereu o eroare, fie rulează la infinit. E și tipul unei valori care „nu poate exista”, cum e `inv` din `default`-ul de mai sus, după ce au fost tratate toate cazurile.

**Exemplu minim**
```ts
function logEvent(name: string): void {
  console.log(name);
}

function fail(message: string): never {
  throw new Error(message);
}
```

**Capcana.** Dacă vezi `never` într-o eroare de compilare, de obicei e un semn că ai restrâns tipul atât de mult încât nu mai rămâne nimic, adică ai ajuns într-o ramură pe care compilatorul o consideră imposibilă. Uită-te la condițiile de deasupra.

## `any` vs `unknown`

**Ce e.** Ambele înseamnă „nu știu tipul”, dar se comportă invers:
- `any` **dezactivează** verificarea. Poți face orice cu valoarea și nimic nu e semnalat.
- `unknown` **forțează** verificarea. Nu poți face nimic cu valoarea până nu demonstrezi ce e.

**Exemplu minim**
```ts
const loose: any = JSON.parse('{"title":"Dune"}');
loose.titel.toUpperCase(); // compiles; crashes at runtime (typo: titel)

const raw: unknown = JSON.parse('{"title":"Dune"}');
raw.title;                 // error: 'raw' is of type 'unknown'
if (typeof raw === "object" && raw !== null && "title" in raw && typeof raw.title === "string") {
  console.log(raw.title.toUpperCase()); // ok: proven safe
}
```

**De ce contează.** `any` se propagă: orice valoare obținută dintr-un `any` e tot `any`, așa că o singură scăpare poate dezactiva verificarea pe un lanț întreg de cod. `unknown` e tipul corect pentru date care vin din afară, până le validezi.

**Cum îl vezi în cod real.** În `catch (err)` eroarea are tipul `unknown` în modul strict, de aceea vezi des `if (err instanceof Error) console.log(err.message)`. Iar validarea manuală de mai sus e lungă și urâtă; în practică o faci cu zod, în [05-validare-cu-zod.md](05-validare-cu-zod.md).

**Capcana.** AI-ul pune `any` când nu reușește să facă tipurile să se potrivească. Fiecare `any` dintr-un diff merită întrebarea: care e tipul real?

<details>
<summary>Întrebare de interviu: Care e diferența dintre any și unknown?</summary>

Amândouă pot primi orice valoare. `any` oprește complet verificarea, deci pot accesa orice proprietate sau apela orice, iar erorile apar abia la runtime. În plus, se propagă la tot ce derivă din valoare. `unknown` mă obligă să restrâng tipul înainte să folosesc valoarea, prin `typeof`, `instanceof` sau o validare cu zod. Folosesc `unknown` pentru date externe și pentru erorile din `catch`, iar `any` îl evit, eventual cu o regulă de lint care îl interzice.
</details>

## Mai departe

- [TypeScript Handbook — Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TypeScript Handbook — More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
