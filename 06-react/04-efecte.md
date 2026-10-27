# Efecte

`useEffect` e hook-ul cel mai folosit greșit din React, de juniori și de AI deopotrivă. Fișierul ăsta te învață când e nevoie de el, cum îl scrii corect și, mai ales, cum recunoști un efect care n-ar trebui să existe.

## Ce e un efect

**Ce e.** Un efect e cod care sincronizează componenta cu ceva **din afara React**: rețeaua, un timer, `localStorage`, un WebSocket, un API al browserului. Rulează *după* ce React a actualizat ecranul.

**De ce contează.** Definiția e testul: dacă nu există niciun „sistem extern” în joc, probabil nu ai nevoie de efect. Codul care doar transformă date React în alte date React nu e un efect.

**Exemplu minim**

```tsx
import { useEffect } from "react";

export function PageTitle({ unread }: { unread: number }) {
  useEffect(() => {
    document.title = unread > 0 ? `(${unread}) Invoicer` : "Invoicer";
  }, [unread]);

  return null;
}
```

Titlul tab-ului e în afara React, deci aici un efect e potrivit.

**Cum îl vezi în cod real.** Efectele legitime din proiectele tale: salvarea rafturilor în `localStorage` (P1), abonarea la schimbările de autentificare Supabase (P3), un `setInterval` pentru un ceas sau un countdown. Datele de la server nu se încarcă cu efect, ci cu TanStack Query.

**Capcana.** Să folosești `useEffect` ca pe un „rulează asta când se schimbă X”. Ăsta e un reflex din alte medii de programare și produce cod cu render-uri în plus, momente cu date vechi pe ecran și bucle.

<details>
<summary>Întrebare de interviu: Când folosești useEffect?</summary>

Când trebuie să sincronizez componenta cu un sistem extern: subscripții, timere, API-uri ale browserului, conexiuni, scriere în `localStorage`. Nu îl folosesc pentru a calcula date din alte date (asta se face în render) și nici pentru acțiuni declanșate de utilizator (acelea merg în event handlers). Pentru fetch de date prefer o bibliotecă precum TanStack Query sau mecanismele framework-ului.
</details>

## Array-ul de dependențe

**Ce e.** Al doilea argument din `useEffect` îi spune lui React când să ruleze din nou efectul:

- **fără array:** după fiecare render;
- **`[]`:** o singură dată, după primul render;
- **`[a, b]`:** după primul render și apoi ori de câte ori se schimbă `a` sau `b` (comparate cu `Object.is`).

**De ce contează.** Dependențele greșite dau cele mai urâte bug-uri din React. Cu o dependență lipsă, efectul rulează cu valori vechi (stale closure, vezi closures din ziua 6). Cu o dependență în plus, poate rula la nesfârșit.

**Exemplu minim**

```tsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("checking availability for", roomId);
  }, 10_000);
  return () => clearInterval(id);
}, [roomId]); // re-subscribe when the room changes
```

**Cum îl vezi în cod real.** Regula de lint `exhaustive-deps` îți subliniază dependențele lipsă. În oxlint, care vine cu template-ul Vite, se numește `react/exhaustive-deps`; în ESLint, `react-hooks/exhaustive-deps`. Ia-o în serios: avertismentul ei aproape întotdeauna înseamnă un bug.

**Capcana.** Un obiect sau o funcție creată în render, pusă ca dependență: `useEffect(..., [options])` unde `options = { page }` e creat la fiecare render. E altă referință de fiecare dată, deci efectul rulează la fiecare render. Pune în dependențe valorile primitive (`page`) sau mută obiectul în interiorul efectului.

<details>
<summary>Întrebare de interviu: Ce se întâmplă dacă lipsește o dependență din array?</summary>

Efectul păstrează valorile din render-ul în care a fost creat, adică o closure veche, și nu se re-rulează când valoarea se schimbă. Rezultatul sunt bug-uri cu date vechi: de exemplu un interval care raportează mereu primul id. Regula de lint exhaustive-deps le prinde. Soluția corectă e să pun dependența sau să restructurez codul, nu să ascund avertismentul.
</details>

## Cleanup

**Ce e.** Funcția pe care o întorci din efect. React o rulează înainte să ruleze efectul din nou și când componenta dispare de pe ecran. Acolo închizi ce ai deschis: oprești timere, te dezabonezi, anulezi request-uri.

**De ce contează.** Fără cleanup apar memory leaks, timere care rulează pentru componente care nu mai există și, la request-uri, *race conditions*: răspunsul vechi ajunge după cel nou și îl suprascrie.

**Exemplu minim** (race condition la căutare, rezolvată)

```tsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`https://openlibrary.org/search.json?q=${encodeURIComponent(query)}`, {
    signal: controller.signal,
  })
    .then((res) => res.json())
    .then((data) => setResults(data.docs))
    .catch((err) => {
      if (err.name !== "AbortError") setError(err);
    });

  return () => controller.abort(); // cancel the previous request
}, [query]);
```

Tastezi „dune”: pleacă request-uri pentru „d”, „du”, „dun”, „dune”. Fără cleanup, dacă răspunsul pentru „dun” vine ultimul, afișezi rezultatele greșite.

**Cum îl vezi în cod real.** Orice `setInterval`, `addEventListener` sau `subscribe` dintr-un efect trebuie să aibă perechea lui în cleanup: `clearInterval`, `removeEventListener`, `unsubscribe`. Dacă lipsește perechea, e bug.

**Capcana.** Să crezi că exemplul de mai sus e „cum se face fetch în React”. E cum se face *corect* cu efecte, dar vezi câtă grijă cere. De aceea în proiecte folosești TanStack Query, care rezolvă race conditions, cache și retry pentru tine.

<details>
<summary>Întrebare de interviu: Ce e o race condition într-un efect de fetch și cum o eviți?</summary>

Când parametrii se schimbă repede, pleacă mai multe request-uri, iar răspunsurile pot sosi în altă ordine, deci un răspuns vechi îl poate suprascrie pe cel nou. O evit cu cleanup: anulez request-ul anterior cu `AbortController` sau ignor răspunsurile vechi cu un flag. În practică folosesc TanStack Query, care leagă răspunsurile de query key și tratează asta automat.
</details>

## StrictMode rulează efectele de două ori

**Ce e.** În development, `<StrictMode>` (pus de template-ul Vite în `main.tsx`) montează fiecare componentă, rulează efectele, face cleanup și le rulează din nou. Intenționat. În producție nu se întâmplă.

**De ce contează.** Vei vedea două request-uri în tab-ul Network sau două loguri și vei crede că ai un bug. Nu e bug: e un test automat care verifică dacă efectul tău are cleanup corect.

**Exemplu minim**

```tsx
useEffect(() => {
  console.log("connect to room", roomId);
  return () => console.log("disconnect from room", roomId);
}, [roomId]);

// In development you'll see:
// connect → disconnect → connect
```

**Cum îl vezi în cod real.** Dacă un efect se comportă greșit sub StrictMode (de exemplu adaugă de două ori același element), înseamnă că lipsește sau e greșit cleanup-ul. Asta e informația pe care ți-o dă.

**Capcana.** Să ștergi `<StrictMode>` ca să „repari” dublura. Ascunzi bug-ul în loc să-l repari. Un efect corect suportă oricâte cicluri de montare și demontare.

<details>
<summary>Întrebare de interviu: De ce rulează useEffect de două ori în development?</summary>

Pentru că `StrictMode` montează, demontează și remontează componentele în development, ca să verifice că efectele au cleanup corect și că pot fi rulate de mai multe ori fără efecte secundare. În producție efectul rulează o dată. Dacă dublura îmi strică ceva, problema e în cleanup, nu în StrictMode.
</details>

## Când NU ai nevoie de un efect

**Ce e.** Lista cazurilor în care un efect e greșit, din ghidul oficial React „You Might Not Need an Effect”. AI-ul le produce pe toate, în special primele trei.

**De ce contează.** E cel mai valoros lucru din capitolul ăsta pentru review la cod generat. Dacă vezi unul dintre tiparele de mai jos, ai găsit un bug sau cel puțin cod de simplificat.

**Exemplu minim**

```tsx
// ❌ 1. Derived data via effect
const [filtered, setFiltered] = useState<Book[]>([]);
useEffect(() => setFiltered(books.filter((b) => b.shelf === shelf)), [books, shelf]);
// ✅ compute during render
const filtered = books.filter((b) => b.shelf === shelf);

// ❌ 2. User action inside an effect
useEffect(() => { if (submitted) createInvoice(form); }, [submitted]);
// ✅ do it in the event handler
function handleSubmit() { createInvoice(form); }

// ❌ 3. Resetting state when a prop changes
useEffect(() => setComment(""), [bookId]);
// ✅ give the component a key, React resets it
<CommentBox key={bookId} bookId={bookId} />
```

**Cum îl vezi în cod real.** Încă două tipare de recunoscut:

- **Lanțuri de efecte:** efectul A setează state, care declanșează efectul B, care setează alt state. Calculează tot ce poți în render, iar ce rămâne pune într-un singur event handler.
- **„Notifică părintele”:** `useEffect(() => onChange(value), [value])`. Apelează `onChange` direct în handler-ul care schimbă valoarea.

**Capcana.** Să crezi că un efect „merge, deci e ok”. Merge, dar cu un render în plus, cu un moment în care ecranul arată date vechi și cu logică mai greu de urmărit. Când vezi un `useEffect`, pune întrebarea: *„ce sistem extern sincronizează?”*. Dacă răspunsul e „niciunul”, îl scoți.

<details>
<summary>Întrebare de interviu: Dă exemple de cazuri în care useEffect e folosit greșit.</summary>

Patru cazuri des întâlnite. Unu: calculul de date derivate, cum ar fi filtrare sau totaluri, care se face direct în render, cu `useMemo` dacă e scump. Doi: acțiuni ale utilizatorului, ca submit sau POST, care merg în event handler. Trei: resetarea state-ului la schimbarea unui prop, unde folosesc `key`. Patru: notificarea părintelui când se schimbă o valoare, unde apelez callback-ul în același handler. Toate produc render-uri în plus și momente cu date desincronizate.
</details>

## Un efect legitim: localStorage în P1

**Ce e.** Rafturile din P1 Shelf stau în `localStorage`, ca să nu se piardă la refresh. Citirea se face o singură dată, la inițializarea state-ului. Scrierea e un efect, pentru că `localStorage` e un sistem extern.

**De ce contează.** E tiparul complet pe care îl vei avea în P1. Arată și diferența dintre inițializare (fără efect) și sincronizare (cu efect).

**Exemplu minim**

```tsx
import { useEffect, useState } from "react";

const STORAGE_KEY = "shelf.v1";

function loadShelf(): ShelfItem[] {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) ?? "[]");
  } catch {
    return []; // corrupted data shouldn't crash the app
  }
}

export function useShelf() {
  const [items, setItems] = useState<ShelfItem[]>(loadShelf); // lazy initializer: runs once

  useEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
  }, [items]);

  return { items, setItems };
}
```

**Cum îl vezi în cod real.** `useState(loadShelf)`, cu funcția trimisă fără paranteze, e un *lazy initializer*: React o apelează doar la primul render. Cu `useState(loadShelf())`, citirea și parsarea s-ar face la fiecare render, apoi rezultatul ar fi ignorat.

**Capcana.** Citirea din `localStorage` într-un efect cu `[]`. Primul render arată o listă goală, apoi „sare” la datele salvate, iar dacă efectul de scriere rulează înainte, poate suprascrie datele cu `[]`. Inițializează din storage, nu sincroniza la mount.

<details>
<summary>Întrebare de interviu: Ce e lazy initialization la useState?</summary>

Dacă dau lui `useState` o funcție în loc de o valoare, React o apelează doar la primul render ca să obțină valoarea inițială. E util când inițializarea e costisitoare, de exemplu citire și parsare din `localStorage`. Altfel, expresia s-ar evalua la fiecare render, chiar dacă rezultatul e ignorat după primul.
</details>

## Mai departe

- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [StrictMode](https://react.dev/reference/react/StrictMode)
