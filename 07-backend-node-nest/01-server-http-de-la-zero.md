# Un server HTTP de la zero

Înainte de Nest, scrii același mini-API de trei ori: cu Node gol, cu Express, apoi te uiți la varianta Nest. Așa vezi exact ce face un framework pentru tine și nu mai pare magie.

Creează un folder de joacă în WSL (`mkdir -p ~/code/playground/server && cd $_`) și rulează exemplele de acolo.

## Ce e un server

**Ce e.** Un program care stă pornit, ascultă pe un port (ex. `3000`) și răspunde la request-uri HTTP. Pentru fiecare request primește metoda (`GET`, `POST`), URL-ul, headerele și eventual un body, și trimite înapoi un status code, headere și un body (de obicei JSON).

**De ce contează.** Orice backend, de la Nest la Django, face doar atât. Restul e organizare. Dacă vezi serverul ca pe o funcție `(request) => response`, fiecare framework devine ușor de citit.

**Exemplu minim** (`server.mjs`, rulezi cu `node server.mjs`, apoi `curl localhost:3000`):

```js
import { createServer } from 'node:http';

const server = createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'hello', method: req.method, url: req.url }));
});

server.listen(3000, () => console.log('Listening on http://localhost:3000'));
```

**Capcana.** Serverul rulează până îl oprești cu `Ctrl+C`. Dacă îl pornești de două ori, al doilea cade cu `EADDRINUSE`, adică portul e deja ocupat. Nu e bug în cod, ai un proces vechi pornit.

<details>
<summary>Întrebare de interviu: Ce se întâmplă, pe scurt, când rulezi `server.listen(3000)`?</summary>

Procesul Node îi cere sistemului de operare să deschidă portul 3000 și să-i trimită conexiunile care vin acolo. Node nu blochează: event loop-ul stă și așteaptă evenimente, iar la fiecare request nou îmi apelează callback-ul cu obiectele `req` și `res`. Un singur proces poate servi multe request-uri în paralel pentru că, cât timp unul așteaptă după baza de date, event loop-ul le procesează pe celelalte.
</details>

## Routing de mână

**Ce e.** Routing înseamnă să decizi ce cod rulează în funcție de metodă + URL. Fără framework, e un șir de `if`-uri.

**Exemplu minim** — un API de facturi în memorie, cu `GET` și `POST`:

```js
import { createServer } from 'node:http';

const invoices = [{ id: 1, client: 'Acme SRL', totalCents: 121000 }];

const server = createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/invoices') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    return res.end(JSON.stringify(invoices));
  }

  if (req.method === 'POST' && req.url === '/invoices') {
    let body = '';
    req.on('data', (chunk) => (body += chunk)); // the body arrives in pieces
    req.on('end', () => {
      const data = JSON.parse(body); // crashes on invalid JSON!
      const invoice = { id: invoices.length + 1, ...data };
      invoices.push(invoice);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify(invoice));
    });
    return;
  }

  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000);
```

Testează din alt tab de terminal:

```bash
curl localhost:3000/invoices
curl -X POST localhost:3000/invoices -H "Content-Type: application/json" -d '{"client":"Beta SRL","totalCents":50000}'
```

**Ce să observi.** Ce lipsește aici e exact ce primești de la un framework:

- **Parsarea body-ului.** Body-ul vine în bucăți (stream) și trebuie lipit și parsat de mână.
- **Parametrii din URL.** `GET /invoices/2` ar cere să tai string-ul URL-ului de mână.
- **Validarea.** Oricine poate trimite `{"totalCents": "abc"}` și îl salvăm.
- **Erorile.** Un JSON invalid în `POST` dărâmă tot procesul, nu doar request-ul ăla.

**Capcana.** Fiecare `if` trebuie să facă `return` după ce a răspuns. Dacă uiți, codul merge mai departe și încearcă să răspundă de două ori, iar Node aruncă `ERR_HTTP_HEADERS_SENT`.

## Același API în Express

**Ce e.** Express e cel mai vechi și mai răspândit framework minimal pentru Node. Îți dă routing, parametri din URL, parsare de JSON și middleware, nimic mai mult.

**Exemplu minim** (`npm init -y && npm i express`, apoi `app.mjs`):

```js
import express from 'express';

const app = express();
app.use(express.json()); // middleware: parses JSON bodies for every request

const invoices = [{ id: 1, client: 'Acme SRL', totalCents: 121000 }];

app.get('/invoices', (req, res) => res.json(invoices));
app.get('/invoices/:id', (req, res) => {
  const invoice = invoices.find((i) => i.id === Number(req.params.id));
  return invoice ? res.json(invoice) : res.status(404).json({ error: 'Not found' });
});
app.post('/invoices', (req, res) => {
  const invoice = { id: invoices.length + 1, ...req.body };
  invoices.push(invoice);
  res.status(201).json(invoice);
});

app.listen(3000);
```

**Ce să observi.** Routing-ul și parsarea au dispărut în framework. Validarea, structura și accesul la baza de date sunt în continuare treaba ta, și fiecare echipă le face altfel.

## Middleware

**Ce e.** O funcție care rulează *înainte* de handler, pentru fiecare request (sau pentru un grup de rute). Primește `req`, `res` și `next`. Poate modifica request-ul, poate răspunde direct (ex. 401), sau cheamă `next()` ca să meargă mai departe.

**De ce contează.** Logging, parsare, autentificare și CORS sunt middleware-uri. Conceptul e peste tot: în Express, în Nest, în Next.js.

**Exemplu minim:**

```js
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => console.log(`${req.method} ${req.url} ${res.statusCode} ${Date.now() - start}ms`));
  next(); // without this, the request hangs forever
});
```

**Capcana.** Ordinea contează. Un middleware definit *după* rute nu rulează pentru ele. Și un middleware care uită `next()` (și nici nu răspunde) lasă request-ul agățat până la timeout.

<details>
<summary>Întrebare de interviu: Ce e un middleware și dă-mi două exemple.</summary>

E o funcție care stă în lanțul de procesare al unui request, înainte de handler. Primește request-ul, poate face ceva cu el și decide dacă îl trimite mai departe sau răspunde direct. Exemple: un logger care măsoară durata fiecărui request și un middleware de autentificare care citește token-ul din header și răspunde cu 401 dacă lipsește. `express.json()` e tot un middleware, care parsează body-ul.
</details>

## Ce adaugă Nest peste toate astea

**Ce e.** NestJS e un framework care rulează *peste* Express (implicit) sau Fastify. Nu înlocuiește serverul, ci îi dă o structură fixă: fiecare funcționalitate e un **modul**, request-urile intră prin **controllers**, logica stă în **services**, iar dependențele sunt injectate automat.

**Același endpoint în Nest:**

```ts
@Controller('invoices')
export class InvoicesController {
  constructor(private readonly invoicesService: InvoicesService) {}

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.invoicesService.findOne(id); // throws NotFoundException if missing -> 404
  }

  @Post()
  create(@Body() dto: CreateInvoiceDto) {
    // dto was already validated before this line runs
    return this.invoicesService.create(dto); // returns 201 by default for POST
  }
}
```

**Ce să observi.**

- Rutele sunt declarate cu decoratori (`@Get`, `@Post`), nu cu `if`-uri.
- `ParseIntPipe` transformă și validează `:id`. Un `GET /invoices/abc` primește automat 400.
- Controller-ul nu știe de baza de date. Doar deleagă către service. Ăsta e tot rostul structurii.

**De ce contează.** Într-o echipă de 10 oameni, Express-ul „liber” ajunge în 10 stiluri diferite. Nest impune unul. De asta îl aleg companiile cu proiecte mari, și de asta îl înveți: când deschizi un proiect Nest străin, știi deja unde e fiecare lucru.

<details>
<summary>Întrebare de interviu: De ce ai alege NestJS în loc de Express?</summary>

Express e minimal și îmi lasă toate deciziile de structură mie. Pe un proiect mic e perfect. Nest vine cu o arhitectură clară (module, controllers, services), dependency injection, validare prin DTO-uri, guards pentru auth și integrare cu testarea, toate standardizate. Pe un proiect care crește și la care lucrează mai mulți oameni, asta înseamnă cod consistent și onboarding rapid. Sub capotă, Nest folosește tot Express, deci nu pierd nimic din ecosistem.
</details>

## Mai departe

- [Node.js — modulul http](https://nodejs.org/api/http.html)
- [Express — Routing](https://expressjs.com/en/guide/routing.html)
- [NestJS — First steps](https://docs.nestjs.com/first-steps)
