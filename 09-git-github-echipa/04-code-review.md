# Code review

Code review înseamnă că un coleg citește schimbările tale înainte să ajungă pe `main`. Nu e un examen. E felul în care o echipă își împarte cunoștințele și prinde problemele cât sunt încă ieftin de reparat. De multe ori și tu înveți mai mult din review-urile primite decât din orice curs.

## De ce există code review

**Ce e.** Un al doilea om citește fiecare schimbare înainte de merge.

**De ce contează.** Review-ul face trei lucruri:

- **prinde bug-uri** pe care autorul nu le mai vede, pentru că știe ce *voia* să scrie;
- **răspândește cunoașterea**: cel puțin doi oameni înțeleg fiecare bucată de cod;
- **păstrează consistența**, adică aceleași tipare și aceleași convenții în tot proiectul.

**Capcana.** Să tratezi review-ul ca pe o poartă de trecut, nu ca pe o conversație. Echipele bune discută pe PR. Nu doar „aprobat” sau „respins”.

## Ce caută un reviewer, în ordinea importanței

**Ce e.** Lista mentală a unui senior care deschide un PR.

1. **Corectitudine.** Face ce scrie în issue? Ce se întâmplă cu lista goală, cu `null`, cu o cantitate de 0, cu un utilizator care nu e logat?
2. **Securitate.** Endpoint-ul verifică al cui e obiectul ([IDOR](../10-securitate-env/05-autentificare-vs-autorizare.md))? Input-ul e validat pe server? A scăpat vreun secret în cod?
3. **Design.** E codul în locul potrivit, de exemplu logica în service și nu în controller? Se repetă ceva ce există deja?
4. **Lizibilitate.** Spun numele ce fac lucrurile? E vreo funcție de 80 de linii care ar trebui spartă?
5. **Teste.** Logica nouă e acoperită? Testele verifică ceva real sau doar trec?
6. **Stil.** Aproape niciodată nu-l comentează un om, pentru că Prettier și linter-ul (oxlint sau ESLint) îl rezolvă automat în CI ([05](05-conventii-si-ci.md)).

**Capcana.** Să comentezi spații și virgule și să ratezi faptul că oricine poate șterge factura oricui. Începi de sus în listă.

<details>
<summary>Întrebare de interviu: La ce te uiți prima dată când faci code review?</summary>

Mai întâi dacă schimbarea face ce trebuie, inclusiv pe cazurile limită, și dacă e sigură: validare pe server, verificări de autorizare, niciun secret în cod. Apoi la design, adică dacă logica e în stratul potrivit și dacă nu duplică ceva existent. Apoi la lizibilitate și teste. Stilul îl las pe seama linter-ului și a formatter-ului din CI.
</details>

## Review la propriul PR, înainte să-l ceri altcuiva

**Ce e.** Îți deschizi PR-ul pe GitHub, în tab-ul *Files changed*, și îl citești ca și cum l-ar fi scris un străin.

**De ce contează.** Așa prinzi singur jumătate din comentarii: un `console.log` uitat, cod comentat, un fișier modificat din greșeală, un TODO. E cel mai ieftin respect pe care îl poți arăta reviewer-ului. La codul generat de AI e obligatoriu, pentru că nu l-ai scris tu rând cu rând.

**Lista ta înainte de „Ready for review”:**

- [ ] am citit tot diff-ul în *Files changed*;
- [ ] pot explica fiecare linie (regula ghidului);
- [ ] fără `console.log`, cod comentat sau fișiere care nu au legătură cu PR-ul;
- [ ] CI-ul e verde;
- [ ] descrierea spune ce, de ce și cum se testează.

**Cu Claude Code.** Înainte să ceri review unui om, cere-i lui Claude un pre-review:

> _„Fă review la diff-ul ăsta ca un senior exigent: `git diff main...HEAD`. Caută bug-uri, probleme de securitate și cod inutil. Nu modifica nimic, doar listează problemele în ordinea gravității.”_

Lista completă de verificări pentru cod generat e în [12-lucrul-cu-ai/02-review-la-cod-generat.md](../12-lucrul-cu-ai/02-review-la-cod-generat.md).

## Cum dai review

**Ce e.** Comentezi pe linii concrete, cu intenția clară. La final alegi una din trei variante:

- **Comment**: doar observații, fără verdict;
- **Approve**: poate intra pe `main`;
- **Request changes**: nu intră până nu se rezolvă ceva important.

**Prefixe utile**, folosite de multe echipe:

- `question:` „De ce ai ales `findFirst` aici?”
- `nit:` un detaliu minor, opțional („nit: `items` ar fi mai clar decât `arr`”)
- `suggestion:` o variantă mai bună, dar nu obligatorie
- `blocking:` trebuie rezolvat înainte de merge

**Suggestion pe GitHub.** În comentariu scrii un bloc `suggestion`, iar autorul îl aplică cu un click:

````markdown
```suggestion
const subtotalCents = lines.reduce((sum, line) => sum + line.quantity * line.unitPriceCents, 0);
```
````

**Din terminal**

```bash
gh pr checkout 14                 # get the PR locally and run it
gh pr diff 14                     # read the diff
gh pr review 14 --approve
gh pr review 14 --request-changes --body "blocking: GET /invoices/:id does not check ownership"
```

**Capcana.** Comentarii despre om, nu despre cod („iar ai uitat…”), sau comentarii vagi („nu-mi place”). Un comentariu bun e specific, spune de ce și, ideal, propune o variantă.

## Cum primești review

**Ce e.** Răspunzi la fiecare comentariu: fie faci schimbarea, fie explici de ce ai lăsat așa. Apoi ceri din nou review (*Re-request review*).

**Bune practici**

- Nu-l lua personal. Comentariul e despre cod, nu despre tine. E ca la pass, când chef-ul gustă sosul înainte să plece farfuria: nu te judecă pe tine, judecă farfuria.
- Fixurile le pui în **commit-uri noi** pe același branch. Reviewer-ul vede exact ce s-a schimbat de la ultima lui trecere. La merge, squash-ul le adună oricum într-un singur commit.
- Dacă nu ești de acord, explici o dată, calm, cu argumente. Dacă discuția se lungește, un call de 5 minute rezolvă mai mult decât 20 de comentarii.
- Comentariul care te-a învățat ceva îl treci în [jurnal](../jurnal.md). Cele mai bune ajung în [senior-tips.md](../senior-tips.md).

**Capcana.** Să marchezi conversațiile ca *Resolved* fără să răspunzi sau fără să schimbi ceva. Reviewer-ul o să creadă că i-ai ignorat comentariul.

<details>
<summary>Întrebare de interviu: Cum reacționezi când un senior îți lasă 20 de comentarii pe un PR?</summary>

Îl iau ca pe o lecție gratuită, nu ca pe o critică. Le iau pe rând, le rezolv pe cele clare, iar unde nu înțeleg sau nu sunt de acord pun o întrebare concretă. Fixurile le fac în commit-uri separate, ca să-i fie ușor să verifice, apoi cer din nou review. Dacă văd că se repetă același tip de comentariu, îl notez ca să nu mai apară la PR-ul următor.
</details>

## Review-urile cu Bogdan

PR-urile tale, mai întâi cele de checkpoint și apoi cele din P3, le verifică Bogdan, la fel cum ar face un senior din echipa ta. Ca să scoți maximum din ele:

- **Ține PR-urile mici.** Un review bun pe 200 de linii valorează mai mult decât unul superficial pe 2.000.
- **Pune întrebări direct în descriere:** „Nu sunt sigur dacă verificarea de suprapunere trebuie în service sau în DB. Ce zici?”
- **După fiecare review, notează** în jurnal un lucru pe care nu-l mai repeți.
- **La CP4** trebuie să ai cel puțin câteva PR-uri cu review real în P3 ([checkpoints/CP4.md](../checkpoints/CP4.md)).

## Mai departe

- [About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews): GitHub Docs
- [Google Engineering Practices — Code Review](https://google.github.io/eng-practices/review/): cum fac review echipele de la Google, scurt și foarte bun
