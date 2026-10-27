# Mock interviews

Un mock interview face pentru interviuri ce face o repetiție generală pentru un eveniment: descoperi problemele când încă nu costă nimic. Claude poate fi un intervievator foarte bun dacă îi spui exact ce rol să joace. Mai jos ai prompturile gata de copiat.

Prompturile sunt în engleză, pentru că mock-ul trebuie să se desfășoare în engleză, ca interviul real.

## Ritmul

| Când | Ce | Cu cine |
|---|---|---|
| Din săpt. 3, o dată pe săptămână | Pe rând: tehnic pe un capitol → proiect → live coding → behavioral | Claude |
| Săpt. 4 ([CP3](../checkpoints/CP3.md)) | Tehnic + prezentarea P2, 45 min | Bogdan |
| Săpt. 7–8 | 2–3 mock-uri pe săptămână, toate tipurile | Claude |
| Săpt. 8 ([CP5](../checkpoints/CP5.md)) | Simulare completă: screening, tehnic, proiect, behavioral, 90 min | Bogdan |

## Regulile unui mock care chiar ajută

1. **Condiții reale:** timer pornit, fără notițe, fără să cauți răspunsuri în timpul mock-ului.
2. **Vorbit, nu scris**, cât se poate. Spune răspunsul cu voce tare, apoi scrie-l pe scurt pentru Claude. Pentru exerciții pur vorbite poți folosi modul vocal din aplicația Claude.
3. **Feedback-ul vine la final**, nu după fiecare răspuns. Așa simulezi presiunea unui interviu adevărat.
4. **Notezi în [jurnal.md](../jurnal.md)** scorul, ce ți-a ieșit prost și ce întrebări noi au apărut. Subiectele slabe intră în lista de repetat.

## 1. Mock tehnic pe un capitol

Rulează-l din rădăcina repo-ului de ghiduri, ca să poată citi fișele.

```text
You are a senior engineer interviewing me for a junior full-stack role (TypeScript, React,
NestJS, Postgres). The interview is in English and lasts about 30 minutes.

Topic: the chapter in [08-baze-de-date-supabase/]. Read its files first, plus the matching
questions in 13-interviuri/04-intrebari-backend-db.md, but don't limit yourself to those.

Rules:
- Ask one question at a time and wait for my answer.
- Ask follow-up questions on my answers, the way a real interviewer digs deeper.
- Mix theory with practical questions ("what would you do if...").
- Don't correct me or give answers during the interview.
- After 8–10 questions, stop and give me the feedback described below.

Feedback: for each question, a score from 1–5 and one line on what was missing. Then my
3 strengths, my 3 biggest gaps with the guide files I should re-read, and a model rewrite
of my weakest answer.
```

## 2. Mock pe proiectul tău

Rulează-l din rădăcina repo-ului proiectului, ca Claude să poată citi codul. E cel mai valoros mock: exact asta se întâmplă la interviurile în care îți prezinți portofoliul.

```text
You are a tech lead interviewing me about this project, which I built and am presenting
in a job interview. The interview is in English.

First, read the codebase: README.md, ARCHITECTURE.md, the main modules, the database
schema and the tests. Don't summarize it to me.

Then:
1. Ask me to present the project in 2 minutes. Wait.
2. Ask 10 questions, one at a time, the way a skeptical but fair tech lead would:
   why I made specific decisions (point to real files and lines), how a request flows
   end to end, what happens in failure cases, security (secrets, authorization, validation),
   what I tested and why, and what I'd change at 10x the users.
3. At least 2 questions must be about specific code: show me a snippet and ask me to
   explain it or spot a problem in it.
4. Ask follow-ups when my answer is vague.

At the end: score my presentation and each answer from 1–5, tell me which answers sounded
like I didn't really understand my own code, and list the files I should walk through again.
```

## 3. Mock de live coding cu timer

```text
You are interviewing me in a live coding round for a junior JavaScript/TypeScript role.
The interview is in English.

Give me one problem of [level 2] difficulty, similar to the katas in
13-interviuri/05-live-coding.md but not one of them. Practical web-style problems
(data transformation, async, a small utility), not hard algorithms.

Rules:
- Present the problem the way an interviewer would, slightly underspecified,
  so I have to ask clarifying questions.
- Answer my clarifying questions briefly.
- I'll solve it in my own editor without AI, with a 25-minute timer, and paste my code
  and my explanation when I'm done (or when time runs out).
- If I ask for a hint, give a small one, not the solution.

After I paste the solution: evaluate my clarifying questions, correctness, edge cases,
readability, complexity, and how I explained my approach. Then show me a clean reference
solution and the 2 most important differences from mine.
```

## 4. Mock behavioral

```text
You are a hiring manager interviewing me for a junior developer role. The interview
is in English. Context about me: Cybernetics degree, wrote C/C++ years ago, then about
ten years as a professional cook, now switching back to software with four deployed
full-stack projects built with a mentor and AI tools.

Ask me 6 behavioral questions, one at a time, including:
"Tell me about yourself", "Why the career switch?", one question about pressure,
one about a mistake, one about conflict or feedback, and one about how I use AI
in my work. Ask one follow-up per answer if it's vague or uses "we" instead of "I".

At the end: for each answer, tell me whether it followed STAR, whether it was specific
enough, and whether it was too long. Point out anything that sounded apologetic about
my background. Then rewrite my weakest answer, keeping my facts. Don't invent new ones.
```

## 5. Mock de system design ușor

```text
You are interviewing me in a system design round adapted for a junior full-stack
developer. The interview is in English and lasts about 30 minutes.

Ask me: "How would you build [a booking system for a sports club with courts,
members and admins]?"

Rules:
- Let me lead. Expect me to ask about requirements first; if I don't, note it.
- Answer my questions about requirements as a product manager would.
- Push on: the data model, the API endpoints, where validation and authorization happen,
  preventing double booking, and how and where it gets deployed.
- Don't expect distributed-systems answers. This is a junior-level round.

At the end: evaluate requirement gathering, structure, data model, security,
trade-off reasoning, and communication, each from 1–5, with concrete advice.
```

Schimbă sistemul din paranteză de la o săptămână la alta, de exemplu un shop online mic, o aplicație de rețete pentru o echipă sau un URL shortener.

## 6. Doar feedback, pentru un răspuns pe care vrei să-l șlefuiești

```text
Here is a question I was asked in an interview and the answer I gave, as close to word
for word as I remember. Evaluate it like a senior interviewer: correctness, structure,
clarity, length, and whether it sounds natural spoken out loud. Then give me an improved
version that keeps my facts and sounds like me, not like a textbook. Question: [...]
My answer: [...]
```

## Mock-urile cu Bogdan

- **La CP3:** 45 de minute. Prezinți P2 Invoicer, apoi primești întrebări tehnice pe capitolele 02, 07, 08 și 10. Bogdan joacă rolul unui tech lead dintr-o firmă care angajează juniori.
- **La CP5:** 90 de minute, simulare completă:
  - screening (10 min);
  - tehnic cu întrebări și un mic live coding (30 min);
  - prezentarea P4 Kitchen AI și întrebări pe cod (30 min);
  - behavioral (15 min);
  - întrebările tale pentru el (5 min).

  Feedback-ul îl treci în `checkpoints/CP5.md`.

Trimite-i lui Bogdan înainte linkurile la repo și la aplicația live, ca să poată pregăti întrebări pe codul tău.

## Mai departe

- [README-ul capitolului](README.md) — flashcards zilnice
- [06 · Povestea ta](06-povestea-ta.md) — materialul pentru mock-ul behavioral
- [05 · Live coding](05-live-coding.md) — metoda pe care o folosești în mock-ul 3
