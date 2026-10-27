# async/await și fetch

Cum arată codul asincron scris azi: `async/await` peste Promise-uri, `fetch` pentru request-uri HTTP și tratarea corectă a erorilor. Exemplele folosesc Open Library, API-ul public din P1, care nu cere cheie. Le poți rula direct cu `node fisier.mjs` în WSL.

## async/await

**Ce e.** Sintaxă peste Promise-uri care face codul asincron să se citească de sus în jos, ca unul sincron. O funcție `async` întoarce **mereu** un Promise. `await` pune pe pauză *funcția*, nu firul, până când Promise-ul se rezolvă.

**De ce contează.** În forma asta vei scrie și citi aproape tot codul asincron: în React, în Nest, în apelurile la Claude API.

**Exemplu minim**
```js
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

async function brew() {
  console.log("start");
  await wait(1000);       // brew() pauses here; the rest of the program keeps running
  console.log("coffee ready");
  return "espresso";      // the caller receives a Promise that resolves to "espresso"
}

brew().then((drink) => console.log(drink));
console.log("meanwhile..."); // printed before "coffee ready"
```

**Capcana.** Uiți `await`, iar codul continuă cu un Promise în loc de valoare: `const books = getBooks(); books.map(...)` → `TypeError: books.map is not a function`. Când vezi `Promise { <pending> }` în consolă, de cele mai multe ori lipsește un `await`.

<details>
<summary>Întrebare de interviu: Ce întoarce o funcție async?</summary>

Mereu un Promise. Dacă funcția face `return x`, Promise-ul se rezolvă cu `x`; dacă aruncă o eroare, Promise-ul e respins cu ea. `await` așteaptă un Promise fără să blocheze firul: funcția curentă se oprește, iar restul programului continuă prin event loop.
</details>

## Erori: try/catch în jurul lui await

**Ce e.** Un Promise respins pe care îl aștepți cu `await` aruncă eroarea exact în locul acela, așa că o prinzi cu `try/catch`, ca pe orice eroare sincronă.

**Exemplu minim**
```js
async function loadInvoice(id) {
  try {
    const invoice = await findInvoice(id); // may reject
    return invoice;
  } catch (err) {
    console.error(`Could not load invoice ${id}:`, err.message);
    throw err; // re-throw so the caller knows it failed
  } finally {
    console.log("loading finished");
  }
}
```

**Capcana.** Un `catch` care doar face `console.log` și nu mai aruncă eroarea mai departe („înghite” eroarea). Funcția întoarce `undefined`, iar codul de mai sus crede că totul a mers. Prinde eroarea doar acolo unde chiar poți face ceva cu ea: afișezi un mesaj, încerci din nou, întorci o valoare implicită. În rest, las-o să urce.

## fetch: request-uri HTTP

**Ce e.** Funcția standard pentru request-uri HTTP, disponibilă în browser și în Node. Întoarce un Promise cu un obiect `Response`, din care citești status-ul și body-ul.

**De ce contează.** Orice frontend vorbește cu backend-ul prin `fetch` sau printr-o bibliotecă construită peste el.

**Exemplu minim**
```js
const res = await fetch("https://openlibrary.org/search.json?q=dune&limit=3");
console.log(res.status, res.ok); // 200 true
const data = await res.json();   // body parsing is async too
for (const book of data.docs) {
  console.log(book.title, book.first_publish_year, book.author_name?.[0]);
}
```

**Capcana, cea mai importantă din fișier.** `fetch` **nu** aruncă eroare la 404 sau 500. Promise-ul e respins doar când nu există deloc răspuns: rețea căzută, domeniu inexistent, request anulat. Un răspuns 404 e tot un răspuns, deci verifici singur `res.ok`, care e `true` pentru status 200–299:

```js
const res = await fetch("https://openlibrary.org/works/OL_NOPE_W.json");
console.log(res.status, res.ok); // 404 false — and no exception was thrown
```

**Cum îl vezi în cod real.** Aproape orice proiect are un mic helper care face verificarea o singură dată:
```ts
export async function apiFetch<T>(url: string, init?: RequestInit): Promise<T> {
  const res = await fetch(url, init);
  if (!res.ok) {
    throw new Error(`Request failed: ${res.status} ${res.statusText}`);
  }
  return res.json() as Promise<T>;
}
```
- Helper-ul transformă „răspuns cu status de eroare” în excepție, ca restul codului să aibă un singur mod de a trata erorile.
- `as Promise<T>` e o promisiune făcută compilatorului, nu o verificare. Despre ce se întâmplă dacă serverul trimite altceva vezi [05-typescript/05-validare-cu-zod.md](../05-typescript/05-validare-cu-zod.md).

<details>
<summary>Întrebare de interviu: fetch aruncă eroare la un răspuns 404?</summary>

Nu. Promise-ul de la `fetch` e respins doar la erori de rețea sau la anulare. Un 404 sau 500 se rezolvă normal, cu `res.ok === false`. De aceea verific `res.ok` sau `res.status` după fiecare `fetch` și arunc eroarea eu, de obicei într-un helper comun.
</details>

## JSON: drumul datelor prin rețea

**Ce e.** JSON e formatul text prin care frontend-ul și backend-ul își trimit date. `JSON.stringify` transformă un obiect în text, `JSON.parse` (sau `res.json()`) face invers.

**Exemplu minim** (cum trimiți date la un API)
```js
const res = await fetch("https://api.example.com/invoices", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ clientId: 7, dueDate: "2026-12-01", items: [] }),
});
```

**Capcana.** JSON nu are tip pentru dată calendaristică: un `Date` devine string ISO (`"2026-12-01T00:00:00.000Z"`), iar la `parse` rămâne string. Același lucru se întâmplă cu `undefined` și funcțiile, care dispar. Dacă uiți header-ul `Content-Type`, multe backend-uri, inclusiv Nest, nu interpretează body-ul ca JSON și primesc un obiect gol.

## Timeout și anulare: AbortController

**Ce e.** Un mecanism prin care oprești un request în curs. `AbortController` îți dă un `signal` pe care îl dai lui `fetch`; când apelezi `abort()`, request-ul se oprește și `fetch` aruncă o eroare cu `name === "AbortError"`. Pentru timeout simplu există `AbortSignal.timeout(ms)`, iar eroarea are atunci `name === "TimeoutError"`.

**De ce contează.** În P1, utilizatorul scrie în căutare „d”, „du”, „dun”, „dune”: patru request-uri, din care doar ultimul te interesează. Răspunsurile pot veni în altă ordine decât au plecat, și fără anulare poți afișa rezultatele pentru „du” peste cele pentru „dune”.

**Exemplu minim**
```js
try {
  const res = await fetch("https://openlibrary.org/search.json?q=dune", {
    signal: AbortSignal.timeout(5000), // give up after 5 seconds
  });
  console.log(res.status);
} catch (err) {
  if (err.name === "TimeoutError") console.log("Open Library is slow, try again");
  else throw err;
}
```

**Cum îl vezi în cod real.** În React nu scrii asta de mână: TanStack Query îți dă un `signal` în `queryFn` și anulează singur request-urile vechi. Vezi [06-react/06-routing-si-date.md](../06-react/06-routing-si-date.md).

## Paralel vs secvențial

**Ce e.** Un `await` pus într-o buclă rulează operațiile pe rând. `Promise.all` le pornește pe toate deodată.

**Exemplu minim**
```js
const ids = ["OL893414W", "OL893461W", "OL893474W"];
const getWork = (id) =>
  fetch(`https://openlibrary.org/works/${id}.json`).then((r) => r.json());

// sequential: total time = sum of all requests
for (const id of ids) {
  const work = await getWork(id);
  console.log(work.title);
}

// parallel: total time ≈ the slowest request
const works = await Promise.all(ids.map(getWork));
console.log(works.map((w) => w.title));
```

**Capcana.** `forEach` cu o funcție `async` **nu așteaptă**:
```js
ids.forEach(async (id) => { await getWork(id); });
console.log("done"); // printed immediately, before any request finishes
```
Folosește `for...of` cu `await` când ordinea contează sau trebuie să respecți un rate limit. Folosește `Promise.all(arr.map(...))` când operațiile sunt independente.

<details>
<summary>Întrebare de interviu: Cum rulezi mai multe request-uri în paralel cu async/await?</summary>

Pornesc toate Promise-urile fără `await`, de obicei cu `arr.map(fn)`, apoi aștept rezultatul cu `await Promise.all(...)`. Timpul total e cel al celui mai lent request, nu suma lor. Când contează ordinea sau API-ul are rate limit, folosesc `for...of` cu `await`. `forEach` nu e potrivit, pentru că ignoră Promise-urile întoarse de callback.
</details>

## Cele patru stări ale datelor din UI

**Ce e.** Orice date încărcate de pe rețea trec prin aceleași stări: **loading** (în curs), **error** (a picat), **success cu date** și **success fără date** (zero rezultate). Un UI bun le tratează pe toate patru.

**Exemplu minim** (logica, fără framework)
```js
async function search(query) {
  render({ status: "loading" });
  try {
    const res = await fetch(
      `https://openlibrary.org/search.json?q=${encodeURIComponent(query)}&limit=10`,
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const { docs } = await res.json();
    render(docs.length ? { status: "success", books: docs } : { status: "empty" });
  } catch (err) {
    render({ status: "error", message: err.message });
  }
}
```

**Cum îl vezi în cod real.** În React, TanStack Query îți dă `isPending`, `isError`, `error` și `data`, iar tu decizi doar ce afișezi pentru fiecare stare.

**Capcana.** Codul generat de AI tratează adesea doar cazul fericit. La review, întreabă-te mereu: ce vede utilizatorul cât se încarcă, când pică rețeaua și când sunt zero rezultate? Și `encodeURIComponent`: fără el, o căutare cu `&` sau `#` strică URL-ul.

## Mai departe

- [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN — async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN — AbortSignal.timeout()](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static)
