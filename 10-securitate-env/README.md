# 10 · Securitate și variabile de mediu

Cele mai multe breșe din aplicațiile scrise de juniori nu vin din atacuri sofisticate. Vin din lucruri banale: o cheie pusă în frontend, un `.env` urcat pe GitHub, un endpoint care nu verifică al cui e obiectul cerut, un total de factură primit de la client fără să fie verificat. Capitolul ăsta acoperă exact ce verifică un senior primul la review. Tot asta întreabă și intervievatorii, ca să vadă dacă ai lucrat pe ceva real.

## La final poți să

- explici unde stă fiecare secret din P2–P4 și de ce acolo;
- arăți în DevTools că tot ce ajunge în frontend e public;
- știi ce faci în primele 10 minute după ce ai scăpat o cheie;
- validezi input-ul pe server și explici pe înțeles SQL injection, XSS și CSRF;
- faci diferența între autentificare și autorizare și scrii endpoint-uri fără IDOR.

## Fișierele, în ordine

- [ ] [01 · Variabile de mediu](01-env-vars.md) — 35 min
- [ ] [02 · Ce ajunge în browser](02-ce-ajunge-in-browser.md) — 40 min, cu demonstrația din DevTools
- [ ] [03 · Secrete scurse](03-secrete-scurse.md) — 25 min
- [ ] [04 · Validare și atacuri](04-validare-si-atacuri.md) — 50 min
- [ ] [05 · Autentificare vs autorizare](05-autentificare-vs-autorizare.md) — 45 min

Îl citești în săptămânile 3–4, odată cu P2. Fișierul 05 îl recitești la începutul lui P3.

## Unde îl folosești

- **P2 Invoicer:** `.env` și `.env.example` pe backend, validare cu DTO-uri, totaluri calculate pe server, CORS doar pentru domeniul tău de frontend.
- **P3 Booking:** autentificare cu Supabase Auth, roluri, verificări de ownership, rate limiting.
- **P4 Kitchen AI:** cheia Claude stă doar pe server, upload de fișiere validat, endpoint-ul de AI protejat ca să nu-ți consume altcineva creditele.
- **Checkpoint-uri:** [CP3](../checkpoints/CP3.md) și [CP4](../checkpoints/CP4.md) întreabă direct despre lucrurile astea.

## Verifică-te

1. De ce cheia publishable Supabase poate sta în frontend, iar cheia secret nu?
2. Ai urcat din greșeală cheia Anthropic pe GitHub. Care sunt primii trei pași?
3. De ce nu e de ajuns validarea din formularul React?
4. Ce e IDOR? Arată-mi cum ar arăta în `GET /invoices/:id` și cum îl repari.
5. Ce diferență e între 401 și 403?
