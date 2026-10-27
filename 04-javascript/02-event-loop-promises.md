# Event loop și Promises

JavaScript rulează pe un singur fir de execuție, dar un server Node servește mii de request-uri, iar o pagină nu se blochează cât așteaptă răspunsul de la API. Fișierul ăsta explică de ce. Event loop-ul e una dintre cele mai frecvente întrebări de interviu pentru orice rol JavaScript.

## Call stack-ul: un singur fir

**Ce e.** Call stack-ul e stiva de apeluri de funcții, la fel ca în C. JavaScript are o singură stivă și execută o singură bucată de cod la un moment dat. Nu există thread-uri care rulează funcțiile tale în paralel, cu memorie comună (cu excepția Web Workers / worker threads, pe care nu le vei folosi curând).

**De ce contează.** Codul tău sincron nu e întrerupt niciodată la jumătate. Prețul e că, dacă o funcție durează mult, nimic altceva nu se mai întâmplă până se termină.

**Exemplu minim**
```js
function c() { console.log("c"); }
function b() { c(); console.log("b"); }
function a() { b(); console.log("a"); }
a(); // c, b, a — each function finishes before the one below it continues
```

**Capcana.** Dacă vii din C, ai putea crede că `setTimeout(fn, 0)` rulează `fn` „imediat, în paralel”. Nu e așa: `fn` rulează abia după ce stiva se golește.

## Cine așteaptă de fapt: Web APIs și libuv

**Ce e.** Când ceri ceva lent (un timer, un request de rețea, un fișier de pe disc), JavaScript nu așteaptă. Dă sarcina mediului în care rulează: browserului (prin Web APIs) sau, în Node, bibliotecii **libuv**, scrisă în C, care vorbește cu sistemul de operare. Când sarcina e gata, callback-ul tău e pus într-o coadă, iar event loop-ul îl mută pe stivă când aceasta e goală.

**De ce contează.** Așa servește Node mii de conexiuni cu un singur fir: cât timp un request așteaptă baza de date, firul se ocupă de alte request-uri.

```
   your code (call stack)          the environment (browser / libuv)
   ┌───────────────┐   fetch(...)  ┌──────────────────────────┐
   │ main()        │ ────────────► │ network, timers, disk    │
   │               │               └────────────┬─────────────┘
   └───────▲───────┘                            │ done → callback
           │                         ┌──────────▼───────────┐
           │  event loop: when the   │ queues (microtasks,  │
           └─ stack is empty, takes  │ then tasks)          │
              the next callback ◄────┴──────────────────────┘
```

Seamănă cu pass-ul din bucătărie: bucătarul nu stă lângă cuptor cât se coace pâinea. Pune timer-ul, trece la altă comandă și se întoarce când sună.

## Microtask queue vs task queue

**Ce e.** Există două cozi cu priorități diferite:
- **microtask queue:** callback-urile de la Promise-uri (`.then`, `await`) și `queueMicrotask`;
- **task queue** (numită și macrotask): `setTimeout`, `setInterval`, evenimente, I/O.

Regula: după fiecare bucată de cod sincron, event loop-ul golește **toate** microtask-urile, abia apoi ia **un** task.

**Exemplu minim** (e întrebarea clasică)
```js
console.log("1 sync");
setTimeout(() => console.log("2 timeout"), 0);
Promise.resolve().then(() => console.log("3 microtask"));
queueMicrotask(() => console.log("4 queueMicrotask"));
console.log("5 sync");
// 1 sync, 5 sync, 3 microtask, 4 queueMicrotask, 2 timeout
```

**Capcana.** Mulți răspund la interviu că `setTimeout(..., 0)` vine înaintea lui `.then`, pentru că „0 înseamnă imediat”. Ține minte ordinea: tot codul sincron, apoi toate microtask-urile, apoi un task.

<details>
<summary>Întrebare de interviu: Explică event loop-ul.</summary>

JavaScript are un singur call stack. Operațiile lente, cum ar fi timer-ele, rețeaua sau discul, sunt delegate mediului (browser sau libuv în Node), iar callback-urile lor ajung în cozi când sunt gata. Event loop-ul așteaptă ca stiva să se golească, apoi rulează toate microtask-urile (Promise-uri, `queueMicrotask`) și după aceea următorul task (`setTimeout`, evenimente, I/O). De aceea un `.then` rulează înaintea unui `setTimeout(fn, 0)`, deși amândouă sunt asincrone.
</details>

## Promise: o valoare care vine mai târziu

**Ce e.** Un Promise e un obiect care reprezintă rezultatul unei operații încă neterminate. Are trei stări: `pending` (în lucru), `fulfilled` (a reușit, are o valoare) și `rejected` (a eșuat, are o eroare). Odată ieșit din `pending`, nu se mai schimbă.

**De ce contează.** Orice operație asincronă din stack-ul tău întoarce un Promise: `fetch`, query-urile Prisma, apelurile la Claude API. Pe majoritatea le vei *consuma*, rar le vei *crea*.

**Exemplu minim**
```js
const getQuote = (supplier, ms, price) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      if (price == null) reject(new Error(`${supplier}: no price`));
      else resolve({ supplier, price });
    }, ms);
  });

getQuote("Metro", 300, 12.4)
  .then((quote) => console.log("got", quote))
  .catch((err) => console.error("failed:", err.message))
  .finally(() => console.log("done either way"));
```

**Cum îl vezi în cod real.** Aproape mereu cu `await` (fișierul următor). `.then` mai apare în cod vechi sau în lanțuri scurte:
```ts
const products = await prisma.product.findMany(); // findMany returns a Promise
```
- `new Promise(...)` scris de mână apare rar, mai ales ca să „promisifici” un API vechi bazat pe callback-uri.
- `.then` întoarce **un Promise nou**. De aceea poți înlănțui, iar ce întorci dintr-un `.then` ajunge în următorul.

**Capcana.** Un Promise respins pe care nu-l prinde nimeni (fără `.catch` sau `try/catch`) e un _unhandled rejection_. În versiunile actuale de Node, asta oprește procesul. În browser apare doar ca eroare roșie în consolă și e ușor de ratat.

<details>
<summary>Întrebare de interviu: Ce stări are un Promise?</summary>

`pending` cât timp operația rulează, apoi `fulfilled` cu o valoare sau `rejected` cu o eroare. Tranziția se face o singură dată: un Promise rezolvat rămâne rezolvat. `.then` primește valoarea, `.catch` eroarea, iar `.finally` rulează în ambele cazuri. Fiecare dintre ele întoarce un Promise nou, de aceea se pot înlănțui.
</details>

## Mai multe Promise-uri deodată: all, allSettled, race, any

**Ce e.** Patru funcții care combină mai multe Promise-uri într-unul:

| Funcție | Se rezolvă când | Eșuează când | Folosești pentru |
|---|---|---|---|
| `Promise.all` | toate reușesc → array cu valori | **primul** eșuează | ai nevoie de toate rezultatele |
| `Promise.allSettled` | toate s-au terminat, oricum | niciodată | vrei rezultatul fiecăruia, chiar dacă unele pică |
| `Promise.race` | primul care se termină (reușită sau eșec) | primul care se termină e eșec | timeout: „operația sau ceasul, care vine primul” |
| `Promise.any` | primul care **reușește** | toate eșuează (`AggregateError`) | prima sursă care răspunde |

**De ce contează.** E diferența dintre a aștepta 3 secunde sau 9 pentru trei operații independente de câte 3 secunde.

**Exemplu minim** (P4: ceri prețuri de la trei furnizori)
```js
// reuses getQuote from above
const quotes = await Promise.allSettled([
  getQuote("Metro", 300, 12.4),
  getQuote("Selgros", 500, 11.9),
  getQuote("Local farm", 200, null), // this one fails
]);
for (const q of quotes) {
  if (q.status === "fulfilled") console.log(q.value.supplier, q.value.price);
  else console.log("skipped:", q.reason.message);
}
```
Cu `Promise.all`, eșecul fermei ar fi aruncat eroare și ai fi pierdut și prețurile de la Metro și Selgros.

**Cum îl vezi în cod real.**
```ts
// P2: dashboard needs three independent queries — run them in parallel
const [unpaid, overdue, paidThisMonth] = await Promise.all([
  invoicesService.sumByStatus("sent"),
  invoicesService.sumOverdue(),
  invoicesService.sumPaidSince(startOfMonth),
]);
```
- Destructuring-ul pe array păstrează ordinea din input, nu ordinea în care s-au terminat.

**Capcana.** `Promise.all` cu 500 de elemente pornește 500 de operații deodată. La un API extern (cum e Claude API în P4) asta înseamnă rate limit și erori `429`. Pentru volume mari procesezi în loturi sau secvențial.

<details>
<summary>Întrebare de interviu: Promise.all vs Promise.allSettled?</summary>

`Promise.all` se rezolvă cu toate valorile dacă toate reușesc, dar e respins imediat ce pică unul, iar rezultatele celelalte se pierd. `Promise.allSettled` așteaptă să se termine toate și îmi dă pentru fiecare `{ status, value }` sau `{ status, reason }`. Folosesc `all` când am nevoie de toate rezultatele ca să continui, și `allSettled` când rezultatele parțiale au valoare, de exemplu la procesarea unui lot de facturi.
</details>

## Nu bloca event loop-ul

**Ce e.** Pentru că există un singur fir, o funcție sincronă lentă blochează tot. În browser, pagina îngheață. În Node, **toți** utilizatorii serverului așteaptă.

**Exemplu minim**
```js
setTimeout(() => console.log("timer fired"), 0);
const start = Date.now();
while (Date.now() - start < 2000) {} // 2 seconds of synchronous work
console.log("loop done"); // "timer fired" appears only after this
```

**Cum îl vezi în cod real.** Rar ca buclă `while`, mai des așa: `JSON.parse` pe un fișier de 200 MB, sortarea unui array uriaș, generarea unui PDF într-un request handler, `fs.readFileSync` în cod de server. Soluțiile sunt variantele asincrone ale API-urilor, munca împărțită în bucăți sau mutată într-un job separat.

**Capcana.** Faptul că funcția e marcată `async` nu face codul din ea neblocant. O buclă grea într-o funcție `async` blochează la fel de mult.

<details>
<summary>Întrebare de interviu: De ce e Node bun pentru I/O, dar slab pentru calcule grele?</summary>

Node rulează JavaScript pe un singur fir, iar I/O-ul e delegat lui libuv și sistemului de operare. Cât timp un request așteaptă baza de date sau rețeaua, firul servește alte request-uri, așa că mii de conexiuni concurente costă puțin. Un calcul CPU-intensiv ține însă firul ocupat și blochează toate request-urile. Pentru asta muți munca în worker threads, într-un serviciu separat sau într-o coadă de job-uri.
</details>

## Mai departe

- [MDN — The event loop / JavaScript execution model](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [MDN — Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [javascript.info — Event loop: microtasks and macrotasks](https://javascript.info/event-loop)
