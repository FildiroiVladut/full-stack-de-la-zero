# Convenții și CI

Convențiile fac istoria și branch-urile ușor de citit pentru oricine din echipă. CI-ul (continuous integration) e un robot care rulează automat lint, typecheck și teste la fiecare PR. Împreună sunt motivul pentru care o echipă de 10 oameni poate face merge de mai multe ori pe zi fără să strice `main`.

## Conventional commits

**Ce e.** Un format standard pentru mesajele de commit: `tip(zonă): descriere`, la prezent, cu literă mică.

| Tip | Când |
|---|---|
| `feat` | funcționalitate nouă |
| `fix` | reparație de bug |
| `refactor` | schimbare de structură, fără schimbare de comportament |
| `test` | adaugi sau modifici teste |
| `docs` | documentație, README |
| `chore` | întreținere: dependențe, configurări |
| `ci` | workflow-uri de CI |
| `perf` | optimizare de performanță |

**Exemplu minim**

```bash
git commit -m "feat(bookings): reject overlapping bookings"
git commit -m "fix(invoices): round VAT per line"
git commit -m "chore: upgrade NestJS"
git commit -m "feat(api)!: rename /bills to /invoices"   # ! = breaking change
```

**De ce contează.** Istoria se citește ca un changelog. Unele unelte generează automat versiuni și note de release din aceste mesaje. Majoritatea echipelor folosesc formatul ăsta sau ceva foarte apropiat.

**Capcana.** `fix: stuff`, `feat: changes`. Formatul e respectat, dar descrierea nu spune nimic. Descrierea trebuie să completeze propoziția: „Dacă aplic commit-ul ăsta, el va…”.

## Nume de branch-uri

**Ce e.** Același tip ca la commit, plus numărul issue-ului și o descriere scurtă. Litere mici, cuvinte separate cu cratimă.

```
feat/12-book-search
fix/31-invoice-total-rounding
chore/upgrade-nest
refactor/extract-pricing-service
```

**De ce contează.** Din lista de branch-uri înțelegi imediat cine lucrează la ce. Numărul issue-ului leagă automat contextul.

## Protejarea lui main cu rulesets

**Ce e.** Un ruleset e un set de reguli pe care GitHub le aplică unui branch: nimeni nu face push direct, totul trece prin PR, CI-ul trebuie să fie verde. Pe repo-urile **publice** sunt disponibile gratuit. Pe cele private îți trebuie un plan plătit.

**Pașii** (pe repo-ul P3, după ce ai pus CI-ul):

1. Repo → **Settings** → **Rules** → **Rulesets** → **New ruleset** → **New branch ruleset**.
2. Dă-i un nume, de exemplu `protect-main`. Pune **Enforcement status** pe *Active*.
3. La **Target branches** alege *Include default branch*.
4. Bifează regulile:
   - **Restrict deletions**: nimeni nu poate șterge `main`;
   - **Block force pushes**: nimeni nu poate rescrie istoria lui `main`;
   - **Require a pull request before merging**: lasă *Required approvals* pe **0**, pentru că pe un repo personal nu-ți poți aproba singur PR-urile și te-ai bloca. Review-ul lui Bogdan rămâne o regulă a voastră, nu una impusă de GitHub;
   - **Require status checks to pass before merging**: adaugă job-ul de CI, de exemplu `check`. Apare în listă după ce a rulat măcar o dată.
5. **Save changes**.

Tot din Settings → **General**, la *Pull Requests*:

- lasă bifat doar **Allow squash merging**;
- bifează **Automatically delete head branches**.

**Capcana.** Să te adaugi pe tine în *Bypass list* „pentru urgențe”. O să folosești bypass-ul pentru fiecare „mică schimbare”, iar exercițiul nu mai are niciun sens.

## GitHub Actions: CI-ul tău

**Ce e.** GitHub Actions rulează scripturi pe o mașină virtuală Linux la fiecare eveniment ales de tine: un PR deschis, un push pe `main` etc. Pentru repo-uri publice, pe runner-ele standard, e gratuit.

Vocabularul:

- **workflow**: un fișier YAML din `.github/workflows/`;
- **trigger** (`on`): când rulează;
- **job**: un set de pași care rulează pe o mașină (**runner**);
- **step**: fie o acțiune gata făcută (`uses:`), fie o comandă (`run:`).

**Exemplu: workflow complet** (`.github/workflows/ci.yml`)

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read           # least privilege: CI only needs to read the code

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - uses: actions/setup-node@v7
        with:
          node-version: lts/*    # or: node-version-file: .nvmrc
          cache: npm

      - run: npm ci              # exact install from the lockfile
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
```

Scripturile din `package.json` pe care le cheamă workflow-ul:

```json
{
  "scripts": {
    "lint": "oxlint",
    "typecheck": "tsc --noEmit",
    "test": "vitest run"
  }
}
```

(Template-urile curente de Vite și NestJS vin cu oxlint și Vitest. În proiectele mai vechi vei vedea `eslint .` și `jest`, iar la Next.js pui scriptul de lint pe care îl generează proiectul. Ideea e aceeași: CI-ul rulează exact comenzile pe care le rulezi și tu local.)

**De ce `npm ci` și nu `npm install`.** `npm ci` șterge `node_modules`, instalează exact ce scrie în lockfile și pică dacă `package.json` și lockfile-ul nu se potrivesc. E reproductibil, exact ce vrei în CI.

**De ce contează.** Fără CI, „am uitat să rulez testele” ajunge pe `main`. Cu CI și cu ruleset-ul de mai sus, codul care nu compilează sau care strică testele nici nu poate intra.

**Capcana.** Un CI care rulează alte comenzi decât cele locale, sau un `continue-on-error: true` pus ca să treacă „temporar”. Dacă CI-ul e roșu, ceva e stricat. Îl repari, nu-l ocolești.

<details>
<summary>Întrebare de interviu: Ce e CI și ce ai pune într-un pipeline pentru un proiect Node?</summary>

Continuous integration înseamnă că fiecare schimbare e verificată automat înainte de merge. Pentru un proiect Node/TypeScript pun install curat din lockfile cu `npm ci`, lint, typecheck cu `tsc --noEmit`, teste și, eventual, build-ul. Pipeline-ul rulează la fiecare PR și e obligatoriu prin regula de branch protection, deci pe `main` nu ajunge cod care nu compilează sau pică testele.
</details>

## Când pică CI-ul

**Ce e.** Un X roșu pe PR. Primul pas e întotdeauna să citești logul, nu să ghicești.

```bash
gh pr checks                 # which check failed
gh run list --limit 5        # recent workflow runs
gh run view --log-failed     # only the logs of the failed steps
```

Apoi reproduci local cu exact aceeași comandă, de exemplu `npm run typecheck`.

**Cauze frecvente de „la mine merge”:**

- **dependență instalată doar local:** ai uitat `npm install pachet`, așa că lipsește din `package.json`;
- **lockfile neactualizat:** `npm ci` pică dacă nu se potrivește cu `package.json`;
- **litere mari și mici în nume de fișiere:** `import { BookCard } from './bookCard'` când fișierul e `BookCard.tsx`. Pe Linux, inclusiv în CI, sunt fișiere diferite. Pe Windows nu sunt, de aceea lucrezi din WSL;
- **variabile de mediu** care există în `.env`-ul tău, dar nu și în CI.

**Cu Claude Code.** _„CI-ul a picat. Rulează `gh run view --log-failed`, explică-mi cauza și propune fixul. Nu modifica nimic până nu confirm.”_

## Dependabot pe scurt

**Ce e.** Un bot GitHub care te anunță când o dependență are o vulnerabilitate cunoscută (*Dependabot alerts*). Dacă vrei, deschide automat PR-uri cu versiuni noi (*version updates*).

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

**De ce contează.** Dependențele vechi sunt una dintre cele mai frecvente surse de vulnerabilități (vezi OWASP în [10-securitate-env/04-validare-si-atacuri.md](../10-securitate-env/04-validare-si-atacuri.md)). PR-urile Dependabot trec prin același CI ca ale tale.

**Capcana.** Să dai merge orbește la un PR Dependabot care urcă o versiune majoră (de exemplu `5.x → 6.x`). Versiunile majore pot schimba API-ul. Citești changelog-ul, iar CI-ul te ajută, dar nu ține locul unei verificări manuale.

## Mai departe

- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/): specificația completă, scurtă
- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) și [Building and testing Node.js](https://docs.github.com/en/actions/tutorials/build-and-test-code/nodejs): GitHub Docs
- [Dependabot quickstart](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide)
