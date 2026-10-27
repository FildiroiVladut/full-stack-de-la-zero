# Node, npm și Bun

Instalezi Node.js printr-un version manager (fnm) și Bun alături de el. Aici doar instalezi și verifici. Ce sunt de fapt un runtime și un package manager, și când alegi `npm` sau `bun`, afli în [03-ecosistem-js](../03-ecosistem-js/).

## De ce un version manager

**Ce e.** Un version manager (fnm, nvm, Volta) instalează mai multe versiuni de Node una lângă alta și o activează pe cea de care are nevoie proiectul curent. Noi folosim **fnm**: e rapid, scris în Rust, și citește automat fișierele `.nvmrc` și `.node-version` din proiecte.

**De ce contează.**

- Proiectele își fixează versiunea de Node (`.nvmrc`, câmpul `engines` din `package.json`). Un proiect vechi la care lucrezi la job poate cere altă versiune decât ultimul tău proiect.
- Node-ul din `apt` e de obicei învechit, iar Microsoft îl descurajează explicit în ghidul pentru WSL.
- Cu un version manager instalezi pachete globale fără `sudo`. Un `sudo npm install -g` strică permisiunile și e un risc de securitate.

**Capcana.** Două instalări de Node care se bat între ele. Dacă `which node` arată o cale care începe cu `/mnt/c/`, folosești Node-ul de Windows. Rezolvarea e să ai Node doar prin fnm, în Ubuntu.

## Pasul 1 — Instalează fnm

`curl` și `unzip` le-ai instalat deja în [01-wsl-si-warp.md](01-wsl-si-warp.md). Rulează:

```bash
curl -fsSL https://fnm.vercel.app/install | bash
source ~/.bashrc
fnm --version
```

Scriptul adaugă la finalul `~/.bashrc` un bloc `# fnm`. Activează și schimbarea automată a versiunii când intri într-un proiect cu `.nvmrc`:

```bash
sed -i 's/fnm env --shell bash/fnm env --use-on-cd --shell bash/' ~/.bashrc
source ~/.bashrc
grep -n "fnm env" ~/.bashrc
```

**Verifică.** Ultima comandă afișează linia `eval "$(fnm env --use-on-cd --shell bash)"`.

**Ce s-a întâmplat de fapt.** fnm s-a instalat în folderul tău de acasă și a adăugat în `~/.bashrc` codul care îl pune în `PATH` la fiecare terminal nou. `sed -i` a înlocuit textul direct în fișier. Cu `--use-on-cd`, fnm verifică la fiecare `cd` dacă folderul are `.nvmrc` și comută singur versiunea.

## Pasul 2 — Instalează Node LTS

```bash
fnm install --lts
fnm list
fnm default
```

**Verifică.**

```bash
node -v
npm -v
which node     # must show a path in your home folder, not /mnt/c/...
```

**Ce s-a întâmplat de fapt.** LTS (Long Term Support) e versiunea pe care rulează producția: primește patch-uri de securitate ani de zile. Prima versiune instalată cu fnm devine automat `default`, cea activă în orice terminal nou. `npm` vine la pachet cu Node, deci nu-l instalezi separat.

**Exemplu minim**: un `.nvmrc` care fixează versiunea pentru un proiect.

```bash
cd ~/code/js-de-la-zero
node -v > .nvmrc      # pins the current version for this project
cat .nvmrc
```

Data viitoare când intri în folder, fnm activează versiunea din fișier. Dacă nu o ai instalată, rulezi `fnm install`, fără argumente.

## Pasul 3 — Instalează Bun

```bash
curl -fsSL https://bun.com/install | bash
source ~/.bashrc
bun --version
```

**Verifică.** `bun --version` afișează un număr de forma `1.x.y`. Dacă primești `command not found`, deschide un tab nou. Dacă tot nu merge, verifică dacă `~/.bun/bin` apare în `echo $PATH`.

**Ce s-a întâmplat de fapt.** Bun e un runtime de JavaScript separat de Node, dar e și package manager, bundler și test runner, totul într-un singur executabil. Scriptul l-a pus în `~/.bun/bin` și a adăugat folderul în `PATH` prin `~/.bashrc`. Bun nu înlocuiește Node pe sistemul tău: le ai pe amândouă și alegi per proiect. Comparația pe îndelete e în [03-ecosistem-js/03-npm-vs-bun.md](../03-ecosistem-js/03-npm-vs-bun.md).

## Verificarea finală

```bash
node -v && npm -v && bun --version && git --version && gh --version
```

Cinci versiuni, nicio eroare: ai uneltele de bază. Dacă vreuna lipsește, întoarce-te la pasul ei.

## Mai departe

- [fnm pe GitHub](https://github.com/Schniz/fnm)
- [Bun — Installation](https://bun.com/docs/installation)
- [Set up Node.js on WSL 2 — Microsoft Learn](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl)
