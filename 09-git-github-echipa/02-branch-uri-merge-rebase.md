# Branch-uri, merge și rebase

Pe branch-uri lucrezi izolat, iar prin merge sau rebase aduci munca înapoi împreună. Aici e și locul unde apar conflictele. Le rezolvi ușor dacă știi ce se întâmplă de fapt.

## De ce lucrezi pe branch-uri

**Ce e.** `main` trebuie să fie mereu într-o stare bună: compilează, testele trec și se poate face deploy oricând. Fiecare schimbare se face pe un branch separat și ajunge pe `main` abia după ce e verificată.

```
main          A ─── B ───────────── M      (always deployable)
                     ╲             ╱
feat/search           C ─── D ─── E        (work in progress, isolated)
```

**De ce contează.** Dacă feature-ul tău e pe jumătate stricat, nu blochezi pe nimeni. Dacă renunți la el, ștergi branch-ul și gata.

**Exemplu minim**

```bash
git switch main
git pull                          # start from the latest main
git switch -c feat/book-search    # new branch from here
```

**Capcana.** Un branch pornit dintr-un `main` vechi de o săptămână. Până termini, `main` a mers mult înainte și te așteaptă conflicte mari. Fă mereu `git pull` pe `main` înainte de `switch -c`.

## Merge: fast-forward sau merge commit

**Ce e.** `git merge` aduce commit-urile altui branch în branch-ul curent. Se poate întâmpla în două feluri.

**1. Fast-forward.** Dacă `main` nu s-a mișcat de când ai pornit branch-ul, Git doar mută eticheta `main` mai în față și nu creează niciun commit nou:

```
înainte:  main → B         după:   main → D
                  ╲                 A ─ B ─ C ─ D
                   C ─ D
```

**2. Merge commit.** Dacă `main` a primit între timp alte commit-uri, Git creează un commit nou, cu doi părinți, care unește cele două istorii:

```
main   A ─── B ─── X ─────── M
              ╲             ╱
feat           C ─── D ────
```

**Exemplu minim**

```bash
git switch main
git merge feat/book-search
```

**Cum îl vezi în cod real.** În echipe aproape nu dai merge local pe `main`. Merge-ul îl faci din PR, pe GitHub, unde ai trei butoane: *Merge commit*, *Squash and merge* și *Rebase and merge*. Cele mai multe echipe aleg **squash**: tot branch-ul devine un singur commit curat pe `main`. Detalii în [03](03-pull-requests.md).

**Capcana.** Să faci merge din `main` în branch-ul tău de zece ori pe zi „ca să fie la zi”. Istoria se umple de commit-uri „Merge branch main”. O dată pe zi ajunge, sau când chiar ai nevoie de ceva nou de pe `main`.

## Rebase: îți rescrii commit-urile peste main-ul nou

**Ce e.** `git rebase` ia commit-urile tale și le reaplică, unul câte unul, peste ultima versiune a altui branch. Rezultatul e o istorie liniară, ca și cum ai fi pornit lucrul abia azi.

```
înainte:                         după git rebase origin/main:
main   A ─ B ─ X ─ Y             main   A ─ B ─ X ─ Y
            ╲                                        ╲
feat         C ─ D               feat                 C' ─ D'
```

`C'` și `D'` sunt commit-uri **noi**, cu alt hash, chiar dacă au același conținut.

**Exemplu minim**

```bash
git switch feat/book-search
git fetch origin
git rebase origin/main
# resolve conflicts if any, then:
git push --force-with-lease       # history changed, so a normal push is refused
```

**De ce contează.** Branch-ul tău rămâne curat și la zi, iar reviewer-ul vede doar commit-urile tale, fără zgomot.

**Capcana, cea mai importantă regulă din fișierul ăsta: nu rescrii istoria publică.** Nu faci rebase pe commit-uri pe care le-au luat deja alții: `main`, `develop`, un branch comun. Rebase-ul schimbă hash-urile, iar colegii rămân cu o istorie care nu se mai potrivește cu a ta. Pe branch-ul tău, înainte de review, rebase-ul e în regulă.

<details>
<summary>Întrebare de interviu: Ce diferență e între merge și rebase?</summary>

Merge-ul păstrează istoria exact cum s-a întâmplat. Unește două linii printr-un merge commit care are doi părinți. Rebase-ul reaplică commit-urile mele peste vârful altui branch și creează commit-uri noi, cu altă identitate, iar istoria devine liniară. Fac rebase doar pe branch-urile mele, care n-au fost preluate de alții, ca să le țin la zi înainte de PR. Pe branch-urile comune folosesc merge, pentru că rebase-ul acolo rescrie istoria pe care o au și colegii.
</details>

## Conflicte: ce sunt și cum le rezolvi

**Ce e.** Un conflict apare când două branch-uri au modificat aceleași linii din același fișier și Git nu poate ghici care variantă e corectă. Te întreabă pe tine și marchează locul în fișier:

```ts
<<<<<<< HEAD
const VAT_RATE = 0.21;
=======
const VAT_RATE = getVatRate(invoice.country);
>>>>>>> feat/vat-per-country
```

Între `<<<<<<<` și `=======` e varianta de pe branch-ul curent. Între `=======` și `>>>>>>>` e varianta care vine.

**Pașii**

1. `git status` îți arată fișierele în conflict (*both modified*).
2. Deschizi fiecare fișier și **înțelegi ce voia fiecare parte**. Uneori răspunsul corect e un amestec al celor două.
3. Ștergi markerii și lași codul final.
4. Rulezi aplicația sau testele.
5. `git add fisier.ts`, apoi:
   - `git commit` dacă erai în mijlocul unui merge;
   - `git rebase --continue` dacă erai în mijlocul unui rebase.

Dacă te-ai încurcat, oprești totul și o iei de la capăt:

```bash
git merge --abort      # or
git rebase --abort
```

**În VS Code.** Deasupra fiecărui conflict apar butoanele *Accept Current Change*, *Accept Incoming Change*, *Accept Both Changes* și *Compare Changes*. Pentru conflicte mai mari, butonul *Resolve in Merge Editor* îți arată cele două variante una lângă alta, iar rezultatul dedesubt.

**Cu Claude Code.** Un prompt bun: _„Am un conflict în `invoices.service.ts`. Explică-mi ce voia să facă fiecare parte, fără să rezolvi încă nimic.”_ Tu decizi, apoi îi ceri să aplice decizia.

**Capcana.**

- „Accept Incoming” peste tot, ca să scapi mai repede. Așa ștergi în tăcere munca unui coleg.
- Markeri `<<<<<<<` uitați în cod. TypeScript îi prinde, dar un fișier `.md` sau `.json` uneori nu.
- Unele conflicte sunt de fapt întrebări de business („TVA fix sau pe țară?”). Atunci îl întrebi pe colegul care a scris cealaltă variantă.

<details>
<summary>Întrebare de interviu: Cum rezolvi un conflict de merge?</summary>

Mă uit cu `git status` care fișiere sunt în conflict și citesc ambele variante ca să înțeleg intenția fiecăreia. Nu aleg orbește una dintre ele. Scriu codul final, șterg markerii, rulez testele, apoi `git add` și continui merge-ul sau rebase-ul. Dacă nu e clar ce voia celălalt, îl întreb înainte să decid.
</details>

## Branch-uri scurte: cel mai bun mod de a evita conflictele

**Ce e.** Un branch care trăiește o zi sau două are puține conflicte. Unul care trăiește trei săptămâni are conflicte garantat.

**De ce contează.** Majoritatea echipelor moderne lucrează cu branch-uri scurte și PR-uri mici. Un feature mare se sparge în bucăți care pot ajunge pe `main` separat, chiar dacă la început nu se văd încă în UI.

**Cum îl vezi în cod real.** În loc de un singur PR „P3: tot sistemul de rezervări”, ai mai multe PR-uri mici:

1. „Add bookings table and migration”
2. „Add POST /bookings with overlap check”
3. „Add booking form”
4. „Add my-bookings page”

**Capcana.** Să lucrezi o săptămână fără push „pentru că nu e gata”. Fă push des pe branch-ul tău. E și backup, și locul unde CI-ul îți dă feedback devreme. Un PR care nu e gata îl deschizi ca draft ([03](03-pull-requests.md)).

## Mai departe

- [Pro Git — Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell): capitolul 3 explică branch-urile, merge-ul și rebase-ul cu desene
- [Resolving a merge conflict on GitHub](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-on-github)
