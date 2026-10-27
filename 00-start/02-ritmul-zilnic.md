# Ritmul zilnic

5 ore pe zi: jumătate dimineața, jumătate seara. 6 zile pe săptămână, a șaptea liberă. Știi din bucătărie ce se întâmplă când lucrezi fără zi liberă.

## Dimineața: lucrurile noi (2,5 h)

Mintea e proaspătă, așa că acum intră conceptele noi și partea grea din proiect.

| Cât | Ce |
|---|---|
| 15 min | Încălzire: 5 întrebări de interviu cu Claude, din [13-interviuri](../13-interviuri/) (din săptămâna 2) |
| 45–60 min | Fișele zilei: citești, rulezi exemplele, spui cu voce tare răspunsul de interviu |
| 75–90 min | Proiect: partea nouă (plan, feature nou) |

## Seara: consolidarea (2,5 h)

| Cât | Ce |
|---|---|
| 90–120 min | Proiect: continuare, apoi code walk pe ce s-a construit azi |
| 20 min | Kata fără AI (de 3 ori pe săptămână) sau break & fix |
| 10 min | Commit și push, trei rânduri în [jurnal.md](../jurnal.md), bifezi progresul |

În săptămâna 1 nu e proiect: tot timpul merge pe setup, fișe și experimente mici în `playground/`.

## Claude Pro și limitele lui

Abonamentul Pro are limite de utilizare care se resetează după câteva ore, plus o limită săptămânală. Detaliile și cum economisești sunt în [01-setup/05-claude-code.md](../01-setup/05-claude-code.md). Ce contează pentru ritm:

- **Dimineața și seara cad natural în ferestre diferite**, deci ritmul tău se potrivește bine cu limitele.
- **Dacă dai de limită, nu ai pierdut timp.** E momentul pentru code walk, pentru `ARCHITECTURE.md` sau pentru fișe, adică lucrurile care se fac oricum fără AI.
- **Un task pe sesiune.** Când treci la altceva, `/clear`. Contextul mic consumă mai puțin și dă rezultate mai bune.

## Jurnalul

Trei rânduri pe zi în [jurnal.md](../jurnal.md):

```
## 2026-11-02 · Ziua 1
- Am făcut: ...
- Am înțeles: ...
- Nu e clar încă: ...
```

Rândul „Nu e clar încă” e cel mai important. Bogdan citește jurnalul înainte de fiecare call și pleacă de la el.

## Săptămâna

- **În ziua dinaintea celei libere, 15 minute:** recitești jurnalul săptămânii și alegi 3 lucruri neclare pentru Claude sau pentru Bogdan.
- **La fiecare checkpoint (săptămânile 1, 2, 4, 6, 8): call cu Bogdan, 30–45 de minute.** Pregătești linkul spre PR-ul de checkpoint, jurnalul și 3 întrebări.
- **Din săptămâna 4 încep aplicările:** 5–10 pe săptămână, cam 3 ore, luate din timpul de proiect. Le notezi în tabelul din [jurnal.md](../jurnal.md).

## Când ritmul nu iese

Unele zile o să iasă mai scurte, unele capitole o să ia mai mult. E normal. Două reguli:

1. **Nu tai din code walk ca să recuperezi.** Mai bine întârzii o zi cu proiectul.
2. **Dacă ai rămas în urmă cu peste 3 zile față de calendar, spune-i lui Bogdan.** Ajustați planul împreună, nu-l înghiți singur.
