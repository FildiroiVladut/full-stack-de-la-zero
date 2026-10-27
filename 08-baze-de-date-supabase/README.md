# 08 · Baze de date și Supabase

Orice aplicație cu care lucrezi la job are o bază de date în spate, și cel mai des e una relațională: Postgres, MySQL, SQL Server. Aici înveți modelul relațional și SQL-ul de care ai nevoie zilnic, apoi Supabase, platforma care îți dă gratuit un Postgres în cloud, plus autentificare și stocare de fișiere. Dacă înțelegi capitolul ăsta, nu mai ai „magie” între backend și date.

## La final poți să…

- desenezi schema unei aplicații (tabele, chei, relații 1-n și n-n) și să explici de ce ai pus fiecare index;
- scrii query-uri cu JOIN, GROUP BY și agregări, și să faci UPDATE/DELETE fără să strici date;
- creezi un proiect Supabase și să alegi connection string-ul potrivit pentru fiecare situație;
- spui ce cheie Supabase are voie în frontend și care niciodată;
- explici ce e o migrație și ce face Row Level Security.

## Fișierele, în ordine

- [ ] [01 · Modelul relațional](01-model-relational.md) — 45 min
- [ ] [02 · SQL esențial](02-sql-esential.md) — 90 min (cu SQL Editor deschis)
- [ ] [03 · Contul Supabase](03-cont-supabase.md) — 30 min
- [ ] [04 · Conectare și chei](04-conectare-si-chei.md) — 45 min
- [ ] [05 · Migrații](05-migratii.md) — 30 min
- [ ] [06 · RLS, Auth, Storage](06-rls-auth-storage.md) — 60 min (săptămâna 5, pentru P3 și P4)

01–05 le citești în săptămânile 3–4, cât faci P2. Ordinea bună e 01 → 03 → 02: faci contul, apoi exersezi SQL-ul direct în proiectul tău.

## Unde îl folosești

- [P2 Invoicer](../proiecte/P2-invoicer.md): schema de facturi, Prisma peste Postgres-ul din Supabase, migrații.
- [P3 Booking](../proiecte/P3-booking.md): Supabase Auth, profiluri și roluri, constrângeri contra rezervărilor duble.
- [P4 Kitchen AI](../proiecte/P4-kitchen-ai.md): relația n-n rețete–ingrediente, Storage pentru facturile urcate, RLS.
- [CP3](../checkpoints/CP3.md) și [CP4](../checkpoints/CP4.md). Pentru interviu: [întrebările de backend și DB](../13-interviuri/04-intrebari-backend-db.md).

## Verifică-te

1. Un client are mai multe facturi, o factură are mai multe linii. Cum arată tabelele și unde stau foreign keys?
2. Care e diferența dintre `INNER JOIN` și `LEFT JOIN`, pe exemplul „clienți și facturile lor”?
3. De ce un server Nest pe Render folosește session pooler-ul și nu conexiunea directă?
4. Cheia publishable e în codul frontend-ului, pe care oricine îl poate citi. De ce e în regulă, și ce trebuie să fie activat ca să fie în regulă?
5. Ce se întâmplă dacă modifici de mână o coloană în Table Editor, pe un proiect care folosește migrații Prisma?
