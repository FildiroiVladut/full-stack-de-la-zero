# Conectare și chei

Un proiect Supabase are două „uși” complet diferite. Prima e **conexiunea directă la Postgres**, cu connection string și parola bazei de date. Pe ea intră Nest prin Prisma în P2 și P3. A doua e **Data API**, cu URL-ul proiectului și o cheie API. Pe ea intră `supabase-js`, în P3 pentru login și în P4 pentru date. Să știi prin care ușă intri și ce cheie ai voie să pui unde e una dintre cele mai importante lecții de securitate din tot ghidul.

## Connection string

**Ce e.** Un singur string care conține tot ce trebuie pentru o conexiune la Postgres: utilizator, parolă, host, port, numele bazei de date.

```
postgresql://postgres.abcdefghij:PAROLA@aws-1-eu-central-1.pooler.supabase.com:5432/postgres
            └── user ────────────┘ └pass┘ └── host ─────────────────────────┘ └port┘ └db──┘
```

**De ce contează.** Cine are string-ul ăsta are acces complet la baza de date: poate citi, modifica și șterge tot. E echivalentul cheii de la ușa din spate a restaurantului. Stă doar în `.env` pe server și în setările de env din Render. Niciodată în frontend, niciodată în git.

**Capcana.** Dacă parola conține caractere speciale (`@`, `#`, `/`, `:`), strică parsarea URL-ului și primești erori ciudate de autentificare sau „invalid URL”. Fie le codezi (`@` devine `%40`), fie generezi o parolă doar din litere și cifre.

## Cele trei feluri de conexiune

Le găsești toate la butonul **Connect** din dashboard. Copiază-le de acolo, nu le construi de mână, pentru că host-ul pooler-ului include un index de cluster pe care nu-l poți ghici.

| Tip | Port | Rețea | Pentru ce |
|---|---|---|---|
| **Direct** (`db.<ref>.supabase.co`) | 5432 | doar IPv6 pe Free (IPv4 e add-on plătit) | migrații, backup-uri, dacă rețeaua ta are IPv6 |
| **Session pooler** (`...pooler.supabase.com`) | 5432 | IPv4 și IPv6 | **servere care stau pornite** (Nest pe Render) |
| **Transaction pooler** (`...pooler.supabase.com`) | 6543 | IPv4 și IPv6 | **serverless** (funcții care pornesc și se opresc des) |

**Ce e un pooler.** Postgres suportă un număr limitat de conexiuni simultane, iar fiecare costă memorie. Un pooler stă în față, ține un set mic de conexiuni reale și le împarte între clienți. În **session mode** primești o conexiune a ta cât timp ești conectat. În **transaction mode** primești una doar pe durata unei tranzacții, apoi o ia altcineva. E mai eficient, dar fără prepared statements și fără setări de sesiune (pentru Prisma pui `?pgbouncer=true` în URL).

**Ce alegi în proiectele tale:**

- **P2/P3 (Nest pe Render):** **session pooler**. Render nu iese pe IPv6, deci conexiunea directă nu merge de acolo pe planul Free. De pe laptop, din WSL, s-ar putea să nu ai nici tu IPv6. Ca să nu ai două configurații, folosești session pooler-ul și local, și pentru migrații.
- **P4 (Next.js pe Vercel):** vorbești cu Supabase prin `supabase-js` (HTTP), nu prin connection string. Dacă vreodată ai nevoie de Postgres direct din funcții serverless, atunci transaction pooler.

**Capcana.** Un mesaj ca `getaddrinfo ENOTFOUND db.xxxx.supabase.co` sau un timeout la conectare, venit de pe Render sau din WSL, înseamnă aproape sigur că folosești conexiunea directă dintr-o rețea fără IPv6. Soluția e session pooler-ul, nu altă parolă.

<details>
<summary>Întrebare de interviu: Ce e connection pooling și de ce e nevoie de el?</summary>

Deschiderea unei conexiuni la Postgres e scumpă, iar baza de date suportă un număr limitat de conexiuni simultane, fiecare consumând memorie. Un pooler ține un set de conexiuni deschise și le refolosește între clienți. Pentru un server care stă pornit, pool-ul din aplicație plus session mode e de ajuns. Pentru serverless, unde pot porni sute de instanțe simultan, fiecare cu conexiunea ei, ai nevoie de transaction mode, ca să nu epuizezi conexiunile bazei de date.
</details>

## Cheile API: publishable și secret

**Ce e.** Pe ușa Data API, Supabase cere o cheie la fiecare request. Le găsești în **Project Settings → API Keys**. Sunt două tipuri:

| Cheie | Arată ca | Rol în Postgres | Unde are voie |
|---|---|---|---|
| **Publishable** | `sb_publishable_...` | `anon` (nelogat) sau `authenticated` (logat) | **frontend**, aplicații mobile, cod public |
| **Secret** | `sb_secret_...` | `service_role`, **ocolește complet RLS** | **doar pe server**: Nest, route handlers din Next.js, scripturi |

Proiectele mai vechi și multe tutoriale folosesc cheile **legacy**: `anon` (echivalentul publishable) și `service_role` (echivalentul secret), ambele JWT-uri lungi care încep cu `eyJ`. Supabase le retrage până la sfârșitul lui 2026. Tu folosești doar cheile noi. Dacă Claude scrie `SUPABASE_ANON_KEY` sau `SERVICE_ROLE_KEY`, cere-i varianta cu publishable/secret.

**De ce publishable poate fi publică.** Cheia publishable nu dă acces la nimic în sine. Doar identifică proiectul. Ce poate face cineva cu ea e decis de **RLS** (vezi [06](06-rls-auth-storage.md)): fără politici care să permită, nu vede niciun rând. Deci o cheie publishable în frontend e în regulă **doar dacă** ai RLS activat pe tabele.

**De ce secret niciodată în frontend.** Cheia secret ocolește RLS. Oricine o găsește în codul JavaScript din browser (și oricine o poate găsi, vezi [10-securitate-env/02](../10-securitate-env/02-ce-ajunge-in-browser.md)) are acces total la toate datele. Supabase refuză cheile secret venite dintr-un browser (răspunde 401), dar asta e o plasă de siguranță, nu o protecție. Cu o cheie scursă, oricine poate face request-uri din afara browserului. Dacă se scurge, o ștergi imediat din dashboard și creezi alta.

**Capcana.** În Next.js (P4), orice variabilă care începe cu `NEXT_PUBLIC_` ajunge în browser. `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` e corect. `NEXT_PUBLIC_SUPABASE_SECRET_KEY` e un incident de securitate. În Vite, același lucru e valabil pentru prefixul `VITE_`.

## Ce pui unde, în fiecare proiect

| Proiect | Frontend (public) | Backend (secret, doar în `.env` și Render/Vercel) |
|---|---|---|
| P2 Invoicer | nimic de la Supabase. Vorbește doar cu Nest | `DATABASE_URL` (session pooler, `?schema=invoicer`) |
| P3 Booking | `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY` (pentru login) | `DATABASE_URL` (session pooler, `?schema=booking`), `SUPABASE_URL` (pentru JWKS) |
| P4 Kitchen AI | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | cheia secret, doar dacă un route handler chiar are nevoie, plus cheia Claude API |

<details>
<summary>Întrebare de interviu: Cheia publishable Supabase e vizibilă în codul frontend-ului. E o problemă de securitate?</summary>

Nu, dacă baza de date e configurată corect. Cheia publishable e gândită să fie publică: identifică proiectul și te pune în rolul `anon` sau `authenticated`. Ce date poți accesa decide Row Level Security, prin politici pe fiecare tabel. Problema reală ar fi un tabel expus fără RLS: atunci oricine cu cheia publică l-ar putea citi sau modifica. Cheia secret, în schimb, ocolește RLS și trebuie să stea exclusiv pe server.
</details>

## `supabase-js`, pe scurt

**Ce e.** Librăria oficială JavaScript pentru ușa Data API. O folosești pentru Auth (P3, P4) și pentru date direct din frontend (P4).

**Exemplu minim:**

```ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,             // https://<ref>.supabase.co
  import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY, // safe in the browser, RLS decides access
);

// login (P3)
const { data, error } = await supabase.auth.signInWithPassword({ email, password });

// data through the Data API (P4) — RLS policies apply to every row
const { data: recipes } = await supabase.from('recipes').select('id, name, selling_price_cents');
```

**Capcana.** `supabase-js` nu aruncă excepții la erori. Întoarce `{ data, error }`. Dacă nu verifici `error`, codul merge mai departe cu `data = null` și cade trei funcții mai încolo, cu un mesaj fără legătură. Verifică mereu `if (error) ...` imediat după apel.

## Mai departe

- [Supabase — Connect to your database](https://supabase.com/docs/guides/database/connecting-to-postgres)
- [Supabase — Understanding API keys](https://supabase.com/docs/guides/api/api-keys)
