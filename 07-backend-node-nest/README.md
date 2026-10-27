# 07 · Backend cu Node.js și NestJS

Frontend-ul arată date, iar backend-ul decide ce date există, cine are voie să le vadă și ce înseamnă „valid”. Aici înveți cum arată un server de la zero, apoi de ce echipele aleg NestJS: o structură fixă (module, controllers, services), validare la intrare, configurare din env, acces la baza de date prin Prisma, autentificare cu guards și teste. E stack-ul din P2 și P3 și unul dintre cele mai cerute la joburile de Node.

## La final poți să…

- explici ce face un server HTTP și ce îți scutește Nest de scris de mână;
- desenezi structura unui modul Nest (controller → service → Prisma) și să spui ce face fiecare strat;
- validezi orice input cu DTO-uri și să returnezi coduri de status corecte;
- conectezi Nest la Postgres (Supabase) prin Prisma, cu relații și tranzacții;
- protejezi rute cu un guard care verifică JWT-ul și rolul user-ului;
- scrii un unit test și un e2e test pentru o regulă de business.

## Fișierele, în ordine

- [ ] [01 · Un server HTTP de la zero](01-server-http-de-la-zero.md) — 45 min
- [ ] [02 · Arhitectura Nest](02-nest-arhitectura.md) — 60 min
- [ ] [03 · DTO, validare, erori](03-dto-validare-erori.md) — 45 min
- [ ] [04 · Config și env](04-config-si-env.md) — 30 min
- [ ] [05 · Baza de date cu Prisma](05-baza-de-date-cu-prisma.md) — 75 min
- [ ] [06 · Guards și auth](06-guards-si-auth.md) — 60 min (săptămâna 5, pentru P3)
- [ ] [07 · Teste](07-teste.md) — 45 min (săptămâna 5–6, pentru P3)

01–05 le citești în săptămânile 3–4, cât construiești P2. 06–07 țin de P3.

## Unde îl folosești

- [P2 Invoicer](../proiecte/P2-invoicer.md): API-ul de clienți, facturi și încasări (01–05).
- [P3 Booking](../proiecte/P3-booking.md): auth, roluri, regula de no double booking și testele (06–07).
- [CP3](../checkpoints/CP3.md) și [CP4](../checkpoints/CP4.md).
- Înainte de interviu: [întrebările de backend](../13-interviuri/04-intrebari-backend-db.md).

## Verifică-te

1. Un request `POST /invoices` intră în aplicația Nest. Prin ce trece, în ordine, până ajunge în baza de date?
2. De ce totalul facturii se calculează în service și nu se primește din frontend?
3. Ce se întâmplă la pornire dacă lipsește `DATABASE_URL`, și de ce e bine așa?
4. Care e diferența dintre un guard care verifică JWT-ul și unul care verifică rolul?
5. Ce testezi cu un unit test și ce cu un e2e test la regula „nu poți rezerva un interval deja ocupat”?
