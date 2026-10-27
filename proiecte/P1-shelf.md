# P1 · Shelf

> Caut cărți, le pun pe rafturi, văd ce am citit.

**Când:** săptămâna 2 (~20 h de proiect) · **Tip:** frontend SPA cu API public · **Domeniu:** consumer / media

## Ce face

O aplicație în care cauți cărți (prin Open Library), le pui pe rafturi (*Want to read*, *Reading*, *Read*), le dai o notă și notițe, și vezi statistici despre ce ai citit.

## Ce înveți pe el

- React și TypeScript pe un proiect real: componente, props, state, efecte, custom hooks.
- Date de pe un API extern cu TanStack Query, cu stările de loading, eroare și „nimic găsit”.
- Routing între pagini: Search, Book, My shelves, Stats.
- Persistență în browser (localStorage) și limitele ei.
- Vite, npm vs bun, linting (oxlint, din template) și formatare (Prettier), deploy pe Vercel.
- Primul tău repo lucrat cu branch-uri și PR-uri.

**Capitole:** [03](../03-ecosistem-js/), [04](../04-javascript/), [05](../05-typescript/), [06](../06-react/) (01–07), [11](../11-deploy/) (01–02), [12](../12-lucrul-cu-ai/).

## Stack

Vite + React + TypeScript (strict), React Router, TanStack Query, Tailwind CSS, zod (pentru validarea răspunsurilor de la API), localStorage. Deploy pe Vercel.

**API: Open Library**, fără cheie. Verifică endpoint-urile curente pe [openlibrary.org/developers/api](https://openlibrary.org/developers/api):

- căutare: `https://openlibrary.org/search.json?q=...`
- detalii carte: `https://openlibrary.org/works/{id}.json`
- coperți: `https://covers.openlibrary.org/b/id/{cover_id}-M.jpg`

E un API gratuit, ținut de o organizație non-profit. Nu-l bombarda cu request-uri: de asta faci debounce și lași TanStack Query să facă cache.

## Funcționalități

**Obligatorii**

1. Căutare cu debounce (~300 ms), cu rezultate care au copertă, titlu, autor și an.
2. Pagina unei cărți: copertă mare, descriere, subiecte, an.
3. Adaugi o carte pe un raft, o muți între rafturi sau o scoți.
4. Notă de la 1 la 5 și notițe pentru cărțile citite.
5. Pagina *My shelves* cu filtrare și sortare (după titlu, data adăugării, notă).
6. Pagina *Stats*: cărți citite pe lună, nota medie, autorii cei mai citiți, total pagini.
7. Persistență în localStorage: după refresh, rafturile rămân.
8. Stări de loading, eroare și „gol” pe fiecare pagină cu date.
9. Merge pe telefon.

**Dacă ai timp**

- Căutarea rămâne în URL (`?q=dune`), ca linkul să poată fi trimis cuiva.
- Export și import al rafturilor ca JSON.
- Dark mode.

## Feature-ul de mână

**Pagina Stats**, adică logica de calcul: grupare pe luni, medii, top autori. E aceeași familie de lucruri ca în ziua 5 din `js-de-la-zero` (`reduce`, `sort`, `filter`), dar pe date reale. Scrii tu funcțiile, pure și tipate; Claude poate face UI-ul după.

## Break & fix

Când ai terminat feature-ul de rafturi, cere-i lui Claude, într-o sesiune nouă:

```
Introdu un singur bug subtil în aplicație, legat de React (key la liste, dependențe la efecte
sau queryKey, ori state modificat direct în loc de copie). Nu-mi spune ce și unde.
Fă commit pe un branch separat numit break-1.
```

Îl găsești cu consola, cu React DevTools și citind diff-ul abia la final.

## Experimentul npm vs bun

La început creezi proiectul de două ori, o dată cu npm și o dată cu bun, cronometrezi instalarea și compari lockfile-urile. Apoi păstrezi varianta npm. Pașii sunt în [03-ecosistem-js/03-npm-vs-bun.md](../03-ecosistem-js/03-npm-vs-bun.md). Notează rezultatele în README, la *Key decisions*.

## Ordinea sugerată a PR-urilor

1. **Setup:** Vite (template `react-ts`) + Tailwind + Prettier + React Router, scripturile `lint` / `typecheck`, layout gol, deploy pe Vercel. Online din prima zi.
2. **Search:** TanStack Query, debounce, stări de loading/eroare/gol.
3. **Book:** pagina de detalii.
4. **Shelves:** un custom hook `useShelves` cu localStorage.
5. **My shelves:** filtrare, sortare, notă, notițe.
6. **Stats:** de mână.
7. **Polish:** responsive, stări goale, README cu screenshot.

## Promptul de start (în plan mode)

```
Citește spec-ul din README.md și CLAUDE.md. Construim Shelf.
Înainte de orice cod, propune un plan:
- structura de foldere și rutele
- componentele principale și unde stă state-ul
- tipurile TypeScript pentru o carte și pentru rafturi
- cum validăm răspunsul de la Open Library cu zod
- cum persistăm în localStorage
- ordinea PR-urilor
Explică fiecare decizie în 1–2 propoziții și spune ce alternativă ai respins și de ce.
Țin să înțeleg tot: păstrează lucrurile simple.
```

## Ce trebuie să poți explica la final (→ [CP2](../checkpoints/CP2.md))

- De ce TanStack Query și nu `fetch` într-un `useEffect`.
- Ce se întâmplă, pas cu pas, de la click pe *Add to shelf* până se schimbă ecranul.
- De ce localStorage nu e o bază de date: un singur browser, limită de spațiu, sincron, nesecurizat.
- Ce produce `npm run build` și ce urcă Vercel.
- Cum ai calculat statisticile și de ce funcțiile alea sunt pure.
