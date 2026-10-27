# Props și state

Cele două surse de date ale unei componente: props vin de la părinte, state e memoria proprie a componentei. Aproape orice bug de React începe cu o confuzie între ele sau cu un state ținut în locul greșit.

## Props tipate

**Ce e.** Props sunt argumentele unei componente. React le primește ca un singur obiect, pe care îl destructurezi. În TypeScript le descrii cu un `type`, ca editorul să-ți spună imediat dacă ai uitat unul sau ai trimis un tip greșit. Props sunt read-only: componenta nu le modifică niciodată.

**De ce contează.** Tipul props-urilor e contractul componentei. La review te uiți întâi la el: îți spune ce face componenta fără să citești corpul.

**Exemplu minim**

```tsx
type InvoiceBadgeProps = {
  status: "draft" | "sent" | "paid" | "overdue";
  size?: "sm" | "md"; // optional
};

export function InvoiceBadge({ status, size = "md" }: InvoiceBadgeProps) {
  return <span className={`badge badge-${size}`}>{status}</span>;
}
```

**Cum îl vezi în cod real.** Două tipare care apar des:

- `children: React.ReactNode` pentru componente care „învelesc” alt conținut (vezi [05-hooks-si-compozitie](05-hooks-si-compozitie.md));
- `React.ComponentProps<"button">` când componenta ta e un `<button>` îmbunătățit și vrei să accepte toate atributele unui buton normal.

**Capcana.** `props: any` sau props netipate. AI-ul pune uneori `any` ca să scape de o eroare TypeScript. Dacă vezi asta la review, cere tipul corect: `any` dezactivează exact verificarea de care ai nevoie.

<details>
<summary>Întrebare de interviu: Care e diferența între props și state?</summary>

Props vin din afară, de la părinte, și sunt read-only pentru componentă. State e intern, deținut de componentă, și se schimbă prin setter. Schimbarea oricăruia declanșează un re-render. Regula practică: dacă o valoare poate fi primită de la părinte sau calculată din altele, nu e state.
</details>

## useState

**Ce e.** Un hook (o funcție specială React, cu nume care începe cu `use`) care dă componentei o valoare care supraviețuiește între render-uri și un setter care o schimbă. Apelul setter-ului îi cere lui React un re-render cu valoarea nouă.

**De ce contează.** E memoria componentei. Fără state, o componentă poate doar să afișeze ce primește.

**Exemplu minim**

```tsx
import { useState } from "react";

export function QuantityStepper() {
  const [quantity, setQuantity] = useState(1);

  function addTwo() {
    setQuantity((q) => q + 1); // functional update: uses the latest value
    setQuantity((q) => q + 1);
  }

  return <button onClick={addTwo}>Quantity: {quantity}</button>;
}
```

**Cum îl vezi în cod real.** Forma `setX(prev => ...)` (functional update) apare când valoarea nouă depinde de cea veche. Forma simplă `setX(valoare)` apare când doar înlocuiești valoarea, de exemplu textul dintr-un input.

**Capcana.** State-ul nu se schimbă pe loc. După `setQuantity(quantity + 1)`, variabila `quantity` are tot valoarea veche până la următorul render. Așa că `setQuantity(quantity + 1)` apelat de două ori adaugă 1, nu 2. React grupează actualizările (batching) și face un singur render.

<details>
<summary>Întrebare de interviu: De ce `setCount(count + 1)` apelat de două ori la rând crește doar cu 1?</summary>

Pentru că `count` e o constantă în render-ul curent, deci ambele apeluri calculează `0 + 1`. React grupează actualizările și re-randează o singură dată. Dacă valoarea nouă depinde de cea veche, folosesc forma funcțională `setCount(c => c + 1)`: React aplică funcțiile în ordine, pe valoarea cea mai recentă.
</details>

## Evenimente

**Ce e.** Atribute ca `onClick`, `onChange`, `onSubmit`, cărora le dai o funcție. React o apelează cu un obiect de eveniment. În TypeScript, evenimentele au tipuri precum `React.ChangeEvent<HTMLInputElement>`.

**De ce contează.** Event handlers sunt locul natural pentru tot ce declanșează utilizatorul: salvare, ștergere, request POST. Mai târziu vei vedea că multe `useEffect`-uri greșite sunt de fapt cod care trebuia pus într-un handler.

**Exemplu minim**

```tsx
export function DeleteClientButton({ onDelete }: { onDelete: () => void }) {
  function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
    e.stopPropagation(); // don't trigger the row's click handler
    onDelete();
  }

  return <button onClick={handleClick}>Delete</button>;
}
```

**Cum îl vezi în cod real.** Handlers scurți scriși inline (`onClick={() => setOpen(true)}`) și handlers cu logică extrași în funcții numite `handleX`. Ambele variante sunt în regulă.

**Capcana.** `onClick={handleDelete()}`, cu paranteze, *apelează* funcția la fiecare render, în loc să o dea lui React. Corect e `onClick={handleDelete}` sau, cu argumente, `onClick={() => handleDelete(id)}`.

<details>
<summary>Întrebare de interviu: Care e diferența între `onClick={fn}` și `onClick={fn()}`?</summary>

`onClick={fn}` îi dă lui React o referință la funcție, iar React o apelează la click. `onClick={fn()}` apelează funcția imediat, în timpul render-ului, și trimite rezultatul ca handler. De obicei asta înseamnă efecte la fiecare render sau chiar o buclă infinită dacă funcția schimbă state. Pentru argumente folosesc o arrow function: `onClick={() => fn(id)}`.
</details>

## Input controlat

**Ce e.** Un input a cărui valoare vine din state (`value={...}`) și se schimbă doar prin `onChange`. React e singura sursă de adevăr pentru ce scrie în câmp.

**De ce contează.** Așa poți valida în timp ce utilizatorul tastează, poți formata (de exemplu să permiți doar cifre) și poți goli formularul după submit. E tiparul standard în formulare.

**Exemplu minim**

```tsx
import { useState } from "react";

export function SearchBar() {
  const [query, setQuery] = useState("");

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search books…"
    />
  );
}
```

**Cum îl vezi în cod real.** În formularele mari vei vedea mai ales react-hook-form (vezi [03-liste-conditii-formulare](03-liste-conditii-formulare.md)), care folosește intern inputuri necontrolate, din motive de performanță. Conceptul rămâne același: datele formularului ajung într-un loc clar înainte de submit.

**Capcana.** `value` fără `onChange`: câmpul devine read-only și nu poți scrie în el. Sau `value={undefined}` care devine apoi un string. React avertizează în consolă că un input trece din necontrolat în controlat. Pornește mereu cu `""`, nu cu `undefined`.

<details>
<summary>Întrebare de interviu: Ce e un input controlat vs. unul necontrolat?</summary>

Valoarea unui input controlat stă în state-ul React și se actualizează prin `onChange`, deci React deține valoarea. Un input necontrolat își ține valoarea în DOM, iar eu o citesc la nevoie, cu un ref sau din `FormData` la submit. Controlatul e mai flexibil pentru validare live și formatare. Necontrolatul e mai simplu și mai performant în formulare mari, de aceea îl folosește react-hook-form.
</details>

## Lifting state up

**Ce e.** Când două componente surori au nevoie de aceeași informație, state-ul se mută în cel mai apropiat părinte comun. Părintele trimite valoarea în jos prin props, iar callback-urile urcă schimbările.

**De ce contează.** E răspunsul la jumătate din întrebările „cum trimit date de la o componentă la alta?”. Nu trimiți date pe orizontală, ci urci state-ul.

**Exemplu minim**

```tsx
import { useState } from "react";

export function SearchPage() {
  const [query, setQuery] = useState(""); // lives in the common parent

  return (
    <>
      <SearchBar value={query} onChange={setQuery} />
      <ResultsList query={query} />
    </>
  );
}
```

`SearchBar` și `ResultsList` nu se cunosc între ele. Amândouă depind de `SearchPage`.

**Cum îl vezi în cod real.** Uneori state-ul urcă până în URL, nu doar într-un părinte. Pentru căutare, Claude va pune adesea query-ul în `?q=` (vezi [06-routing-si-date](06-routing-si-date.md)), ca linkul să poată fi trimis cuiva. E tot lifting state up, doar că mai sus.

**Capcana.** Două copii ale aceluiași state, câte una în fiecare soră, sincronizate cu efecte. Vor ajunge să se contrazică. O singură sursă de adevăr, în părinte.

<details>
<summary>Întrebare de interviu: Cum partajezi state între două componente surori?</summary>

Mut state-ul în cel mai apropiat părinte comun. Părintele trimite valoarea ca props și callback-uri pentru modificare. Dacă părintele comun e foarte sus și props-urile trec prin multe niveluri, mă uit la compoziție, la Context sau, pentru date de la server, la un cache precum TanStack Query.
</details>

## Update imutabil de array și obiect

**Ce e.** Nu modifici array-ul sau obiectul din state. Creezi o copie nouă cu modificarea și o dai setter-ului. React compară referințele (`===`): dacă îi dai același array modificat pe dinăuntru, nu vede nicio schimbare.

**De ce contează.** Ai învățat în ziua 4 diferența dintre referință și copie. Aici devine regulă de lucru: mutarea directă a state-ului dă bug-uri în care ecranul nu se actualizează.

**Exemplu minim**

```tsx
type Line = { id: string; description: string; quantity: number };

const [lines, setLines] = useState<Line[]>([]);

// add
setLines((prev) => [...prev, { id: crypto.randomUUID(), description: "", quantity: 1 }]);

// update one
setLines((prev) => prev.map((l) => (l.id === id ? { ...l, quantity: 3 } : l)));

// remove
setLines((prev) => prev.filter((l) => l.id !== id));
```

**Cum îl vezi în cod real.** Exact aceste trei forme: spread pentru adăugare, `map` cu spread pe obiect pentru modificare, `filter` pentru ștergere. Le vei vedea în formularul de linii de factură din P2. În cod nou mai apar `toSorted()` și `toSpliced()`, care întorc copii în loc să modifice array-ul.

**Capcana.** `lines.push(newLine); setLines(lines);` sau `lines.sort()`, care sortează pe loc. AI-ul scapă uneori un `.sort()` pe un array din state. Caută-l la review.

<details>
<summary>Întrebare de interviu: De ce nu modifici direct un array din state?</summary>

React decide dacă trebuie să re-randeze comparând referința veche cu cea nouă. Dacă modific array-ul pe loc și îl dau înapoi, referința e aceeași, așa că React poate considera că nu s-a schimbat nimic. În plus, mutarea strică predictibilitatea: alte componente sau render-uri anterioare văd date schimbate pe sub ele. Creez mereu o copie nouă, cu spread, `map` sau `filter`.
</details>

## State derivat vs. state duplicat

**Ce e.** Dacă o valoare poate fi calculată din props sau din alt state, nu o pui în state. O calculezi direct în render. State-ul trebuie să fie minim: doar ce nu poate fi dedus.

**De ce contează.** Fiecare bucată de state în plus e o bucată care se poate desincroniza. E una dintre cele mai frecvente greșeli în codul generat de AI și una dintre cele mai frecvente întrebări de interviu de nivel mediu.

**Exemplu minim**

```tsx
// ❌ duplicated state, kept in sync by hand
const [lines, setLines] = useState<Line[]>([]);
const [total, setTotal] = useState(0);

// ✅ derived during render
const totalCents = lines.reduce((sum, l) => sum + l.quantity * l.unitPriceCents, 0);
```

**Cum îl vezi în cod real.** La fel cu selecția: ții `selectedId` în state și calculezi `const selected = books.find(b => b.id === selectedId)`, nu ții în state o copie a obiectului `selectedBook`. Dacă cartea se actualizează în listă, copia ar rămâne veche.

**Capcana.** `useEffect(() => setTotal(...), [lines])`. Un efect care doar copiază date dintr-un state în altul e aproape mereu greșit: face un render în plus și o fereastră în care ecranul arată totalul vechi. Detalii în [04-efecte](04-efecte.md).

<details>
<summary>Întrebare de interviu: Ce e derived state și de ce e de evitat să-l ții în useState?</summary>

E o valoare care poate fi calculată din alte date, de exemplu totalul facturii din linii sau lista filtrată din listă plus filtru. Dacă o țin în state separat, trebuie să o sincronizez manual și există momente în care cele două nu se potrivesc. O calculez direct în render. Dacă e foarte costisitoare, o memorez cu `useMemo`, dar nu o duplic în state.
</details>

## Mai departe

- [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [State: A Component's Memory](https://react.dev/learn/state-a-components-memory)
- [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)
