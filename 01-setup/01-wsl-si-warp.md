# WSL și Warp

Aici îți faci mediul de lucru: Ubuntu care rulează în Windows (WSL) și un terminal modern (Warp) care se deschide direct în el. La final știi comenzile de bază și unde stau fișierele tale.

## Terminal, shell, comandă

**Ce e.** **Terminalul** e fereastra (Warp, Windows Terminal). **Shell-ul** e programul care rulează în ea, citește ce tastezi și execută: în Ubuntu e `bash`. O **comandă** e un program (`ls`, `git`, `node`) căruia îi dai argumente (`ls -la ~/code`).

**De ce contează.** Toate uneltele din job-ul ăsta se folosesc din terminal: Git, npm, Claude Code, deploy-ul. Un junior care se mișcă repede în terminal arată din prima că a mai lucrat.

**Capcana.** Să confunzi shell-urile. PowerShell (Windows) și bash (Linux) au comenzi diferite. În ghidul ăsta, **toate comenzile sunt de bash și se rulează în Ubuntu**, în afară de cele marcate explicit „în PowerShell”.

## De ce WSL și nu Windows direct

**Ce e.** WSL (Windows Subsystem for Linux) rulează un Linux adevărat (Ubuntu) în Windows, fără mașină virtuală de configurat și fără dual boot. WSL 2 e versiunea actuală, cu kernel Linux real.

**De ce contează.** Microsoft spune deschis că pentru un începător Node pe Windows e mai simplu de instalat. Noi alegem totuși WSL, din trei motive:

- **Serverele sunt Linux.** Render, Vercel și containerele din CI rulează Linux. Dacă lucrezi pe același sistem, scapi de bug-urile de tip „pe mașina mea merge”. Și Microsoft recomandă WSL când faci deploy pe servere Linux.
- **Tutorialele și documentațiile presupun bash.** Orice comandă din docs-ul de la Supabase, Nest sau Vite merge copiată ca atare.
- **Claude Code e mai bine acasă în WSL 2.** Acolo are bash nativ și sandboxing, care pe Windows nativ nu există.

**Capcana.** Să lucrezi „pe jumătate”: proiectul în `C:\Users\...` și comenzile din WSL, sau invers. E lent și strică permisiunile și line endings. Regula e simplă: **cod în Linux, comenzi în Linux.**

## Pasul 1 — Instalează WSL

1. Deschide meniul Start, scrie `PowerShell`, apoi click dreapta → **Run as administrator**.
2. Rulează (în PowerShell):

   ```powershell
   wsl --install
   ```

   Comanda activează WSL și instalează Ubuntu, distribuția implicită.
3. **Repornește laptopul.**
4. După restart se deschide o fereastră Ubuntu care despachetează fișierele. Îți cere un **username** (litere mici, fără spații, de exemplu `vlad`) și o **parolă**. Când tastezi parola nu apare nimic pe ecran, nici steluțe, și asta e normal în Linux. Parola o vei folosi la `sudo`, deci n-o uita.
5. Actualizează sistemul și instalează câteva unelte de bază (în Ubuntu):

   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y curl unzip build-essential
   ```

**Verifică.** În PowerShell (nu e nevoie de admin):

```powershell
wsl -l -v
```

Trebuie să vezi `Ubuntu` cu `VERSION 2`. Dacă scrie `1`, rulează `wsl --set-version Ubuntu 2`.

**Ce s-a întâmplat de fapt.** `sudo` rulează o comandă ca administrator (îți cere parola). `apt` e managerul de pachete al Ubuntu, adică magazinul de programe al sistemului de operare. Mai târziu o să lucrezi cu `npm`, care face același lucru pentru pachetele JavaScript. `build-essential` aduce `gcc`, `make` și compania: compilatoarele de C/C++ pe care le știi de la facultate. Unele pachete npm compilează cod nativ la instalare și au nevoie de ele.

## Pasul 2 — Instalează Warp și deschide-l în Ubuntu

1. Pe **Windows**, descarcă Warp de pe [warp.dev/download](https://www.warp.dev/download) și rulează installer-ul. Alternativ, în PowerShell: `winget install Warp.Warp`.
2. Deschide Warp din Start. La primul pornit îți propune să-ți faci cont. **Login-ul e opțional**, așa că poți sări peste el.
3. Setează Ubuntu ca shell implicit: **Settings → Features → Session → „Startup shell for new sessions”** și alege intrarea WSL (Ubuntu). Schimbarea se aplică doar sesiunilor noi, deci deschide un tab nou.
4. Oprește AI-ul propriu al lui Warp: **Settings → Agents → Warp Agent**, apoi dezactivează funcțiile AI. Motivul: înveți să lucrezi cu un singur agent, Claude Code, și nu vrei două AI-uri care îți sugerează lucruri diferite în aceeași fereastră. Dacă meniul s-a mutat între versiuni, caută „AI” în bara de căutare din Settings.

**Verifică.** Într-un tab nou din Warp:

```bash
uname -a     # contains "microsoft" and "WSL2"
pwd          # /home/<your-username>
```

**Planul B.** Dacă Warp îți face figuri (nu pornește în Ubuntu, Claude Code se afișează ciudat), folosește **Windows Terminal**. E preinstalat pe Windows 11, e gratuit și e terminalul recomandat de Microsoft pentru WSL. În Settings → Startup → Default profile alege Ubuntu. Restul ghidului rămâne neschimbat, pentru că tot ce contează se întâmplă în Ubuntu, nu în fereastră.

## Comenzile de bază

| Comandă | Ce face | De reținut |
|---|---|---|
| `pwd` | Unde sunt acum (print working directory) | Rulează-o oricând te-ai pierdut |
| `ls -la` | Listează tot, inclusiv fișierele ascunse | Fișierele care încep cu `.` sunt ascunse (`.git`, `.env`) |
| `cd folder` / `cd ..` / `cd ~` / `cd -` | Intri în folder / urci un nivel / acasă / înapoi unde erai | |
| `mkdir -p a/b/c` | Creează foldere, cu tot cu părinții | `-p` nu dă eroare dacă există deja |
| `touch notes.md` | Creează un fișier gol | |
| `cat file` | Afișează un fișier scurt | |
| `less file` | Citește un fișier lung, cu scroll | `q` ca să ieși, `/text` ca să cauți |
| `cp a b` / `mv a b` | Copiază / mută sau redenumește | `cp -r` pentru foldere |
| `rm file` / `rm -r folder` | Șterge **definitiv**, fără Recycle Bin | Citește de două ori înainte de Enter |
| `clear` sau `Ctrl+L` | Curăță ecranul | |
| `history` | Comenzile anterioare | `Ctrl+R` caută în istoric |
| `comandă --help` | Ajutor rapid pentru o comandă | `man comandă` pentru manualul complet |

Patru reflexe care îți economisesc ore:

- **`Tab`** completează nume de fișiere și comenzi. Nu scrie niciodată o cale întreagă de mână.
- **`↑` / `↓`** aduc comenzile anterioare.
- **`Ctrl+C`** oprește comanda care rulează (un server, un script blocat). Nu copiază text ca în Windows.
- **`rm -rf`** e singura comandă la care nu te grăbești niciodată. Mai ales cu variabile sau cu `/` în cale.

## Căi, `~` și `PATH`

**Ce e.** O **cale absolută** pornește din rădăcină: `/home/vlad/code`. O cale **relativă** pornește de unde ești: `code/shelf`, `../notes.md`. `~` e scurtătura pentru folderul tău de acasă (`/home/vlad`), iar `.` înseamnă folderul curent. **`PATH`** e o variabilă de mediu care conține lista de foldere în care shell-ul caută programe când tastezi o comandă.

**De ce contează.** Când instalezi o unealtă și terminalul răspunde `command not found`, aproape mereu folderul ei nu e în `PATH`. Vei vedea asta la Claude Code, la fnm și la Bun.

**Exemplu minim**

```bash
echo $PATH | tr ':' '\n'     # one folder per line
which node                   # which folder the node command comes from
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc             # reload the config in this terminal
```

**Cum îl vezi în cod real.** Scripturile de instalare îți spun la final ceva de genul „add this line to your `~/.bashrc`”. `~/.bashrc` e fișierul pe care bash îl citește la fiecare terminal nou. Orice modificare acolo se aplică abia după `source ~/.bashrc` sau într-un tab nou.

**Capcana.** WSL importă implicit și `PATH`-ul din Windows. Dacă ai avut Node instalat pe Windows, `which node` poate răspunde `/mnt/c/Program Files/nodejs/node`, adică folosești Node-ul de Windows din Linux. Orice cale care începe cu `/mnt/c/` la o unealtă de dezvoltare e semn că ceva e greșit.

<details>
<summary>Întrebare de interviu: Ce se întâmplă când tastezi o comandă în terminal și apeși Enter?</summary>

Shell-ul (bash, de exemplu) desparte linia în numele comenzii și argumente. Dacă e o comandă internă a shell-ului (`cd`, `echo`), o execută direct. Altfel caută un executabil cu numele ăla, pe rând, în folderele din `PATH`. Primul găsit e pornit ca proces nou, cu argumentele primite, și moștenește variabilele de mediu. Shell-ul așteaptă să termine și primește un exit code: 0 înseamnă succes, orice altceva înseamnă eroare.
</details>

## Două sisteme de fișiere

**Ce e.** Din WSL ai acces la două lumi:

```text
/home/vlad/...            Linux file system   <- your code lives here
/mnt/c/Users/Vlad/...     Windows C: drive    <- Downloads, Desktop
```

Din Windows, fișierele Linux se văd în File Explorer la adresa `\\wsl$` (sau în secțiunea „Linux” din bara laterală).

**De ce contează.** Microsoft recomandă explicit: când lucrezi din linia de comandă Linux, ține fișierele în sistemul Linux, pentru viteză. Diferența e mare. `npm install` pe `/mnt/c` poate dura de câteva ori mai mult.

**Exemplu minim**

```bash
mkdir -p ~/code       # all your projects go here
cd ~/code
explorer.exe .        # opens this folder in Windows File Explorer
```

**Cum îl vezi în practică.** Descarci ceva în Windows (o arhivă, un PDF) și îl copiezi în Linux: `cp /mnt/c/Users/<user-windows>/Downloads/file.zip ~/code/`. Folosești `/mnt/c` ca să aduci lucruri, nu ca să lucrezi pe ele.

**Capcana.** Să editezi fișierele din WSL cu programe de Windows (Notepad, alt editor deschis direct pe `\\wsl$`). Pot schimba line endings și permisiunile. Pentru cod folosești VS Code deschis cu `code .` din Ubuntu, care știe să lucreze corect în WSL ([04-vscode.md](04-vscode.md)).

## Mai departe

- [Install WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Working across file systems — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- [Supported shells — Warp docs](https://docs.warp.dev/getting-started/supported-shells)
