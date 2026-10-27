# VS Code

În munca cu AI, editorul e mai mult locul unde **citești** cod decât unde îl scrii. Instalezi VS Code pe Windows, îl conectezi la WSL și înveți scurtăturile cu care te miști printr-un proiect pe care nu l-ai scris tu.

## Pasul 1 — Instalează VS Code și extensia WSL

1. Pe **Windows**, descarcă VS Code de pe [code.visualstudio.com](https://code.visualstudio.com) și instalează-l. Lasă bifată opțiunea **„Add to PATH”** (e implicită): de ea depinde comanda `code` din WSL.
2. Deschide VS Code, apasă `Ctrl+Shift+X` (Extensions), caută **WSL** (publisher Microsoft, id `ms-vscode-remote.remote-wsl`) și instaleaz-o.
3. Închide VS Code. În Warp, în Ubuntu:

   ```bash
   cd ~/code/js-de-la-zero
   code .
   ```

**Verifică.** Prima dată durează câteva secunde, pentru că VS Code instalează în Ubuntu componenta de server. În colțul din stânga jos trebuie să vezi **WSL: Ubuntu**. Dacă nu scrie asta, ai deschis folderul „ca Windows” și pierzi IntelliSense corect, debugging și extensiile din Linux.

**Ce s-a întâmplat de fapt.** VS Code rulează împărțit în două. Interfața (ferestrele, tema) rulează pe Windows. Serverul (citirea fișierelor, terminalul, Git, extensiile de limbaj) rulează în Ubuntu, lângă cod. De aceea merge rapid și nu strică line endings.

## Pasul 2 — Extensii și setări

Cu un folder din WSL deschis, instalează din Extensions (`Ctrl+Shift+X`). VS Code le pune automat în partea WSL:

| Extensie | Id | La ce folosește |
|---|---|---|
| Oxc | `oxc.oxc-vscode` | Subliniază problemele raportate de oxlint, linter-ul care vine acum în template-urile Vite și NestJS |
| ESLint | `dbaeumer.vscode-eslint` | Același lucru pentru ESLint, pe care îl vei întâlni în multe proiecte existente și în Next.js |
| Prettier | `esbenp.prettier-vscode` | Formatează codul la salvare, ca să nu mai discuți niciodată despre spații |
| Error Lens (opțional) | `usernamehw.errorlens` | Afișează erorile direct pe linie, nu doar la hover |

Apoi deschide setările în format JSON: `Ctrl+Shift+P` → **Preferences: Open User Settings (JSON)**. Adaugă:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

Setează VS Code ca editor pentru Git (pentru mesaje de commit lungi și rebase), în Ubuntu:

```bash
git config --global core.editor "code --wait"
```

## Scurtăturile pentru citit cod

**Ce e.** Un set mic de scurtături care îți permit să urmărești codul de la un apel la definiție și înapoi fără să cauți cu mouse-ul prin foldere.

**De ce contează.** La code walk (pasul 4 din [bucla de lucru](../12-lucrul-cu-ai/01-workflow.md)) treci prin zeci de fișiere scrise de Claude. Diferența dintre 5 minute și 30 de minute pe fișier o fac scurtăturile astea.

| Scurtătură | Ce face |
|---|---|
| `Ctrl+P` | Deschide un fișier după nume (scrii câteva litere) |
| `Ctrl+Shift+P` | Command Palette: orice comandă VS Code, după nume |
| `F12` sau `Ctrl+Click` | Go to Definition: sari la locul unde e definită funcția sau tipul |
| `Alt+←` / `Alt+→` | Înapoi / înainte, după ce ai sărit |
| `Shift+F12` | Peek References: cine folosește funcția asta, într-un panou mic |
| `Shift+Alt+F12` | Find All References, în panoul lateral |
| `Ctrl+Shift+O` | Simbolurile din fișierul curent (funcții, clase) |
| `Ctrl+T` | Caută un simbol în tot proiectul |
| `Ctrl+Shift+F` | Caută text în tot proiectul |
| `F2` | Redenumește un simbol peste tot unde e folosit |
| `` Ctrl+` `` | Terminalul integrat (tot în Ubuntu) |
| Hover pe o variabilă | Îi vezi tipul. În TypeScript e cel mai ieftin mod de a înțelege codul |

Panoul **Outline** (jos, în Explorer) îți arată scheletul fișierului curent: ce funcții și clase are, în ordine.

**Exemplu minim.** Deschide `ziua7/05-inventar.js` din `js-de-la-zero`. Pune cursorul pe `valoareTotala` și apasă `Shift+F12` ca să vezi unde e apelată. Apoi `Ctrl+Shift+O` și sari la `stocSubMinim`.

**Capcana.** Să citești un proiect deschizând fișierele în ordine alfabetică. Pornește de la punctul de intrare (`main.tsx`, `main.ts`) și urmărește apelurile cu `F12`.

## Source Control: unde vezi ce a schimbat Claude

**Ce e.** Panoul Source Control (`Ctrl+Shift+G`) arată fișierele modificate față de ultimul commit. Click pe un fișier și vezi diff-ul lângă original: roșu e ce s-a șters, verde ce s-a adăugat.

**De ce contează.** E unealta ta principală de review. După fiecare pas făcut de Claude Code, te uiți aici înainte de commit. Poți face stage doar la o parte din modificări (click dreapta pe o selecție → **Stage Selected Ranges**) și poți arunca restul.

**Capcana.** Commit fără să fi deschis diff-ul. Dacă nu l-ai citit, nu știi ce ai pus în istorie. Checklist-ul de review e în [12-lucrul-cu-ai/02-review-la-cod-generat.md](../12-lucrul-cu-ai/02-review-la-cod-generat.md).

## Debugger-ul, minimul necesar

**Ce e.** În loc de `console.log` peste tot, oprești programul pe o linie (breakpoint) și te uiți la toate variabilele în momentul ăla. Apoi avansezi pas cu pas.

**De ce contează.** E cea mai rapidă cale să înțelegi cod străin și să găsești bug-uri subtile, exact ce faci la pasul „break & fix”. La interviu, „folosesc debugger-ul” sună a om care a lucrat pe proiecte reale.

**Exemplu minim**

1. `Ctrl+Shift+P` → **Debug: JavaScript Debug Terminal**. Se deschide un terminal special.
2. Click în stânga numărului de linie de la `return` din `stocSubMinim` și apare un punct roșu: ăsta e breakpoint-ul.
3. În terminalul ăla rulează `node ziua7/05-inventar.js`.
4. Programul se oprește la breakpoint. În stânga vezi **Variables**, **Watch** și **Call Stack**.
5. `F10` trece la linia următoare, `F11` intră în funcție, `F5` continuă până la următorul breakpoint.

Exercițiu de detectiv: adaugă un produs cu `cantitate` 12 și `minim` 20. Ar trebui să apară în lista de stoc sub minim. Apare? Urmărește cu debugger-ul și găsește de ce.

**Capcana.** Breakpoint pus într-un fișier, dar programul pornit din terminalul obișnuit. Breakpoint-urile funcționează doar în **JavaScript Debug Terminal** (sau cu o configurație de launch).

<details>
<summary>Întrebare de interviu: Cum depanezi un bug pe care nu-l înțelegi?</summary>

Întâi îl reproduc sigur: pași exacți, input exact. Citesc tot mesajul de eroare și stack trace-ul, de sus, ca să găsesc primul fișier care e al meu. Îmi formulez o ipoteză despre cauză și o verific, cu un breakpoint în debugger sau cu un log țintit, nu cu zeci de `console.log`. Dacă ipoteza pică, restrâng zona: comentez bucăți sau caut ultimul commit bun cu `git bisect`. La final scriu un test care reproduce bug-ul, ca să nu revină.
</details>

## Mai departe

- [Developing in WSL — VS Code docs](https://code.visualstudio.com/docs/remote/wsl)
- [Node.js debugging in VS Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)
