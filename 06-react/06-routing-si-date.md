# Routing și date

Două lucruri pe care le are orice aplicație reală: mai multe pagini cu URL-uri proprii și date care vin de la un server. Aici înveți cum le leagă React Router și TanStack Query, adică exact combinația din P1, P2 și P3.

## Client-side routing

**Ce e.** Un SPA (Single Page Application) are un singur fișier HTML, `index.html`. Când navighezi de la `/` la `/books/OL123W`, browserul **nu** cere o pagină nouă de la server. Un router din JavaScript schimbă URL-ul (prin History API) și afișează altă componentă.

**De ce contează.** Navigarea e instantanee și state-ul aplicației se păstrează. Are însă o consecință la deploy: dacă cineva deschide direct `/books/OL123W` sau dă refresh, serverul trebuie să-i răspundă tot cu `index.html`, altfel dă 404. De aceea pe Vercel adaugi un rewrite (vezi [02-vercel](../11-deploy/02-vercel.md)).

**Exemplu minim**

```
User clicks <Link to="/books/OL123W">
  → router updates the URL (no request to the server)
  → router renders <BookPage />
  → BookPage reads "OL123W" from the URL and loads the data

User refreshes on /books/OL123W
  → browser asks the server for /books/OL123W
  → server must answer with index.html (rewrite), then the router takes over
```

**Cum îl vezi în cod real.** În loc de `<a href>` pentru navigarea internă vei vedea `<Link to>`. Un `<a href>` obișnuit ar reîncărca toată pagina.

**Capcana.** Aplicația merge perfect local (serverul de dev al lui Vite face rewrite automat) și dă 404 în producție la refresh. E una dintre cele mai frecvente probleme la primul deploy.

<details>
<summary>Întrebare de interviu: Cum funcționează routing-ul într-un SPA și ce trebuie configurat pe server?</summary>

Routerul din browser interceptează navigarea, schimbă URL-ul prin History API și afișează componenta potrivită fără să ceară o pagină nouă. La un refresh sau la un link deschis direct, cererea ajunge totuși la server. Serverul trebuie configurat să întoarcă `index.html` pentru orice rută necunoscută, iar apoi routerul din client preia controlul.
</details>

## React Router în modul declarativ

**Ce e.** React Router e cel mai folosit router pentru React. Are trei moduri. Pentru un SPA Vite folosești cel mai simplu, **declarativ**: rutele se scriu ca JSX cu `<BrowserRouter>`, `<Routes>` și `<Route>`. Pachetul se instalează cu `npm i react-router`.

**De ce contează.** E prima structură pe care o citești într-o aplicație nouă: rutele îți spun ce pagini există.

**Exemplu minim**

```tsx
// main.tsx
import { BrowserRouter, Routes, Route } from "react-router";

createRoot(document.getElementById("root")!).render(
  <BrowserRouter>
    <Routes>
      <Route element={<AppLayout />}>
        <Route index element={<SearchPage />} />
        <Route path="books/:bookId" element={<BookPage />} />
        <Route path="shelf" element={<ShelfPage />} />
        <Route path="*" element={<NotFound />} />
      </Route>
    </Routes>
  </BrowserRouter>,
);

// AppLayout.tsx: shared header, the active page renders in <Outlet />
import { NavLink, Outlet } from "react-router";

export function AppLayout() {
  return (
    <>
      <nav>
        <NavLink to="/">Search</NavLink> <NavLink to="/shelf">My shelf</NavLink>
      </nav>
      <Outlet />
    </>
  );
}
```

**Cum îl vezi în cod real.** Un *layout route* (`<Route element={<AppLayout />}>` fără `path`) învelește paginile cu header-ul comun. `<Outlet />` e locul unde apare pagina curentă. `NavLink` știe dacă e activ, ca să-l poți colora diferit.

**Capcana.** În tutoriale și în cod mai vechi (v6) vei vedea `import ... from "react-router-dom"`. În versiunile noi totul se importă din `react-router`. Dacă Claude amestecă cele două pachete, cere-i să folosească doar `react-router`. Modurile *data* și *framework* sunt pentru aplicații mai complexe, cu încărcare de date în router. În P1–P3, datele le încarcă TanStack Query.

<details>
<summary>Întrebare de interviu: Ce e un nested route sau layout route?</summary>

O rută părinte care afișează un layout comun (header, sidebar) și un `<Outlet />` în care apar rutele copil. Navigarea între copii schimbă doar conținutul din outlet, iar layout-ul rămâne montat. Așa evit să repet header-ul în fiecare pagină și păstrez state-ul layout-ului între pagini.
</details>

## Parametri în URL și search params

**Ce e.** Două feluri de a pune date în URL. **Path params** (`/books/:bookId`) identifică resursa și se citesc cu `useParams()`. **Search params** (`/?q=dune&page=2`) descriu filtre, căutare, paginare și se citesc și se scriu cu `useSearchParams()`.

**De ce contează.** Ce e în URL se poate trimite cuiva, se păstrează la refresh și funcționează cu butonul Back. Un filtru ținut doar în `useState` se pierde la refresh.

**Exemplu minim**

```tsx
import { useParams, useSearchParams } from "react-router";

export function BookPage() {
  const { bookId } = useParams(); // "OL123W"
  // ...
}

export function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const q = searchParams.get("q") ?? "";

  return <input value={q} onChange={(e) => setSearchParams({ q: e.target.value })} />;
}
```

**Cum îl vezi în cod real.** În P2, lista de facturi filtrată după status și lună: `/invoices?status=overdue&month=2026-10`. Contabilul îi poate trimite linkul clientului și vede exact același filtru.

**Capcana.** `useParams()` întoarce mereu string-uri, posibil `undefined`. Un id numeric trebuie convertit și validat (`Number(id)`, verificat cu `Number.isInteger`), iar pagina trebuie să trateze cazul în care parametrul lipsește sau e invalid.

<details>
<summary>Întrebare de interviu: Când pui state în URL în loc de useState?</summary>

Când state-ul descrie *ce* vede utilizatorul și ar trebui să se păstreze la refresh, să poată fi trimis prin link sau să meargă cu Back/Forward: căutare, filtre, paginare, tab-ul activ, resursa deschisă. State-ul efemer de UI, ca un dropdown deschis sau textul dintr-un formular nesalvat, rămâne în `useState`.
</details>

## De ce nu fetch în useEffect peste tot

**Ce e.** Datele de la server nu sunt state obișnuit. Sunt o *copie* a ceva care trăiește în altă parte și se poate schimba oricând. Ca să le încarci corect cu `fetch` și `useEffect`, trebuie să rezolvi singur o listă lungă de probleme.

**De ce contează.** Orice listă de facturi, rezervări sau cărți are aceste probleme. Ori le rezolvi de mână în fiecare componentă, ori folosești o unealtă făcută pentru ele.

**Exemplu minim.** Ce ai de rezolvat cu `useEffect` + `fetch`:

```
□ loading / error / success state for every request
□ race conditions when params change (AbortController, see 04-efecte)
□ cache: going back to a page shouldn't reload everything
□ deduplication: two components need the same data → one request
□ refetch after a mutation (new invoice → list must update)
□ retry on network failure
□ refetch when the user comes back to the tab (data may be stale)
```

**Cum îl vezi în cod real.** În proiectele de 2026, aproape niciodată nu mai vezi `fetch` direct într-un `useEffect` pentru date de afișat. Vezi TanStack Query (în aplicații Vite) sau mecanismele framework-ului (Server Components în Next.js, vezi [08-nextjs](08-nextjs.md)).

**Capcana.** AI-ul încă generează uneori `useEffect` + `fetch` + trei `useState` (data, loading, error). Merge pentru o demonstrație, dar strică ceva la prima interacțiune reală. Cere-i varianta cu TanStack Query.

<details>
<summary>Întrebare de interviu: Ce e server state și de ce e diferit de client state?</summary>

Server state e date care trăiesc pe server (facturi, utilizatori), iar ce ține frontend-ul e doar o copie care se poate învechi. Are nevoie de cache, invalidare, refetch, deduplicare și retry. Client state e date care există doar în browser: un modal deschis, textul dintr-un input, tema. Pentru server state folosesc o bibliotecă dedicată, ca TanStack Query, iar pentru client state ajung de obicei `useState` și Context.
</details>

## TanStack Query: useQuery

**Ce e.** TanStack Query (fost React Query) e o bibliotecă pentru server state. Îi spui *cum* se încarcă datele (`queryFn`) și *sub ce cheie* stau în cache (`queryKey`). Ea se ocupă de loading, erori, cache, deduplicare, retry și refetch.

**De ce contează.** E standardul de facto pentru date în aplicațiile React fără framework. Combinația React + TanStack Query apare foarte des în anunțuri.

**Exemplu minim**

```tsx
// main.tsx: one client for the whole app
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

// <QueryClientProvider client={queryClient}><App /></QueryClientProvider>
```

```tsx
// features/books/api.ts
import { useQuery } from "@tanstack/react-query";

export function useBookSearch(query: string) {
  return useQuery({
    queryKey: ["books", "search", query], // every input of the request goes in the key
    queryFn: () => searchBooks(query),
    enabled: query.trim().length >= 2, // don't search for "" or "a"
  });
}

// component
const { data, isPending, isError, error } = useBookSearch(debouncedQuery);
```

**Cum îl vezi în cod real.** Hook-uri mici, unul pe fiecare tip de date (`useInvoices`, `useInvoice(id)`, `useClients`), în `features/*/api.ts`. Componentele nu apelează niciodată `fetch` direct. Stările vin gata făcute: `isPending` (încă nu avem date), `isError`/`error`, `data`, plus `isFetching` (se reîncarcă în fundal, dar avem deja date de afișat).

**Capcana.** O cheie care nu conține toți parametrii: `queryKey: ["invoices"]` pentru o listă filtrată după status. Când schimbi filtrul, TanStack Query crede că sunt aceleași date și îți arată lista veche. Regula: tot ce intră în `queryFn` intră și în `queryKey`.

<details>
<summary>Întrebare de interviu: Ce rol are queryKey în TanStack Query?</summary>

E identitatea datelor în cache. Două componente cu aceeași cheie împart același request și aceleași date. Când cheia se schimbă, de exemplu cu un filtru nou, biblioteca încarcă automat datele pentru cheia nouă. Cheia trebuie să conțină toți parametrii de care depinde request-ul, altfel primesc date greșite din cache. Tot prin cheie invalidez datele după o modificare.
</details>

## useMutation și invalidateQueries

**Ce e.** `useQuery` e pentru citit. `useMutation` e pentru scris: creare, modificare, ștergere. După ce o mutație reușește, îi spui lui TanStack Query ce date s-au învechit cu `invalidateQueries`, iar el le reîncarcă.

**De ce contează.** E bucla completă din P2: creezi o factură și lista se actualizează singură, fără `window.location.reload()` și fără să ții lista de mână în state.

**Exemplu minim**

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

export function useCreateInvoice() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: CreateInvoiceInput) => api.post<Invoice>("/invoices", input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["invoices"] }); // all invoice lists refetch
    },
  });
}

// component
const createInvoice = useCreateInvoice();
// <button disabled={createInvoice.isPending} onClick={() => createInvoice.mutate(form)}>Create</button>
// {createInvoice.isError && <p role="alert">{createInvoice.error.message}</p>}
```

**Cum îl vezi în cod real.** `invalidateQueries({ queryKey: ["invoices"] })` invalidează și `["invoices", { status: "paid" }]` și orice altă cheie care începe cu `"invoices"`, pentru că potrivirea se face pe prefix. De aceea cheile se construiesc ierarhic, de la general la particular.

**Capcana.** Lipsa invalidării: mutația reușește, dar lista rămâne neschimbată până la refresh. Sau eroarea mutației neafișată: utilizatorul apasă „Save”, nu se întâmplă nimic vizibil și nu știe că a eșuat.

<details>
<summary>Întrebare de interviu: Cum actualizezi o listă după ce ai creat un element nou?</summary>

Cu TanStack Query: fac mutația cu `useMutation`, iar în `onSuccess` invalidez query-urile afectate (`invalidateQueries` cu cheia listei). Biblioteca le reîncarcă de pe server, deci lista reflectă adevărul de acolo. Ca optimizare pot face un update optimist: modific cache-ul imediat și revin dacă request-ul eșuează. Dar varianta de bază e invalidarea.
</details>

## Un strat de API

**Ce e.** Un fișier mic (`lib/api.ts`) care învelește `fetch`: pune URL-ul de bază, header-ele comune (token-ul în P3) și transformă răspunsurile cu status de eroare în excepții. Toate apelurile trec prin el.

**De ce contează.** `fetch` **nu** aruncă excepție la 404 sau 500, doar la erori de rețea. Fără un strat care verifică `res.ok`, TanStack Query crede că un 500 e un succes și îți dă în `data` mesajul de eroare.

**Exemplu minim**

```ts
// lib/api.ts
const BASE_URL = import.meta.env.VITE_API_URL; // e.g. https://invoicer-api.onrender.com

async function request<T>(path: string, init?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE_URL}${path}`, {
    ...init,
    headers: { "Content-Type": "application/json", ...init?.headers },
  });
  if (!res.ok) {
    const body = await res.json().catch(() => null);
    throw new Error(body?.message ?? `Request failed with ${res.status}`);
  }
  return res.status === 204 ? (undefined as T) : res.json();
}

export const api = {
  get: <T>(path: string) => request<T>(path),
  post: <T>(path: string, data: unknown) => request<T>(path, { method: "POST", body: JSON.stringify(data) }),
};
```

**Cum îl vezi în cod real.** `VITE_API_URL` e o variabilă de mediu *publică*. Prefixul `VITE_` înseamnă că ajunge în codul trimis browserului. E în regulă pentru un URL, dar **niciodată** pentru o cheie secretă (vezi [02-ce-ajunge-in-browser](../10-securitate-env/02-ce-ajunge-in-browser.md)).

**Capcana.** `VITE_API_URL=http://localhost:3000` rămas în build-ul de producție. Pe telefonul unui recrutor, `localhost` e telefonul lui, nu serverul tău. Setezi valoarea corectă în Vercel, pe fiecare mediu.

<details>
<summary>Întrebare de interviu: Aruncă fetch eroare la un răspuns 404?</summary>

Nu. Promise-ul de la `fetch` e respins doar la erori de rețea, de exemplu fără conexiune sau DNS eșuat. La 404 sau 500 se rezolvă normal, cu `res.ok` pe `false`. Trebuie să verific `res.ok` sau `res.status` și să arunc eu eroarea. De aceea pun un mic wrapper peste `fetch`, folosit de toate apelurile.
</details>

## Mai departe

- [React Router: Declarative Mode](https://reactrouter.com/start/declarative/installation)
- [TanStack Query: Overview](https://tanstack.com/query/latest/docs/framework/react/overview)
- [TanStack Query: Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys)
