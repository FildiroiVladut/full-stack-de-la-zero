# Cum gândește React

În [DOM minim](../04-javascript/05-dom-minim.md) ai văzut cum e să actualizezi pagina de mână: găsești elementul, îi schimbi textul, ții minte ce ai schimbat. React schimbă complet modul de lucru. Aici e modelul mental pe care se sprijină tot restul capitolului.

## UI = f(state)

**Ce e.** În React nu spui „schimbă textul din paragraful ăsta”. Descrii cum arată interfața pentru niște date (state-ul), iar React se ocupă să aducă DOM-ul în starea aia. Interfața e o funcție de state: aceleași date dau aceeași interfață.

**De ce contează.** E diferența dintre a da instrucțiuni pas cu pas și a descrie rezultatul. Majoritatea bug-urilor de UI din codul vechi apăreau din DOM-ul care rămânea desincronizat de date. În React, dacă datele sunt corecte, ecranul e corect. Dacă ecranul e greșit, cauți problema în date, nu în DOM.

**Exemplu minim**

```tsx
import { useState } from "react";

export function ReadToggle() {
  const [isRead, setIsRead] = useState(false);

  return (
    <button onClick={() => setIsRead(!isRead)}>
      {isRead ? "Read ✓" : "Mark as read"}
    </button>
  );
}
```

Nu există nicio linie de genul `button.textContent = ...`. Schimbi `isRead`, iar React recalculează ce trebuie afișat.

**Cum îl vezi în cod real.** Componentele generate de Claude au mereu aceeași formă: întâi datele (state, props, date din API), apoi calcule pe ele, la final un `return` cu JSX care le afișează. Când citești o componentă, întreabă-te: „ce date intră aici și cum arată ecranul pentru fiecare combinație?”

**Capcana.** Să încerci să modifici DOM-ul direct (`document.querySelector(...)`) dintr-o componentă React. React nu află de schimbare și o suprascrie la următorul render. AI-ul face asta rar, dar dacă vezi `document.` într-o componentă, oprește-te și întreabă de ce.

<details>
<summary>Întrebare de interviu: Ce înseamnă că React e declarativ?</summary>

Descriu cum trebuie să arate interfața pentru un anumit state, nu pașii prin care se modifică DOM-ul. Când state-ul se schimbă, React calculează din nou interfața și aplică în DOM doar diferențele. Asta elimină o clasă întreagă de bug-uri în care pagina ajunge desincronizată de date.
</details>

## Componenta

**Ce e.** O funcție care primește date (props) și întoarce JSX. Numele începe cu literă mare (`BookCard`, nu `bookCard`), ca React să o deosebească de un tag HTML. O aplicație e un arbore de componente compuse una în alta.

**De ce contează.** Componenta e unitatea în care gândești, împarți munca și faci review. Un PR bun atinge câteva componente clare, nu o funcție de 400 de linii.

**Exemplu minim**

```tsx
type BookCardProps = {
  title: string;
  author: string;
  year?: number;
};

export function BookCard({ title, author, year }: BookCardProps) {
  return (
    <article>
      <h3>{title}</h3>
      <p>
        {author}
        {year !== undefined && ` · ${year}`}
      </p>
    </article>
  );
}
```

O folosești ca pe un tag: `<BookCard title="Dune" author="Frank Herbert" year={1965} />`.

**Cum îl vezi în cod real.** De obicei câte o componentă pe fișier, cu numele fișierului identic (`BookCard.tsx`) și export cu nume (`export function`). Componentele mari se sparg în altele mici când o bucată are o responsabilitate clară, de exemplu `InvoiceTable` → `InvoiceRow`.

**Capcana.** Componente definite *în interiorul* altor componente. Se recreează la fiecare render și își pierd state-ul. Dacă vezi `function Row() {...}` scrisă în corpul lui `Table`, mut-o afară.

<details>
<summary>Întrebare de interviu: Ce e o componentă în React?</summary>

O funcție care primește props și întoarce o descriere a interfeței, adică JSX. Trebuie să fie pură în timpul render-ului: aceleași props și același state dau același rezultat, fără efecte secundare. Componentele se compun una în alta și formează un arbore.
</details>

## JSX

**Ce e.** O sintaxă care seamănă cu HTML-ul, dar e JavaScript. La build, `<h3>{title}</h3>` devine un apel de funcție, ceva de genul `jsx("h3", { children: title })`. În acolade `{}` pui orice expresie JavaScript.

**De ce contează.** Dacă știi că JSX e doar JavaScript, înțelegi de ce nu poți pune un `if` direct în el (`if` e instrucțiune, nu expresie) și de ce folosești `map` în loc de `for`.

**Exemplu minim**

```tsx
const user = { name: "Ana", unpaidInvoices: 3 };

const element = (
  <>
    <h2 className="title">Hello, {user.name}</h2>
    <p>You have {user.unpaidInvoices} unpaid invoices.</p>
  </>
);
```

**Cum îl vezi în cod real.** Diferențele față de HTML pe care le vei vedea peste tot:

- `className` în loc de `class`;
- `htmlFor` în loc de `for` la `<label>`;
- `onClick={handler}` cu camelCase;
- `<>...</>` (Fragment), când vrei să întorci mai multe elemente fără un `<div>` în plus.

**Capcana.** Să crezi că JSX-ul „se execută” de sus în jos ca un script. E o valoare, o descriere. Codul care are efecte (fetch, scriere în `localStorage`) nu-și are locul în corpul componentei. Locul lui e în event handlers sau, rar, în efecte (vezi [04-efecte](04-efecte.md)).

<details>
<summary>Întrebare de interviu: Ce este JSX și ce devine la build?</summary>

O extensie de sintaxă pentru JavaScript care arată ca HTML. Compilatorul (Babel, esbuild sau SWC, prin Vite) o transformă în apeluri de funcții care creează elemente React, adică obiecte simple care descriu ce trebuie afișat. Pentru că e JavaScript, în acolade pot pune orice expresie: variabile, apeluri de funcții, ternare.
</details>

## Render și re-render

**Ce e.** *Render* înseamnă că React îți apelează funcția componentei ca să afle ce trebuie afișat. După render urmează *commit*: React compară rezultatul nou cu cel vechi și aplică în DOM doar diferențele (procesul se numește reconciliation). O componentă se re-randează în trei situații:

1. i s-a schimbat state-ul (s-a apelat un setter);
2. s-a re-randat părintele ei;
3. s-a schimbat un context pe care îl citește (vezi [05-hooks-si-compozitie](05-hooks-si-compozitie.md)).

**De ce contează.** Juniorii se sperie de re-render-uri și împânzesc codul cu optimizări. Un re-render e doar un apel de funcție. Dacă DOM-ul nu se schimbă, nu se atinge nimic pe pagină. Optimizezi abia când măsori o problemă.

**Exemplu minim**

```tsx
import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  console.log("render, count =", count);

  return <button onClick={() => setCount(count + 1)}>Clicked {count} times</button>;
}
```

Deschide consola din browser și apasă butonul: vezi un log la fiecare click. În development, logul poate apărea de două ori. E `StrictMode`, care rulează intenționat render-ul de două ori ca să prindă componentele care nu sunt pure.

**Cum îl vezi în cod real.** În React DevTools (extensie de browser, în Chrome sau Edge pe Windows), la tab-ul *Components*, poți activa „Highlight updates when components render”. Vezi exact ce se re-randează când tastezi într-un câmp de căutare.

**Capcana.** Să modifici date în timpul render-ului: `props.items.push(...)` sau un `let` global incrementat în corpul componentei. Render-ul trebuie să fie pur. Exact asta prinde `StrictMode` când rulează render-ul de două ori.

<details>
<summary>Întrebare de interviu: Ce declanșează un re-render în React?</summary>

Schimbarea state-ului componentei, re-render-ul părintelui și schimbarea unui context pe care îl consumă. Render-ul înseamnă doar că React apelează funcția componentei. Apoi compară rezultatul cu cel anterior și modifică în DOM numai ce s-a schimbat. Re-render-urile sunt ieftine de obicei, așa că optimizez abia când măsor o problemă reală.
</details>

## Fluxul de date într-o singură direcție

**Ce e.** Datele coboară prin props, de la părinte la copil. Evenimentele urcă prin funcții primite tot ca props (callbacks). Un copil nu modifică niciodată direct datele părintelui: îl anunță, iar părintele decide.

**De ce contează.** Când ceva e greșit pe ecran, știi în ce direcție să cauți: urci în arbore până găsești componenta care deține state-ul. Fără regula asta, orice componentă ar putea modifica orice, iar debugging-ul ar deveni ghicit.

**Exemplu minim**

```tsx
type ShelfPickerProps = {
  value: "want" | "reading" | "read";
  onChange: (shelf: "want" | "reading" | "read") => void;
};

export function ShelfPicker({ value, onChange }: ShelfPickerProps) {
  return (
    <select value={value} onChange={(e) => onChange(e.target.value as ShelfPickerProps["value"])}>
      <option value="want">Want to read</option>
      <option value="reading">Reading</option>
      <option value="read">Read</option>
    </select>
  );
}
```

`ShelfPicker` nu știe unde se salvează raftul. Doar raportează alegerea prin `onChange`.

**Cum îl vezi în cod real.** Convenția de nume: datele se numesc după ce sunt (`value`, `book`, `invoice`), iar callback-urile încep cu `on` (`onChange`, `onDelete`, `onSubmit`). În părinte, funcțiile care le implementează încep de obicei cu `handle` (`handleDelete`).

**Capcana.** Să trimiți un setter brut în jos prin cinci niveluri, ca oricine să poată scrie orice. Mai sănătos e un callback cu intenție clară (`onMarkAsRead(bookId)`) decât `setBooks` dat din mână în mână.

<details>
<summary>Întrebare de interviu: Cum comunică un copil cu părintele în React?</summary>

Prin callbacks primite ca props. Părintele deține state-ul și trimite în jos atât datele, cât și o funcție, de exemplu `onChange`. Copilul apelează funcția la un eveniment, iar părintele își actualizează state-ul, ceea ce re-randează copilul cu datele noi. Datele merg mereu în jos, iar evenimentele în sus.
</details>

## Cum citești un arbore de componente generat de Claude

**Ce e.** O metodă de lucru, nu un API. Când Claude îți generează un frontend întreg, te uiți la zeci de fișiere. Ai nevoie de o ordine de citire.

**De ce contează.** La code walk (pasul 4 din [metodă](../00-start/01-metoda.md)) trebuie să poți explica tot proiectul. Ordinea de mai jos îți dă harta în 15 minute.

**Exemplu minim.** Arborele pentru P1 Shelf arată cam așa:

```
main.tsx                    ← intrarea: montează <App /> în #root, plus providerii
└── App.tsx                 ← layout + rute
    ├── SearchPage          ← state: query (în URL), rezultatele vin din TanStack Query
    │   ├── SearchBar       ← primește value + onChange
    │   └── ResultsList
    │       └── BookCard    ← primește un book, emite onAddToShelf
    ├── BookPage            ← citește :id din URL, încarcă detaliile
    └── ShelfPage           ← citește rafturile din context/localStorage
        └── StatsPanel      ← calcule derivate (nimic în state)
```

**Cum îl vezi în cod real.** Ordinea de citire:

1. `main.tsx`: ce provideri învelesc aplicația (router, QueryClient, context).
2. `App.tsx` sau fișierul de rute: ce pagini există.
3. Pentru fiecare pagină: **unde stă state-ul?** Caută `useState`, `useQuery`, `useContext`.
4. Urmărește props-urile în jos și callback-urile în sus.
5. Abia la final te uiți la componentele mici de prezentare.

Pune-i lui Claude întrebarea: *„Desenează-mi arborele de componente și spune-mi pentru fiecare unde stă state-ul și de unde vin datele.”* Apoi verifică harta în cod.

**Capcana.** Să citești fișierele în ordine alfabetică sau să te pierzi în CSS. Întâi datele, apoi forma.

<details>
<summary>Întrebare de interviu: Cum te-ai orienta într-un codebase React nou?</summary>

Pornesc de la punctul de intrare, ca să văd providerii (router, cache de date, context), apoi trec la rute, ca să văd paginile. Pentru fiecare pagină caut unde stă state-ul și de unde vin datele: state local, server state prin TanStack Query sau context. Urmăresc apoi props-urile în jos și callback-urile în sus. Componentele de prezentare le citesc la final. React DevTools mă ajută să văd arborele live.
</details>

## Mai departe

- [Thinking in React](https://react.dev/learn/thinking-in-react)
- [Render and Commit](https://react.dev/learn/render-and-commit)
- [Keeping Components Pure](https://react.dev/learn/keeping-components-pure)
