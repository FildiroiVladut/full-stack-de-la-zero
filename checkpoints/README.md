# Checkpoints

Cinci opriri în care verifici că ai înțeles cu adevărat, înainte să mergi mai departe. Un checkpoint nu e un examen pe care îl treci sau îl pici. E momentul în care găsești golurile cât sunt mici.

| | Când | Ce acoperă |
|---|---|---|
| [CP1](CP1.md) | Finalul săpt. 1 | Setup, web, ecosistem, JavaScript async |
| [CP2](CP2.md) | Finalul săpt. 2 | TypeScript, React, P1 Shelf |
| [CP3](CP3.md) | Finalul săpt. 4 | Backend, baze de date, securitate, P2 Invoicer, primele aplicări |
| [CP4](CP4.md) | Finalul săpt. 6 | Auth, teste, Git în echipă, P3 Booking |
| [CP5](CP5.md) | Finalul săpt. 8 | Next.js, AI, P4 Kitchen AI, pregătirea pentru interviu |

## Cele 4 părți ale fiecărui checkpoint

1. **Poți explica fără notițe.** O listă de lucruri. Bifezi un rând doar după ce l-ai explicat cu voce tare, lui Claude sau cuiva din casă.
2. **Quiz cu Claude.** 10 întrebări, cu promptul de mai jos. Ținta e 8/10. Ce ai greșit intră în jurnal, iar fișele respective le recitești.
3. **Livrabile.** Linkuri spre ce ai construit.
4. **Call cu Bogdan.** 30–45 de minute: el te întreabă, tu explici, apoi ajustați planul.

## Promptul pentru quiz

Deschide Claude Code în repo-ul ăsta (știe ce are de făcut din [CLAUDE.md](../CLAUDE.md)) și scrie:

```
Fă-mi quiz-ul pentru CP1. Citește checkpoints/CP1.md și capitolele pe care le acoperă.
Pune-mi 10 întrebări, una câte una, de la ușor la greu, ca la un interviu de junior:
6 de explicat, 2 de tipul „ce afișează codul ăsta” și 2 de tipul „găsește bug-ul”.
Nu-mi da răspunsul până nu răspund eu. Dacă răspunsul meu e incomplet, pune-mi o întrebare ajutătoare.
La final: scorul, ce am răspuns slab și ce fișe să recitesc.
```

Schimbi `CP1` cu checkpoint-ul la care ești.

## Fluxul prin PR

Checkpoint-urile le bifezi prin pull request, la fel cum ai lucra într-o echipă:

```bash
git switch -c cp1
# completezi checkpoints/CP1.md: bifezi, pui linkurile, scrii scorul de la quiz
# bifezi capitolele terminate în README.md
git add README.md checkpoints/CP1.md jurnal.md
git commit -m "checkpoint: CP1"
git push -u origin cp1
gh pr create --title "Checkpoint 1" --body "Scor quiz: 8/10. Neclar încă: ..." --reviewer itsbogdann
```

Bogdan lasă comentarii pe ce nu e clar, faceți call-ul, apoi dai merge. Pașii explicați sunt în [09-git-github-echipa/03-pull-requests.md](../09-git-github-echipa/03-pull-requests.md).

**Dacă la quiz ai sub 8/10:** recitești fișele indicate și refaci quiz-ul a doua zi. Nu treci mai departe cu goluri mari, dar nici nu stai pe loc mai mult de două zile. Restul se discută la call.
