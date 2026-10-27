# Hooks și compoziție

Cum îți organizezi logica și componentele când aplicația crește: custom hooks pentru logica reutilizabilă, Context pentru datele de care are nevoie toată lumea, compoziție pentru componente flexibile și o structură de foldere în care te descurci și după o lună.

## Regulile hook-urilor

**Ce e.** Hook-urile (`useState`, `useEffect`, `useContext`, ale tale…) au două reguli:

1. le apelezi doar la nivelul de sus al unei componente sau al altui hook, nu în `if`, `for` sau după un `return` timpuriu;
2. le apelezi doar din componente React sau din custom hooks.

Numele unui hook începe obligatoriu cu `use`.

**De ce contează.** React își amintește state-ul fiecărui hook după *ordinea* apelurilor. Dacă un hook e apelat doar uneori, ordinea se strică și state-urile se amestecă.

**Exemplu minim**

```tsx
// ❌ conditional hook
function BookPage({ id }: { id?: string }) {
  if (!id) return <p>Missing id</p>;
  const [notes, setNotes] = useState(""); // called only sometimes → broken order
}

// ✅ hooks first, conditions after
function BookPage({ id }: { id?: string }) {
  const [notes, setNotes] = useState("");
  if (!id) return <p>Missing id</p>;
  // ...
}
```

**Cum îl vezi în cod real.** Regulile de hooks ale linter-ului (plugin-ul `react-hooks` în ESLint; regulile `react/*` în oxlint) prind încălcările. Erorile lui nu sunt stilistice: sunt bug-uri.

**Capcana.** Un `return` timpuriu pus *înaintea* unui hook. E cel mai frecvent mod de a încălca regula fără să-ți dai seama.

<details>
<summary>Întrebare de interviu: De ce nu poți apela un hook într-un if?</summary>

React leagă state-ul fiecărui hook de poziția lui în ordinea apelurilor dintr-o componentă. Dacă un hook e apelat condiționat, ordinea diferă între render-uri, iar React atribuie state-ul greșit hook-urilor următoare. De aceea hook-urile se apelează mereu la nivelul de sus, în aceeași ordine.
</details>

## Custom hooks

**Ce e.** O funcție al cărei nume începe cu `use` și care folosește alte hook-uri. Extragi în ea logica ce se repetă sau care încarcă o componentă. Fiecare componentă care o folosește primește propriul state.

**De ce contează.** E principalul instrument de organizare a logicii în React. Componentele rămân despre ce se afișează, iar hook-urile despre cum se obțin datele.

**Exemplu minim** (`useDebounce`, din P1: căutarea pornește abia după ce utilizatorul se oprește din tastat)

```tsx
import { useEffect, useState } from "react";

export function useDebounce<T>(value: T, delayMs = 400): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delayMs);
    return () => clearTimeout(id); // new keystroke → cancel the pending update
  }, [value, delayMs]);

  return debounced;
}

// usage
const debouncedQuery = useDebounce(query);
const results = useBookSearch(debouncedQuery); // fires only after 400ms of silence
```

**Cum îl vezi în cod real.** Hook-uri precum `useBooks()`, `useInvoices(filters)`, `useAuth()`, `useShelf()` care învelesc TanStack Query, Context sau `localStorage`. Componentele ajung să arate așa: `const { data, isPending } = useInvoices({ status })`.

**Capcana.** Un custom hook care nu folosește niciun hook e doar o funcție. Nu-i pune prefixul `use`. Și invers: dacă îl numești `use...`, linter-ul îi aplică regulile de hook.

<details>
<summary>Întrebare de interviu: Ce e un custom hook și când îl creezi?</summary>

O funcție cu prefixul `use` care compune alte hook-uri. O creez când aceeași logică cu state sau efecte apare în mai multe componente sau când o componentă are prea multă logică amestecată cu UI-ul. Hook-ul partajează logica, nu state-ul: fiecare componentă care îl apelează are propriile valori.
</details>

## useRef

**Ce e.** Un hook care îți dă un obiect `{ current: ... }` care persistă între render-uri, dar a cărui modificare **nu** declanșează re-render. Se folosește în două feluri: ca referință la un element DOM și ca „cutie” pentru o valoare care trebuie ținută minte, dar nu afișată.

**De ce contează.** E unealta corectă pentru focus pe un input, scroll la un element sau păstrarea id-ului unui timer. Cu `useState` în locul lui ai avea re-render-uri inutile.

**Exemplu minim**

```tsx
import { useRef } from "react";

export function QuickAdd() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <>
      <input ref={inputRef} placeholder="Client name" />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

**Cum îl vezi în cod real.** `ref={...}` pe elemente din formulare și modale, plus refs care țin valori „de lucru”, de exemplu id-ul unui interval sau valoarea anterioară a unui prop. În React 19, `ref` se trimite componentelor ca un prop normal (fără `forwardRef`, pe care îl vei vedea în cod mai vechi).

**Capcana.** Să citești sau să scrii `ref.current` în timpul render-ului ca să afișezi ceva. Ecranul nu se actualizează când ref-ul se schimbă. Dacă valoarea trebuie să apară pe ecran, e state.

<details>
<summary>Întrebare de interviu: Care e diferența între useRef și useState?</summary>

Amândouă păstrează o valoare între render-uri. Schimbarea state-ului declanșează un re-render. Schimbarea `ref.current` nu declanșează nimic. Folosesc `useRef` pentru acces la DOM și pentru valori care nu apar pe ecran (id-uri de timer, instanțe externe), iar `useState` pentru tot ce influențează ce se afișează.
</details>

## useMemo, useCallback și când NU

**Ce e.** `useMemo(() => calc, [deps])` memorează *rezultatul* unui calcul. `useCallback(fn, [deps])` memorează *o funcție*, ca să rămână aceeași referință între render-uri. Ambele sunt optimizări.

**De ce contează.** AI-ul le pune peste tot, „preventiv”. Fiecare adaugă cod, dependențe de întreținut și o comparație la fiecare render. În plus, **React Compiler** (opțiunea `react-compiler-ts` la `npm create vite@latest`) face memorarea automat la build, așa că în proiectele noi rareori le scrii de mână.

**Exemplu minim**

```tsx
import { useMemo } from "react";

// ✅ worth it: thousands of items, recomputed only when inputs change
const stats = useMemo(() => computeReadingStats(allBooks), [allBooks]);

// ❌ pointless: trivial calculation
// const fullName = useMemo(() => `${first} ${last}`, [first, last]);
// ✅ just compute it
const fullName = `${first} ${last}`;
```

**Cum îl vezi în cod real.** Folosiri legitime: un calcul măsurat ca lent, o valoare folosită ca dependență de efect care altfel s-ar schimba la fiecare render, sau o funcție trimisă unei componente învelite în `React.memo`. Dacă proiectul are React Compiler, majoritatea dispar.

**Capcana.** Să optimizezi înainte să măsori. Întâi faci codul corect și simplu. Dacă pare lent, deschizi Profiler-ul din React DevTools, găsești componenta care chiar costă și abia acolo memorezi.

<details>
<summary>Întrebare de interviu: Când folosești useMemo și useCallback?</summary>

Când am o problemă de performanță măsurată: un calcul costisitor care se repetă inutil sau o componentă memoizată care se re-randează pentru că primește la fiecare render o funcție nouă. Și când trebuie să stabilizez o valoare folosită ca dependență de efect. Nu le pun preventiv: au și ele un cost și complică codul. Cu React Compiler, memorarea se face automat în cele mai multe cazuri.
</details>

## Context

**Ce e.** Un mecanism prin care o componentă de sus pune o valoare la dispoziția oricărei componente de dedesubt, fără s-o trimită prin props pe fiecare nivel. Creezi contextul cu `createContext`, îl furnizezi sus și îl citești jos cu `useContext` (sau, în React 19, cu `use`).

**De ce contează.** E soluția pentru date de care au nevoie multe componente aflate departe una de alta: utilizatorul logat (P3), tema, limba. Fără el ajungi la *prop drilling*, adică props trimise prin cinci componente care nu le folosesc.

**Exemplu minim**

```tsx
import { createContext, useContext } from "react";

type Session = { userId: string; role: "admin" | "member" } | null;

const SessionContext = createContext<Session>(null);

export function SessionProvider({ session, children }: { session: Session; children: React.ReactNode }) {
  return <SessionContext value={session}>{children}</SessionContext>; // React 19 syntax
}

export function useSession() {
  return useContext(SessionContext);
}
```

**Cum îl vezi în cod real.** Aproape mereu cu un custom hook alături (`useSession`, `useAuth`), ca restul codului să nu atingă contextul direct. În cod mai vechi vei vedea `<SessionContext.Provider value={...}>`. E același lucru, cu sintaxa dinainte de React 19.

**Capcana.** Să folosești Context ca bază de date globală pentru tot. Fiecare schimbare a valorii re-randează toți consumatorii. Datele de la server stau în TanStack Query, nu în Context. Context e pentru valori care se schimbă rar.

<details>
<summary>Întrebare de interviu: Când folosești Context și când nu?</summary>

Îl folosesc pentru valori globale care se schimbă rar și de care au nevoie multe componente: utilizatorul curent, tema, limba. Nu îl folosesc pentru date de la server (pentru ele am o bibliotecă de cache, ca TanStack Query) și nici pentru state care se schimbă des, pentru că fiecare schimbare re-randează toți consumatorii. Adesea, prop drilling-ul se rezolvă și prin compoziție, trimițând componente ca `children`.
</details>

## Compoziție cu `children`

**Ce e.** O componentă poate primi alte elemente prin prop-ul special `children` și le poate afișa unde vrea. Așa construiești componente „cadru” (layout, card, modal) care nu știu nimic despre conținutul lor.

**De ce contează.** E felul în care React reutilizează UI. Rezolvă și multe cazuri de prop drilling: în loc să trimiți date prin componente intermediare, trimiți direct componenta deja construită.

**Exemplu minim**

```tsx
export function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="rounded-lg border p-4">
      <h2 className="font-semibold">{title}</h2>
      {children}
    </section>
  );
}

// usage
<Card title="Unpaid invoices">
  <InvoiceTable invoices={unpaid} />
</Card>
```

**Cum îl vezi în cod real.** `Layout` cu header și sidebar care primește pagina ca `children`, `Modal`, `Card`, `PageHeader`. În componentele shadcn/ui (vezi [07-styling](07-styling.md)) compoziția e peste tot: `<Dialog><DialogTrigger/><DialogContent/></Dialog>`.

**Capcana.** Componente-„monstru” cu 15 props booleene (`showHeader`, `showFooter`, `isCompact`, `hasBorder`…) în loc de compoziție. Când vezi asta, cere o variantă cu `children` sau cu componente mai mici.

<details>
<summary>Întrebare de interviu: Ce e compoziția în React și de ce e preferată moștenirii?</summary>

Compoziția înseamnă să construiești componente din alte componente, trimițând elemente ca `children` sau ca props. React nu folosește moștenire între componente: tot ce ai face prin moștenire faci mai flexibil prin compoziție. Un `Card` sau un `Layout` nu știe ce conține, doar unde îl așază, iar asta îl face reutilizabil peste tot.
</details>

## Structura de foldere

**Ce e.** Cum aranjezi fișierele. Nu există o regulă oficială React, dar în 2026 majoritatea echipelor organizează **pe funcționalități** (features), nu pe tipuri de fișiere.

**De ce contează.** Când lucrezi la facturi, vrei tot ce ține de facturi într-un singur loc. Și când îi ceri lui Claude o funcționalitate nouă, o structură clară îl face să pună codul unde trebuie.

**Exemplu minim**

```
src/
├── main.tsx
├── App.tsx               ← routes
├── components/ui/        ← generic building blocks (Button, Card, Input)
├── features/
│   ├── invoices/
│   │   ├── api.ts        ← fetch functions + TanStack Query hooks
│   │   ├── schemas.ts    ← zod schemas + inferred types
│   │   ├── InvoiceTable.tsx
│   │   └── InvoiceForm.tsx
│   └── clients/
├── lib/                  ← api client, formatters (money, dates)
└── pages/                ← one component per route, thin, composes features
```

**Cum îl vezi în cod real.** Paginile sunt „subțiri”: compun componente din `features/`. Logica de date stă în `api.ts` sau în hook-uri, nu în componente. Formatarea banilor și a datelor stă o singură dată în `lib/`.

**Capcana.** Să schimbi structura la jumătatea proiectului sau să lași fiecare cerere către Claude să inventeze un folder nou. Descrie structura în `CLAUDE.md`-ul proiectului (vezi [templates](../templates/CLAUDE.template.md)), ca s-o respecte de fiecare dată.

<details>
<summary>Întrebare de interviu: Cum ai structura folderele într-o aplicație React medie?</summary>

Pe funcționalități. Fiecare feature (facturi, clienți) are folderul ei cu componentele, apelurile de API, schemele de validare și tipurile. Separat țin componentele UI generice, utilitarele comune (clientul de API, formatarea) și paginile, care doar compun feature-urile. Așa, codul care se schimbă împreună stă împreună, iar ștergerea unui feature înseamnă ștergerea unui folder.
</details>

## Mai departe

- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- [React Compiler](https://react.dev/learn/react-compiler)
