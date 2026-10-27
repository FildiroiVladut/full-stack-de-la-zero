# 09 · Git și GitHub în echipă

În `js-de-la-zero` ai lucrat singur, direct pe `main`: `add`, `commit`, `push`. La un job nu mai merge așa. Lucrezi pe branch-uri, deschizi pull request-uri, primești review de la colegi, iar un robot (CI) verifică automat codul înainte să ajungă pe `main`. Capitolul ăsta îți dă modelul mental al lui Git și fluxul exact pe care îl folosesc echipele. Te ajută din prima săptămână, pentru că și checkpoint-urile le bifezi prin PR.

## La final poți să

- explici fără bâlbâieli ce sunt un commit, un branch și HEAD;
- duci un feature de la branch la PR deschis din terminal cu `gh`, apoi până la merge;
- rezolvi un conflict de merge calm, înțelegând ce voia fiecare parte;
- faci review la PR-ul altcuiva și primești review fără să-l iei personal;
- configurezi un CI cu GitHub Actions care blochează merge-ul când pică testele;
- repari aproape orice „am stricat ceva în git”.

## Fișierele, în ordine

- [ ] [01 · Cum gândește Git](01-cum-gandeste-git.md) — 40 min · săptămâna 1
- [ ] [02 · Branch-uri, merge și rebase](02-branch-uri-merge-rebase.md) — 45 min
- [ ] [03 · Pull requests](03-pull-requests.md) — 45 min · citește-l devreme, îți trebuie la CP1
- [ ] [04 · Code review](04-code-review.md) — 30 min
- [ ] [05 · Convenții și CI](05-conventii-si-ci.md) — 45 min
- [ ] [06 · Când o dai în bară](06-cand-o-dai-in-bara.md) — 40 min

Calendarul pune 01 în săptămâna 1, iar restul în săptămânile 5–6, odată cu P3. Fișierul 03 îl poți citi în diagonală încă din prima săptămână. Fluxul de PR îl aplici de la [CP1](../checkpoints/CP1.md).

## Unde îl folosești

- **Checkpoint-urile:** fiecare CP e un PR în repo-ul ăsta, cu review de la Bogdan ([checkpoints](../checkpoints/README.md)).
- **P1 și P2:** commit-uri mici, câte un branch pentru fiecare feature.
- **P3:** fluxul complet, adică issue → branch → PR → CI → review → merge, cu `main` protejat. [CP4](../checkpoints/CP4.md) cere exact asta.
- **Interviuri:** „Descrie-mi cum lucrați cu Git în echipă” e o întrebare aproape sigură. Vezi și [întrebările de backend și tooling](../13-interviuri/04-intrebari-backend-db.md).

## Verifică-te

1. Ce conține de fapt un commit? Dar un branch?
2. Prin ce diferă `git merge` de `git rebase` și când NU ai voie să faci rebase?
3. Ce se întâmplă, pas cu pas, între „am terminat feature-ul” și „e pe main”?
4. Ai făcut commit cu un `.env` și ai dat push. Ce faci și în ce ordine?
5. De ce primește un PR de 40 de linii un review mai bun decât unul de 1.500?
