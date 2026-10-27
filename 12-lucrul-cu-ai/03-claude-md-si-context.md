# CLAUDE.md și contextul

Claude Code începe fiecare sesiune fără memorie. Două lucruri decid cât de bine lucrează: ce îi dai permanent (`CLAUDE.md`) și cât de curat ții ce se adună pe parcurs (contextul).

## Ce e CLAUDE.md

**Ce e.** Un fișier Markdown pe care Claude Code îl încarcă automat la începutul fiecărei sesiuni. Conține ce nu poate ghici citind codul: comenzile proiectului, convențiile, regulile de lucru cu tine.

**Unde poate sta**

| Fișier | Pentru ce | Intră în Git? |
|---|---|---|
| `./CLAUDE.md` (rădăcina proiectului) | Regulile proiectului: stack, comenzi, convenții | Da, e al echipei |
| `~/.claude/CLAUDE.md` | Preferințele tale, pentru toate proiectele | Nu, e pe laptopul tău |
| `./CLAUDE.local.md` | Preferințele tale doar pentru proiectul ăsta | Nu, îl pui în `.gitignore` |

Fișierele `CLAUDE.md` din folderele de deasupra celui în care pornești se încarcă la start. Cele din subfoldere se încarcă abia când Claude lucrează cu fișiere de acolo.

**Cum îl creezi.** `/init` analizează proiectul și generează un prim `CLAUDE.md`, pe care apoi îl tai și îl ajustezi. Pentru proiectele tale pornești de la template-ul din ghid: [templates/CLAUDE.template.md](../templates/CLAUDE.template.md). `/memory` îți deschide fișierele ca să le editezi.

**Capcana.** Să crezi că `CLAUDE.md` e o regulă pe care Claude o respectă garantat. E context, nu configurare impusă. Ce trebuie să se întâmple **mereu** (de exemplu „nu citi `.env`”) pui în permisiuni ([01-setup/05](../01-setup/05-claude-code.md#siguranța)), nu în text.

## Ce pui și ce nu pui

**De ce contează.** `CLAUDE.md` se încarcă la **fiecare** cerere. Unul lung costă limite pe Pro și, ce e mai grav, regulile importante se pierd în zgomot. Documentația oficială recomandă sub 200 de linii. Al tău de proiect ar trebui să aibă 40–80.

| Pui | Nu pui |
|---|---|
| Comenzile pe care Claude nu le poate ghici (`npm run typecheck`, cum pornești DB-ul local) | Ce poate afla citind codul |
| Reguli care diferă de default (bani în cenți, date în UTC) | Convenții standard pe care le știe oricum |
| Cum vrei să lucreze cu tine (plan întâi, pași mici, explicații în română) | Tutoriale sau documentație lungă (pui un link) |
| Ce scrii tu de mână („Built by hand”) | Descrierea fiecărui fișier |
| Capcanele proiectului care nu se văd din cod | Informații care se schimbă des |
| | Secrete. Niciodată. |

Testul pentru fiecare rând, din documentația oficială: **„Dacă scot rândul ăsta, va greși Claude?”** Dacă nu, îl scoți.

**Exemplu minim**: secțiunea de reguli specifice din `CLAUDE.md`-ul lui P2 Invoicer.

```markdown
## Conventions

- Money is stored as integer cents (`amountCents: number`). Never use floats for money.
- Invoice totals are always computed on the server from line items. Ignore totals sent by the client.
- Dates are stored in UTC and shown in Europe/Bucharest.
- Every new endpoint gets a DTO with class-validator rules and an e2e test.

## Built by hand

"Overdue" status calculation — I write this one myself. Review only.
```

## @-mentions și import-uri

**Ce e.** În prompt, `@src/invoices/invoices.service.ts` îi spune lui Claude să citească exact fișierul ăla (cu autocomplete pe `Tab`). În `CLAUDE.md`, `@README.md` **importă** fișierul, adică îl încarcă în fiecare sesiune.

**De ce contează.** Cu `@` în prompt economisești context: Claude nu mai caută prin tot proiectul ca să găsească fișierul despre care vorbești.

**Capcana.** Import-uri în `CLAUDE.md` „ca să știe tot”: un README de 300 de linii importat înseamnă 300 de linii în plus la fiecare cerere. Ca să menționezi o cale fără s-o imporți, o pui între backticks.

## Contextul e resursa principală

**Ce e.** Contextul (context window) e tot ce vede modelul acum: `CLAUDE.md`, mesajele tale, fișierele citite, output-ul fiecărei comenzi. Se umple repede. O singură sesiune de debugging poate aduna zeci de mii de tokeni.

**De ce contează.**

- **Calitatea scade când contextul se umple.** Claude „uită” instrucțiunile de la început și greșește mai des.
- **Costul crește.** Fiecare mesaj retrimite tot contextul, deci o întrebare de un rând într-o sesiune de trei ore consumă cât toată sesiunea.

**Exemplu minim**

```text
/context
```

Vezi un grid colorat cu ce ocupă loc și sugestii de optimizare.

**Capcana.** Să lași sesiunea deschisă toată ziua „ca să știe tot ce am făcut”. Ce trebuie să știe pe termen lung e în cod, în `CLAUDE.md` și în `ARCHITECTURE.md`, nu în istoricul conversației.

## `/clear` vs `/compact` vs rewind

| Comandă | Ce face | Când |
|---|---|---|
| `/clear` | Conversație nouă, context gol. Nu costă nimic | Ai terminat un task. Începi altceva. L-ai corectat de două ori degeaba |
| `/compact [instrucțiuni]` | Rezumă conversația și continuă. E el însuși o cerere mare | Ești la jumătatea unui task lung și vrei să continui pe același fir |
| `Esc Esc` / `/rewind` → „Summarize from here” | Rezumă doar de la un mesaj încolo | Un debugging lung pe care vrei să-l comprimi, păstrând începutul |
| `/btw întrebare` | Răspunde fără să adauge în conversație | O întrebare pe lângă |

`/clear` nu pierde munca: codul e pe disc și în Git. Dacă ai nevoie de conversația veche, `/resume` o aduce înapoi.

**Capcana.** `/compact` folosit în loc de `/clear` din frică să nu se piardă ceva. Dacă task-ul e gata, `/clear` e mai ieftin și mai curat.

<details>
<summary>Întrebare de interviu: Ce e context window-ul unui LLM și de ce contează când construiești cu AI?</summary>

E cantitatea maximă de text, măsurată în tokeni, pe care modelul o poate lua în calcul într-o cerere: instrucțiunile, istoricul conversației, documentele și rezultatele uneltelor. Contează din trei motive. Totul trebuie să încapă, deci nu poți trimite o bază de cod întreagă, și de aici apare RAG-ul. Costul și latența cresc cu fiecare token trimis. Iar calitatea scade când contextul e plin de informație irelevantă. În practică, dai modelului doar contextul relevant pentru task-ul curent și începi sesiuni noi pentru task-uri noi.
</details>

## Mai departe

- [How Claude remembers your project — Claude Code docs](https://code.claude.com/docs/en/memory)
- [Manage costs effectively — Claude Code docs](https://code.claude.com/docs/en/costs)
