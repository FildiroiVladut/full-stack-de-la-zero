# 01 · Setup

Înainte de primul rând de cod îți trebuie atelierul: un terminal bun, Linux în Windows (WSL), Git legat de GitHub, Node, Bun, un editor în care citești cod și Claude Code. Setup-ul îl faci o singură dată, cu atenție, și apoi nu te mai gândești la el. Unul făcut pe jumătate te costă zece minute pe zi, la nesfârșit.

Laptopul tău e pe Windows, dar vei lucra aproape tot timpul în Linux (Ubuntu, prin WSL). De ce, afli din primul fișier.

## La final poți să…

- deschizi Warp direct în Ubuntu și să te miști prin fișiere doar din tastatură;
- faci push pe GitHub prin SSH, fără parole;
- instalezi și schimbi versiunea de Node dintr-o comandă și să rulezi atât `npm`, cât și `bun`;
- deschizi orice proiect din WSL în VS Code cu `code .` și să sari prin cod cu scurtături;
- pornești Claude Code în modul potrivit (manual, plan, accept edits) și să-ți drămuiești limitele planului Pro;
- publici repo-ul ăsta pe GitHub-ul tău și să bifezi un checkpoint printr-un pull request.

## Fișierele, în ordine

- [ ] [01 · WSL și Warp](01-wsl-si-warp.md) — 60–90 min (include un restart)
- [ ] [02 · Git și GitHub](02-git-si-github.md) — 30–45 min
- [ ] [03 · Node, npm și Bun](03-node-npm-bun.md) — 20–30 min
- [ ] [04 · VS Code](04-vscode.md) — 30–45 min
- [ ] [05 · Claude Code](05-claude-code.md) — 45–60 min
- [ ] [06 · Publică ghidul](06-publica-ghidul.md) — 30 min

Total: cam 4–5 ore. Fă-le în aceeași zi și în ordine, pentru că fiecare pas se sprijină pe cel dinainte.

## Unde îl folosești

Peste tot. Concret: [P1 Shelf](../proiecte/P1-shelf.md) e primul proiect construit integral cu uneltele de aici, iar [CP1](../checkpoints/CP1.md) îl bifezi printr-un PR deschis exact ca în [06-publica-ghidul.md](06-publica-ghidul.md). Cum folosești Claude Code în proiecte, pas cu pas, e în [12-lucrul-cu-ai](../12-lucrul-cu-ai/).

## Verifică-te

1. De ce stau proiectele tale în `~/code` și nu în `C:\Users\...`?
2. Ce e `PATH` și ce înseamnă când o comandă răspunde cu „command not found”?
3. Care e diferența dintre cheia SSH privată și cea publică? Care dintre ele ajunge pe GitHub?
4. De ce instalezi Node printr-un version manager și nu cu `apt install nodejs`?
5. În ce mod pornești Claude Code când vrei să vezi și să aprobi fiecare comandă? Cum ajungi în plan mode?
