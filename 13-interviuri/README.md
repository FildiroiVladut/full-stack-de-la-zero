# 13 · Interviuri

Piața din 2026 nu mai plătește potențial, plătește dovezi. Proiectele tale sunt dovada, iar interviul e locul unde dovada trebuie să sune a înțelegere, nu a prompt bine dat. Capitolul ăsta te antrenează pentru fiecare etapă: povestea ta, întrebările tehnice, live coding fără AI, prezentarea proiectelor și pair programming cu AI pe față.

**Limba.** Majoritatea interviurilor pe care le vei da, cu clienți străini și cu multe firme din România, se țin în engleză. De aceea întrebările și răspunsurile-model sunt în engleză, exact cum le vei auzi și cum trebuie să le spui. Explicațiile și sfaturile din jur sunt în română.

## La final poți să

- explici cu voce tare, în engleză și fără notițe, conceptele de bază din JS/TS, React, backend și baze de date;
- rezolvi o problemă de live coding fără AI, gândind cu voce tare;
- îți spui povestea în 90 de secunde fără să sune a scuză;
- prezinți oricare din cele 4 proiecte în 2 minute și răspunzi la „de ce?” pentru fiecare decizie;
- folosești AI-ul în fața unui intervievator așa cum vrea el să te vadă: verificând tot ce-ți dă.

## Fișierele

- [ ] [01 · Cum arată interviurile](01-cum-arata-interviurile.md) — 30 min
- [ ] [02 · Întrebări JavaScript și TypeScript](02-intrebari-js-ts.md) — bancă de întrebări, se repetă
- [ ] [03 · Întrebări React și Next.js](03-intrebari-react.md) — bancă de întrebări, se repetă
- [ ] [04 · Întrebări backend, baze de date, securitate, git](04-intrebari-backend-db.md) — bancă de întrebări, se repetă
- [ ] [05 · Live coding](05-live-coding.md) — metoda + 20 de kata-uri
- [ ] [06 · Povestea ta](06-povestea-ta.md) — 2–3 ore de scris, apoi repetat
- [ ] [07 · Mock interviews](07-mock-interviews.md) — prompturi gata de copiat

## Cum folosești capitolul

Nu se citește o dată și gata. E un antrenament care merge în paralel cu restul planului.

| Când | Ce faci | Cât |
|---|---|---|
| Săpt. 1 | Citești doar 01, ca să știi spre ce mergi | 30 min |
| Săpt. 2–6 | Încălzire dimineața: 5 întrebări din bancă, pe tema săptămânii | 15 min/zi |
| Din săpt. 2 | Kata-uri fără AI din 05 | 20 min, de 3 ori pe săpt. |
| Din săpt. 3 | Un mock interview cu Claude din 07 | 45 min/săpt. |
| Săpt. 4 | Scrii povestea și prezentarea pentru P1–P2 (06), înainte de primele aplicări | 2–3 ore |
| Săpt. 7–8 | Intensiv: mock-uri, prezentarea tuturor proiectelor, kata-urile picate | 45–60 min/zi |

Tema încălzirii, pe săptămâni:

| Săpt. | Banca |
|---|---|
| 2 | [02](02-intrebari-js-ts.md) — fundamente, async, TypeScript |
| 3–4 | [04](04-intrebari-backend-db.md) — HTTP/REST, Node/Nest, SQL |
| 5–6 | [03](03-intrebari-react.md) — React, apoi [04](04-intrebari-backend-db.md) — auth, securitate, git |
| 7–8 | Tot, amestecat, plus partea de Next.js din [03](03-intrebari-react.md) |

### Flashcards cu Claude

Deschide Claude Code în rădăcina repo-ului de ghiduri și dă-i promptul ăsta (schimbă fișierul și secțiunea):

```text
Read 13-interviuri/02-intrebari-js-ts.md. Quiz me on 5 questions from the "Async" section,
one at a time, in English, without showing me the model answer. Wait for my answer.
Then score it 1–5, tell me what was missing compared to the model answer, and show me
a better phrasing in at most 3 sentences. At the end, list the questions I scored
below 4 so I can add them to my review list.
```

Trei reguli care fac diferența:

1. **Răspunzi cu voce tare înainte să scrii.** La interviu vorbești, nu scrii. Spune răspunsul, apoi tastează-l pe scurt pentru Claude.
2. **Ce ia sub 4 intră în lista de repetat** din [jurnal.md](../jurnal.md). Revine a doua zi, apoi după 3 zile, apoi după o săptămână.
3. **Nu învăța răspunsurile pe de rost.** Răspunsul-model îți arată ce idei trebuie să conțină răspunsul. Formularea trebuie să fie a ta, altfel sună recitat și se rupe la prima întrebare de follow-up.

## Unde îl folosești

- **Săpt. 4 ([CP3](../checkpoints/CP3.md)):** primele aplicări și primul mock cu Bogdan, pe P2.
- **Săpt. 8 ([CP5](../checkpoints/CP5.md)):** mock complet cu Bogdan: screening, tehnic, proiect, behavioral.
- **După fiecare interviu real:** notezi în jurnal ce te-au întrebat și unde te-ai împiedicat. Întrebările noi intră în bancă.

## Verifică-te

- Care sunt etapele de interviu la care te simți cel mai slab acum și ce faci săptămâna asta pentru fiecare?
- Poți spune în 90 de secunde, în engleză, cine ești și de ce te-ai întors la programare?
- Cum îi arăți unui intervievator că ai verificat codul generat de AI, nu doar că l-ai acceptat?
- Care e întrebarea din bănci care ți se pare cea mai grea și de ce?
