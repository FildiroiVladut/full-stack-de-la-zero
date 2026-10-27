# Render

Render e locul unde rulează API-ul NestJS din P2 și P3. Spre deosebire de Vercel, aici aplicația ta e un **proces care pornește și ascultă pe un port**, exact ca pe laptop cu `npm run start:prod`. Render are un plan gratuit pentru web services, cu limitări pe care trebuie să le știi (le găsești mai jos).

Baza de date **nu** stă pe Render, ci pe Supabase. Postgres-ul gratuit de la Render expiră după 30 de zile, deci nu e o opțiune pentru portofoliu.

## Pregătește codul înainte de deploy

Patru schimbări mici în proiectul Nest. Cere-i lui Claude să le facă, apoi verifică-le tu în diff.

### 1. Portul și host-ul din mediu

Render îți dă portul în variabila `PORT` (implicit 10000) și cere ca serverul să asculte pe `0.0.0.0`, nu doar pe `localhost`.

```ts
// src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors({
    origin: process.env.CORS_ORIGIN?.split(",") ?? "http://localhost:5173",
  });

  await app.listen(process.env.PORT ?? 3000, "0.0.0.0");
}
bootstrap();
```

### 2. Un endpoint de health

Un endpoint simplu, pe care Render (și tu) îl poate întreba „ești viu?”:

```ts
// src/health/health.controller.ts
import { Controller, Get } from "@nestjs/common";

@Controller("health")
export class HealthController {
  @Get()
  check() {
    return { status: "ok" };
  }
}
```

(Pentru verificări mai serioase, de exemplu „merge și baza de date?”, Nest are modulul `@nestjs/terminus`. E opțional aici.)

### 3. Versiunea de Node fixată

Render alege versiunea de Node în ordinea: variabila `NODE_VERSION` → fișierul `.node-version` → `.nvmrc` → câmpul `engines` din `package.json`. Cu fnm ai deja un `.nvmrc` în proiect (vezi [03-node-npm-bun](../01-setup/03-node-npm-bun.md)). Verifică doar că e comis în Git:

```bash
node --version > .nvmrc   # e.g. v24.x.x — same version you develop with
git add .nvmrc
```

### 4. Prisma în procesul de build

Clientul Prisma trebuie generat la build, iar migrațiile aplicate pe baza de producție. Pe planul gratuit Render nu ai *Pre-Deploy Command*, deci migrațiile rulează în build command (vezi pasul 2 de mai jos). Detalii despre migrații în [05-migratii](../08-baze-de-date-supabase/05-migratii.md).

**Verifică local, înainte de deploy:**

```bash
npm run build && PORT=4000 npm run start:prod
curl http://localhost:4000/health     # → {"status":"ok"}
```

## Pașii

### 1. Contul și serviciul

Intră pe [render.com](https://render.com) din browserul din Windows și fă-ți cont cu GitHub. Din dashboard: **New → Web Service**, apoi conectezi repo-ul API-ului (de exemplu `invoicer-api`).

### 2. Configurarea serviciului

| Câmp | Valoare |
|---|---|
| **Language** | Node |
| **Branch** | `main` |
| **Region** | Frankfurt (EU Central), aproape de Supabase, dacă proiectul Supabase e tot în Frankfurt |
| **Build Command** | `npm ci && npx prisma generate && npx prisma migrate deploy && npm run build` |
| **Start Command** | `npm run start:prod` |
| **Instance Type** | Free |

`npm run start:prod` e scriptul pus de Nest CLI în `package.json` (`node dist/main`). Verifică-l înainte.

### 3. Variabilele de mediu

În secțiunea **Environment Variables**:

```
DATABASE_URL   = postgresql://postgres.xxxx:PAROLA@aws-...pooler.supabase.com:5432/postgres
CORS_ORIGIN    = https://invoicer.vercel.app
```

- **`DATABASE_URL`:** folosește connection string-ul **Session pooler** din Supabase (butonul **Connect** din dashboard). Conexiunea directă e pe IPv6 în planul gratuit și s-ar putea să nu fie accesibilă din Render, iar pooler-ul în session mode merge pe IPv4. Explicația completă e în [04-conectare-si-chei](../08-baze-de-date-supabase/04-conectare-si-chei.md). Dacă proiectul Prisma folosește și o variabilă separată pentru migrații, o adaugi și pe aceea, conform capitolului 08.
- **`CORS_ORIGIN`:** URL-ul exact al frontend-ului, fără `/` la final.
- **`PORT`:** nu-l seta. Îl dă Render.
- **`NODE_ENV=production`:** nu-l pune ca variabilă de build (vezi capcanele).

### 4. Health check

În **Advanced → Health Check Path** pui `/health`. Render nu trimite trafic către o versiune nouă până nu răspunde cu 2xx sau 3xx pe acest path. Dacă versiunea nouă nu răspunde, deploy-ul e anulat, iar versiunea veche rămâne online.

### 5. Deploy și verificare

Confirmi crearea serviciului cu butonul de la finalul formularului. Urmărești tab-ul **Logs**: vezi `npm ci`, migrațiile, build-ul, apoi mesajul de pornire al lui Nest.

```bash
curl https://invoicer-api.onrender.com/health   # → {"status":"ok"}
```

### 6. Leagă frontend-ul

În Vercel, la proiectul frontend-ului, setezi `VITE_API_URL=https://invoicer-api.onrender.com` (Production și Preview) și faci redeploy (vezi [02-vercel](02-vercel.md)). Deschizi site-ul, faci o acțiune care cheamă API-ul și verifici în tab-ul Network că request-ul pleacă spre Render și primește 200.

De acum, fiecare push pe `main` în repo-ul API-ului face deploy automat.

## Limitările planului gratuit

Verificate pe documentația Render (render.com/docs/free) la momentul scrierii:

- **Adormire după 15 minute fără trafic.** Serviciul se oprește, iar la următorul request pornește din nou, în **aproximativ un minut** (cold start).
- **750 de ore de instanță gratuite pe lună**, per workspace. Un serviciu care doarme nu consumă ore. Dacă le termini, serviciile gratuite sunt suspendate până luna viitoare.
- **Fișierele locale se pierd** la redeploy, restart sau adormire. Uploads-urile merg în Supabase Storage, niciodată pe disc.
- **Fără shell sau SSH** pe instanță și o singură instanță, fără scalare.
- **Render poate reporni serviciul oricând.** Aplicația trebuie să pornească curat de fiecare dată.

Același tip de grijă și pentru baza de date: un proiect **Supabase gratuit se pune pe pauză** dacă are activitate prea mică într-o perioadă de 7 zile. Primești un email de avertizare înainte și îl poți restaura din dashboard. Ce faci ca să nu găsească un recrutor demo-ul mort: în [04-loguri-si-productie](04-loguri-si-productie.md).

## Ce s-a întâmplat de fapt

```
git push → GitHub notifies Render
  → Render clones the repo into a fresh build environment
  → runs the Build Command: install, prisma generate, migrate deploy, nest build
  → starts the Start Command: node dist/main
  → waits until /health answers 2xx (and the app binds to $PORT)
  → switches traffic to the new instance, stops the old one
```

Spre deosebire de Vercel, unde rezultatul build-ului sunt fișiere statice, aici rezultatul e un **proces care rulează**. Logurile lui (tot ce scrie Nest în consolă) apar live în tab-ul **Logs**.

## Capcane

- **„No open ports detected” sau deploy anulat la health check:** aplicația ascultă pe un port fix (`3000`) sau doar pe `localhost`. Folosește `process.env.PORT` și `"0.0.0.0"`.
- **Build-ul pică cu „nest: not found” sau lipsește `typescript`:** ai setat `NODE_ENV=production` ca variabilă de mediu. Atunci `npm ci` sare peste `devDependencies`, adică exact uneltele de build. Scoate variabila sau folosește `npm ci --include=dev`.
- **Eroare CORS în browser, dar curl merge:** `CORS_ORIGIN` nu se potrivește exact cu originea frontend-ului: `http` vs `https`, un `/` la final, URL-ul de preview în loc de cel de producție. CORS e verificat de browser, de aceea curl nu-l vede (vezi [05-auth-cookies-cors](../02-cum-functioneaza-webul/05-auth-cookies-cors.md)).
- **Eroare de conexiune la baza de date:** connection string-ul direct în loc de pooler, parola cu caractere speciale neencodate în URL sau proiectul Supabase pus pe pauză.
- **Prima cerere durează un minut:** nu e bug, e cold start-ul planului gratuit.

<details>
<summary>Întrebare de interviu: De ce o aplicație Node trebuie să asculte pe `process.env.PORT` și pe `0.0.0.0` când e deployată?</summary>

Platforma de hosting decide portul pe care rutează traficul spre container și îl comunică prin variabila `PORT`. Un port fix poate să nu fie cel așteptat. `0.0.0.0` înseamnă „toate interfețele de rețea”: dacă serverul ascultă doar pe `localhost`, acceptă conexiuni doar din interiorul aceleiași mașini, iar proxy-ul platformei nu ajunge la el.
</details>

## Mai departe

- [Render: Deploy for Free](https://render.com/docs/free)
- [Render: Web Services](https://render.com/docs/web-services)
- [Render: Health Checks](https://render.com/docs/health-checks)
