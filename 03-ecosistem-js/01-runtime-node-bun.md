# Runtime: Node.js, Bun și Deno

Limbajul JavaScript și locul în care rulează sunt lucruri diferite. Dacă ai clară distincția asta, înțelegi de ce același cod merge în browser și crapă în Node (sau invers) și de ce un dev de frontend are nevoie de Node, deși aplicația lui rulează în browser.

## Engine vs runtime

**Ce e.** **Engine-ul** e programul care citește și execută cod JavaScript. Cele mai cunoscute sunt V8 (în Chrome, Edge și Node), JavaScriptCore (în Safari și Bun) și SpiderMonkey (în Firefox). Limbajul în sine nu știe să citească fișiere, să facă request-uri sau să afișeze ceva pe ecran. Nici măcar `setTimeout` nu face parte din limbaj. Le primește de la gazdă.

**Runtime-ul** = engine + API-urile gazdei + event loop-ul. Browserul îți dă `document`, `window`, `localStorage`, `fetch`. Node îți dă `fs` (fișiere), `http`, `process`, `path`.

O comparație cu ce știi din C: JavaScript-ul pur e ca C-ul fără biblioteca standard și fără sistem de operare. Runtime-ul e „libc-ul”.

**De ce contează.** Când un cod crapă cu `window is not defined` sau `process is not defined`, problema nu e în JavaScript. Codul rulează în alt runtime decât crezi.

**Exemplu minim**

```js
// where-am-i.js — run it with `node where-am-i.js`, then paste it into the browser console
console.log(typeof window);  // "object" in the browser, "undefined" in Node
console.log(typeof process); // "undefined" in the browser, "object" in Node
console.log(typeof fetch);   // "function" in both — Node has had fetch built in for years
```

**Cum îl vezi în cod real.** În Vite (browser), variabilele de mediu vin din `import.meta.env.VITE_API_URL`. În NestJS (Node), vin din `process.env.DATABASE_URL`. Sunt runtime-uri diferite, cu API-uri diferite.

**Capcana.** Codul care folosește `window` sau `localStorage` și rulează pe server, de exemplu într-un Server Component din Next.js în P4, crapă cu `ReferenceError: window is not defined`. Invers, `process.env` în cod de browser făcut cu Vite e `undefined`.

<details>
<summary>Întrebare de interviu: Ce e Node.js?</summary>

Un runtime de JavaScript construit pe engine-ul V8 de la Chrome, care rulează JavaScript în afara browserului. Adaugă API-uri pentru fișiere, rețea și procese și are un event loop care face operațiile de I/O fără să blocheze. Îl folosești pentru servere și API-uri, dar și pentru toate uneltele de dezvoltare: Vite, TypeScript, ESLint, Prisma.
</details>

## Ce adaugă Node: fs, http, process, path

**Ce e.** Modulele de bază ale Node se importă cu prefixul `node:`, ca să fie clar că sunt incluse în runtime și nu vin din npm:

- `node:fs/promises`: citești și scrii fișiere;
- `node:http`: un server HTTP (vezi [07-backend-node-nest/01-server-http-de-la-zero.md](../07-backend-node-nest/01-server-http-de-la-zero.md));
- `node:path`: lucrul cu căi de fișiere;
- `process`: variabile de mediu (`process.env`), argumente (`process.argv`), oprirea programului (`process.exit`).

**De ce contează.** Toate framework-urile de backend sunt construite peste ele. NestJS folosește Express, Express folosește `node:http`.

**Exemplu minim**

```js
// count-books.mjs — run: node count-books.mjs books.json
import { readFile } from "node:fs/promises";

const file = process.argv[2] ?? "books.json";
const books = JSON.parse(await readFile(file, "utf8")); // top-level await works in ES modules
console.log(`${books.length} books in ${file}`);
console.log(`NODE_ENV = ${process.env.NODE_ENV ?? "not set"}`);
```

**Cum îl vezi în cod real.** Ultima linie din `main.ts`, în orice proiect NestJS:

```ts
await app.listen(process.env.PORT ?? 3000); // Render sets PORT; locally it falls back to 3000
```

**Capcana.** Node rulează codul tău JavaScript pe **un singur thread**. Un calcul greu, de exemplu o buclă care procesează un milion de rânduri, blochează **toate** request-urile până se termină. I/O-ul (baza de date, fișierele, rețeaua) nu blochează, dar calculul pur blochează. Detalii în [04-javascript/02-event-loop-promises.md](../04-javascript/02-event-loop-promises.md).

<details>
<summary>Întrebare de interviu: Node e single-threaded. Cum servește mii de request-uri în același timp?</summary>

Codul JavaScript rulează pe un singur thread, dar operațiile lente de I/O (baza de date, rețeaua, fișierele) sunt date sistemului de operare sau unui thread pool intern. Cât timp așteaptă răspunsul, thread-ul principal servește alte request-uri. Când vine răspunsul, callback-ul sau promisiunea intră în coada event loop-ului. Merge foarte bine pentru aplicații care așteaptă mult I/O, cum sunt API-urile, și prost pentru calcul intens pe CPU, care blochează thread-ul.
</details>

## Node e și unealtă, nu doar server

**Ce e.** Cele mai multe programe pe care le rulezi cu Node nu sunt servere de producție, ci unelte de dezvoltare: Vite, compilatorul TypeScript (`tsc`), linter-ele (oxlint, ESLint), Prettier, Prisma CLI, Vitest. Toate sunt programe instalate din npm. Majoritatea sunt scrise în JavaScript și rulează în Node. Unele, ca oxlint, sunt scrise în Rust și vin ca executabile native, dar le instalezi și le pornești tot prin npm.

**De ce contează.** De asta ai nevoie de Node și pentru un proiect doar de frontend, ca P1. Aplicația rulează în browser, dar uneltele care o construiesc rulează în Node.

**Exemplu minim**

```bash
node --version          # which Node you have
node                    # interactive REPL — type JS, see results; exit with Ctrl+D
node --watch server.mjs # restarts automatically when the file changes
```

Versiunile recente de Node pot rula direct și fișiere `.ts` simple (`node script.ts`): pur și simplu șterg tipurile, fără să le verifice. E util pentru scripturi mici. Pentru proiecte reale, verificarea o face `tsc`, iar rularea o face framework-ul (Vite, NestJS) sau `npx tsx`.

**Capcana.** „Am nevoie de Node doar pentru backend.” Fără Node nu poți nici măcar să pornești serverul de dezvoltare Vite.

## Versiunile de Node și LTS

**Ce e.** Pe nodejs.org vezi mereu două variante: **LTS** (Long Term Support: stabilă, primește patch-uri de securitate mult timp, e cea pentru proiecte) și **Current** (cea mai nouă, cu funcționalități proaspete). Folosește LTS.

**De ce contează.** Proiecte diferite pot cere versiuni diferite de Node. De asta l-ai instalat prin `fnm`, un version manager care schimbă versiunea per proiect (vezi [01-setup/03-node-npm-bun.md](../01-setup/03-node-npm-bun.md)).

**Exemplu minim**

```bash
fnm list                 # versions installed on your machine
node --version > .nvmrc  # pin the current version for this project (fnm reads .nvmrc)
fnm use                  # switch to the version from .nvmrc
```

**Cum îl vezi în cod real.** Render și Vercel te lasă să alegi versiunea de Node. Dacă local ai o versiune și pe server alta, apar bug-uri de tipul „la mine merge”. Un `.nvmrc` sau câmpul `"engines"` din `package.json` le aliniază.

**Capcana.** Să instalezi Node direct, cu `sudo apt install nodejs`. Primești o versiune veche din depozitele Ubuntu, iar instalările globale cer `sudo` și strică permisiunile. Cu `fnm` nu ai nevoie de `sudo`.

## Bun și Deno

**Ce e.** Două runtime-uri alternative la Node:

- **Bun** e construit pe JavaScriptCore și scris în Zig. Pe lângă runtime, e și **package manager**, **bundler** și **test runner**, toate într-un singur executabil. Rulează TypeScript direct și e foarte rapid, mai ales la instalarea pachetelor. Vrea să fie un înlocuitor direct pentru Node și cele mai multe pachete npm merg, dar unele API-uri Node sunt implementate doar parțial.
- **Deno** e făcut de creatorul Node, Ryan Dahl. Pune accent pe securitate: implicit, un script nu are acces la fișiere sau la rețea fără permisiune explicită. Rulează TypeScript direct și e compatibil cu pachetele npm.

| | Node | Bun | Deno |
|---|---|---|---|
| engine | V8 | JavaScriptCore | V8 |
| TypeScript | type stripping simplu, altfel unelte separate | nativ | nativ |
| package manager | npm (inclus) | `bun install` (inclus, foarte rapid) | inclus, compatibil npm |
| folosit în joburi | standardul | în creștere | de nișă |

**De ce contează.** O să auzi de ele la interviuri și în echipe. Trebuie să știi ce sunt și de ce ai alege sau nu ai alege unul dintre ele.

**Cum îl vezi în cod real.** În ghidul ăsta, Node + npm sunt standardul pentru toate proiectele. Bun îl încerci în P1 ca să simți diferența, în [03-npm-vs-bun.md](03-npm-vs-bun.md).

**Capcana.** „Bun e mai rapid, deci e mai bun.” În producție contează compatibilitatea cu framework-ul (NestJS e testat pe Node), ce suportă hosting-ul, ce știe echipa și cât de ușor faci debugging. Viteza de instalare e un confort pentru developer, nu un argument de arhitectură.

<details>
<summary>Întrebare de interviu: Ai auzit de Bun? Ce e și l-ai folosi?</summary>

Bun e un runtime de JavaScript alternativ la Node, construit pe JavaScriptCore. Vine la pachet cu package manager, bundler și test runner și rulează TypeScript direct. Instalarea pachetelor e mult mai rapidă decât cu npm. L-aș folosi liniștit ca package manager sau pentru scripturi și proiecte noi. Pentru un backend în producție aș verifica întâi compatibilitatea cu framework-ul și cu hosting-ul, pentru că Node rămâne standardul cu cel mai bun suport.
</details>

## Mai departe

- [Node.js — Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)
- [Bun — Docs](https://bun.com/docs)
