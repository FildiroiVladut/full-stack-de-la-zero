# Live coding

Construiești cu AI în fiecare zi. Unele firme te vor pune totuși să scrii cod fără el, într-un editor partajat, cu cineva care se uită. Nu caută geniu algoritmic, caută un om cu care ar putea lucra: unul care gândește clar, vorbește în timp ce lucrează și își verifică singur codul.

## Metoda în 6 pași

Fiecare pas are frazele pe care le poți spune cu voce tare. Învață structura, nu frazele exacte.

### 1. Repeți problema și clarifici

Înainte să tastezi ceva, spune problema cu cuvintele tale și întreabă ce nu e clar: ce intră, ce iese, ce cazuri limită există, ce constrângeri sunt.

> _"So I get an array of invoices and I need to return the total per client, right? Can the array be empty? Can amounts be negative, like credit notes? Should the result be sorted?"_

### 2. Rezolvi un exemplu de mână

Ia un input mic și spune rezultatul așteptat. Așa verifici că ai înțeles, și ai deja un test.

> _"If I have two invoices for client A, 100 and 50, and one for B, 30, I expect `{ A: 150, B: 30 }`."_

### 3. Spui planul înainte de cod

Câteva comentarii sau pseudo-cod, cu voce tare. Intervievatorul te poate corecta acum, nu după 15 minute.

> _"My plan: loop once over the invoices, keep an object keyed by client ID, and add each amount to it. That's O(n). Does that sound OK before I start?"_

### 4. Întâi varianta simplă, apoi o îmbunătățești

O soluție simplă care merge bate o soluție elegantă neterminată.

> _"I'll write the straightforward version first, then we can see if it needs optimizing."_

### 5. Testezi

Treci exemplul de la pasul 2 prin cod, linie cu linie, sau rulează-l. Apoi cazurile limită: input gol, un singur element, duplicate, valori invalide.

> _"Let me run it with the example… OK. Now an empty array: I get `{}`, which I think is right. And one invoice with amount zero…"_

### 6. Vorbești despre complexitate și ce ai face mai departe

> _"This is O(n) time and O(k) space, where k is the number of clients. If this ran on millions of rows, I'd do the aggregation in SQL with GROUP BY instead of loading everything into memory."_

## Ce spui când te blochezi

Blocajele sunt normale. Tăcerea nu e. Ordinea în care ieși din blocaj:

1. **Spune că te-ai blocat și unde:** _"I'm stuck on how to handle the overlap check, let me think out loud."_
2. **Întoarce-te la exemplu** și rezolvă-l de mână, pas cu pas. Codul vine din pașii ăia.
3. **Rezolvă o versiune mai mică:** fără cazurile limită, cu un singur element, doar cu numere pozitive.
4. **Cere un hint pe ceva anume:** _"Could I get a hint on the data structure?"_ Să ceri un hint e normal și arată maturitate. Să stai în tăcere 10 minute nu.
5. **Pune `console.log`** dacă poți rula codul. E debugging real și e perfect acceptabil.

## Mediul

- **Fără AI, uneori fără autocomplete.** Din când în când, exersează într-un editor simplu sau în CoderPad, nu doar în VS Code.
- **Știi să rulezi rapid:** `node kata.js` pentru JavaScript, `npx tsx kata.ts` pentru TypeScript.
- **Pentru componentele React** ai nevoie de un proiect Vite gol, pregătit dinainte, ca să nu pierzi timp cu setup-ul.
- **Algoritmică clasică:** unele firme, de obicei cele mari, încă dau probleme în stilul LeetCode. Minimul de știut: hash maps (obiecte și `Map`), doi pointeri, stivă, recursivitate simplă, sortare și ce înseamnă O(n) vs O(n²). Kata-urile de mai jos acoperă nivelul ăsta.

## Cum exersezi kata-urile

- **Când și cât:** de 3 ori pe săptămână, câte 20 de minute, cu timer, fără AI. Într-un folder separat, de exemplu `~/code/kata`.
- **Cu voce tare**, cu metoda de mai sus, de parcă ar fi cineva lângă tine. Înregistrează-te din când în când; e neplăcut, dar o să auzi exact unde taci.
- **După ce termini** sau după ce expiră timpul, îi dai codul lui Claude pentru review:

  ```text
  Here is my solution to this kata, written without AI in 20 minutes: [problem] [code].
  Review it like an interviewer: correctness, edge cases I missed, readability,
  complexity. Don't rewrite it; point out the 3 most important issues.
  ```

- **Ce n-a ieșit** treci în [jurnal.md](../jurnal.md) și refaci peste o săptămână, de la zero.

## Cele 20 de kata-uri

Fără soluții, intenționat. Ordinea e crescătoare ca dificultate.

### Nivelul 1 — încălzire

1. **Coșul.** Primești `[{ name, priceCents, qty }]`. Întoarce totalul în bani/cenți. Apoi ignoră produsele cu `qty <= 0`.
2. **Numără cuvintele.** Pentru un text, întoarce un obiect `cuvânt → număr de apariții`, fără să ții cont de majuscule și de punctuație.
3. **Tag-uri unice.** Primești o listă de tag-uri cu duplicate. Întoarce-le unice și sortate alfabetic, fără să modifici array-ul original.
4. **Facturi restante.** Primești `[{ number, dueDate, status }]` și o dată de referință. Întoarce numerele facturilor neplătite cu `dueDate` trecut.
5. **Formatează prețul.** `formatPrice(123450, 'RON')` trebuie să întoarcă prețul formatat în stil românesc, cu `Intl.NumberFormat`. Apoi aceeași funcție pentru `EUR`.

### Nivelul 2 — transformări de date

6. **groupBy.** Scrie `groupBy(items, keyFn)` fără `Object.groupBy`. Folosește-o ca să grupezi cărți după autor. Apoi arată varianta cu `Object.groupBy`.
7. **Top 3.** Din `[{ title, rating, pages }]`, întoarce primele 3 cărți după rating. La egalitate câștigă cea cu mai puține pagini.
8. **Paranteze echilibrate.** Verifică dacă un string cu `()[]{}` e echilibrat, de exemplu `"{[()]}"` e valid și `"([)]"` nu.
9. **Rezervări suprapuse.** Primești rezervările unei săli, `[{ start, end }]`, ca ore. Întoarce `true` dacă oricare două se suprapun. Discută complexitatea înainte și după ce sortezi.
10. **Diff de liste.** Primești două liste de obiecte cu `id`: lista veche și lista nouă. Întoarce `{ added, removed, changed }`.
11. **Arbore în listă.** Primești categorii imbricate (`{ name, children: [...] }`). Întoarce toate căile ca stringuri: `"Books > Fiction > Sci-fi"`.

### Nivelul 3 — async și utilitare

12. **debounce.** Implementează `debounce(fn, ms)`. Explică unde l-ai folosi.
13. **fetchJson.** Scrie `fetchJson(url)`: aruncă o eroare clară dacă `!res.ok`, renunță după 5 secunde (`AbortSignal.timeout`) și întoarce JSON-ul parsat.
14. **retry.** Scrie `retry(fn, attempts)`, care reîncearcă o funcție async la eșec, cu pauze de 200, 400, 800 ms.
15. **Paralel cu limită.** Primești o listă de funcții async. Rulează-le câte maximum 3 deodată și întoarce rezultatele în ordinea inițială.
16. **memoize.** Scrie `memoize(fn)` pentru funcții cu un singur argument. Ce se întâmplă dacă argumentul e un obiect?

### Nivelul 4 — React, backend, SQL

17. **Listă filtrabilă (React).** Un input și o listă de cărți. Filtrezi după titlu în timp ce scrii și afișezi „No results” când nu găsești nimic. Input controlat, key-uri corecte.
18. **Coș de cumpărături (React).** Adaugi și scoți produse, vezi totalul, butonul „−” e dezactivat la 0. State-ul e un array de obiecte actualizat imutabil.
19. **Endpoint cu validare.** `POST /bookings`, în Express sau Nest, primește `{ roomId, start, end }`. Validează că există toate câmpurile și că `end > start`. Întoarce `400` cu un mesaj clar sau `201` cu obiectul creat. Datele stau în memorie.
20. **SQL.** Ai tabelele `clients(id, name)` și `invoices(id, client_id, total_cents, status, issued_at)`. Scrie query-ul pentru primii 5 clienți după suma încasată în 2026, cu numărul de facturi. Apoi o variantă care include și clienții fără nicio factură.

## Mai departe

- [02 · Întrebări JS și TS](02-intrebari-js-ts.md) — teoria din spatele kata-urilor 12–16
- [07 · Mock interviews](07-mock-interviews.md) — mock de live coding cu timer
- [MDN — Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)
