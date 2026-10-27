# Claude Code

Claude Code e un agent care lucrează în terminal: citește fișierele proiectului, rulează comenzi, editează cod și verifică ce a făcut. Aici îl instalezi în WSL, te loghezi cu contul Pro și înveți modurile, scurtăturile și regulile de siguranță. Cum lucrezi efectiv cu el pe un proiect, pas cu pas, e în [12-lucrul-cu-ai](../12-lucrul-cu-ai/).

## Ce e, de fapt, Claude Code

**Ce e.** Un chat îți dă un răspuns și se oprește. Claude Code primește un task și lucrează într-o buclă: citește fișiere ca să înțeleagă contextul, își face un plan, editează și rulează comenzi (teste, build, `git`), se uită la rezultat și corectează. Totul se întâmplă pe laptopul tău, în folderul în care l-ai pornit.

**De ce contează.** Pe job, „știu să lucrez eficient cu un agent de cod” e criteriu de angajare. Dar agentul execută ce i-ai cerut, nu neapărat ce ai vrut. Tu rămâi cel care decide, verifică și răspunde de cod.

**Capcana.** Să-l tratezi ca pe un coleg care „știe ce face” și să aprobi orice. E rapid și convingător, dar poate greși cu aceeași siguranță în glas. Ce poți face când se întâmplă e în [05-cand-ai-ul-greseste.md](../12-lucrul-cu-ai/05-cand-ai-ul-greseste.md).

## Pasul 1 — Instalează Claude Code în WSL

În Warp, în Ubuntu (nu în PowerShell):

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Comanda nu arată progres cât descarcă, așa că așteaptă să termine. Apoi deschide un **tab nou** și rulează:

```bash
claude --version
claude doctor
```

**Verifică.** `claude --version` afișează ceva de forma `2.x.y (Claude Code)`. `claude doctor` face un diagnostic al instalării și îți spune dacă e ceva de reparat.

Dacă primești `command not found: claude`, folderul `~/.local/bin` nu e în `PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Ce s-a întâmplat de fapt.** Installer-ul nativ a pus executabilul în `~/.local/share/claude/` și un link în `~/.local/bin/claude`. Instalarea nativă se actualizează singură în fundal, deci nu trebuie să faci nimic ca să ai ultima versiune.

## Pasul 2 — Login cu contul Pro

Fă-ți un folder de joacă, ca prima sesiune să nu fie pe un proiect real:

```bash
mkdir -p ~/code/sandbox-claude && cd ~/code/sandbox-claude
git init
claude
```

La prima pornire îți cere login. Alegi contul de **claude.ai** (abonamentul Pro).

- Dacă browserul nu se deschide singur, apasă `c` ca să copiezi link-ul, apoi deschide-l în browserul din Windows.
- Dacă după login browserul îți arată un **cod** în loc să te trimită înapoi, lipește-l în terminal la `Paste code here if prompted`. Se întâmplă des în WSL 2 și e normal.
- La final vezi `Login successful`. Apasă `Enter`.

**Verifică.** În sesiune, scrie `/status`. Trebuie să vezi contul tău și metoda de login.

**Ce s-a întâmplat de fapt.** Credențialele s-au salvat în `~/.claude/.credentials.json`, cu permisiuni doar pentru userul tău. Planul gratuit de pe claude.ai nu include Claude Code, dar Pro îl include. Abonamentul e același pentru chat-ul de pe claude.ai și pentru Claude Code, deci și limitele sunt comune.

## Pasul 3 — Prima sesiune

În `~/code/sandbox-claude`, cu `claude` pornit, încearcă pe rând:

```text
creează un fișier hello.js care afișează data de azi în format românesc
```

Claude îți arată ce vrea să scrie și îți cere voie. Citește, apoi aprobă.

```text
! node hello.js
```

`!` la începutul liniei rulează comanda direct în shell, fără să treacă prin Claude, dar rezultatul intră în conversație.

```text
ce fișiere am modificat? fă un commit cu un mesaj descriptiv
```

La final, ieși cu `/exit` sau cu `Ctrl+D` de două ori.

## Modurile de permisiuni

**Ce e.** Modul decide ce poate face Claude fără să te întrebe. Le schimbi cu `Shift+Tab` (pe Windows, dacă nu merge: `Alt+M`). Modul activ apare în bara de jos.

| Mod (în bara de jos) | Ce face fără să întrebe | Când îl folosești |
|---|---|---|
| `⏸ manual mode on` | Doar citește. Te întreabă la fiecare editare și comandă | Primele 2 săptămâni și orice lucru delicat |
| `⏵⏵ accept edits on` | Editează fișiere și rulează comenzi simple de fișiere (`mkdir`, `mv`) | Când urmărești pas cu pas un plan pe care l-ai aprobat |
| `⏸ plan mode on` | Citește și explorează, dar **nu editează**. Îți propune un plan | Înainte de orice feature. Pornești direct cu `/plan` |
| `⏵⏵ auto mode on` | Aproape tot, cu un model separat care blochează acțiunile riscante | Mai târziu, pe task-uri lungi pe care le înțelegi |

**De ce contează.** Versiunile noi de Claude Code pornesc implicit în **auto mode**. E comod, dar în primele săptămâni pierzi exact lecția importantă: să vezi fiecare comandă pe care o rulează agentul și să înțelegi de ce. Pornește în manual mode până când comenzile pe care le aprobi nu te mai surprind.

**Exemplu minim**: setezi modul de pornire și stilul de răspuns (explicat mai jos) pentru toate proiectele.

```bash
code ~/.claude/settings.json
```

Dacă fișierul e gol, lipește tot. Dacă are deja conținut, adaugă doar cheile lipsă în obiectul existent.

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "outputStyle": "Explanatory"
}
```

**Capcana.** `bypassPermissions` („Yes, don't ask again” la tot) pe laptopul tău. Documentația oficială îl recomandă doar în containere izolate. Nu-l folosi.

## Scurtături și comenzi pe care le folosești zilnic

| Ce apeși / scrii | Ce face |
|---|---|
| `Shift+Tab` (sau `Alt+M`) | Schimbi modul: manual → accept edits → plan → … |
| `Esc` | Oprești ce face Claude chiar acum. Ce a făcut până atunci rămâne |
| `Esc` `Esc` (cu input gol) | Meniul de rewind: întorci codul și/sau conversația la un mesaj anterior |
| `Ctrl+C` | Întrerupi, sau golești ce ai scris. Apăsat de două ori pe input gol, iese |
| `@cale/fișier` | Îi dai un fișier ca context, cu autocomplete |
| `!comandă` | Rulezi o comandă de shell, iar output-ul intră în conversație |
| `Shift+Enter` | Linie nouă în prompt (merge nativ în Warp și Windows Terminal) |
| `Ctrl+R` | Cauți în prompturile anterioare |
| `Alt+V` | Lipești o imagine (screenshot) din clipboard, în WSL |
| `Ctrl+G` | Deschizi prompt-ul (sau planul) într-un editor, pentru texte lungi |

| Comandă | Ce face |
|---|---|
| `/init` | Generează un `CLAUDE.md` pentru proiect (vezi [12/03](../12-lucrul-cu-ai/03-claude-md-si-context.md)) |
| `/plan descriere` | Intri direct în plan mode, cu task-ul dat |
| `/clear` | Conversație nouă, context gol. Folosește-l între task-uri |
| `/compact` | Rezumă conversația ca să elibereze context |
| `/context` | Vezi cât din context e ocupat și cu ce |
| `/rewind` | Același lucru ca `Esc Esc` |
| `/resume` | Reiei o conversație anterioară |
| `/model` | Schimbi modelul (Opus, Sonnet, Haiku) |
| `/usage` | Consumul din limitele planului |
| `/status` | Versiune, model, cont |
| `/permissions` | Reguli: ce e permis, ce e interzis |
| `/output-style` | Schimbi stilul de răspuns (mai jos) |
| `/btw întrebare` | O întrebare pe lângă, care nu intră în conversație |
| `/help` | Toate comenzile |

## Output styles: Explanatory și Learning

**Ce e.** Un output style schimbă felul în care Claude îți răspunde pe toată sesiunea. Două stiluri sunt făcute pentru tine:

- **Explanatory**: lucrează normal și adaugă blocuri scurte `★ Insight` care explică de ce a făcut alegerile din cod.
- **Learning**: aceleași explicații, plus că la deciziile interesante (o validare, o structură de date, o regulă de business) lasă un `TODO(human)` în fișier și te pune pe tine să scrii bucata aia. Apoi îți comentează codul.

**De ce contează.** E diferența dintre „Claude a construit, eu m-am uitat” și „am înțeles fiecare decizie”. Explanatory e default-ul bun pentru proiecte. Learning e exact ce-ți trebuie la pasul 6 din buclă, feature-ul scris de mână.

**Exemplu minim**

```text
/output-style learning
```

Fără argument, `/output-style` îți arată lista. Alegerea se salvează la nivel de proiect, în `.claude/settings.local.json`. Default-ul pentru toate proiectele l-ai pus mai sus, în `~/.claude/settings.json`.

**Capcana.** În `settings.json`, valoarea e sensibilă la majuscule: `"Explanatory"` merge, `"explanatory"` te lasă tacit pe stilul default. Comanda `/output-style` nu ține cont de majuscule.

## Planul Pro și limitele lui

**Ce e.** Pro are o **limită pe sesiune, pe ferestre de cinci ore**, și o **limită săptămânală**. Ambele sunt comune cu chat-ul de pe claude.ai. Anthropic nu publică un număr fix de mesaje, pentru că consumul depinde de cât context trimiți și de model.

**De ce contează.** Fiecare cerere trimite **toată conversația** de până atunci. O sesiune lungă și încărcată consumă mult mai repede decât una curată. Modelul implicit e Opus, cel mai capabil și cel mai scump. Documentația spune clar că Sonnet face bine majoritatea task-urilor de cod și costă mai puțin.

**Cum economisești**

- `/model sonnet` pentru construit zilnic. Alegerea rămâne default pentru sesiunile noi. Pentru task-uri grele de arhitectură comuți pe Opus. `opusplan` folosește Opus în plan mode și Sonnet la execuție, un compromis bun.
- **Un task pe sesiune.** Gata un feature: `/clear`. E gratuit, pe când `/compact` e el însuși o cerere mare.
- **Plan mode înainte de schimbări mari.** Refacerile după o direcție greșită costă de câteva ori mai mult decât planul.
- **Prompturi precise.** „Adaugă validare pe `amount` în `create-invoice.dto.ts`” e mult mai ieftin decât „îmbunătățește validarea”.
- **Nu lipi loguri de sute de linii.** Lipești doar eroarea și stack trace-ul relevant.
- **`CLAUDE.md` scurt.** Se încarcă la fiecare cerere.
- `/usage` îți arată barele de consum și ce anume a consumat cel mai mult.

**Ritmul tău.** Sesiunea de dimineață și cea de seară sunt la mai mult de cinci ore distanță, deci de obicei cad în ferestre diferite. Când dai de limită, Claude Code îți spune când se resetează (`Usage limit reached · limit resets …`) și poate continua singur task-ul după reset. Timpul dintre ele nu e pierdut: e exact timpul pentru code walk, adică să citești ce s-a construit și să-ți scrii `ARCHITECTURE.md`. Sau citești fișele zilei.

**Capcana.** „Usage credits” (consum peste limită, plătit separat) se activează din setările contului. Nu le porni din reflex: limita e un semnal că ai sesiuni prea încărcate, nu că ai nevoie de mai mult.

## Siguranța

**Ce e.** Claude Code rulează comenzi reale pe laptopul tău, cu drepturile tale. Plasa de siguranță ai construit-o deja: Git și modurile de permisiuni.

**Reguli**

- **Commit înainte de orice schimbare mare.** Rewind-ul (`Esc Esc`) întoarce doar fișierele editate prin uneltele lui Claude. Schimbările făcute prin comenzi (`rm`, scripturi) nu le urmărește. Git le acoperă pe toate.
- **Citește comanda înainte de „Yes”.** Te oprești și întrebi „de ce?” la: `rm -rf`, `git push --force`, `git reset --hard`, `sudo`, `curl … | bash`, `DROP TABLE`, `npm install` cu un pachet de care n-ai auzit, orice modificare în `.env`.
- **Nu-i da secrete în prompt.** Chei API, parole, connection string-uri: nu le lipești în conversație.
- **Interzice-i să citească `.env`.** În `~/.claude/settings.json` sau în proiect, în `.claude/settings.json`:

  ```json
  {
    "permissions": {
      "deny": ["Read(.env)", "Read(.env.*)"]
    }
  }
  ```

  E o barieră utilă, dar regula de bază rămâne: secretele stau în `.env`, care e în `.gitignore`. Detalii în [10-securitate-env](../10-securitate-env/01-env-vars.md).

<details>
<summary>Întrebare de interviu: Cum folosești AI-ul în munca ta de zi cu zi?</summary>

Folosesc Claude Code în terminal, cu un flux fix. Scriu eu cerințele. Îi cer un plan în plan mode și îl discut înainte de orice cod. Construim pe pași mici, cu un commit după fiecare pas care merge. Fac review la fiecare diff ca la codul unui coleg și verific cu teste sau rulând aplicația. La final trec prin cod până îl pot explica: dacă nu pot explica o bucată, nu intră în commit. AI-ul îmi scurtează execuția, dar deciziile și verificarea rămân la mine.
</details>

## Mai departe

- [Quickstart — Claude Code docs](https://code.claude.com/docs/en/quickstart)
- [Permission modes — Claude Code docs](https://code.claude.com/docs/en/permission-modes)
- [Best practices — Claude Code docs](https://code.claude.com/docs/en/best-practices)
