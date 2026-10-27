# Cum arată interviurile în 2026

Ca să te pregătești bine, trebuie să știi ce se testează la fiecare etapă. Aproape niciodată nu se testează ce crezi tu.

## Drumul tipic

```text
Aplici ──► Screening ──► Tehnic ─────────► (Take-home) ──► Discuția finală ──► Ofertă
           recruiter     live coding sau     proiect mic      manager / CTO
           20–30 min     pair cu AI          + discuție pe    behavioral,
                         60–90 min           codul tău        cultural fit
```

Variantele pe care le vei întâlni:

- **Firme mari și corporații:** 4–5 etape, uneori și un test online automat înainte de screening.
- **Startup-uri și agenții:** 2–3 etape. Adesea un singur tehnic, apoi o discuție cu fondatorul.
- **Contracte cu clienți străini:** de multe ori un call cu tech lead-ul, în care discutați un proiect de-al tău sau rezolvați împreună ceva mic, apoi o perioadă de probă plătită.

## Screening-ul cu recruiterul

**Cum arată.** Un call de 20–30 de minute, în engleză, cu cineva care nu e tehnic: cine ești, de ce cauți, ce stack știi, de când poți începe, ce așteptări salariale ai.

**Ce evaluează de fapt.** Dacă vorbești clar și coerent, dacă povestea ta are sens, dacă ești în bugetul lor și dacă engleza ta merge pentru echipă. Nu te va întreba ce e un closure.

**Cum te pregătești.**

- Povestea ta de 90 de secunde ([06](06-povestea-ta.md)), repetată cu voce tare până iese natural.
- Un interval salarial pe care îl poți susține. Briefing-ul tău are cifrele pe piață; actualizează-le cu ce vezi în anunțuri.
- Două-trei propoziții despre fiecare proiect, ca să-l poți descrie unui om care nu e developer.

**Capcana.** Să te scuzi pentru schimbarea de carieră („I know I'm not a typical candidate, but…”). Pentru recruiter, povestea ta e un plus: e memorabilă. Spune-o ca pe o decizie, nu ca pe o problemă.

## Interviul tehnic cu live coding

**Cum arată.** Un editor partajat (CoderPad, un VS Code pe share screen sau ceva similar), 45–60 de minute, o problemă practică. Pentru roluri de web, problemele sunt mai des „transformă datele astea”, „repară bug-ul ăsta” sau „fă o componentă care…” decât algoritmi grei.

**Ce evaluează de fapt.** Cum gândești, nu dacă ajungi la soluția perfectă:

- dacă pui întrebări înainte să tastezi;
- dacă spargi problema în pași;
- dacă explici ce faci;
- dacă îți testezi codul;
- cum reacționezi când primești un hint.

O soluție parțială explicată bine bate o soluție completă scrisă în tăcere.

**Cum te pregătești.** Metoda și kata-urile din [05-live-coding.md](05-live-coding.md). Unele firme încă dau probleme clasice de algoritmică, cu hash maps, doi pointeri sau stive. Kata-urile acoperă nivelul de bază.

**Capcana.** Tăcerea. Zece minute în care tastezi fără să spui nimic sunt, pentru intervievator, zece minute în care nu știe ce se întâmplă. Și nici nu te poate ajuta.

## Take-home

**Cum arată.** Primești o temă: o aplicație mică sau un feature, cu o durată sugerată de 2–6 ore. Apoi urmează o discuție pe codul tău, în care adesea ți se cere să schimbi ceva live.

**Ce evaluează de fapt.** Structura, README-ul, testele, istoricul de commit-uri și, mai ales, discuția de după. În 2026, majoritatea firmelor presupun că folosești AI la take-home. Ștacheta s-a mutat: codul trebuie să fie curat, iar tu trebuie să poți apăra fiecare decizie și să-l modifici pe loc.

**Cum te pregătești.** Exact ce faci în proiecte: commit-uri mici, README cu secțiunea „Decisions and trade-offs” și „What I'd do with more time”, câteva teste pe logica importantă.

**Capcana.** Trei, de fapt:

- predai cod generat pe care nu-l poți explica, iar discuția te demască în 5 minute;
- faci over-engineering (microservicii pentru un CRUD);
- stai 20 de ore pe o temă de 4.

Respectă timpul sugerat și scrie în README ce ai lăsat deoparte.

## Pair programming cu AI permis

**Cum arată.** Un format tot mai des întâlnit. Primești un repo și un task și ai voie să folosești Claude, Copilot sau ce folosești zilnic. Intervievatorul se uită *cum* lucrezi cu AI-ul.

**Ce evaluează de fapt.** Judecata ta: dacă citești ce generează AI-ul, dacă îl testezi, dacă prinzi greșelile, dacă pui la îndoială cerințele, dacă știi când să-l oprești și să scrii tu.

**Cum validezi output-ul AI în fața lor**, în cinci pași pe care îi spui cu voce tare:

1. **Spune ce ceri și de ce:** _„I'll ask it to add validation to this endpoint, because right now it accepts any body.”_
2. **Citește diff-ul cu voce tare:** _„OK, it added a DTO with class-validator… it marks `email` as optional, but our spec says it's required. I'll fix that.”_
3. **Verifică edge cases:** _„What happens with an empty array here? Let me run the test with that.”_
4. **Rulează:** testele, aplicația sau măcar un `curl`. Nu declara nimic gata fără să-l fi văzut mergând.
5. **Respinge când e greșit:** _„This uses a package I don't recognise. Let me check it actually exists before installing it.”_ AI-urile mai inventează pachete și API-uri. Vezi [12-lucrul-cu-ai/02-review-la-cod-generat.md](../12-lucrul-cu-ai/02-review-la-cod-generat.md).

**Capcana.** Fie accepți primul răspuns fără să-l citești, fie, invers, refuzi AI-ul ca să „demonstrezi” că știi să scrii cod, deși ei ți-au cerut explicit să-l folosești. Exact asta vor să vadă: cum lucrezi cu el.

## System design pentru junior

**Cum arată.** Nu sisteme distribuite la scară Google. O întrebare de tipul _„How would you build an invoicing app for freelancers?”_ sau _„Walk me through how your Booking app works, end to end.”_

**Ce evaluează de fapt.** Dacă înțelegi cum se leagă bucățile unei aplicații și dacă iei decizii cu motiv.

**Structura unui răspuns bun**, exemplificată pe Invoicer:

1. **Clarifici cerințele.** _„Who are the users? One freelancer or teams? Roughly how many invoices? Do we need PDF export?”_
2. **Desenezi cutiile:**

   ```text
   [React SPA, Vercel] ──HTTPS + JSON──► [NestJS API, Render] ──SQL──► [Postgres, Supabase]
         UI, forms,                      validation, auth,
         client-side checks              business rules (totals, numbering)
   ```

3. **Modelul de date:** `clients`, `invoices` (cu `client_id`, `number` unic, `status`, `due_date`) `invoice_items` (cu `invoice_id`, `quantity`, `unit_price_cents`) și `payments`. Cota de TVA stă pe factură, nu pe linie. Explici relațiile 1-n.
4. **Endpoint-urile cheie:** `GET /invoices?status=overdue&page=2`, `POST /invoices` (creează factura și liniile într-o singură tranzacție), `PATCH /invoices/:id/status`.
5. **Securitatea:** totalurile se calculează pe server, nu se acceptă de la client. Cheile stau doar pe backend. Fiecare query e filtrat după utilizatorul logat.
6. **Edge cases:** banii ținuți ca întregi în bani/cenți, numerotare fără duplicate (constraint `UNIQUE`), ce se întâmplă dacă doi utilizatori editează aceeași factură.
7. **Deploy și monitorizare:** unde rulează fiecare bucată, unde stau env vars, unde te uiți la loguri.
8. **Ce ai face mai departe:** paginare cu cursor, index pe `(user_id, status)`, joburi pentru remindere la facturile restante.

**Capcana.** Să sari direct la nume de tehnologii („I'd use Kafka and Redis”) fără să fi întrebat ce trebuie să facă aplicația. Cerințele întâi, tehnologia după.

## Behavioral

**Cum arată.** _„Tell me about a time you…”_: un conflict, o greșeală, o situație sub presiune, ceva învățat repede, un feedback greu.

**Ce evaluează de fapt.** Cum lucrezi cu oamenii, dacă îți asumi greșelile și dacă înveți din ele. Pentru un junior contează la fel de mult ca partea tehnică, pentru că asta e partea pe care n-o poate repara un mentor.

**Cum te pregătești.** Șase povești pregătite după structura STAR, în [06-povestea-ta.md](06-povestea-ta.md). Bucătăria e o mină de aur aici: ani de presiune reală, echipă, termene care nu se negociază.

**Capcana.** Răspunsuri generice („I'm a team player”) și „we” în loc de „I”. Intervievatorul vrea să știe ce ai făcut *tu*.

## Ce faci când nu știi

Se va întâmpla la fiecare interviu, și e în regulă. Intervievatorii din 2026 sapă intenționat până găsesc limita ta. Contează ce faci acolo.

- **Nu blufa.** Se vede imediat, iar următoarea întrebare de follow-up te expune complet.
- **Spune ce știi și raționează cu voce tare:** _„I haven't used Redis in a project, but from what I understand it's an in-memory store, so I'd expect it to be used for caching or sessions. Is that the kind of use you have in mind?”_
- **Leagă de ceva ce știi:** _„I haven't worked with GraphQL, but I've built REST APIs in NestJS. The main difference I know of is…”_
- **Spune cum ai afla:** _„I don't know off the top of my head. I'd check the Postgres docs for the exact syntax, but the idea is…”_
- **Pune o întrebare de clarificare.** Uneori nu știi pentru că întrebarea e vagă.

## Pe scurt: ce crezi că se evaluează vs. ce se evaluează de fapt

| Etapa | Ce crezi | Ce evaluează de fapt |
|---|---|---|
| Screening | Experiența | Claritate, coerență, buget, engleză |
| Live coding | Soluția corectă | Procesul: întrebări, pași, comunicare, testare |
| Take-home | Cod perfect | Decizii pe care le poți apăra, plus modificarea live |
| Pair cu AI | Dacă știi să dai prompt | Judecata: verifici, prinzi greșeli, respingi |
| System design | Tehnologii la modă | Cerințe → componente → date → trade-offs |
| Behavioral | Să pari perfect | Asumare, învățare, lucru cu oamenii |

## Mai departe

- [05 · Live coding](05-live-coding.md), [06 · Povestea ta](06-povestea-ta.md), [07 · Mock interviews](07-mock-interviews.md)
- [senior-tips.md](../senior-tips.md)
