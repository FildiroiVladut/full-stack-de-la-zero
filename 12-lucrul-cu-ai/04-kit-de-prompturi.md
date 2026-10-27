# Kit de prompturi

Prompturi gata de copiat pentru momentele care se repetă: când vrei să înțelegi, să verifici, să exersezi sau să depanezi. Înlocuiești ce e între `[...]`. Merg în română; pentru mock interviews treci pe engleză. Ține fișierul deschis într-un tab.

Ce au în comun prompturile bune: **un singur scop, contextul exact (`@fișier`) și ce vrei să primești înapoi.**

## Planificare

**Spec prin interviu**: când pornești un proiect sau un feature mare și nu știi încă toate cazurile.

```text
Vreau să construiesc [descriere scurtă]. Intervievează-mă în detaliu, o întrebare pe rând,
despre ce trebuie să facă, cazurile limită, ce NU intră și cum arată "gata".
Nu întreba lucruri evidente. Insistă pe părțile la care probabil nu m-am gândit.
La final scrie spec-ul complet în SPEC.md, dar deciziile le iau eu.
```

**Plan pe pași mici** (în plan mode):

```text
Pe baza @SPEC.md, propune un plan de implementare în pași mici; fiecare pas trebuie
să fie un commit care rulează și se poate testa. Pentru fiecare pas: fișierele atinse,
ce verificare rulăm, și o alternativă pe care n-ai ales-o și de ce.
```

## Înțelegere

**Explică un fișier**

```text
Explică-mi @[cale/fișier] pentru cineva care știe C și JavaScript de bază, dar e nou în [React/Nest/...].
Întâi rolul fișierului în aplicație (2-3 propoziții), apoi bucată cu bucată.
Nu modifica nimic.
```

**Explică diff-ul**

```text
Explică-mi diff-ul curent (git diff) fișier cu fișier: ce s-a schimbat și de ce era nevoie.
Marchează orice schimbare care nu ține direct de task-ul nostru.
```

**Urmărește un request cap-coadă**

```text
Urmărește ce se întâmplă când utilizatorul apasă [butonul X], de la componenta React
până la query-ul din baza de date și înapoi. Pentru fiecare pas: fișierul, funcția,
ce date intră și ce date ies. Desenează fluxul ca diagramă ASCII.
```

**Harta proiectului**

```text
Dă-mi harta proiectului ăstuia: folderele principale, ce responsabilitate are fiecare
și care e punctul de intrare. Maxim 15 rânduri. Apoi recomandă-mi ordinea în care să citesc fișierele.
```

**Un concept, pe codul meu**

```text
Explică-mi [conceptul, ex. dependency injection] folosind exemple din proiectul ăsta, nu exemple generice.
Arată-mi unde apare în cod și ce s-ar strica dacă n-ar exista.
```

## Verificare

**Ascultă-mi explicația** (pasul de code walk):

```text
Îți explic eu cum funcționează [feature/fișier]. Nu mă întrerupe.
După ce termin: 1) ce am spus greșit, 2) ce am omis și e important, 3) trei întrebări de follow-up
cum ar pune un intervievator. Nu-mi da răspunsurile la întrebări până nu încerc.
```

**Pune-mi întrebări**

```text
Pune-mi 5 întrebări despre codul din @[cale], de la ușor la greu, una câte una.
Așteaptă răspunsul meu la fiecare, apoi spune-mi dacă e corect și ce lipsea.
```

**Quiz de checkpoint**: folosește promptul din [checkpoints/README.md](../checkpoints/README.md). Îl rulezi în repo-ul de ghiduri, unde `CLAUDE.md` îi spune deja lui Claude să fie examinator.

## Exersare

**Break & fix** (într-o sesiune nouă, după `/clear`):

```text
Introdu un singur bug subtil în [zona/feature-ul], fără să-mi spui unde și fără comentarii.
Fă commit pe un branch nou drill/[n]. Când scriu "indiciu", dă-mi câte un indiciu, de la vag la precis.
Când cred că l-am găsit, îți explic cauza și tu confirmi sau infirmi.
```

**Kata fără AI, cu corectură după**

```text
Dă-mi un exercițiu de 20 de minute despre [array methods / async / o componentă React / un endpoint],
nivel interviu junior. Doar enunțul și 3 exemple de input/output. Nu scrie soluția.
```

Apoi, după ce l-ai rezolvat singur:

```text
Uite soluția mea: [lipește codul]. Evaluează-o ca la un interviu: corectitudine, cazuri limită,
lizibilitate. Ce ar fi întrebat un intervievator în plus?
```

## Review

**Review ca un senior la PR-ul meu**

```text
Fă-mi review la diff-ul de pe branch-ul curent față de main, ca un senior exigent dar corect.
Pentru fiecare problemă: fișierul și linia, de ce e o problemă, și ce ai propune.
Împarte în: trebuie reparat / ar fi bine / preferință. Nu modifica nimic.
```

**Doar securitate**

```text
Uită-te la @[cale] doar din perspectiva securității: validare pe server, autorizare
(poate un user să acceseze datele altuia?), secrete, ce ajunge în browser, injection.
Nu modifica nimic, doar listează.
```

**Mai simplu**

```text
Care e varianta cea mai simplă a codului din @[cale] care face exact același lucru?
Arată-mi ce ai scoate și de ce. Nu modifica până nu-ți spun.
```

## Depanare

**Explică eroarea, nu o repara**

```text
Am eroarea asta: [lipește eroarea + stack trace-ul relevant].
Explică-mi ce înseamnă, de unde vine (fișier și linie) și care sunt 2-3 cauze posibile.
Nu repara nimic. Vreau să găsesc eu cauza.
```

**Ipoteze, nu fix-uri**

```text
[Descrie simptomul]: ce mă așteptam, ce s-a întâmplat, ce am încercat deja.
Dă-mi ipoteze ordonate după probabilitate și, pentru fiecare, cum o verific rapid.
```

**Întâi testul care pică**

```text
Scrie un test care reproduce bug-ul ăsta: [descriere]. Rulează-l și arată-mi că pică.
Abia apoi propune fix-ul, și rulează testul din nou.
```

## Decizii

```text
Compară două abordări pentru [problema]: [A] și [B]. Pentru fiecare: avantaje, dezavantaje,
când aș alege-o, și ce ar alege o echipă tipică în 2026. Fără cod. Recomandă una și spune de ce.
```

## Interviu

**Mock interview** (în engleză, pentru că interviurile cu clienți străini sunt în engleză):

```text
Act as an interviewer for a junior full-stack developer role (TypeScript, React, Node/NestJS, Postgres).
Ask me one question at a time about [topic or my project: link/README]. Wait for my answer.
After each answer: score it 1-5, say what a strong answer would add, then ask a follow-up.
After 6 questions, give me an overall summary and the 3 topics to review.
```

**Povestea proiectului**

```text
Ascultă-mă cum prezint proiectul [P2 Invoicer] în 2 minute, ca la un interviu.
Apoi spune-mi ce a sunat vag, ce a lipsit (decizii tehnice, dificultăți, ce aș face altfel)
și pune-mi întrebările de follow-up pe care le-ar pune un tech lead.
```

Mai multe, plus cum organizezi mock interviews, în [13-interviuri/07-mock-interviews.md](../13-interviuri/07-mock-interviews.md).

## Mai departe

- [Best practices: provide specific context — Claude Code docs](https://code.claude.com/docs/en/best-practices)
