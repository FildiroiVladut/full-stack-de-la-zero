# Git și GitHub

Configurezi Git în Ubuntu, legi laptopul de contul tău GitHub prin SSH și instalezi `gh`, unealta oficială GitHub din terminal. La final îți clonezi din nou `js-de-la-zero`, de data asta în WSL.

## Git vs GitHub

**Ce e.** **Git** e programul care ține istoria codului pe laptopul tău: commit-uri, branch-uri, diff-uri. Funcționează și fără internet. **GitHub** e un serviciu online care găzduiește repo-uri Git și adaugă ce ține de echipă: pull request-uri, code review, issues, CI.

**De ce contează.** La interviu, confuzia dintre ele e un semnal rău. În munca de zi cu zi, Git e plasa ta de siguranță când Claude Code modifică zece fișiere deodată. Modelul mental complet e în [09-git-github-echipa](../09-git-github-echipa/01-cum-gandeste-git.md). Aici doar instalezi.

<details>
<summary>Întrebare de interviu: Care e diferența dintre Git și GitHub?</summary>

Git e un sistem de version control distribuit: rulează local și fiecare clonă are istoria completă. GitHub e o platformă care găzduiește repo-uri Git și adaugă colaborarea: pull request-uri, review, issues, Actions pentru CI. Poți folosi Git fără GitHub (cu GitLab, Bitbucket sau doar local), dar nu poți folosi GitHub fără Git.
</details>

## Pasul 1 — Configurează Git

Ubuntu vine de obicei cu Git instalat. Verifică:

```bash
git --version
```

Dacă primești `command not found`: `sudo apt install -y git`.

Spune-i lui Git cine ești. Numele și emailul apar în fiecare commit:

```bash
git config --global user.name "Prenume Nume"
git config --global user.email "ID+username@users.noreply.github.com"
git config --global init.defaultBranch main
```

**De unde iei emailul.** Pe GitHub: **Settings → Emails**, bifezi „Keep my email addresses private” și copiezi adresa `...@users.noreply.github.com` afișată acolo. Commit-urile sunt publice, deci așa nu-ți expui emailul personal boților care colectează adrese din repo-uri.

**Verifică.**

```bash
git config --global --list
```

**Ce s-a întâmplat de fapt.** `--global` scrie setările în `~/.gitconfig` și ele se aplică tuturor repo-urilor tale. `init.defaultBranch main` face ca branch-ul principal al repo-urilor noi să se numească `main`, convenția actuală.

## Chei SSH, pe scurt

**Ce e.** O pereche de chei asimetrice. **Cheia privată** (`~/.ssh/id_ed25519`) nu părăsește niciodată laptopul. **Cheia publică** (`~/.ssh/id_ed25519.pub`) o pui pe GitHub. La fiecare `git push`, GitHub îți trimite o provocare pe care doar cine are cheia privată o poate semna. E criptografia cu cheie publică de la facultate, folosită la ceva concret.

**De ce contează.** Fără parole la fiecare push. Același mecanism îl vei folosi mai târziu ca să intri pe servere.

**Capcana.** Să trimiți cuiva, să lipești într-un chat sau să urci într-un repo fișierul **fără** `.pub`. Cheia privată e echivalentul cheii de la casă: dacă a văzut-o altcineva, o regenerezi.

## Pasul 2 — Instalează `gh` (GitHub CLI)

Comenzile oficiale pentru Ubuntu. Copiază-le pe toate odată:

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && sudo mkdir -p -m 755 /etc/apt/sources.list.d \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

**Verifică.** `gh --version`

**Ce s-a întâmplat de fapt.** Ai adăugat în `apt` repository-ul oficial GitHub, împreună cu cheia lui de semnătură, apoi ai instalat `gh` din el. E tiparul standard pentru orice unealtă care nu există în magazinul implicit al Ubuntu. Același tipar apare și în instalarea Claude Code prin apt.

## Pasul 3 — Login și cheie SSH, dintr-o mișcare

```bash
gh auth login
```

Răspunde la întrebări așa:

1. **Where do you use GitHub?** → `GitHub.com`
2. **Preferred protocol for Git operations?** → `SSH`
3. **Generate a new SSH key?** → `Yes`. `gh` creează `~/.ssh/id_ed25519` și îl urcă pe contul tău.
4. **Passphrase** → pune una. E parola cheii, cerută când o folosești.
5. **Title for your SSH key** → de exemplu `laptop-wsl`.
6. **How would you like to authenticate?** → `Login with a web browser`. Primești un cod de 8 caractere. Dacă browserul nu se deschide singur din WSL, deschide tu pe Windows [github.com/login/device](https://github.com/login/device) și introdu codul.

Ca să nu tastezi passphrase-ul la fiecare push, instalează `keychain`. Îți cere passphrase-ul o singură dată după fiecare pornire a laptopului:

```bash
sudo apt install -y keychain
echo 'eval "$(keychain --eval --quiet id_ed25519)"' >> ~/.bashrc
source ~/.bashrc
```

**Verifică.**

```bash
gh auth status
ssh -T git@github.com
```

Prima dată, `ssh` te întreabă dacă ai încredere în fingerprint-ul serverului. Răspunde `yes`. La final trebuie să vezi:

```text
Hi FildiroiVladut! You've successfully authenticated, but GitHub does not provide shell access.
```

Mesajul „does not provide shell access” e normal. Înseamnă că autentificarea a mers.

**Ce s-a întâmplat de fapt.** `gh` are acum un token de acces (pentru repo-uri, PR-uri și issues din terminal), iar Git folosește cheia SSH pentru push și pull. Fingerprint-ul acceptat la primul `ssh` e salvat în `~/.ssh/known_hosts`, așa că data viitoare SSH verifică singur că vorbești cu GitHub-ul adevărat.

## Pasul 4 — Clonează `js-de-la-zero` în WSL

Dacă ai modificări necommise în copia de pe Windows, fă-le commit și push de acolo întâi. Apoi, în Ubuntu:

```bash
cd ~/code
git clone git@github.com:FildiroiVladut/js-de-la-zero.git
cd js-de-la-zero
git log --oneline | head -5
```

**Verifică.** Vezi ultimele commit-uri (`Ziua 7: clase si erori...`). De acum lucrezi doar pe copia asta. Pe cea din Windows o poți șterge după ce te-ai convins că pe GitHub e tot.

## Line endings: CRLF vs LF

**Ce e.** Windows termină liniile din fișierele text cu două caractere (`\r\n`, CRLF). Linux și macOS folosesc unul singur (`\n`, LF). Același fișier poate arăta identic în editor și totuși să difere la nivel de bytes.

**De ce contează.** Dacă amesteci unelte de Windows cu unelte de Linux pe același repo, Git vede modificate fișiere pe care nu le-ai atins: fiecare linie apare schimbată în diff. Unele scripturi de shell cu CRLF nici nu mai rulează (`/bin/bash^M: bad interpreter`).

**Cum îl eviți.** Lucrezi doar din WSL, cu Git din WSL și VS Code deschis cu `code .`, și nu ai problema. În proiecte vei vedea adesea un fișier `.gitattributes` cu linia `* text=auto eol=lf`, care fixează regula pentru toată echipa, indiferent de sistemul fiecăruia.

**Capcana.** GitHub Desktop sau alt client Git de Windows deschis peste un repo din `\\wsl$`. Alegi o singură lume, WSL, și rămâi în ea.

## Mai departe

- [Connecting to GitHub with SSH — GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [gh auth login — GitHub CLI manual](https://cli.github.com/manual/gh_auth_login)
- [Get started using Git on WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git)
