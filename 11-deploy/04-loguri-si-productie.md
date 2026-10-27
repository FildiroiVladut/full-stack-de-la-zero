# Loguri și producție

În producție nu ai debugger și nu poți pune un `console.log` și da refresh. Ai loguri, tab-ul Network din browser și o metodă de lucru. Fișierul ăsta e despre metodă.

## Unde sunt logurile

**Ce e.** Fiecare bucată din aplicație își scrie logurile în alt loc. Primul pas la orice eroare e să știi unde să te uiți.

**De ce contează.** Juniorii caută eroarea în locul greșit: se uită în Render la o eroare de JavaScript din browser, care nu ajunge niciodată pe server.

**Exemplu minim**

| Unde a apărut eroarea | Unde o vezi |
|---|---|
| În browser (React, fetch, CORS) | **DevTools din browser**: tab-urile Console și Network. Vercel nu vede erorile din browser. |
| La build pe Vercel | Pagina deployment-ului → **Build Logs** |
| În partea de server a lui Next.js (P4) | Vercel → **Logs** (runtime logs). Pe Hobby se păstrează puțin timp (o oră, la momentul scrierii). |
| În API-ul Nest | Render → serviciul → **Logs** (live, tot ce scrie aplicația în consolă) |
| În baza de date, Auth, Storage | Supabase dashboard → secțiunea **Logs** (API, Postgres, Auth) |

**Cum îl vezi în cod real.** O eroare care traversează tot sistemul lasă urme în mai multe locuri: un 500 în tab-ul Network, stack trace-ul în logurile Render și, poate, o eroare de constrângere în logurile Postgres. Le legi prin **oră** și **endpoint**.

**Capcana.** Să ceri ajutor (lui Claude sau lui Bogdan) cu „nu merge” fără log. Copiază întotdeauna eroarea completă, cu stack trace-ul, plus ce request ai făcut și ce răspuns ai primit.

<details>
<summary>Întrebare de interviu: Utilizatorul spune că butonul „Save” nu face nimic. De unde începi?</summary>

Din browser: deschid DevTools, mă uit în Console după erori de JavaScript și în Network la request-ul trimis de buton. Văd dacă a plecat, ce status a primit și ce body a venit înapoi. Dacă e 4xx, problema e în datele trimise sau în autorizare. Dacă e 5xx, merg în logurile backend-ului, la ora și endpoint-ul respectiv. Dacă request-ul nici n-a plecat, problema e în frontend.
</details>

## Loguri bune în Nest

**Ce e.** În loc de `console.log` împrăștiat prin cod, folosești `Logger`-ul din Nest. Scrie mesaje cu nivel (`log`, `warn`, `error`), cu context (numele clasei) și cu timestamp.

**De ce contează.** În logurile din Render vei avea sute de linii. Fără context și niveluri, nu găsești nimic. Cu ele, filtrezi după `ERROR` sau după `InvoicesService`.

**Exemplu minim**

```ts
import { Injectable, Logger } from "@nestjs/common";

@Injectable()
export class InvoicesService {
  private readonly logger = new Logger(InvoicesService.name);

  async markAsPaid(invoiceId: string, userId: string) {
    this.logger.log(`Marking invoice ${invoiceId} as paid (user ${userId})`);
    try {
      // ...
    } catch (err) {
      this.logger.error(`Failed to mark invoice ${invoiceId} as paid`, (err as Error).stack);
      throw err;
    }
  }
}
```

**Cum îl vezi în cod real.** Loguri la începutul operațiilor importante, cu id-urile relevante (invoice, user), și `error` cu stack trace la excepții. Excepțiile neprinse sunt logate automat de Nest.

**Capcana.** Să loghezi date sensibile: parole, token-uri JWT, chei de API, body-uri întregi cu date personale. Logurile sunt citite de mai mulți oameni și păstrate în servicii terțe. Loghează id-uri, nu conținut.

<details>
<summary>Întrebare de interviu: Ce pui și ce nu pui în loguri?</summary>

Pun evenimentele importante cu context: ce operație, pe ce resursă (id-uri), pentru ce utilizator (id), rezultatul și durata. La erori pun stack trace-ul. Nu pun parole, token-uri, chei, date de card sau date personale în clar, pentru că logurile ajung la mai mulți oameni și în servicii externe. Folosesc niveluri (info, warn, error) ca să pot filtra.
</details>

## „Merge la mine”: checklist de diferențe

**Ce e.** Când ceva merge local și nu merge în producție, cauza e aproape întotdeauna o *diferență de mediu*, nu codul. Lista de mai jos e în ordinea frecvenței.

**De ce contează.** Îți economisește ore. Verifici lista, nu ghicești.

**Exemplu minim**

```
1. Env vars       — missing in Vercel/Render? different name? old value (no redeploy)?
2. URLs           — frontend still calling localhost? CORS_ORIGIN matches exactly?
3. Database       — migrations applied in prod? prod DB empty (no seed data)? project paused?
4. Node version   — .nvmrc committed? same major version as your laptop?
5. Dependencies   — package used locally but missing from package.json? devDeps skipped?
6. Build vs dev   — code that only works in dev mode (e.g. relies on Vite's dev server)?
7. Cold start     — first request after sleep takes ~1 min → frontend timeout?
```

**Cum îl vezi în cod real.** Cele mai multe probleme de la primul deploy al lui P2 vor fi punctele 1–3. Punctul 3 are o variantă vicleană: API-ul merge, dar lista e goală, pentru că baza de producție nu are datele de test pe care le ai local.

**Capcana.** Să schimbi codul ca să „repari” o problemă de configurare, de exemplu să scrii URL-ul API-ului direct în cod. Repari configurarea, nu codul.

<details>
<summary>Întrebare de interviu: Care sunt cauzele tipice pentru care o aplicație merge local, dar nu în producție?</summary>

Variabile de mediu lipsă sau diferite, URL-uri care încă indică spre localhost, CORS configurat pentru alt origin, migrații neaplicate sau o bază de date de producție fără datele de test, altă versiune de Node, dependențe instalate global local și lipsă din `package.json`, diferențe între modul de dezvoltare și build-ul de producție. Le verific pe rând, pornind de la loguri.
</details>

## Cum citești o eroare din producție

**Ce e.** O metodă în cinci pași, aceeași de fiecare dată.

**De ce contează.** E diferența dintre a repara în 10 minute și a face 15 deploy-uri la întâmplare. Și e exact ce vrea să audă un interviewer la „povestește-mi un bug greu”.

**Exemplu minim**

```
1. Reproduce   — what exact action? which page? which user? does it happen every time?
2. Observe     — Network tab: which request, status code, response body
3. Correlate   — backend logs at the same time, same endpoint → find the stack trace
4. Locate      — read the stack trace top-down until the first line that is YOUR code
5. Fix + prove — reproduce locally, fix in a branch, PR, deploy, repeat step 1 in prod
```

Un stack trace din Render arată cam așa:

```
ERROR [ExceptionsHandler] Cannot read properties of undefined (reading 'unitPriceCents')
TypeError: Cannot read properties of undefined (reading 'unitPriceCents')
    at InvoicesService.computeTotal (/opt/render/project/src/dist/invoices/invoices.service.js:41:38)
    at InvoicesService.create (/opt/render/project/src/dist/invoices/invoices.service.js:22:28)
    at ...node_modules/@nestjs/core/...
```

Prima linie cu codul tău e `invoices.service.js:41`, în `computeTotal`. Acolo începi. Liniile din `node_modules` sunt aproape mereu doar drumul până la tine.

**Cum îl vezi în cod real.** Când îi dai lui Claude o eroare de producție, dă-i: acțiunea care a declanșat-o, request-ul și răspunsul din Network și stack trace-ul complet din loguri. Cere-i întâi *o ipoteză și cum o verifici*, nu direct un fix.

**Capcana.** Să citești doar ultima linie a erorii sau să te pierzi în liniile din `node_modules`. Caută prima linie din codul tău.

<details>
<summary>Întrebare de interviu: Povestește-mi cum ai depanat o problemă în producție.</summary>

Structura răspunsului: ce am observat (simptomul, cum a fost raportat), cum am reprodus, ce am găsit în tab-ul Network și în logurile backend-ului, care a fost cauza reală, cum am reparat (fix-ul și testul adăugat) și ce am schimbat ca să nu se repete (validare, alertă, test). Folosește un exemplu real din proiectele tale. Primul deploy din P2 îți va da sigur unul.
</details>

## Health checks și demo-ul de portofoliu

**Ce e.** Un *health check* e un endpoint simplu (`/health`) care spune dacă aplicația e vie. Platforma îl folosește la deploy, iar tu îl poți folosi ca să ții aplicația trează.

**De ce contează.** Pe planurile gratuite ai două probleme pentru un portofoliu:

- **Render** adoarme după 15 minute fără trafic, iar primul request durează aproximativ un minut;
- **Supabase** pune proiectul pe pauză dacă are activitate prea mică într-o perioadă de 7 zile.

Un recrutor care dă click pe demo și vede o eroare închide tab-ul.

**Exemplu minim.** Un health check care atinge și baza de date (cu Prisma), deci confirmă tot lanțul:

```ts
@Get()
async check() {
  await this.prisma.$queryRaw`SELECT 1`; // proves the DB connection works
  return { status: "ok" };
}
```

Și un workflow GitHub Actions programat care îl apelează o dată pe zi, în perioada în care aplici la joburi:

```yaml
# .github/workflows/keep-alive.yml
name: keep-alive
on:
  schedule:
    - cron: "0 7 * * *" # every day at 07:00 UTC
  workflow_dispatch:
jobs:
  ping:
    runs-on: ubuntu-latest
    steps:
      - run: curl --fail --max-time 120 https://invoicer-api.onrender.com/health
```

**Cum îl vezi în cod real.** Ce mai faci pentru demo:

- În README-ul proiectului scrii sincer: *„Free-tier demo: the first request may take up to a minute while the API wakes up.”*
- Frontend-ul afișează un mesaj clar în timpul cold start-ului („Waking up the server…”), nu un spinner infinit.

**Capcana.** Un request pe zi ține Supabase activ, dar nu ține Render treaz: acolo primul vizitator tot așteaptă cold start-ul. Mai e un detaliu: GitHub dezactivează workflow-urile programate dintr-un repo public după 60 de zile fără activitate în repo, așa că verifică din când în când tab-ul Actions.

<details>
<summary>Întrebare de interviu: Ce verifică un endpoint de health și de ce contează?</summary>

Minimum: că procesul rulează și răspunde. Mai util: că dependențele critice merg, de exemplu conexiunea la baza de date. Platforma de hosting îl folosește ca să nu trimită trafic spre o versiune nouă care nu pornește corect și ca să repornească instanțele blocate. Tot pe el se sprijină monitorizarea și alertele de uptime. Trebuie să fie rapid și să nu expună informații interne.
</details>

## Un tracker de erori (opțional)

**Ce e.** Un serviciu (cel mai cunoscut e Sentry) care prinde automat excepțiile din frontend și backend și ți le arată grupate, cu stack trace, browser, utilizator și versiunea de cod. Are un plan gratuit pentru proiecte mici.

**De ce contează.** Logurile îți arată ce s-a întâmplat dacă știi unde să cauți. Un tracker îți spune singur că s-a întâmplat ceva, inclusiv erorile din browserul utilizatorilor, pe care altfel nu le vezi niciodată. În companii aproape orice aplicație are unul.

**Exemplu minim**

```
User hits a bug in production
  → SDK in the app captures the exception + context
  → Sentry groups it with similar errors, emails you
  → you see: stack trace, browser, URL, release, how many users are affected
```

**Cum îl vezi în cod real.** Un `Sentry.init({ dsn: ... })` la pornirea aplicației, în `main.tsx` și în `main.ts`. DSN-ul din frontend e public prin design: identifică proiectul, nu dă acces la date.

**Capcana.** Să adaugi trackerul înainte să ai obiceiul de a citi logurile. E un bonus bun pentru P4, nu o cerință pentru P1–P3.

<details>
<summary>Întrebare de interviu: Cum afli de erorile care apar în browserul utilizatorilor?</summary>

Logurile serverului nu le văd, pentru că erorile de JavaScript din browser nu ajung acolo. Folosesc un tracker de erori, ca Sentry, al cărui SDK prinde excepțiile din frontend și le trimite cu context: stack trace, browser, pagină, versiune. Pentru stack trace-uri lizibile încarc source maps la build. Le corelez apoi cu logurile backend-ului când eroarea vine dintr-un request.
</details>

## Mai departe

- [Vercel: Logs](https://vercel.com/docs/logs)
- [Render: Logging](https://render.com/docs/logging)
- [Supabase: Logging](https://supabase.com/docs/guides/telemetry/logs)
