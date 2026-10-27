# 04 · JavaScript

Fundamentele le-ai făcut în [js-de-la-zero](https://github.com/FildiroiVladut/js-de-la-zero). Capitolul ăsta face două lucruri. Întâi strânge tot ce știi deja pe o pagină, împreună cu capcanele de interviu. Apoi acoperă ce lipsește ca să poți lucra pe un proiect real: codul asincron (event loop, Promises, `async/await`, `fetch`), modulele, DOM-ul cât trebuie ca să înțelegi de ce există React, și tiparele de JS modern pe care le vezi zilnic în cod. Fără exerciții. Citești, rulezi exemplele, schimbi ceva în ele și treci mai departe.

## La final poți să

- explici ordinea în care rulează codul sincron, Promise-urile și `setTimeout`, și de ce;
- scrii și citești cod cu `async/await` și `fetch` care tratează corect erorile, timeout-urile și cazul 404;
- recunoști diferența dintre ESM și CommonJS și unde le întâlnești;
- explici de ce sincronizarea manuală a DOM-ului devine iad și ce problemă rezolvă React;
- lucrezi corect cu bani (în bani, nu în lei cu zecimale), cu date calendaristice și fusuri orare, și cu update-uri imutabile.

## Fișierele, în ordine

- [ ] [01 · Recap zilele 1–7](01-recap-zilele-1-7.md) — 45 min
- [ ] [02 · Event loop și Promises](02-event-loop-promises.md) — 60 min
- [ ] [03 · async/await și fetch](03-async-await-fetch.md) — 60 min
- [ ] [04 · Module](04-module.md) — 30 min
- [ ] [05 · DOM, minimul necesar](05-dom-minim.md) — 30 min
- [ ] [06 · JS-ul pe care îl vezi în cod real](06-js-in-cod-real.md) — 60 min

Total: cam 4 ore și jumătate, cu rulatul exemplelor.

Exemplele le rulezi în WSL. Faci un folder de lucru (de exemplu `~/code/playground`), salvezi exemplul ca `fisier.mjs` și îl rulezi cu `node fisier.mjs`. Extensia `.mjs` îi spune lui Node că fișierul e modul ES, deci poți folosi `import` și `await` direct la nivelul fișierului.

## Unde îl folosești

- **P1 Shelf:** `fetch` spre Open Library, stările de loading/eroare, `AbortController` la căutare.
- **P2 Invoicer:** bani ca întregi, `Intl.NumberFormat`, update-uri imutabile.
- **P3 Booking:** date, ore și fusuri orare.
- **P4 Kitchen AI:** `Promise.all` și `Promise.allSettled` când procesezi mai multe facturi.
- **[CP1](../checkpoints/CP1.md):** event loop-ul și drumul unui request.

## Verifică-te

1. Ce afișează `console.log("A"); setTimeout(() => console.log("B"), 0); Promise.resolve().then(() => console.log("C")); console.log("D");` și de ce?
2. De ce `fetch` nu intră în `catch` când serverul răspunde 404, și cum scrii codul ca să trateze cazul?
3. Când alegi `Promise.allSettled` în loc de `Promise.all`?
4. De ce `[10, 9, 1, 100].sort()` nu sortează numeric?
5. De ce ții prețurile în bani (întregi) și nu în lei cu zecimale?
