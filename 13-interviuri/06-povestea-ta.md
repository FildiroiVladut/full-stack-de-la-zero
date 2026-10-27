# Povestea ta

Ai un avantaj pe care majoritatea juniorilor nu-l au: o poveste memorabilă. Cibernetică, C/C++, ani de bucătărie profesionistă, apoi întoarcerea la cod cu un mentor și 4 proiecte livrate. Prost spusă, sună a „am încercat altceva și n-a mers”. Bine spusă, sună a „om matur, care a ales asta conștient și livrează sub presiune”. Fișierul ăsta te ajută s-o spui bine.

Scrie-ți versiunile tale în engleză, într-un document separat, și repetă-le cu voce tare până nu mai sună citite. Textele de mai jos sunt schelete: ce e între `[paranteze]` completezi cu fapte reale. Nu inventa nimic. La interviu, orice detaliu poate primi o întrebare de follow-up.

## „Tell me about yourself”

E aproape mereu prima întrebare și dă tonul. Țintește 60–90 de secunde și o structură în trei timpi: **cine ești acum → cum ai ajuns aici → de ce rolul ăsta.**

> _"I'm a full-stack developer working with TypeScript, React and NestJS. Over the past few months I've built and deployed four projects. The one I'm proudest of is Kitchen AI: it reads supplier invoices with an LLM and recalculates recipe costs, and I built it for a problem I know from the inside._
>
> _My background is a bit unusual. I studied Cybernetics at ASE in Bucharest, where I wrote C and C++ and did some early web work. Then I spent [about ten] years in professional kitchens, up to [your role]: running service under pressure, coordinating a team, managing costs and suppliers._
>
> _This year I came back to software deliberately, with a senior engineer as my mentor, and I learned the way teams work now: AI-assisted development, but every line reviewed and understood, pull requests, CI, deployed apps._
>
> _I'm looking for a role where I can ship real features in a team and keep growing fast, and [company] interests me because [one specific, real reason]."_

**Ce să eviți:**

- să începi cu „So, I was born in…”;
- să-ți ceri scuze pentru pauza de la programare;
- să citezi tot CV-ul;
- să termini fără să legi povestea de rolul lor.

## „Why the career switch?”

Spune ce te-a atras înapoi, nu de ce fugi. Nu vorbi urât despre bucătărie: e experiența ta, și intervievatorul se întreabă cum vei vorbi despre firma lui peste doi ani.

> _"I always liked building things that work. That's what drew me to Cybernetics and to programming in the first place. Kitchens taught me a lot: discipline, working under real pressure, being part of a team where everyone depends on everyone. But I missed solving problems with code, and I could see how much software could change even the kitchens I worked in. So I made a deliberate plan, found a mentor, and spent the last months building real projects. It wasn't a whim; it was a decision with a plan behind it."_

## „Why should we hire you?” / „Why hire a junior?”

Răspunsul nu e „because I'm passionate”. E o listă scurtă de dovezi:

> _"Three reasons. First, proof: I have four deployed projects, and I can walk you through any file in any of them. Second, I ramp up fast and I'm used to owning things under pressure; in a kitchen, nobody finishes your station for you. Third, I already work the way modern teams do: with AI tools, but with judgment. I review what they produce, test it, and I know where they tend to be wrong. Plus I bring business sense from years of dealing with costs, suppliers and customers."_

## Cum vorbești despre faptul că ai construit cu AI

Vor întreba. Uneori direct („How much of this code did AI write?”), alteori indirect, cerându-ți să explici un fișier. Două greșeli de evitat: să pretinzi că ai scris totul de mână, sau să spui „AI-ul a făcut-o, eu doar am cerut”.

Răspunsul onest, care e și cel mai bun:

> _"A lot of the code was generated with Claude Code. That's how I work, and how most teams work now. But my process was strict: I wrote the spec myself, reviewed and challenged the plan before any code was written, built feature by feature with small commits, and after every feature I walked through every file and explained it back until I really understood it. Some parts I wrote by hand on purpose, like [a concrete feature]. And I caught real problems in the generated code. For example, [a bug you actually found, like a missing ownership check or totals calculated on the client]. Ask me about any file. I'm happy to walk you through it."_

Ultima propoziție e importantă: invită-i să verifice. Funcționează doar dacă e adevărată, și e adevărată pentru că ai făcut code walk pe fiecare proiect.

## Prezentarea unui proiect în 2 minute

Pentru fiecare proiect, pregătește o prezentare cu structura asta:

| Timp | Ce spui | Exemplu pentru Invoicer |
|---|---|---|
| 20 s | **Problema** | „Freelancers track invoices in spreadsheets and lose track of who owes what.” |
| 20 s | **Ce face** (cu demo, dacă se poate) | „You manage clients, create invoices with lines and VAT, track status, and see what's overdue.” |
| 30 s | **Arhitectura și stack-ul** | „React SPA on Vercel, a NestJS REST API on Render, Postgres on Supabase through Prisma.” |
| 30 s | **O decizie grea sau un trade-off** | „I store money as integer cents and compute totals on the server, never trusting the client. I also create an invoice and its lines in one transaction.” |
| 20 s | **Ce ai face altfel sau mai departe** | „I'd add authentication from day one; I left it for the next project, and adding it later would mean migrating data.” |

**După prezentare vin întrebările de detaliu.** Pentru fiecare proiect, pregătește răspunsuri la:

- De ce ai ales [tehnologia X] și nu [Y]?
- Cum circulă un request, de la click până în baza de date și înapoi?
- Ce se întâmplă dacă [două rezervări simultane / API-ul extern pică / utilizatorul trimite date invalide]?
- Unde stau secretele și cum ajung în producție?
- Ce ai testat și de ce ai ales să testezi tocmai lucrurile alea?
- Care a fost cel mai greu bug și cum l-ai găsit?
- Ce ai schimba dacă ar avea 10.000 de utilizatori?

`ARCHITECTURE.md`-ul pe care l-ai scris în fiecare proiect e materialul tău de pregătire pentru întrebările astea.

## Poveștile pentru behavioral (STAR)

Pregătește **șase povești** pe care le poți adapta la aproape orice întrebare _„Tell me about a time…”_. Patru din bucătărie, două din drumul spre cod:

1. **Presiune:** o seară în care totul a mers prost și ai ținut serviciul în picioare.
2. **Conflict:** un dezacord cu un coleg sau un șef și cum l-ați rezolvat.
3. **Greșeală:** o greșeală a ta, cum ți-ai asumat-o și ce ai schimbat după.
4. **Ai condus sau ai învățat pe cineva:** un junior pe care l-ai format, un post pe care l-ai organizat.
5. **Ai învățat ceva repede:** din proiecte, o tehnologie nouă pe care ai livrat-o într-o săptămână.
6. **Feedback:** un review de la Bogdan cu care n-ai fost de acord sau care te-a schimbat.

**Structura STAR:** _Situation_ (contextul, scurt), _Task_ (ce trebuia să faci tu), _Action_ (ce ai făcut tu, cu „I”, nu „we”: partea cea mai lungă), _Result_ (cum s-a terminat, cu cifre dacă le ai, plus ce ai învățat).

Exemplu de *formă* (folosește o poveste reală de-a ta, nu pe asta):

> _**Situation:** "We had a [120-cover] private event on a Saturday, and two hours before service our main supplier told us half the fish order wasn't coming."_
> _**Task:** "I was responsible for the hot section, and the menu was already printed."_
> _**Action:** "I called two backup suppliers, found a substitute that worked with the same garnish, adjusted portions so the cost per plate stayed the same, briefed the team in five minutes, and told the front of house exactly what to say if guests asked."_
> _**Result:** "Service went out on time, no complaints, and afterwards we set up a second supplier for every critical product. The lesson I keep using: have a plan B before you need it, and communicate the change clearly to everyone affected."_

Observă cum lecția de la final se transferă direct în software: dependențe externe, plan de rezervă, comunicare.

## Întrebări de pus angajatorului

La final te întreabă aproape mereu „Do you have any questions for us?”. „No” e un răspuns slab. Alege 2–3:

- _"What would my first three months look like? What does success look like at the end of them?"_
- _"How does code review work in the team? Who would review my pull requests?"_
- _"How do you use AI tools in development, and are there any guidelines around them?"_
- _"How often do you deploy, and what happens when something breaks in production?"_
- _"Is there someone who'd act as a mentor or a go-to person for me?"_
- _"What's the biggest challenge the team is facing right now?"_

## Red flags

Semne că un loc nu e bun pentru un junior, oricât de bine sună oferta:

- **Ai fi singurul developer** sau n-ar avea cine să-ți facă review la cod.
- **Nu au code review, teste sau CI** și nu par să-și dorească.
- **Proba de lucru e lungă și neplătită.** Un take-home de 3–4 ore e normal. Trei zile de „probă” gratuită pe produsul lor nu sunt.
- **Nimeni nu-ți poate descrie o zi obișnuită** sau ce vei face concret.
- **Interviul a fost doar trivia**, fără nicio discuție despre cum lucrezi.
- **Presiune să accepți pe loc**, sau neclarități despre contract și plată.

Pentru detaliile despre contracte, SRL și taxe, vorbește cu Bogdan și cu un contabil. Nu sunt subiectul ghidului ăstuia.

## Despre salariu

- Când te întreabă de așteptări, dă un **interval** pe care îl poți susține, bazat pe ce vezi în anunțuri și pe cifrele din briefing. Partea de jos a intervalului trebuie să fie o sumă cu care chiar ai fi mulțumit.
- Dacă vrei să afli întâi bugetul lor: _"I'd like to understand the role better first. What's the budgeted range for this position?"_ Dacă insistă, dai intervalul tău.
- Prima slujbă plătită, chiar sub țintă, valorează mult: briefing-ul tău explică de ce.

## Mai departe

- [01 · Cum arată interviurile](01-cum-arata-interviurile.md)
- [07 · Mock interviews](07-mock-interviews.md) — repeți povestea și prezentările cu Claude
