# Proiectele

Patru proiecte, patru domenii, patru tipuri de aplicații. Cine deschide GitHub-ul tău vede un frontend curat, un full-stack de business, o aplicație cu conturi, teste și CI, și un produs AI dintr-un domeniu pe care îl cunoști ca puțini alții. Fiecare demonstrează altceva, așa că nu se repetă.

| | Proiect | Domeniu | Tip | Ce demonstrează | Când |
|---|---|---|---|---|---|
| P1 | [Shelf](P1-shelf.md) | Cărți, consumer | Frontend SPA + API public | React, TypeScript, date asincrone, UX corect | Săpt. 2 |
| P2 | [Invoicer](P2-invoicer.md) | Facturare, B2B | Full-stack: React + NestJS + Postgres | API-uri, baze de date, reguli de business, deploy pe două servicii | Săpt. 3–4 |
| P3 | [Booking](P3-booking.md) | Programări, SaaS | Multi-user cu roluri | Auth, autorizare, concurență, teste, CI, lucru prin PR-uri | Săpt. 5–6 |
| P4 | [Kitchen AI](P4-kitchen-ai.md) | Bucătărie profesională | Produs AI full-stack | Next.js, integrare LLM, evals, utilizatori reali | Săpt. 7–8 |

Numele sunt de lucru. Dacă îți vine unul mai bun, schimbă-l.

## Reguli comune

- **Repo separat și public pe GitHub-ul tău**, cu nume în engleză.
- **Totul în engleză:** codul, commit-urile, issues, PR-urile și README-ul. Cu Claude vorbești în română.
- **La început, copiezi din [templates](../templates/):**
  - `CLAUDE.template.md` → `CLAUDE.md` (și îl completezi pentru proiect)
  - `ARCHITECTURE.md` → `ARCHITECTURE.md`
  - `README-proiect.md` → `README.md`
  - `pull_request_template.md` → `.github/pull_request_template.md`
- **Bucla în 7 pași** din [00-start/01-metoda.md](../00-start/01-metoda.md).
- **Deploy din prima zi.** Primul PR al fiecărui proiect pune online o pagină goală. Ce urmează se vede live la fiecare merge.

## Definition of done (pentru toate)

Un proiect e gata când:

- [ ] rulează local după instrucțiunile din README, pe un clone curat;
- [ ] e live, cu linkul în README și în descrierea repo-ului;
- [ ] README-ul în engleză are: ce face, screenshot sau GIF, stack, cum îl rulezi, deciziile importante și ce ai face în continuare;
- [ ] `ARCHITECTURE.md` e scris de tine;
- [ ] `.env.example` e complet și niciun secret nu e în git (nici în istorie);
- [ ] lint și typecheck trec fără erori;
- [ ] feature-ul de mână e marcat în README („Built by hand: …”);
- [ ] Bogdan a făcut review la cel puțin un PR;
- [ ] repo-ul e pinned pe profilul tău de GitHub.

## Portofoliul

- **La CP3** (când începi să aplici): un README de profil pe GitHub (repo cu numele tău de utilizator), primele repo-uri pinned, LinkedIn și CV în engleză actualizate.
- **La CP5:** profilul complet cu toate 4 proiectele. Un site de portofoliu separat e opțional: o zi de lucru, Next.js pe Vercel. Un profil de GitHub bine făcut plus LinkedIn ajung.
