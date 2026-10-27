# Cum gândește Git

Git pare o colecție de comenzi magice până înțelegi ideile de mai jos. După aceea aproape orice comandă devine logică, iar mesajele de eroare încep să aibă sens.

## Commit = snapshot

**Ce e.** Un commit e o fotografie a întregului proiect la un moment dat, plus metadate: autor, dată, mesaj și o referință la commit-ul de dinainte (părintele). Fiecare commit are un ID unic, un hash, din care vezi de obicei doar primele caractere, de exemplu `a7c481d`.

**De ce contează.** Istoria e un lanț de snapshot-uri, deci te poți întoarce oricând la orice punct și poți compara oricare două stări. Diff-ul („ce s-a schimbat”) Git îl calculează abia când i-l ceri.

**Exemplu minim**

```bash
git log --oneline
# 11d5927 Ziua 7: clase si erori, clasa Inventar
# 7338622 Ziua 6: closures si this
# a7c481d Ziua 5: bucle si metode de lista

git show 7338622 --stat   # which files changed in that commit
```

**Cum îl vezi în cod real.** Rulează `git log --oneline` în repo-ul unei echipe și o să vezi ceva de genul:

```
3f9c2e1 feat(invoices): compute totals on the server
b71d0aa fix(invoices): round VAT per line, not per invoice
9e04c55 test(invoices): cover zero-quantity lines
```

- Fiecare commit face un singur lucru.
- Mesajul spune ce s-a schimbat. Corpul mesajului, când există, spune și de ce.
- Formatul `tip(zonă): descriere` se numește conventional commits și îl găsești în [05](05-conventii-si-ci.md).

**Capcana.** Un singur commit uriaș pe zi, cu mesajul „update” sau „fix”. Nu poți anula jumătate de commit, iar cine citește istoria nu înțelege nimic din el. Regula e simplă: un commit e o schimbare pe care o poți descrie într-o propoziție.

<details>
<summary>Întrebare de interviu: Ce e un commit în Git?</summary>

Un snapshot al proiectului, identificat printr-un hash, cu autor, dată, mesaj și o referință la commit-ul părinte. Istoria e un graf de astfel de snapshot-uri. De aceea poți reveni oricând la o stare anterioară, poți compara două stări și poți lucra în paralel pe mai multe linii de dezvoltare.
</details>

## Cele trei zone: working directory, staging, repository

**Ce e.** Git lucrează cu trei zone:

- **working directory**: fișierele de pe disc, așa cum le vezi în editor;
- **staging area** (sau index): lista cu ce intră în următorul commit;
- **repository**: istoria salvată, adică folderul ascuns `.git`.

```
 working directory ──git add──▶ staging ──git commit──▶ repository
        ▲                                                    │
        └──────────────── git restore / git switch ──────────┘
```

**De ce contează.** Staging-ul te lasă să pui în commit doar o parte din modificări. Să zicem că ai reparat un bug și, în trecere, ai redenumit o variabilă în alt fișier. Le poți pune în două commit-uri separate, fiecare cu mesajul lui.

**Exemplu minim**

```bash
git status                  # what changed, what is staged
git add src/invoices.ts     # stage one file
git commit -m "fix(invoices): handle empty line list"
```

**Cum îl vezi în cod real.** Claude Code face commit cu `git add` pe fișiere anume, nu cu `git add .`. După commit verifică ce a intrat:

```bash
git show --stat HEAD        # files in the last commit
```

**Capcana.** Un `git add .` dat orbește trimite în istorie `.env`, fișiere de debug sau tot `node_modules` dacă lipsește `.gitignore`. Înainte de orice commit, uită-te la `git status`.

<details>
<summary>Întrebare de interviu: Ce e staging area și la ce folosește?</summary>

E zona intermediară dintre fișierele modificate și istorie, unde alegi exact ce intră în următorul commit. Așa poți împărți modificările în commit-uri mici și coerente, chiar dacă le-ai făcut pe toate odată, și poți verifica cu `git diff --staged` ce urmează să salvezi.
</details>

## Branch = pointer, HEAD = unde ești acum

**Ce e.** Un branch e doar o etichetă care arată spre un commit. Când faci un commit nou pe branch, eticheta se mută pe el. `HEAD` înseamnă „unde ești acum” și de obicei arată spre branch-ul curent.

```
              main
               │
  A ─── B ─── C
               ╲
                D ─── E
                      │
            feat/invoice-totals ◀── HEAD
```

**De ce contează.** Un branch nou nu costă nimic: e un fișier mic care conține un hash. De aceea echipele fac câte un branch pentru fiecare feature, oricât de mic.

**Exemplu minim**

```bash
git branch                        # list branches, * marks the current one
git switch -c feat/book-search    # create a branch and move HEAD onto it
git log --oneline --graph --all   # all branches, drawn as a graph
```

**Cum îl vezi în cod real.** În VS Code, numele branch-ului curent apare în colțul din stânga jos. Dacă dai click pe el, schimbi branch-ul. Un dev senior se uită acolo înainte de fiecare commit, ca să nu ajungă pe `main` din greșeală.

**Capcana.** „Mi-am pierdut commit-urile!” De obicei nu le-ai pierdut, doar ești pe alt branch sau în „detached HEAD”. `git log --all` și `git reflog` le găsesc aproape mereu. Vezi [06](06-cand-o-dai-in-bara.md).

<details>
<summary>Întrebare de interviu: Ce e un branch în Git?</summary>

Un pointer mobil spre un commit. Când faci commit pe el, pointerul înaintează. Crearea unui branch e practic gratuită, de aceea fiecare feature sau fix stă izolat pe branch-ul lui până e gata de merge. HEAD indică branch-ul (sau commit-ul) pe care lucrezi acum.
</details>

## Cele trei comenzi de citit: status, diff, log

**Ce e.** Comenzile pe care le dai cel mai des. Nu schimbă nimic, doar îți arată starea.

```bash
git status                   # where am I, what changed
git diff                     # changes not yet staged
git diff --staged            # what goes into the next commit
git log --oneline -10        # last 10 commits
git log -p -- src/app.ts     # history of one file, with changes
git blame src/app.ts         # which commit last changed each line
```

**De ce contează.** `git diff --staged` e ultima ta șansă să vezi ce intră în istorie. Regula ghidului spune că nu faci commit la cod pe care nu-l poți explica, iar diff-ul e locul unde verifici asta.

**Cum îl vezi în cod real.** În VS Code, panoul Source Control (`Ctrl+Shift+G`) arată același diff, vizual, fișier cu fișier. Pentru început e mai ușor de citit decât în terminal.

**Capcana.** Să citești doar lista de fișiere, nu și conținutul diff-ului. AI-ul modifică uneori și fișiere pe care nu le-ai cerut, cum ar fi un config sau un test „reparat” prin ștergere.

## restore: anulezi modificări locale

**Ce e.** `git restore` aduce un fișier înapoi la ultima versiune salvată în Git.

```bash
git restore src/app.ts            # discard unstaged changes (they are gone)
git restore --staged src/app.ts   # unstage, keep the changes in the file
```

**Capcana.** Modificările pe care nu le-ai pus niciodată într-un commit nu sunt nicăieri în Git. După `git restore` ele dispar pentru totdeauna. Cu cât faci commit mai des, cu atât ai mai multe puncte de salvare.

## .gitignore: ce nu intră niciodată în Git

**Ce e.** Un fișier text cu tipare de căi pe care Git le ignoră.

**Exemplu minim** (pentru un proiect Node/React):

```gitignore
node_modules/
dist/
build/
coverage/
.env
.env.*
!.env.example
*.local
.DS_Store
```

**Ce intră și ce nu:**

| Intră în Git | Nu intră |
|---|---|
| codul sursă, `package.json` | `node_modules/` (se reface cu `npm ci`) |
| lockfile-ul: `package-lock.json` sau `bun.lock` | `dist/`, `build/` (se refac cu `npm run build`) |
| `.env.example` (chei fără valori) | `.env` (secretele reale) |
| configurări: `tsconfig.json`, ESLint, Prettier | fișiere de editor și de sistem |
| migrațiile bazei de date | rapoarte de teste (`coverage/`) |

**De ce lockfile-ul intră.** Lockfile-ul ține versiunile exacte instalate. Fără el, colegul tău și CI-ul instalează alte versiuni decât tine, și apare clasicul „la mine merge”. Detalii în [03-ecosistem-js/02-npm-si-pachete.md](../03-ecosistem-js/02-npm-si-pachete.md).

**Capcana.** `.gitignore` nu scoate din Git un fișier pe care Git îl urmărește deja. Dacă ai făcut commit la `.env`, nu ajunge să-l adaugi în `.gitignore`. Trebuie `git rm --cached .env` și un commit nou. Dacă ajunsese pe GitHub, **rotești cheile**: vezi [06](06-cand-o-dai-in-bara.md) și [10-securitate-env/03-secrete-scurse.md](../10-securitate-env/03-secrete-scurse.md).

<details>
<summary>Întrebare de interviu: Ce pui în .gitignore într-un proiect Node și de ce lockfile-ul rămâne în Git?</summary>

Ignor tot ce se poate regenera sau e secret: `node_modules`, build output-ul, fișierele `.env` și fișierele locale de editor. Lockfile-ul rămâne în Git pentru că fixează versiunile exacte ale tuturor dependențelor, inclusiv ale celor indirecte. Așa toată echipa, CI-ul și serverul instalează exact același lucru cu `npm ci`.
</details>

## Line endings pe Windows: de ce lucrezi doar din WSL

**Ce e.** Windows termină liniile din fișiere cu două caractere (CRLF), iar Linux și macOS cu unul (LF). Dacă același repo e atins și de Git-ul din Windows, și de cel din WSL, apar diff-uri pe fișiere întregi în care „nu s-a schimbat nimic”. Pe deasupra, scripturile shell nu mai rulează.

**De ce contează.** Serverele de deploy și CI-ul rulează pe Linux. Tu lucrezi în WSL tocmai ca să ai același mediu.

**Regula.**

- Repo-urile stau în `~/code` din WSL, nu în `/mnt/c/...`.
- Folosești doar Git-ul din WSL, pe acele repo-uri. Nu le deschizi cu GitHub Desktop sau cu Git-ul instalat pe Windows.
- VS Code îl deschizi din WSL cu `code .`, ca să folosească și el Git-ul din Linux.

**Capcana.** Când vezi un diff în care fiecare linie apare ștearsă și adăugată la loc identic, aproape sigur e o problemă de line endings, nu o modificare reală. Nu face commit la așa ceva. Întreabă-l pe Claude ce s-a întâmplat.

## Mai departe

- [Pro Git, capitolele 1–3](https://git-scm.com/book/en/v2): gratuită, cea mai bună carte despre Git
- [Ignoring files](https://docs.github.com/en/get-started/git-basics/ignoring-files): GitHub Docs
