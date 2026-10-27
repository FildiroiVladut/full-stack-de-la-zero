# Next.js

Citești fișierul ăsta în săptămâna 7, înainte de P4. Până atunci ai lucrat cu două aplicații separate: un frontend React (Vite) și un backend Nest, cu o graniță foarte clară între ele. Next.js pune frontend-ul și o parte de backend în același proiect, uneori chiar în același fișier. Granița există în continuare, doar că acum trebuie să știi tu unde e.

## Ce aduce Next.js față de un SPA Vite

**Ce e.** Un framework construit peste React. Un SPA Vite trimite browserului un HTML gol plus JavaScript, iar totul se randează în browser. Next.js poate randa pe server, are routing pe bază de fișiere și te lasă să scrii cod de server (citire din baza de date, apeluri cu chei secrete) în același proiect cu interfața.

**De ce contează.** E cel mai cerut framework React în anunțuri. În P4 îți permite să apelezi Claude API și Supabase cu chei secrete *fără* un backend separat, pentru că acel cod rulează pe server.

**Exemplu minim**

```
SPA (Vite) — P1–P3                          Next.js — P4
──────────────────────                      ──────────────────────
Browser                                     Browser
  │ downloads index.html + JS                 │ gets ready HTML for the page
  │ renders everything client-side            │ JS hydrates the interactive parts
  │ fetch → separate API (Nest on Render)     │ calls server code in the same app
  ▼                                            ▼
Nest API → Supabase                         Next.js server → Supabase / Claude API
```

**Cum îl vezi în cod real.** Un proiect Next.js are folderul `app/` cu fișiere `page.tsx` și `layout.tsx` și nu are `main.tsx` sau `index.html`. Unele fișiere încep cu `"use client"`, altele cu `"use server"`, iar majoritatea cu nimic: acelea rulează pe server.

**Capcana.** Să tratezi Next.js ca pe un SPA cu alt nume și să pui `"use client"` peste tot ca să „meargă ca înainte”. Pierzi exact avantajele lui și riști să muți cod de server în browser.

<details>
<summary>Întrebare de interviu: Când ai alege Next.js în loc de un SPA cu Vite?</summary>

Când contează SEO și timpul până la primul conținut afișat, de exemplu pagini publice sau marketing, sau când vreau logică de server (chei secrete, acces la baza de date) în același proiect fără un backend separat. Pentru un dashboard intern în spatele unui login, consumat de un API existent, un SPA cu Vite e mai simplu. Next.js aduce putere, dar și complexitate: trebuie să știi mereu unde rulează fiecare bucată de cod.
</details>

## App Router: routing pe fișiere

**Ce e.** În Next.js, structura folderului `app/` *este* routing-ul. Fiecare folder e un segment de URL, iar `page.tsx` face ruta accesibilă. `layout.tsx` învelește paginile de dedesubt, iar folderele cu paranteze drepte (`[id]`) sunt segmente dinamice.

**De ce contează.** Nu mai ai un fișier de rute ca în React Router. Ca să înțelegi o aplicație, citești arborele de foldere.

**Exemplu minim**

```
app/
├── layout.tsx                 ← root layout: <html>, <body>, header
├── page.tsx                   ← /
├── recipes/
│   ├── page.tsx               ← /recipes
│   └── [id]/
│       ├── page.tsx           ← /recipes/abc123
│       └── loading.tsx        ← shown while the page loads
└── invoices/
    └── upload/page.tsx        ← /invoices/upload
```

```tsx
// app/recipes/[id]/page.tsx
export default async function RecipePage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params; // params is a Promise in current Next.js
  const recipe = await getRecipe(id);
  return <h1>{recipe.name}</h1>;
}
```

**Cum îl vezi în cod real.** Alte fișiere speciale pe care le vei întâlni: `loading.tsx` (UI de așteptare), `error.tsx` (UI la eroare), `not-found.tsx` și `route.ts` (endpoint-uri de API, vezi mai jos).

**Capcana.** Tutorialele vechi folosesc *Pages Router* (folderul `pages/`, `getServerSideProps`) sau `params` ca obiect sincron. Dacă Claude amestecă cele două stiluri, cere-i explicit App Router și API-ul versiunii curente.

<details>
<summary>Întrebare de interviu: Cum funcționează routing-ul în Next.js App Router?</summary>

E bazat pe fișiere: fiecare folder din `app/` e un segment de URL, iar un `page.tsx` face segmentul accesibil ca pagină. `layout.tsx` învelește paginile copil și rămâne montat la navigare. Segmentele dinamice sunt foldere ca `[id]`, iar valoarea lor vine în `params`. Fișiere speciale ca `loading.tsx` și `error.tsx` definesc stările de încărcare și eroare pentru acel segment.
</details>

## Server Components (implicit)

**Ce e.** În App Router, orice componentă e **Server Component** dacă nu spui altfel. Rulează doar pe server: poate fi `async`, poate citi direct din baza de date și poate folosi chei secrete, iar codul ei **nu ajunge** în browser. Browserul primește doar rezultatul.

**De ce contează.** E locul sigur pentru tot ce ține de date și secrete în P4. Și e motivul pentru care Next.js trimite mai puțin JavaScript decât un SPA.

**Exemplu minim**

```tsx
// app/recipes/page.tsx — Server Component (no directive)
import { createClient } from "@/lib/supabase/server";

export default async function RecipesPage() {
  const supabase = await createClient();
  const { data: recipes } = await supabase.from("recipes").select("id, name, cost_per_portion_cents");

  return (
    <ul>
      {recipes?.map((r) => <li key={r.id}>{r.name}</li>)}
    </ul>
  );
}
```

**Cum îl vezi în cod real.** Pagini `async` care încarcă date direct, fără `useEffect` și fără TanStack Query. Au și limite: în Server Components **nu** poți folosi `useState`, `useEffect`, event handlers (`onClick`) sau API-uri de browser (`window`, `localStorage`).

**Capcana.** Eroarea „You're importing a component that needs useState…” apare când pui interactivitate într-un Server Component. Soluția *nu* e să marchezi toată pagina cu `"use client"`. Extragi doar bucata interactivă într-o componentă separată.

<details>
<summary>Întrebare de interviu: Ce sunt React Server Components?</summary>

Componente care rulează doar pe server. Pot fi asincrone, pot accesa direct baza de date sau secrete, iar codul lor nu e trimis în browser, ci doar rezultatul randat. Nu pot avea state, efecte sau event handlers. În Next.js App Router sunt implicite. Interactivitatea o adaug prin Client Components, pe care le folosesc doar unde e nevoie.
</details>

## Client Components și `"use client"`

**Ce e.** Un fișier care începe cu `"use client"` declară o **graniță**: componenta din el și tot ce importă ajung în bundle-ul trimis browserului. Acolo poți folosi state, efecte și event handlers. Componenta e pre-randată ca HTML pe server, apoi „hidratată” (React îi atașează event handlers) în browser.

**De ce contează.** Trebuie să știi exact ce ajunge în browser. Tot ce e importat într-un fișier `"use client"` e public, la fel ca într-un SPA Vite.

**Exemplu minim**

```tsx
// app/invoices/upload/upload-form.tsx
"use client";

import { useState } from "react";

export function UploadForm({ onUpload }: { onUpload: (fd: FormData) => Promise<void> }) {
  const [fileName, setFileName] = useState<string | null>(null);

  return (
    <form action={onUpload}>
      <input type="file" name="invoice" accept="application/pdf,image/*"
        onChange={(e) => setFileName(e.target.files?.[0]?.name ?? null)} />
      {fileName && <p>Selected: {fileName}</p>}
      <button type="submit">Analyze invoice</button>
    </form>
  );
}
```

**Cum îl vezi în cod real.** Tiparul sănătos: pagina e Server Component, încarcă datele și le trimite ca props unor Client Components mici, aflate la marginea arborelui (un buton, un formular, un grafic interactiv). Props-urile trebuie să fie serializabile: date simple, nu funcții obișnuite sau instanțe de clase. Excepție fac Server Actions, care pot fi trimise ca props.

**Capcana.** `"use client"` pus în `layout.tsx` sau în pagini mari „ca să meargă”. Tot arborele de sub ele devine cod de client, iar orice import de cod de server (cu chei) devine o problemă. Ține granița cât mai jos în arbore.

<details>
<summary>Întrebare de interviu: Ce face directiva "use client"?</summary>

Marchează o graniță între graful de module de server și cel de client. Fișierul respectiv și tot ce importă devin parte din bundle-ul de JavaScript trimis browserului, deci pot folosi state, efecte și event handlers. Componentele sunt totuși pre-randate pe server și apoi hidratate. O pun cât mai jos în arbore, doar pe componentele interactive, ca să trimit cât mai puțin JavaScript și să nu expun cod de server.
</details>

## Server Actions

**Ce e.** Funcții `async` marcate cu `"use server"` care rulează pe server, dar pe care le poți apela din formulare sau din Client Components ca pe niște funcții obișnuite. În spate, Next.js face un request POST către ele.

**De ce contează.** În P4, confirmarea liniilor extrase de AI dintr-o factură e o Server Action: primește datele, verifică utilizatorul, scrie în Supabase și reîmprospătează pagina. Nu mai ai nevoie de un endpoint REST separat.

**Exemplu minim**

```ts
// app/invoices/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { createClient } from "@/lib/supabase/server";

export async function confirmInvoiceLines(invoiceId: string, lineIds: string[]) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) throw new Error("Unauthorized"); // always check, this is a public endpoint

  // ...validate input (zod), check the invoice belongs to this user, update prices...

  revalidatePath("/recipes"); // recipe costs changed → refresh that page
}
```

**Cum îl vezi în cod real.** `<form action={serverAction}>` în Server Components sau o Server Action importată și apelată dintr-un Client Component. Pentru starea de „se trimite” vei vedea `useActionState` sau `useTransition`.

**Capcana.** Să crezi că o Server Action e „privată” pentru că nu are URL vizibil. Documentația Next.js spune explicit că e accesibilă prin request-uri POST directe, nu doar din interfața ta. **Fiecare** Server Action verifică autentificarea, autorizarea (e factura *lui*?) și validează input-ul, exact ca un endpoint Nest.

<details>
<summary>Întrebare de interviu: Ce sunt Server Actions și ce trebuie să ai grijă la ele?</summary>

Funcții async marcate cu `"use server"` care rulează pe server și pot fi apelate din formulare sau din componente de client. Next.js le expune ca endpoint-uri POST. Tocmai de aceea sunt publice: oricine le poate apela direct. În fiecare verific autentificarea și autorizarea și validez input-ul. După mutație reîmprospătez datele cu `revalidatePath`, `revalidateTag` sau `refresh`.
</details>

## Route Handlers

**Ce e.** Endpoint-uri HTTP clasice într-un proiect Next.js: un fișier `route.ts` care exportă funcții numite după metodele HTTP (`GET`, `POST`…) și lucrează cu obiectele standard `Request` și `Response`.

**De ce contează.** Pentru ce nu e o acțiune din propria interfață: webhooks de la alte servicii, un endpoint apelat de altă aplicație, răspunsuri în streaming (de exemplu textul generat de AI afișat pe măsură ce vine).

**Exemplu minim**

```ts
// app/api/health/route.ts
export async function GET() {
  return Response.json({ status: "ok" });
}

// app/api/webhooks/supplier/route.ts
export async function POST(request: Request) {
  const payload = await request.json();
  // verify the signature, validate, process...
  return new Response(null, { status: 204 });
}
```

**Cum îl vezi în cod real.** Folder `app/api/...` cu `route.ts`. Regula practică: Server Actions pentru mutațiile din propria interfață, Route Handlers pentru tot ce vine din afară sau are nevoie de control fin asupra răspunsului HTTP.

**Capcana.** Să construiești în Route Handlers un API REST complet doar ca să-l apelezi cu `fetch` din propriile Server Components. E un drum inutil prin rețea: un Server Component poate apela direct funcția care citește din baza de date.

<details>
<summary>Întrebare de interviu: Când folosești un Route Handler și când o Server Action?</summary>

Server Action pentru mutațiile declanșate din propria interfață: formulare, butoane. Route Handler când am nevoie de un endpoint HTTP clasic, apelat de altcineva: webhooks, alte aplicații, clienți mobili. Și tot Route Handler când vreau control asupra răspunsului: streaming, headere, status-uri. Pentru citirea datelor în propriile pagini nu folosesc niciunul: le citesc direct în Server Components.
</details>

## Unde stau cheile secrete și capcana `NEXT_PUBLIC_`

**Ce e.** În Next.js, variabilele de mediu sunt implicit **doar pe server**. Cele cu prefixul `NEXT_PUBLIC_` sunt copiate literal în JavaScript-ul trimis browserului, la build. O variabilă fără prefix, citită într-un Client Component, devine șir gol.

**De ce contează.** E capcana numărul unu de securitate în P4. `ANTHROPIC_API_KEY` și cheia secretă Supabase stau doar pe server. Dacă una ajunge cu `NEXT_PUBLIC_` în față, oricine o poate copia din DevTools și o poate folosi pe banii tăi.

**Exemplu minim**

```bash
# .env.local (never committed)
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co        # public by design
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_...   # public by design, protected by RLS
SUPABASE_SECRET_KEY=sb_secret_...                         # server only
ANTHROPIC_API_KEY=sk-ant-...                              # server only
```

```ts
// lib/ai.ts
import "server-only"; // build error if a Client Component ever imports this file
import Anthropic from "@anthropic-ai/sdk";

export const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
```

**Cum îl vezi în cod real.** `import "server-only"` în fișierele care folosesc secrete. E o plasă de siguranță: dacă cineva, inclusiv Claude, importă din greșeală fișierul într-o componentă de client, build-ul pică în loc să publice cheia. Detalii despre ce e public și ce nu în [02-ce-ajunge-in-browser](../10-securitate-env/02-ce-ajunge-in-browser.md).

**Capcana.** Claude vede `undefined` într-un Client Component și „repară” punând `NEXT_PUBLIC_` în fața cheii secrete. Codul merge, iar cheia e publică. Soluția corectă e să muți apelul pe server (Server Component, Server Action sau Route Handler).

<details>
<summary>Întrebare de interviu: Cum funcționează variabilele de mediu în Next.js?</summary>

Implicit sunt disponibile doar pe server, în Server Components, Server Actions și Route Handlers. Cele cu prefixul `NEXT_PUBLIC_` sunt înlocuite la build cu valoarea lor direct în bundle-ul de client, deci devin publice și nu se mai schimbă după build. Secretele (chei de API, chei de bază de date) nu au niciodată prefixul. Fișierele care le folosesc le marchez cu `import "server-only"`, ca un import accidental în client să dea eroare la build.
</details>

## Caching pe scurt

**Ce e.** Next.js poate păstra rezultatele randării și ale încărcărilor de date ca să răspundă mai repede. În proiectele noi create cu `create-next-app` (setările recomandate activează *Cache Components*), **nimic nu e pus în cache dacă nu ceri explicit**, cu directiva `"use cache"`. Datele care trebuie să fie proaspete la fiecare request se pun într-un `<Suspense>`, cu un fallback de încărcare.

**De ce contează.** Când „am salvat, dar pagina arată datele vechi”, primul suspect e cache-ul. Trebuie să știi că există și cum îl reîmprospătezi după o modificare (`revalidatePath`, `revalidateTag`, `refresh` din `next/cache`).

**Exemplu minim**

```tsx
import { Suspense } from "react";

export default function DashboardPage() {
  return (
    <>
      <h1>Kitchen dashboard</h1>
      <Suspense fallback={<p>Loading margins…</p>}>
        <MarginAlerts /> {/* fresh on every request */}
      </Suspense>
    </>
  );
}
```

**Cum îl vezi în cod real.** În tutoriale și în proiecte mai vechi vei vedea alt model de caching: opțiuni pe `fetch` și `export const revalidate = 60`. E modelul anterior. Când citești cod, verifică ce versiune și ce configurare are proiectul (`next.config.ts`).

**Capcana.** Să ceri lui Claude să „repare caching-ul” fără să știi ce model folosește proiectul. Spune-i versiunea de Next.js și dacă proiectul are `cacheComponents: true`, ca să nu amestece cele două modele. Pentru P4, regula simplă: datele specifice utilizatorului nu se pun în cache.

<details>
<summary>Întrebare de interviu: Ai salvat date printr-o Server Action, dar pagina arată în continuare valorile vechi. Ce verifici?</summary>

Mai întâi dacă datele paginii sunt puse în cache, cu `"use cache"` sau prin modelul vechi de caching pe `fetch`. Apoi dacă Server Action-ul invalidează ce trebuie după mutație: `revalidatePath` pentru pagină, `revalidateTag` sau `updateTag` pentru datele etichetate, `refresh` pentru routerul de client. Verific și că mutația chiar a reușit: eroarea poate fi înghițită, iar datele pot să nu se fi schimbat deloc.
</details>

## Mai departe

- [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Mutating Data (Server Actions)](https://nextjs.org/docs/app/getting-started/mutating-data)
- [Environment Variables](https://nextjs.org/docs/app/guides/environment-variables)
