# Contul Supabase

Supabase îți dă, gratuit, un Postgres adevărat în cloud, plus autentificare, stocare de fișiere și API-uri generate automat. Pentru P2–P4 e baza de date, iar pentru P3–P4 și sistemul de login. Aici îți faci contul și primul proiect și înveți unde e fiecare lucru în dashboard.

## Planul: două proiecte Supabase, în total

Planul Free permite **două proiecte active**. P2, P3 și P4 au nevoie de baze de date live, ca să poată fi deschise de recrutori, așa că le împarți așa:

| Proiect Supabase | Cine îl folosește | Ce conține |
|---|---|---|
| `portfolio-db` | **P2 Invoicer** și **P3 Booking** | schema Postgres `invoicer` (tabelele P2), schema `booking` (tabelele P3), plus Auth pentru user-ii din P3 |
| `kitchen-ai` | **P4 Kitchen AI** | tabelele în `public` (frontend-ul vorbește direct cu ele, protejat de RLS), plus Auth și Storage |

O **schemă** e un spațiu de nume pentru tabele în aceeași bază de date (vezi [01](01-model-relational.md#schemă-postgres)). P2 și P3 stau în aceeași bază, dar nu se văd și nu se calcă una pe alta: fiecare are tabelele și migrațiile ei. Primul proiect îl creezi acum, pentru P2. Al doilea abia la P4.

## Ce e Supabase

**Ce e.** O platformă construită în jurul unui Postgres. Fiecare proiect Supabase îți dă:

| Componentă | Ce face | Unde o folosești |
|---|---|---|
| **Database** | un Postgres complet, al tău | P2, P3, P4 |
| **Auth** | conturi, login (email/parolă, magic link, Google), JWT-uri | P3, P4 |
| **Storage** | fișiere (poze, PDF-uri) în bucket-uri, cu reguli de acces | P4 (facturile urcate) |
| **Data API** | endpoint-uri REST generate automat din tabele, folosite de `supabase-js` | P4 |
| **Realtime** | notificări când se schimbă date | opțional în P3 |

**De ce contează.** Nu e o bază de date „de jucărie”: e Postgres, deci tot ce înveți aici (SQL, indecși, constrângeri) se aplică la orice job cu Postgres. Supabase doar îți scutește administrarea serverului.

**Capcana.** Supabase poate fi folosit în două moduri foarte diferite. Fie frontend-ul vorbește direct cu baza de date prin `supabase-js` și securitatea stă în RLS (asta faci în P4), fie ai un backend propriu (Nest) care vorbește cu Postgres-ul și frontend-ul nu atinge baza de date (P2, P3). Să știi mereu în ce mod ești.

## Pașii: contul și primul proiect

1. **Contul.** Intră pe [supabase.com](https://supabase.com) (din browserul de pe Windows) → **Start your project** → **Continue with GitHub**. Folosește același cont de GitHub pe care ții proiectele.
2. **Organizația.** Ți se cere o organizație. Pune numele tău și alege planul **Free**.
3. **Proiectul nou** (**New project**):
   - **Name:** `portfolio-db` (îl împart P2 și P3, vezi planul de mai sus);
   - **Database password:** apasă **Generate a password** și salveaz-o imediat într-un password manager (Bitwarden e gratuit). Parola asta intră în connection string și dă acces complet la baza de date;
   - **Region:** **Central EU (Frankfurt)**, cea mai apropiată de tine și de user-ii tăi;
   - dacă formularul are și opțiuni de securitate (Data API, expunerea automată a tabelelor), lasă valorile implicite. Le înțelegi în [06](06-rls-auth-storage.md).
4. Apasă **Create new project** și așteaptă câteva minute până se pornește.

**Verifică.** Ajungi pe pagina proiectului, cu meniul lateral (Table Editor, SQL Editor, Authentication, Storage...) și fără mesaje de „setting up”.

**Ce s-a întâmplat de fapt.** Supabase ți-a pornit o mașină cu Postgres în Frankfurt, a creat utilizatorul de bază de date `postgres` cu parola ta și a configurat în fața ei Auth, Storage, Data API și un connection pooler.

**Capcana.** Dacă pierzi parola bazei de date, o poți reseta din setările proiectului. Dar orice aplicație care o folosea (`.env` local, Render) trebuie actualizată, altfel cade. De aceea o ții într-un singur loc sigur, nu în notițe.

## Turul dashboard-ului

**Ce găsești unde:**

- **Table Editor:** vezi și editezi datele ca într-un Excel. Bun pentru verificat ce a scris aplicația. Selectorul de schemă e sus-stânga.
- **SQL Editor:** rulezi SQL direct. Aici exersezi [02](02-sql-esential.md) și verifici ce face Prisma.
- **Database:** tabele, funcții, indecși, extensii, backup-uri.
- **Authentication:** user-ii înregistrați, metodele de login, template-urile de email (P3).
- **Storage:** bucket-urile cu fișiere (P4).
- **Advisors:** Security Advisor și Performance Advisor. Îți arată probleme reale, de exemplu tabele expuse fără RLS sau foreign keys fără index. Uită-te aici după fiecare migrație.
- **Butonul Connect** (sus): connection string-urile pentru baza de date. Detalii în [04](04-conectare-si-chei.md).
- **Project Settings → API Keys:** cheile publishable și secret.

**Capcana.** Table Editor e foarte comod pentru *date*, dar nu-l folosi ca să modifici *structura* (coloane, tabele) într-un proiect cu migrații Prisma. Schema din baza de date nu mai corespunde cu migrațiile, iar următorul `prisma migrate dev` îți propune să resetezi tot. Vezi [05](05-migratii.md).

## Limitele planului Free

Verificate pe [supabase.com/pricing](https://supabase.com/pricing) în octombrie 2026. Pot să se schimbe, deci verifică acolo dacă ceva nu se potrivește.

| Ce | Limita Free |
|---|---|
| Proiecte active | **2** |
| Baza de date | 500 MB per proiect |
| Storage (fișiere) | 1 GB, maximum 50 MB per fișier |
| Egress (trafic ieșit) | 5 GB |
| User-i activi lunar (Auth) | 50.000 |
| Inactivitate | proiectul e **pus pe pauză după o săptămână** fără activitate |

**Ce înseamnă pentru tine:**

- **Pauza.** Un proiect de portofoliu pe care nu-l accesează nimeni o săptămână se oprește. Când un recrutor dă click pe demo, primește erori. Îl repornești din dashboard (**Restore project**). Înainte să trimiți aplicări, intră în fiecare proiect și verifică-l.
- **Două proiecte active.** De aici vine [planul de mai sus](#planul-două-proiecte-supabase-în-total). Separarea pe scheme e o setare în Prisma, explicată în [07/05](../07-backend-node-nest/05-baza-de-date-cu-prisma.md). Tu verifici că fiecare proiect are migrațiile în schema lui și că nu vede tabelele celuilalt.
- **500 MB** ajung cu mult peste ce vei pune în proiectele de portofoliu.

<details>
<summary>Întrebare de interviu: Ce e Supabase și când l-ai folosi în loc de un backend propriu?</summary>

Supabase e o platformă construită peste Postgres, care adaugă autentificare, stocare de fișiere, API-uri generate automat și realtime. Îl folosesc când vreau să livrez repede: frontend-ul poate vorbi direct cu baza de date prin `supabase-js`, iar securitatea o pun în politici Row Level Security. Când am logică de business complexă, integrări externe sau vreau control complet asupra API-ului, pun un backend propriu, de exemplu NestJS, și folosesc Supabase doar ca Postgres gestionat și pentru Auth. În portofoliu am făcut ambele variante.
</details>

## Mai departe

- [Supabase — Docs, pagina principală](https://supabase.com/docs)
- [Supabase — Pricing și limite](https://supabase.com/pricing)
