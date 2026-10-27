# API-uri REST

Un API e contractul dintre frontend și backend. REST e setul de convenții după care sunt construite majoritatea API-urilor HTTP pe care o să le întâlnești la job. Exemplele sunt din P2 Invoicer.

## Ce e un API

**Ce e.** API (Application Programming Interface) e lista de operații pe care un program le pune la dispoziția altor programe, împreună cu formatul exact al cererilor și răspunsurilor. La un API web, asta înseamnă URL-uri + metode + JSON.

**De ce contează.** Frontend-ul nu știe și nu trebuie să știe cum își ține backend-ul datele. Cunoaște doar contractul. Așa pot lucra doi oameni în paralel, unul pe frontend și unul pe backend, și așa poți schimba baza de date fără să atingi interfața.

**Exemplu minim.** Open Library e un API public, pe care îl folosești în P1:

```bash
curl -s "https://openlibrary.org/search.json?q=dune&limit=1&fields=key,title,first_publish_year"
# {"numFound":48308,"start":0,...,"docs":[{"first_publish_year":1965,"key":"/works/OL893414W","title":"Dune"}]}
```

**Cum îl vezi în cod real.** În P2, NestJS îți generează documentația API-ului (Swagger/OpenAPI) la o adresă de tipul `/docs`, cu toate endpoint-urile, câmpurile și codurile de răspuns. Acolo contractul devine vizibil pentru oricine din echipă.

**Capcana.** Să folosești „API” și „backend” ca sinonime. API-ul e interfața, backend-ul e implementarea din spatele ei. Același backend poate expune mai multe API-uri: unul public, unul intern, unul de admin.

<details>
<summary>Întrebare de interviu: Explică ce e un API cuiva care nu e programator.</summary>

Ca într-un restaurant: meniul e API-ul, adică lista a ce poți cere și în ce formă. Bucătăria e backend-ul: clientul nu intră acolo și nu știe cum se gătește. Ospătarul e HTTP-ul, care duce comanda și aduce farfuria. Câtă vreme meniul rămâne același, bucătăria își poate schimba oricând furnizorii sau rețetele.
</details>

## Resurse și CRUD

**Ce e.** În REST, URL-urile numesc **resurse** (substantive, la plural), iar metoda HTTP spune acțiunea. CRUD = Create, Read, Update, Delete, cele patru operații de bază.

| Acțiune | Request | Răspuns tipic |
|---|---|---|
| listează clienții | `GET /clients` | `200` + listă |
| creează un client | `POST /clients` | `201` + clientul creat |
| un client | `GET /clients/7` | `200` sau `404` |
| modifică un client | `PATCH /clients/7` | `200` + clientul modificat |
| șterge un client | `DELETE /clients/7` | `204` |
| liniile unei facturi | `GET /invoices/42/lines` | `200` + listă |
| adaugă o linie | `POST /invoices/42/lines` | `201` |

**De ce contează.** Cine cunoaște convenția îți poate ghici API-ul fără documentație, inclusiv Claude, care va genera cod mai bun pentru un API previzibil.

**Exemplu minim.** Controller-ul NestJS pentru tabelul de mai sus:

```ts
@Controller("clients")
export class ClientsController {
  constructor(private readonly clients: ClientsService) {}

  @Get()
  findAll() {
    return this.clients.findAll();
  }

  @Get(":id")
  findOne(@Param("id", ParseIntPipe) id: number) {
    return this.clients.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateClientDto) {
    return this.clients.create(dto);
  }

  @Delete(":id")
  @HttpCode(204)
  remove(@Param("id", ParseIntPipe) id: number) {
    return this.clients.remove(id);
  }
}
```

**Capcana.** Verbe în URL: `/getClients`, `/createInvoice`, `/deleteClient?id=7`. Există o excepție acceptată: acțiunile care nu sunt CRUD, ca `POST /invoices/42/send` (trimite factura pe email). Uneori e mai clar așa decât un `PATCH` cu un status „magic”.

<details>
<summary>Întrebare de interviu: Cum ai proiecta API-ul pentru facturi care au mai multe linii?</summary>

Două resurse: `/invoices` și liniile ca sub-resursă, `/invoices/:id/lines`. `GET /invoices/:id` poate întoarce factura cu liniile incluse, pentru că așa o afișezi aproape mereu. Totalul nu se trimite din client: serverul îl calculează din linii. Pentru o acțiune ca trimiterea pe email aș folosi `POST /invoices/:id/send`, pentru că nu e o simplă modificare de câmp.
</details>

## Path params vs query params

**Ce e.** Path param-ul identifică _care_ resursă (`/invoices/42`). Query param-urile spun _cum_ vrei să vezi o colecție: filtrare, sortare, paginare (`/invoices?status=overdue&sort=dueDate&page=2`).

Regula simplă: dacă scoți parametrul și se schimbă _resursa_, e path. Dacă se schimbă doar _felul în care vezi lista_, e query.

**De ce contează.** E una dintre primele decizii pe care le vei lua la orice endpoint nou și una dintre primele întrebări la un review.

**Exemplu minim**

```ts
@Get(":id") // GET /invoices/42
findOne(@Param("id", ParseIntPipe) id: number) { /* ... */ }

@Get() // GET /invoices?status=overdue&page=2
findAll(@Query("status") status?: string, @Query("page") page = "1") {
  const pageNumber = Number(page); // query params always arrive as strings
  /* ... */
}
```

**Capcana.** Query param-urile vin **mereu ca string-uri**. `page + 1` cu `page = "2"` dă `"21"`, nu `3`. În NestJS rezolvi cu pipes și DTO-uri (vezi [07-backend-node-nest/03-dto-validare-erori.md](../07-backend-node-nest/03-dto-validare-erori.md)).

<details>
<summary>Întrebare de interviu: Când folosești path params și când query params?</summary>

Path params pentru identitatea resursei: `/clients/7`, `/invoices/42/lines`. Query params pentru opțiuni aplicate unei colecții: filtre, sortare, paginare, căutare. Un URL cu path param arată spre un singur lucru. Query-ul doar schimbă felul în care vezi o listă.
</details>

## Paginare și filtrare

**Ce e.** Nu întorci niciodată toată tabela. Clientul cere o pagină (`page`, `pageSize`, numite uneori `limit` și `offset`), iar serverul întoarce datele împreună cu informațiile necesare pentru navigare.

**De ce contează.** Cu 50 de facturi merge orice. Cu 50.000, un `GET /invoices` fără paginare îngheață și serverul, și browserul.

**Exemplu minim.** Un răspuns paginat:

```json
{
  "data": [{ "id": 42, "number": "INV-2026-0042", "status": "overdue" }],
  "total": 137,
  "page": 2,
  "pageSize": 20
}
```

**Cum îl vezi în cod real.** Service-ul din P2, cu Prisma:

```ts
async findAll({ status, page = 1, pageSize = 20 }: ListInvoicesQuery) {
  const take = Math.min(pageSize, 100); // never trust the client's page size
  const where = status ? { status } : {};
  const [data, total] = await this.prisma.$transaction([
    this.prisma.invoice.findMany({ where, skip: (page - 1) * take, take, orderBy: { dueDate: "asc" } }),
    this.prisma.invoice.count({ where }),
  ]);
  return { data, total, page, pageSize: take };
}
```

- `Math.min(pageSize, 100)` limitează mărimea paginii, ca nimeni să nu poată cere un milion de rânduri.
- `count` folosește același `where`, ca totalul să corespundă filtrului.

**Capcana.** Lipsa unei limite maxime la `pageSize`. Și ordonarea lipsă: fără `orderBy`, baza de date nu garantează nicio ordine, așa că paginile pot avea rânduri duplicate sau sărite.

<details>
<summary>Întrebare de interviu: Ce e paginarea cu offset și ce e paginarea cu cursor?</summary>

Cu offset spui „sari peste 40, dă-mi 20”. E simplă, permite sărituri direct la pagina 7, dar devine lentă pe tabele uriașe și poate sări sau dubla rânduri dacă datele se schimbă între pagini. Cu cursor spui „dă-mi 20 după elementul cu id-ul X”. E stabilă și rapidă, e folosită la feed-uri infinite, dar nu poți sări direct la o pagină anume. Pentru un dashboard de facturi, offset e perfect.
</details>

## Coduri corecte și erori consistente

**Ce e.** Fiecare endpoint întoarce codul potrivit situației, iar erorile au mereu aceeași formă, ca frontend-ul să le poată trata într-un singur loc.

| Situație în P2/P3 | Cod |
|---|---|
| listă / detaliu | `200` |
| creat | `201` |
| șters | `204` |
| validare picată (email invalid, sumă negativă) | `400` |
| factura 999 nu există | `404` |
| slotul e deja rezervat, numărul de factură există deja | `409` |
| lipsește token-ul | `401` |
| un `member` încearcă o acțiune de `admin` | `403` |

**De ce contează.** Frontend-ul ia decizii pe baza codului. Un `404` primit ca `200` cu `null` în body produce bug-uri ciudate în interfață.

**Exemplu minim.** Forma implicită a unei erori în NestJS:

```json
{ "statusCode": 400, "message": ["email must be an email"], "error": "Bad Request" }
```

**Cum îl vezi în cod real**

```ts
async findOne(id: number) {
  const client = await this.prisma.client.findUnique({ where: { id } });
  if (!client) throw new NotFoundException(`Client ${id} not found`); // → 404 with the shape above
  return client;
}
```

**Capcana.** Să lași o eroare a bazei de date să ajungă necontrolat la client. Utilizatorul primește un `500` cu stack trace și detalii despre tabele, ceea ce e și urât, și periculos.

<details>
<summary>Întrebare de interviu: Ce cod întorci când cineva încearcă să rezerve un slot deja ocupat?</summary>

`409 Conflict`, pentru că request-ul e valid ca format, dar intră în conflict cu starea curentă a datelor. Body-ul trebuie să explice clar ce s-a întâmplat, ca frontend-ul să poată arăta „slotul tocmai a fost rezervat de altcineva”. Garanția reală vine dintr-o constrângere în baza de date, nu doar dintr-un `if` în cod, pentru că două request-uri pot ajunge în același timp.
</details>

## Idempotență pe scurt

**Ce e.** O operație e idempotentă dacă, executată o dată sau de zece ori, lasă datele în aceeași stare. `GET`, `PUT` și `DELETE` sunt idempotente. `POST` nu e.

**De ce contează.** Rețelele cad și request-urile se repetă: utilizatorul dă dublu-click, browserul reîncearcă, un proxy retrimite. `DELETE /clients/7` de două ori e inofensiv. `POST /invoices` de două ori creează două facturi.

**Exemplu minim**

```text
DELETE /clients/7   → 204
DELETE /clients/7   → 404   (state is the same: client 7 is gone)

POST /invoices      → 201  id 42
POST /invoices      → 201  id 43   ← duplicate invoice
```

**Cum îl vezi în cod real.** Trei apărări, de la simplu la serios:

1. butonul se dezactivează cât timp request-ul e în curs (`isPending` din TanStack Query);
2. o constrângere `UNIQUE` în baza de date, de exemplu pe numărul facturii;
3. o „idempotency key”: clientul trimite un ID unic pe request, iar serverul ignoră repetările. Așa face Stripe, de exemplu.

**Capcana.** Să te bazezi doar pe butonul dezactivat. Protejează de dublu-click, dar nu și de un retry de rețea.

<details>
<summary>Întrebare de interviu: Ce înseamnă că o metodă HTTP e idempotentă?</summary>

Că dacă repeți request-ul, starea finală e aceeași ca după primul. `GET`, `PUT` și `DELETE` sunt idempotente, `POST` nu e, iar `PATCH` depinde de cum e implementat. Contează pentru retry-uri: un request idempotent poate fi retrimis fără grijă, unul neidempotent are nevoie de protecție suplimentară, cum ar fi constrângeri unice sau idempotency keys.
</details>

## Cum arată un API bine făcut

O listă de verificat la fiecare endpoint pe care ți-l generează Claude:

- [ ] resurse la plural, fără verbe în URL (cu excepția acțiunilor clare, ca `/send`);
- [ ] metodele și codurile de status corecte;
- [ ] input validat pe server, cu mesaje clare (`400`);
- [ ] liste paginate, cu o limită maximă și ordonare explicită;
- [ ] erori cu aceeași formă peste tot;
- [ ] date calendaristice ca string ISO 8601 în UTC (`"2026-11-30T00:00:00.000Z"`);
- [ ] bani ca întregi în cea mai mică unitate (`"totalCents": 125000`), nu `1250.00` ca float (vezi [04-javascript/06-js-in-cod-real.md](../04-javascript/06-js-in-cod-real.md));
- [ ] nu scapă câmpuri interne (hash-uri de parole, note de admin, chei);
- [ ] documentat (Swagger în P2).

Mai auzi și de **GraphQL** (clientul cere exact câmpurile de care are nevoie, printr-un singur endpoint) și de **tRPC** (frontend-ul TypeScript apelează funcțiile backend-ului ca pe niște funcții locale). Sunt bune în anumite contexte, dar REST e ce trebuie să stăpânești întâi.

## Mai departe

- [MDN — HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)
- [NestJS — Controllers](https://docs.nestjs.com/controllers)
