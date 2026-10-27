# Module

Până acum ai scris tot codul într-un singur fișier. Un proiect real are sute de fișiere, iar modulele sunt felul în care acestea își dau unele altora funcții, componente și tipuri. Fișierul ăsta e scurt, dar o să-l aplici în fiecare fișier pe care îl deschizi.

## ES modules: import și export

**Ce e.** Un modul e un fișier cu scope propriu: nimic din el nu e vizibil în afară decât dacă îl exporți explicit cu `export`. Alt fișier îl folosește cu `import`. Sistemul ăsta, numit **ES modules** (ESM), e standardul limbajului și merge la fel în browser, în Node și în bundlere precum Vite.

**De ce contează.** Fiecare componentă React, fiecare service din Nest și fiecare helper e un modul. Primele linii din aproape orice fișier sunt import-uri.

**Exemplu minim**
```js
// money.js
export const VAT_RATE = 0.21;
export function toCents(lei) {
  return Math.round(lei * 100);
}

// invoice.js
import { VAT_RATE, toCents } from "./money.js";
console.log(toCents(12.4), VAT_RATE);
```

**Capcana.** Import-urile sunt statice: stau sus în fișier și sunt rezolvate înainte să ruleze codul. Dacă ai nevoie să încarci un modul doar la nevoie, există `await import("./heavy.js")` (import dinamic). Vite îl folosește ca să împartă aplicația în bucăți care se descarcă separat.

## Named export vs default export

**Ce e.** Un modul poate avea oricâte **named exports** (`export function x`) și cel mult un **default export** (`export default ...`).

**Exemplu minim**
```js
// BookCard.jsx
export default function BookCard() { /* ... */ }  // default
export function BookCover() { /* ... */ }         // named

// elsewhere
import BookCard, { BookCover } from "./BookCard.jsx";
import Whatever from "./BookCard.jsx"; // also works — a default import can be renamed freely
```

**Cum îl vezi în cod real.**
- Multe echipe preferă named exports: numele e același peste tot, editorul le găsește și le redenumește automat, iar import-urile greșite dau eroare.
- Default exports apar unde **framework-ul le cere**. În Next.js (P4), fișierele `page.tsx` și `layout.tsx` trebuie să aibă `export default`. Și componentele generate de unele template-uri React vin cu default.

**Capcana.** La un default export, numele de la import e ales liber, deci `import Invoice from "./ClientCard"` compilează fără eroare. Așa ajungi cu același lucru numit în trei feluri prin proiect.

<details>
<summary>Întrebare de interviu: Ce diferență e între named și default export?</summary>

Named exports sunt importate cu numele lor exact, între acolade, și un modul poate avea oricâte. Default export e unul singur pe modul și se importă fără acolade, sub orice nume. Prefer named exports pentru consistență și refactorizare, și folosesc default unde convenția framework-ului o cere, cum e la paginile din Next.js.
</details>

## Cum decide Node: `"type": "module"`

**Ce e.** Node înțelege două sisteme de module: ESM și CommonJS (cel vechi). Decide pe care îl folosește după câmpul `"type"` din `package.json` și după extensie:
- `"type": "module"` → fișierele `.js` sunt ESM;
- lipsă sau `"type": "commonjs"` → fișierele `.js` sunt CommonJS;
- `.mjs` e mereu ESM, `.cjs` e mereu CommonJS. De aceea exemplele din ghid se salvează ca `.mjs`.

**Exemplu minim**
```json
{
  "name": "shelf",
  "type": "module",
  "scripts": { "dev": "vite" }
}
```

**Capcana.** În ESM, Node cere extensia completă la import-urile relative: `import { toCents } from "./money.js"`, nu `"./money"`. În proiectele Vite și Next.js poți omite extensia, pentru că acolo import-urile sunt rezolvate de bundler, nu de Node. Dacă vezi `ERR_MODULE_NOT_FOUND` la un fișier care există, aproape sigur lipsește extensia.

## Să recunoști CommonJS

**Ce e.** Sistemul de module cu care a pornit Node, înainte să existe ESM. Nu-l vei scrie în proiecte noi, dar îl vei citi în cod mai vechi, în documentație și în fișiere de configurare.

**Exemplu minim**
```js
// CommonJS
const express = require("express");
function toCents(lei) { return Math.round(lei * 100); }
module.exports = { toCents };

// the same thing in ESM
import express from "express";
export function toCents(lei) { return Math.round(lei * 100); }
```

**Cum îl vezi în cod real.** Proiectele noi generate cu Nest CLI sunt ESM (au `"type": "module"`). Multe proiecte Nest existente, din firmele unde vei aplica, sunt însă CommonJS, iar TypeScript le transformă `import`-urile în `require` la compilare. Vei mai vedea CommonJS în fișiere de config ca `*.config.cjs` și în tutoriale vechi.

**Capcana.** Combinarea celor două sisteme dă erori de tipul `require is not defined in ES module scope` sau `Cannot use import statement outside a module`. Ambele înseamnă același lucru: fișierul e tratat în alt sistem decât cel în care e scris. Te uiți la `"type"` și la extensie.

<details>
<summary>Întrebare de interviu: Ce diferență e între CommonJS și ES modules?</summary>

CommonJS e sistemul istoric al lui Node: `require()` e o funcție apelată la runtime și încarcă sincron, iar exporturile se fac prin `module.exports`. ESM e standardul limbajului: `import`/`export` sunt statice, analizate înainte de execuție, ceea ce permite tree-shaking, adică eliminarea codului nefolosit la build, și merge nativ și în browser. În Node, alegerea se face prin `"type"` din `package.json` sau prin extensiile `.mjs`/`.cjs`.
</details>

## Barrel files

**Ce e.** Un fișier `index.ts` care doar re-exportă din alte fișiere ale folderului, ca import-urile din afară să fie mai scurte.

**Exemplu minim**
```ts
// components/index.ts
export { BookCard } from "./BookCard";
export { ShelfList } from "./ShelfList";
export { SearchBar } from "./SearchBar";

// elsewhere: one import instead of three
import { BookCard, SearchBar } from "./components";
```

**Cum îl vezi în cod real.** Des în proiecte React și Nest. Claude le generează adesea fără să i le ceri.

**Capcana.** Barrel-urile mari pot produce **import-uri circulare** (A importă din `index`, care importă din B, care importă din A), cu erori ciudate de tipul „X is undefined” la pornire, și pot încetini build-ul și testele. Păstrează-le mici sau evită-le în folderele unde fișierele se importă între ele.

## Ce poate importa Vite și ce nu poate Node

**Ce e.** Într-un proiect Vite vei vedea import-uri care nu sunt JavaScript:

```ts
import "./index.css";            // side-effect import: just load the CSS
import logo from "./logo.svg";   // gives you a URL to the file
import { useQuery } from "@tanstack/react-query"; // a package from node_modules (no ./ prefix)
```

Primul tip nu importă nicio valoare, doar încarcă fișierul. Importul de CSS sau SVG funcționează pentru că **Vite** le transformă la build, nu pentru că așa ar merge JavaScript-ul. Rulat direct cu Node, același fișier ar da eroare. Un import fără `./` sau `../` în față înseamnă un pachet din `node_modules`; despre asta vezi [03-ecosistem-js/02-npm-si-pachete.md](../03-ecosistem-js/02-npm-si-pachete.md).

## Mai departe

- [MDN — JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [javascript.info — Modules, introduction](https://javascript.info/modules-intro)
