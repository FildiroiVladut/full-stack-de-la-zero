# HTTP

HTTP e limba în care vorbesc clientul și serverul. Fiecare pagină încărcată, fiecare `fetch` și fiecare apel de API e un request HTTP urmat de un response. Dacă știi să citești un request și un response, rezolvi majoritatea problemelor din web.

## Request și response

**Ce e.** Mesaje text cu o structură fixă:

- **request**: metodă + path + headers + body opțional;
- **response**: status code + headers + body opțional.

HTTP e **stateless**: fiecare request e independent, iar serverul nu ține minte nimic despre tine între două request-uri dacă nu îi trimiți tu ceva care să te identifice (un cookie sau un token).

**De ce contează.** Când ceva nu merge, întrebarea e mereu aceeași: _ce request a plecat și ce response s-a întors?_ Restul sunt detalii.

**Exemplu minim.** Un request și un response, exact cum circulă (simplificat):

```http
POST /invoices HTTP/1.1
Host: api.invoicer.app
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...

{"clientId": 7, "dueDate": "2026-11-30"}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /invoices/42

{"id": 42, "number": "INV-2026-0042", "status": "draft"}
```

**Cum îl vezi în cod real.** În NestJS (P2) nu scrii HTTP de mână. Framework-ul îl parsează pentru tine:

```ts
@Controller("invoices")
export class InvoicesController {
  constructor(private readonly invoices: InvoicesService) {}

  @Post() // handles POST /invoices; Nest answers 201 by default for POST
  create(@Body() dto: CreateInvoiceDto) {
    return this.invoices.create(dto); // the returned object becomes the JSON body
  }
}
```

- `@Body()` e body-ul request-ului, deja transformat din JSON în obiect.
- Ce returnezi devine body-ul response-ului.

**Capcana.** Să presupui că serverul „știe” cine ești pentru că te-ai logat acum un minut. Nu știe. Fiecare request trebuie să-și ducă singur dovada de identitate.

<details>
<summary>Întrebare de interviu: HTTP e stateless. Atunci cum rămâi logat pe un site?</summary>

Clientul trimite la fiecare request ceva care îl identifică. Varianta clasică e un cookie cu un session ID, pe care serverul îl caută în baza de date. Varianta modernă e un token (de obicei JWT) în headerul `Authorization`, pe care serverul îl verifică criptografic. Protocolul rămâne stateless, starea e dusă de client la fiecare request.
</details>

## Metodele

**Ce e.** Metoda spune _ce vrei să faci_ cu resursa din path.

| Metodă | Ce face | Are body? | Exemplu în P2 |
|---|---|---|---|
| `GET` | citește | nu | `GET /invoices?status=overdue` |
| `POST` | creează sau declanșează o acțiune | da | `POST /invoices` |
| `PUT` | înlocuiește resursa întreagă | da | `PUT /clients/7` |
| `PATCH` | modifică parțial | da | `PATCH /invoices/42` cu `{ "status": "sent" }` |
| `DELETE` | șterge | de obicei nu | `DELETE /clients/7` |
| `OPTIONS` | „ce am voie să fac?” (îl trimite browserul la CORS) | nu | automat |

**De ce contează.** Metoda corectă face API-ul previzibil, iar unele unelte (cache, browser, proxy-uri) se bazează pe ea. Un `GET` nu trebuie să schimbe date, niciodată.

**Exemplu minim**

```js
await fetch("http://localhost:3000/invoices/42", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ status: "sent" }),
});
```

**Capcana.** `GET /invoices/42/delete` sau un link `<a>` care șterge ceva. Browserele fac prefetch, crawlerele urmează linkuri, iar proxy-urile pun `GET`-urile în cache. Datele dispar „de la sine”.

<details>
<summary>Întrebare de interviu: Care e diferența dintre PUT și PATCH?</summary>

`PUT` înlocuiește resursa întreagă cu ce trimiți: dacă lipsește un câmp, el se pierde sau e resetat. `PATCH` modifică doar câmpurile trimise. În practică, majoritatea API-urilor folosesc `PATCH` pentru editare, pentru că formularul trimite doar ce s-a schimbat.
</details>

## Status codes care contează

**Ce e.** Un număr din trei cifre care spune cum s-a terminat request-ul. Prima cifră e categoria: `2xx` a mers, `3xx` mergi în altă parte, `4xx` clientul a greșit, `5xx` serverul a greșit.

| Cod | Când |
|---|---|
| `200 OK` | a mers, uite datele |
| `201 Created` | am creat resursa (după `POST`) |
| `204 No Content` | a mers, nu am ce să-ți trimit (des după `DELETE`) |
| `301` / `302` | redirect |
| `304 Not Modified` | ai deja versiunea asta în cache |
| `400 Bad Request` | date invalide (validarea a picat) |
| `401 Unauthorized` | nu știu cine ești: lipsește token-ul sau e expirat |
| `403 Forbidden` | știu cine ești, dar n-ai voie |
| `404 Not Found` | resursa nu există |
| `409 Conflict` | conflict cu starea curentă (slot deja rezervat, număr de factură duplicat) |
| `422 Unprocessable Content` | date corecte ca format, dar invalide semantic (unele API-uri îl folosesc în loc de 400) |
| `429 Too Many Requests` | rate limit |
| `500 Internal Server Error` | bug pe server |
| `502` / `503` | serverul din spate nu răspunde sau e oprit (des la cold start pe Render) |

**De ce contează.** Codul corect îi permite frontend-ului să reacționeze corect: la `401` duci utilizatorul la login, la `409` îi arăți „slot ocupat”, la `500` un mesaj generic.

**Exemplu minim**

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://openlibrary.org/search.json?q=hobbit   # 200
curl -s -o /dev/null -w "%{http_code}\n" https://openlibrary.org/this-does-not-exist    # 404
```

**Capcana.** API-ul care răspunde `200` cu `{ "error": "not found" }` în body, sau `500` la o validare picată. Și pe partea cealaltă: **`fetch` nu aruncă eroare la 404 sau 500**, aruncă doar când rețeaua cade. Trebuie verificat `res.ok` (vezi [04-javascript/03-async-await-fetch.md](../04-javascript/03-async-await-fetch.md)).

<details>
<summary>Întrebare de interviu: Care e diferența dintre 401 și 403?</summary>

`401` înseamnă că serverul nu știe cine ești: lipsește token-ul, e invalid sau a expirat. Soluția e să te autentifici. `403` înseamnă că te-a identificat, dar n-ai drepturi pentru acțiunea asta, de exemplu un `member` care încearcă o operație de `admin`. Numele `Unauthorized` e istoric și derutant: de fapt înseamnă „unauthenticated”.
</details>

## Headers

**Ce e.** Metadate despre request sau response, sub formă de perechi `Nume: valoare`. Numele nu țin cont de litere mari și mici.

| Header | Unde | La ce folosește |
|---|---|---|
| `Content-Type` | ambele | formatul body-ului (`application/json`) |
| `Accept` | request | ce format accepți înapoi |
| `Authorization` | request | token-ul: `Bearer eyJ...` |
| `Cookie` / `Set-Cookie` | request / response | cookies (vezi [05-auth-cookies-cors.md](05-auth-cookies-cors.md)) |
| `Cache-Control` | ambele | reguli de cache |
| `Origin` | request | de pe ce site vine request-ul (pentru CORS) |
| `Access-Control-Allow-*` | response | răspunsurile CORS ale serverului |
| `Location` | response | unde e resursa nou creată sau unde e redirectul |

**De ce contează.** Jumătate din bug-urile de integrare frontend–backend sunt de fapt un header lipsă sau greșit.

**Exemplu minim**

```bash
curl -I https://openlibrary.org/search.json?q=hobbit   # -I = show only the response headers
```

**Cum îl vezi în cod real.** În P3 Booking, fiecare request spre API duce token-ul utilizatorului:

```ts
const res = await fetch(`${API_URL}/bookings`, {
  headers: { Authorization: `Bearer ${session.access_token}` },
});
```

**Capcana.** `POST` cu body JSON, dar fără `Content-Type: application/json`. Serverul nu știe cum să citească body-ul și controller-ul primește un obiect gol. Mesajul de eroare nu-ți spune niciodată direct „ți-a lipsit headerul”.

<details>
<summary>Întrebare de interviu: Ce face headerul Content-Type și ce se întâmplă dacă lipsește?</summary>

Spune ce format are body-ul, ca cealaltă parte să știe cum să-l parseze. Dacă trimiți JSON fără `Content-Type: application/json`, majoritatea serverelor (Express, NestJS) nu-l parsează și body-ul ajunge gol în cod. În response, browserul folosește același header ca să știe dacă primește HTML, JSON sau o imagine.
</details>

## Body și JSON

**Ce e.** Body-ul e conținutul mesajului. În API-urile moderne e aproape mereu JSON, un format text care seamănă cu obiectele JavaScript. `JSON.stringify` transformă obiectul în text, iar `JSON.parse` sau `res.json()` face drumul invers.

**De ce contează.** JSON nu poate reprezenta tot ce are JavaScript. Ce nu încape se pierde sau se schimbă pe drum, fără nicio eroare.

**Exemplu minim**

```js
const invoice = { id: 42, dueDate: new Date("2026-11-30"), note: undefined };

const text = JSON.stringify(invoice);
console.log(text); // {"id":42,"dueDate":"2026-11-30T00:00:00.000Z"}  — note disappeared

const back = JSON.parse(text);
console.log(typeof back.dueDate); // "string" — no longer a Date
```

Ce se pierde: `Date` devine string ISO, `undefined` dispare, funcțiile dispar, `Map` și `Set` devin `{}`, iar `BigInt` aruncă eroare.

**Capcana.** În TypeScript declari `dueDate: Date`, dar din API vine un string. Tipul te minte, iar `invoice.dueDate.getTime()` crapă la runtime. Datele care vin din exterior se validează (vezi [05-typescript/05-validare-cu-zod.md](../05-typescript/05-validare-cu-zod.md)).

<details>
<summary>Întrebare de interviu: Ce se pierde când trimiți un obiect JavaScript prin JSON?</summary>

Tot ce nu e string, număr, boolean, `null`, array sau obiect simplu. Datele calendaristice devin string-uri ISO și trebuie convertite înapoi, câmpurile `undefined` dispar, funcțiile dispar, iar `Map` și `Set` nu se serializează. Din cauza asta, datele care vin dintr-un API trebuie parsate sau validate, nu doar „castate” la un tip.
</details>

## DevTools: tab-ul Network

**Ce e.** Unealta din browser care îți arată fiecare request făcut de pagină. O folosești în Chrome sau Edge, pe Windows.

1. Deschide aplicația și apasă `F12`, apoi mergi la tab-ul **Network**.
2. Bifează **Preserve log**, ca să nu se șteargă lista la navigare, și filtrează pe **Fetch/XHR**, ca să vezi doar apelurile de API.
3. Fă acțiunea din aplicație, de exemplu o căutare în P1.
4. Dă click pe request și citește tab-urile: **Headers** (URL, metodă, status, headers), **Payload** (ce ai trimis), **Response** (ce s-a întors), **Timing** (cât a durat).
5. Click dreapta pe request → **Copy → Copy as cURL (bash)** și îl lipești în terminalul din WSL. Așa reproduci exact același request fără browser.

**Verifică.** Deschide în browser `https://openlibrary.org/search.json?q=hobbit&limit=1`. Primești JSON direct, adică exact ce primește aplicația ta din P1.

**Capcana.** Să te uiți doar în Console. Adevărul e în Network: _a plecat request-ul? la ce URL? cu ce body? ce status s-a întors?_ Când un buton „nu face nimic”, Network-ul îți spune în 10 secunde dacă problema e în frontend sau în backend.

## curl din terminal

**Ce e.** `curl` e un client HTTP în linie de comandă. Trimite request-uri fără browser, deci fără CORS, fără cache și fără frontend.

**De ce contează.** Îți testezi backend-ul înainte să existe frontend-ul și izolezi bug-urile. Dacă merge cu `curl` și nu merge din browser, problema e în frontend sau în CORS.

**Exemplu minim**

```bash
# GET with response headers (-i)
curl -i "https://openlibrary.org/search.json?q=the+hobbit&limit=1&fields=key,title,author_name,first_publish_year"

# POST JSON to your local NestJS API (P2)
curl -i -X POST http://localhost:3000/clients \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme SRL", "email": "billing@acme.ro"}'

# pretty-print JSON (install once: sudo apt install jq)
curl -s "https://openlibrary.org/search.json?q=hobbit&limit=2" | jq '.docs[].title'
```

Primul răspuns arată cam așa:

```http
HTTP/2 200
content-type: application/json

{"numFound":461,"start":0,"docs":[{"author_name":["J.R.R. Tolkien"],"first_publish_year":1937,"key":"/works/OL27482W","title":"The Hobbit"}], ...}
```

**Capcana.** Ghilimelele. În bash, JSON-ul stă între ghilimele simple (`'...'`), iar cheile din el între ghilimele duble. În PowerShell regulile sunt altele, încă un motiv să lucrezi în WSL.

<details>
<summary>Întrebare de interviu: Cum îți dai seama dacă un bug e în frontend sau în backend?</summary>

Încep cu tab-ul Network. Mă uit dacă request-ul a plecat, cu URL-ul, metoda și body-ul corecte, și ce status și ce body s-au întors. Dacă request-ul e greșit, bug-ul e în frontend. Dacă request-ul e corect și răspunsul e greșit, reproduc cu `curl`, apoi caut în logurile și codul backend-ului. Dacă răspunsul e corect, dar interfața arată altceva, bug-ul e iar în frontend, la cum folosesc datele.
</details>

## Mai departe

- [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [MDN — HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [Chrome DevTools — Network features reference](https://developer.chrome.com/docs/devtools/network/reference)
