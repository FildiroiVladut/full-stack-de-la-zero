# Full-stack de la zero

Drumul meu de la bazele JavaScript la primul job de full-stack developer: TypeScript, React, Node.js/NestJS, Supabase, Git în echipă, deploy și lucrul cu AI, plus 4 proiecte publicate.

E continuarea lui [js-de-la-zero](https://github.com/FildiroiVladut/js-de-la-zero), unde am refăcut fundamentele JavaScript.

## Cum funcționează

Două tipuri de lucru, nimic altceva:

1. **Fișe de concept.** Fiecare concept important explicat scurt: ce e, de ce contează, un exemplu minim, cum arată în cod real, capcana și întrebarea de interviu. Le citesc și rulez exemplele. Fără seturi de exerciții.
2. **Proiecte cap-coadă cu Claude Code.** Construiesc cu AI, apoi trec prin tot codul până pot explica fiecare fișier. Fiecare proiect e un repo separat, cu README în engleză, gata de portofoliu.

Regula de bază: **nu fac commit la cod pe care nu-l pot explica.**

Totul pe larg în [00-start](00-start/).

## Calendarul

5 ore pe zi (jumătate dimineața, jumătate seara), 6 zile pe săptămână. Calendarul e o direcție, nu un contract: dacă un capitol cere o zi în plus, o ia.

| Săpt. | Citesc | Construiesc | Checkpoint |
|---|---|---|---|
| 1 | [00](00-start/), [01](01-setup/), [02](02-cum-functioneaza-webul/), [03](03-ecosistem-js/), [04](04-javascript/), [09](09-git-github-echipa/) (doar 01) | — | [CP1](checkpoints/CP1.md) |
| 2 | [05](05-typescript/), [06](06-react/) (01–07), [12](12-lucrul-cu-ai/), [11](11-deploy/) (01–02) | [P1 Shelf](proiecte/P1-shelf.md) | [CP2](checkpoints/CP2.md) |
| 3–4 | [07](07-backend-node-nest/) (01–05), [08](08-baze-de-date-supabase/) (01–05), [10](10-securitate-env/), [11](11-deploy/) (03–04) | [P2 Invoicer](proiecte/P2-invoicer.md) | [CP3](checkpoints/CP3.md) · încep să aplic |
| 5–6 | [07](07-backend-node-nest/) (06–07), [08](08-baze-de-date-supabase/) (06), [09](09-git-github-echipa/) (02–06) | [P3 Booking](proiecte/P3-booking.md) | [CP4](checkpoints/CP4.md) |
| 7–8 | [06](06-react/) (08 — Next.js), [13](13-interviuri/) intensiv | [P4 Kitchen AI](proiecte/P4-kitchen-ai.md) | [CP5](checkpoints/CP5.md) |

Din săptămâna 2: 15 minute pe zi de întrebări de interviu din [13-interviuri](13-interviuri/).

## Proiectele

| | Proiect | Ce face | Stack |
|---|---|---|---|
| P1 | [Shelf](proiecte/P1-shelf.md) | Caut cărți, îmi fac rafturi de lectură, văd statistici | React, TypeScript, Vite, TanStack Query |
| P2 | [Invoicer](proiecte/P2-invoicer.md) | Facturare pentru freelanceri: clienți, facturi, încasări | React + NestJS + Prisma + Supabase |
| P3 | [Booking](proiecte/P3-booking.md) | Rezervări de săli/terenuri, cu conturi și roluri | + Supabase Auth, teste, CI, flux de PR-uri |
| P4 | [Kitchen AI](proiecte/P4-kitchen-ai.md) | Urc factura furnizorului, AI-ul o citește, costurile rețetelor se recalculează | Next.js, Supabase, Claude API |

## Progres

**Capitole**

- [ ] [00 · Start](00-start/) — metoda, ritmul, cum arată o fișă
- [ ] [01 · Setup](01-setup/) — WSL, Warp, Git, Node, Bun, VS Code, Claude Code
- [ ] [02 · Cum funcționează web-ul](02-cum-functioneaza-webul/) — client/server, HTTP, REST, frontend vs backend
- [ ] [03 · Ecosistemul JS](03-ecosistem-js/) — runtime, npm, Bun, Vite, harta uneltelor
- [ ] [04 · JavaScript](04-javascript/) — recap + async, module, DOM
- [ ] [05 · TypeScript](05-typescript/) — tipuri, narrowing, generics, zod
- [ ] [06 · React](06-react/) — componente, state, efecte, routing, date, Next.js
- [ ] [07 · Backend cu Node și NestJS](07-backend-node-nest/) — server, Nest, validare, Prisma, auth, teste
- [ ] [08 · Baze de date și Supabase](08-baze-de-date-supabase/) — SQL, Supabase, chei, migrații, RLS
- [ ] [09 · Git și GitHub în echipă](09-git-github-echipa/) — branch-uri, PR, code review, CI
- [ ] [10 · Securitate și env](10-securitate-env/) — ce ajunge în browser, secrete, atacuri, autorizare
- [ ] [11 · Deploy](11-deploy/) — Vercel, Render, loguri
- [ ] [12 · Lucrul cu AI](12-lucrul-cu-ai/) — workflow, review la cod generat, prompturi
- [ ] [13 · Interviuri](13-interviuri/) — întrebări, live coding, povestea mea, mock interviews

**Proiecte**

- [ ] P1 Shelf — live: _link_ · repo: _link_
- [ ] P2 Invoicer — live: _link_ · repo: _link_
- [ ] P3 Booking — live: _link_ · repo: _link_
- [ ] P4 Kitchen AI — live: _link_ · repo: _link_

**Checkpoints** (le bifez prin PR, vezi [checkpoints](checkpoints/))

- [ ] CP1 · [ ] CP2 · [ ] CP3 · [ ] CP4 · [ ] CP5

## Fișiere utile

- [senior-tips.md](senior-tips.md) — lucrurile pe care seniorii le știu din experiență
- [jurnal.md](jurnal.md) — trei rânduri pe zi
- [templates/](templates/) — `CLAUDE.md`, `ARCHITECTURE.md`, README și PR template pentru proiecte
- [CLAUDE.md](CLAUDE.md) — instrucțiunile pentru Claude Code când lucrez în repo-ul ăsta

Plan construit împreună cu mentorul meu, Bogdan.
