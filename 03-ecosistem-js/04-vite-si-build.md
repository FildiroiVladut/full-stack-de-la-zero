# Vite și build-ul

Codul pe care îl scrii (TypeScript, JSX, sute de fișiere, importuri din npm) nu e codul pe care îl primește browserul. Între ele stă Vite. Trebuie să înțelegi ce face, ca să nu te sperie primul „merge în dev, nu merge după build”.

## De ce ai nevoie de un bundler

**Ce e.** Browserul înțelege doar JavaScript, CSS și HTML. Nu înțelege TypeScript și nici JSX și nu știe unde e `"react"` când scrii `import { useState } from "react"`. **Bundler-ul**:

1. pornește de la fișierul de intrare (`src/main.tsx`) și urmărește toate `import`-urile;
2. transformă TypeScript și JSX în JavaScript;
3. găsește pachetele în `node_modules`;
4. le combină în câteva fișiere și le micșorează (minificare);
5. le pune în nume un hash (`index-BdQq_4o_.js`), ca browserul să le poată ține în cache fără grijă.

**De ce contează.** Codul din `src/` e pentru oameni, iar fișierele din `dist/` sunt pentru browser. Când depanezi o problemă de producție, te uiți la al doilea set.

**Exemplu minim.** Înainte și după:

```text
src/                              dist/
├── main.tsx                      ├── index.html
├── App.tsx            build      └── assets/
├── components/      ───────▶         ├── index-BdQq_4o_.js   (all your code + React, minified)
│   └── BookCard.tsx                  └── index-D8b4DHJx.css
└── index.css
```

<details>
<summary>Întrebare de interviu: Ce face un bundler și de ce avem nevoie de el?</summary>

Pornește de la punctul de intrare al aplicației, urmărește toate importurile, inclusiv cele din `node_modules`, transformă codul (TypeScript, JSX) în JavaScript pe care browserul îl înțelege și produce un număr mic de fișiere optimizate: minificate, cu cod nefolosit eliminat și cu hash în nume pentru cache. Fără el, browserul ar trebui să descarce sute de fișiere separate și n-ar înțelege TypeScript sau importurile de pachete după nume.
</details>

## Transpilare: TypeScript și JSX devin JavaScript

**Ce e.** **Transpilarea** înseamnă transformarea codului dintr-o formă în alta la același nivel, nu în cod mașină. Din TypeScript se **șterg tipurile**. JSX-ul devine apeluri de funcții:

```tsx
// what you write
const card = <BookCard title="Dune" year={1965} />;

// roughly what the browser receives
import { jsx } from "react/jsx-runtime";
const card = jsx(BookCard, { title: "Dune", year: 1965 });
```

**De ce contează.** **Vite transpilează, dar nu verifică tipurile.** Aplicația pornește și merge în browser chiar dacă ai erori de TypeScript. Erorile le vezi doar în VS Code și când rulezi `tsc`.

**Cum îl vezi în cod real.** Uită-te la scriptul `build` din șablonul Vite:

```json
"build": "tsc -b && vite build"
```

`tsc -b` verifică tipurile, iar dacă pică, build-ul se oprește. Abia dacă trece, `vite build` construiește. Două unelte, două roluri.

**Capcana.** „Merge în dev, deci e corect.” Ai erori de tip ignorate, iar build-ul pe Vercel pică. Rulează `npm run build` local înainte de push, iar în P3 îl pui și în CI.

<details>
<summary>Întrebare de interviu: Vite verifică tipurile TypeScript?</summary>

Nu. Vite doar șterge tipurile ca să obțină JavaScript, pentru viteză. Verificarea o face compilatorul TypeScript (`tsc`), de aceea șablonul are `tsc -b && vite build` la build, iar în CI se rulează un pas separat de typecheck. În dezvoltare, erorile de tip le vezi în editor.
</details>

## Dev server și HMR

**Ce e.** `npm run dev` pornește un server pe `http://localhost:5173`. În dezvoltare, Vite nu construiește tot bundle-ul. Trimite fișierele pe rând, transformate la cerere, ca ES modules native. De asta pornește aproape instant.

**HMR** (Hot Module Replacement) înseamnă că, la salvarea unui fișier, Vite înlocuiește în browser doar modulul modificat. Pagina nu se reîncarcă, iar state-ul componentelor (textul din formular, filtrul ales) rămâne.

**De ce contează.** Bucla „salvez → văd” durează sub o secundă. Când HMR nu mai merge, pierzi mult timp fără să-ți dai seama de ce.

**Exemplu minim**

```bash
npm run dev              # http://localhost:5173
npm run dev -- --host    # also listen on the network (rarely needed in WSL)
```

Schimbă un text în `src/App.tsx` și salvează. Browserul se actualizează singur.

**Capcana.** Pe WSL, proiectul trebuie să stea în home-ul Linux (`~/code/...`), **nu** pe `/mnt/c/...`. Pe partiția Windows, Vite poate să nu observe modificările (documentația Vite pomenește explicit WSL2), iar totul merge mult mai încet. Și: dacă deschizi `index.html` direct din File Explorer (`file://`), primești o pagină albă. Aplicația merge doar prin serverul Vite.

## Build și preview

**Ce e.** `npm run build` produce folderul `dist/`: un `index.html` și fișierele din `assets/`, cu hash în nume. `npm run preview` servește `dist/` local, pe `http://localhost:4173`, ca să testezi exact ce urcă pe server. Deploy-ul unui SPA înseamnă să pui `dist/` pe un hosting static, ca Vercel.

**De ce contează.** Dev-ul și build-ul sunt două moduri diferite. Unele bug-uri apar doar în build.

**Exemplu minim**

```bash
npm run build
npm run preview
```

Output-ul build-ului arată cam așa:

```text
vite v8.x.x building for production...
✓ 32 modules transformed.
dist/index.html                   0.46 kB │ gzip:  0.30 kB
dist/assets/index-D8b4DHJx.css    1.39 kB │ gzip:  0.71 kB
dist/assets/index-BdQq_4o_.js   195.06 kB │ gzip: 61.20 kB
✓ built in 412ms
```

**Capcana.** Importurile cu litere greșite: `import BookCard from "./bookcard"` când fișierul e `BookCard.tsx`. Pe Windows și macOS merge, pentru că sistemul de fișiere nu face diferența. Pe Linux, unde rulează build-ul pe Vercel, pică cu `Module not found`. Lucrând în WSL, prinzi problema din prima, pe laptop. `dist/` nu intră în git: se reface la fiecare build.

<details>
<summary>Întrebare de interviu: Care e diferența dintre `npm run dev` și `npm run build`?</summary>

`dev` pornește un server de dezvoltare care transformă fișierele la cerere, cu HMR, fără optimizări, pentru o buclă de lucru rapidă. `build` produce varianta de producție în `dist/`: cod transpilat, combinat, minificat, cu fișiere cu hash pentru cache. Pe aceea o urci pe hosting. `preview` servește local build-ul, ca să-l verifici înainte de deploy.
</details>

## Variabile de mediu în Vite

**Ce e.** Vite citește fișierele `.env` și pune la dispoziție variabilele în `import.meta.env`. Regulile:

- doar variabilele cu prefixul **`VITE_`** ajung în codul de browser. Restul sunt ignorate;
- fișierele: `.env` (mereu), `.env.local` (mereu, nu intră în git), `.env.development` sau `.env.production` (doar în acel mod);
- toate valorile sunt **string-uri**;
- la build, `import.meta.env.VITE_API_URL` e **înlocuit textual** cu valoarea, adică valoarea e copiată în bundle.

Vite îți dă și câteva variabile predefinite: `import.meta.env.DEV`, `PROD`, `MODE`.

**De ce contează.** Așa îi spui frontend-ului din P2 unde e API-ul: local `http://localhost:3000`, în producție adresa de pe Render.

**Exemplu minim**

```bash
# .env.local  (never committed)
VITE_API_URL=http://localhost:3000
```

```ts
const API_URL = import.meta.env.VITE_API_URL; // "http://localhost:3000" in dev
```

Ca să ai autocomplete și tipuri pentru variabilele tale, adaugi `src/vite-env.d.ts`:

```ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

**Cum îl vezi în cod real.** Pe Vercel setezi `VITE_API_URL` în setările proiectului. Fiind înlocuită la build, **dacă o schimbi, trebuie să faci deploy din nou**: valoarea veche e deja scrisă în bundle.

**Capcana.** `VITE_SUPABASE_SECRET_KEY` sau `VITE_ANTHROPIC_API_KEY`. Prefixul `VITE_` înseamnă literalmente „publică asta în browser”. AI-ul face asta când îi ceri să „repare” o eroare de tipul „variabila e undefined”. Secretele stau doar pe server (vezi [10-securitate-env/02-ce-ajunge-in-browser.md](../10-securitate-env/02-ce-ajunge-in-browser.md)). După ce modifici `.env.local`, repornește `npm run dev`.

<details>
<summary>Întrebare de interviu: Cum folosești variabile de mediu într-o aplicație Vite și ce risc există?</summary>

Le pui în fișiere `.env` și le citești din `import.meta.env`. Doar cele cu prefixul `VITE_` ajung în cod. Riscul e că sunt copiate textual în bundle la build, deci oricine le poate citi în browser. Acolo pun doar valori publice, ca URL-ul API-ului sau o cheie publishable. Secretele rămân pe server. Și pentru că sunt înlocuite la build, o schimbare de valoare cere un build nou.
</details>

## Ce e în proiectul generat

**Ce e.** Așa arată un proiect proaspăt creat cu `npm create vite@latest -- --template react-ts` (detaliile se pot schimba de la o versiune la alta):

```text
shelf/
├── index.html           # the page the browser loads; it includes <script src="/src/main.tsx">
├── package.json
├── vite.config.ts       # Vite config: plugins (React), dev server, build options
├── tsconfig.json        # points to the two configs below
├── tsconfig.app.json    # TS rules for the code in src/ (browser)
├── tsconfig.node.json   # TS rules for vite.config.ts (runs in Node)
├── .oxlintrc.json       # linter config (oxlint, see 05-harta-ecosistemului.md)
├── .gitignore           # node_modules, dist, .env.local...
├── public/              # files copied as-is (favicon); referenced as /favicon.svg
└── src/
    ├── main.tsx         # entry point: mounts <App /> into <div id="root">
    ├── App.tsx          # the first component
    ├── App.css, index.css
    └── assets/          # images imported from code
```

**De ce contează.** E primul proiect pe care Claude ți-l generează în P1. Cu harta asta știi unde să cauți fiecare lucru.

**Capcana.** Două fișiere `tsconfig` pentru două lumi: codul din `src/` rulează în browser, iar `vite.config.ts` rulează în Node. De asta au setări diferite. Nu le „unifica” la sugestia AI-ului.

## Mai departe

- [Vite — Getting Started](https://vite.dev/guide/)
- [Vite — Env Variables and Modes](https://vite.dev/guide/env-and-mode)
- [Vite — Building for Production](https://vite.dev/guide/build)
