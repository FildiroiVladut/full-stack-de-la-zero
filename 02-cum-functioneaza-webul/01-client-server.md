# Client și server

Orice aplicație web e o conversație între două programe care, de obicei, rulează pe calculatoare diferite. Înainte de React și NestJS, trebuie să știi la orice linie de cod cine vorbește cu cine și unde rulează ea.

## Client și server

**Ce e.** Clientul e programul care cere (de obicei browserul). Serverul e programul care răspunde. „Server” nu înseamnă un calculator special: e orice program care ascultă pe un port și răspunde la request-uri. Request = cererea trimisă de client; response = răspunsul serverului. În dezvoltare, laptopul tău e și client, și server, în același timp.

**De ce contează.** Fiecare bucată de cod pe care o scrii rulează într-un singur loc: fie în browserul utilizatorului, fie pe server. Dacă nu știi unde rulează, nu știi ce poate vedea utilizatorul, ce acces are codul la baza de date și ce se pierde la refresh.

**Exemplu minim**

```js
// server.mjs — run in WSL with: node server.mjs
import { createServer } from "node:http";

const server = createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ message: "Hello from the server", path: req.url }));
});

server.listen(3000, () => console.log("Listening on http://localhost:3000"));
```

Pornește-l în WSL și deschide `http://localhost:3000/abc` în browserul din Windows. Browserul e clientul, procesul `node` e serverul. Oprește serverul cu `Ctrl+C` și dă refresh: primești eroare de conexiune, pentru că nu mai ascultă nimeni.

**Cum îl vezi în cod real.** În P2 Invoicer o să ai două terminale deschise tot timpul:

```bash
# terminal 1 — frontend (Vite serves the React app)
cd ~/code/invoicer/web && npm run dev        # http://localhost:5173

# terminal 2 — backend (NestJS API)
cd ~/code/invoicer/api && npm run start:dev  # http://localhost:3000
```

- Sunt două servere diferite. Vite doar livrează fișierele aplicației React către browser, iar codul React rulează **în browser**.
- NestJS rulează în procesul Node din terminalul 2 și doar el vorbește cu baza de date.

**Capcana.** Să crezi că un cod rulează pe server doar pentru că stă în același proiect cu serverul. Întreabă-te mereu: _cine execută linia asta, browserul sau procesul Node?_

<details>
<summary>Întrebare de interviu: Ce e un server, de fapt?</summary>

Un program care ascultă pe un port și răspunde la request-uri, de obicei prin HTTP. Nu e neapărat o mașină dedicată: același laptop poate rula un server Vite, un server NestJS și un browser care e client pentru amândouă. În producție, serverele rulează pe mașini din cloud, dar principiul e același.
</details>

## Anatomia unui URL

**Ce e.** URL-ul e adresa completă a unei resurse. Fiecare bucată are un rol:

```
https://api.invoicer.app:8443/invoices/42/lines?status=paid&page=2#totals
└─┬─┘   └──────┬───────┘ └┬─┘└───────┬────────┘└───────┬───────┘└──┬──┘
protocol     host       port       path            query string  fragment
```

- **protocol**: `http` sau `https` (criptat). În producție, mereu `https`.
- **host**: domeniul (`api.invoicer.app`), unde `api` e un subdomeniu.
- **port**: implicit 443 pentru https și 80 pentru http, deci de obicei nu apare.
- **path**: ce resursă vrei.
- **query string**: parametri de filtrare, după `?`, separați cu `&`.
- **fragment**: după `#`. **Nu se trimite la server**, îl folosește doar browserul.

**De ce contează.** Path-ul și query-ul sunt exact ce citește backend-ul tău. Protocolul, host-ul și portul formează **origin-ul**, iar de origin depinde CORS ([05-auth-cookies-cors.md](05-auth-cookies-cors.md)).

**Exemplu minim**

```js
const url = new URL("https://api.invoicer.app:8443/invoices/42/lines?status=paid&page=2#totals");
console.log(url.origin);                     // https://api.invoicer.app:8443
console.log(url.pathname);                   // /invoices/42/lines
console.log(url.searchParams.get("page"));   // "2"  (a string, not a number)
console.log(url.hash);                       // #totals
```

**Cum îl vezi în cod real.** În P1 Shelf construiești URL-ul de căutare spre Open Library:

```ts
const params = new URLSearchParams({ q: query, page: String(page), limit: "20" });
const res = await fetch(`https://openlibrary.org/search.json?${params}`);
```

- `URLSearchParams` face encoding-ul corect: spațiul, `&` și diacriticele nu mai strică URL-ul.

**Capcana.** URL-ul construit prin concatenare: `` `?q=${query}` ``. Dacă utilizatorul caută „Tom & Jerry”, `&` taie query-ul în două. AI-ul scrie des varianta asta. Cere-i `URLSearchParams` sau `encodeURIComponent`.

<details>
<summary>Întrebare de interviu: Ce e un origin?</summary>

Combinația dintre protocol, host și port, de exemplu `https://app.invoicer.app:443`. Două URL-uri au același origin doar dacă toate trei coincid. `http://localhost:5173` și `http://localhost:3000` sunt origin-uri diferite, din cauza portului. Browserul folosește origin-ul pentru same-origin policy și CORS.
</details>

## DNS pe scurt

**Ce e.** DNS (Domain Name System) e agenda telefonică a internetului: transformă un nume (`openlibrary.org`) într-o adresă IP (de forma `207.241.224.2`), adică numărul la care calculatorul chiar se conectează. Răspunsurile sunt ținute în cache un timp, numit TTL (time to live).

**De ce contează.** La deploy primești un domeniu de la Vercel sau Render. Dacă vrei domeniul tău, adaugi o înregistrare DNS (un `A` sau un `CNAME`) la firma de la care l-ai cumpărat.

**Exemplu minim**

```bash
nslookup openlibrary.org        # if missing: sudo apt install dnsutils
```

**Capcana.** Schimbi o înregistrare DNS și site-ul tot nu merge, deși setările sunt corecte. Cache-urile vechi trăiesc până expiră TTL-ul. Așteaptă puțin înainte să schimbi altceva.

<details>
<summary>Întrebare de interviu: Ce se întâmplă când scrii un URL în browser și apeși Enter?</summary>

1. Browserul află IP-ul domeniului prin DNS (întâi din cache, apoi întreabă un resolver).
2. Deschide o conexiune TCP spre IP-ul ăla, iar pentru https negociază criptarea (TLS).
3. Trimite un request HTTP `GET` pentru path și primește un response, de obicei HTML.
4. Parsează HTML-ul și face request-uri noi pentru CSS, JS și imagini.
5. Execută JavaScript-ul, care la rândul lui poate face request-uri `fetch` spre un API.
6. Construiește pagina și o desenează pe ecran.

Nu trebuie să știi detaliile TCP. Contează să poți povesti drumul cap-coadă.
</details>

## localhost și porturi

**Ce e.** `localhost` (adică `127.0.0.1`) înseamnă „calculatorul ăsta”. Portul e un număr între 0 și 65535 care spune _cărui program_ de pe calculator îi e destinată conexiunea. Pe un port poate asculta un singur program odată.

| Port | Cine ascultă de obicei |
|---|---|
| 80 / 443 | HTTP / HTTPS în producție |
| 3000 | NestJS, Express (convenție) |
| 5173 | Vite dev server |
| 4173 | `vite preview` |
| 5432 | PostgreSQL |

**De ce contează.** O să rulezi mereu mai multe servere în paralel: frontend, backend, poate un preview. Fiecare are portul lui.

**Notă pentru WSL.** Serverele pornite în WSL pe `localhost` se văd din browserul din Windows tot pe `localhost:PORT`, pentru că WSL redirecționează automat. Dacă rar se întâmplă să nu meargă, pornește Vite cu `npm run dev -- --host`: așa ascultă pe toate interfețele, nu doar pe localhost.

**Exemplu minim**

```bash
node server.mjs   # in one terminal
node server.mjs   # in a second terminal → Error: listen EADDRINUSE: address already in use :::3000

ss -ltnp | grep 3000   # which process is listening on port 3000?
kill <PID>             # stop it (PID is the number shown by ss)
```

**Capcana.** `EADDRINUSE`: ai uitat un server pornit într-un tab vechi de terminal. Înainte să schimbi portul, caută-l pe cel uitat.

<details>
<summary>Întrebare de interviu: De ce aplicația ta merge pe localhost, dar nu și de pe telefonul din aceeași rețea?</summary>

`localhost` înseamnă „mașina asta”. Pe telefon, `localhost` e telefonul însuși. Trebuie folosit IP-ul laptopului din rețeaua locală, iar serverul trebuie să asculte pe toate interfețele (`0.0.0.0`), nu doar pe `127.0.0.1`. Vite face asta cu `--host`. În WSL mai e un strat de rețea la mijloc, deci e și mai complicat. Pentru test pe telefon, cel mai simplu e un deploy de preview.
</details>

## Ce rulează unde

**Ce e.** Harta pe care o ții în cap pentru fiecare proiect:

| Cod / date | Rulează / stă în | Cine îl poate vedea |
|---|---|---|
| Componentele React (P1–P3) | browserul utilizatorului | **oricine** (bundle-ul se descarcă) |
| Controllere și services NestJS (P2–P3) | serverul de pe Render | doar tu |
| Datele din Postgres | serverul Supabase | cine are credențialele |
| `localStorage` (P1) | browserul utilizatorului, per origin | doar el, doar pe acel device |
| Server Components / route handlers Next.js (P4) | server | doar tu |
| Client Components Next.js (P4) | browser | oricine |

**De ce contează.** De harta asta depind securitatea ([10-securitate-env](../10-securitate-env/)) și unde pui fiecare bucată de logică.

**Exemplu minim.** În P1, rafturile tale de cărți stau în `localStorage`. Deschide aplicația pe alt browser și sunt goale. Nu e un bug: datele nu au ajuns niciodată pe un server. Asta e motivul pentru care P2 are backend și bază de date.

**Capcana.** „Dacă ascund butonul de ștergere în UI, utilizatorul nu poate șterge.” Ba poate: oricine poate trimite request-ul direct cu `curl`, fără interfața ta. Interfața e doar o comoditate. Regulile se aplică pe server.

<details>
<summary>Întrebare de interviu: Unde ai calcula totalul unei facturi și de ce?</summary>

Pe server. Orice vine din browser poate fi modificat: cineva poate edita request-ul și trimite un total fals. Frontend-ul poate arăta un total provizoriu, ca să răspundă repede interfața, dar valoarea salvată în baza de date o calculează backend-ul din liniile facturii.
</details>

## Mai departe

- [MDN — How the web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Web_standards/How_the_web_works)
- [MDN — What is a URL?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL)
