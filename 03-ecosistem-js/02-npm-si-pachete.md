# npm și pachetele

Aproape tot codul dintr-un proiect modern e scris de alții și vine din npm. Trebuie să știi ce instalezi, unde ajunge, cum se fixează versiunile și ce riscuri îți asumi.

## Ce e un pachet

**Ce e.** Un **pachet** e un folder de cod cu un `package.json`, publicat într-un **registry** ca să-l poată instala oricine. Când cineva zice „o librărie npm” (o „npm library”), se referă la un pachet pe care îl imporți în cod: `react`, `zod`, `@tanstack/react-query`. Alte pachete sunt programe de linie de comandă (CLI-uri): `vite`, `prisma`, `@nestjs/cli`.

„npm” înseamnă trei lucruri:

1. **registry-ul** (npmjs.com), depozitul public cu milioane de pachete;
2. **CLI-ul** `npm`, programul care vine cu Node și le instalează;
3. firma care le administrează, acum parte din GitHub.

Comparat cu C: e ca o bibliotecă pe care o legi de program, doar că descărcarea, versiunile și dependențele dependențelor se rezolvă automat.

**De ce contează.** Fiecare `npm install` aduce cod străin pe calculatorul tău și în aplicația ta. Trebuie să știi ce faci.

**Exemplu minim**

```bash
npm install zod     # downloads zod into node_modules/ and adds it to package.json
```

```ts
import { z } from "zod"; // a bare name → Node and Vite look it up in node_modules/
```

<details>
<summary>Întrebare de interviu: Ce face `npm install`?</summary>

Fără argumente, citește `package.json` și `package-lock.json`, descarcă din registry toate dependențele, inclusiv dependențele lor, la versiunile din lockfile, și le pune în `node_modules`. Cu un nume de pachet, îl adaugă în `package.json`, îl instalează și actualizează lockfile-ul. Poate rula și scripturi de instalare ale pachetelor, motiv pentru care contează ce instalezi.
</details>

## package.json câmp cu câmp

**Ce e.** Fișa de identitate a proiectului. Așa arată cea generată de Vite pentru un proiect React + TypeScript (versiunile vor fi altele când îl generezi tu):

```json
{
  "name": "vite-react-typescript-starter",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "oxlint",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.3.1",
    "react-dom": "^19.3.1"
  },
  "devDependencies": {
    "@types/node": "^24.19.2",
    "@types/react": "^19.3.0",
    "@types/react-dom": "^19.3.0",
    "@vitejs/plugin-react": "^6.1.2",
    "oxlint": "^1.88.0",
    "typescript": "~6.0.3",
    "vite": "^8.3.4"
  }
}
```

| Câmp | Ce înseamnă |
|---|---|
| `name`, `version` | identitatea pachetului, obligatorii doar dacă îl publici |
| `private: true` | npm refuză să-l publice din greșeală |
| `type: "module"` | fișierele `.js` sunt ES modules (`import`/`export`), nu CommonJS (`require`). Vezi [04-javascript/04-module.md](../04-javascript/04-module.md) |
| `scripts` | comenzi rulate cu `npm run <nume>` |
| `dependencies` | ce trebuie ca aplicația să **ruleze** |
| `devDependencies` | ce trebuie doar ca să **dezvolți, să construiești și să testezi** |
| `@types/...` | doar tipuri TypeScript pentru pachete scrise în JavaScript |
| `engines` (opțional) | ce versiune de Node cere proiectul |

**De ce contează.** Când intri într-un proiect nou la job, `package.json` e primul fișier pe care-l citești. În două minute știi stack-ul și comenzile.

**Capcana.** Să editezi versiunile de mână în `package.json` și să nu rulezi `npm install`. `package.json` și lockfile-ul ajung să spună lucruri diferite, iar `npm ci` din CI pică.

## dependencies vs devDependencies

**Ce e.** `npm install pachet` pune pachetul în `dependencies`. `npm install -D pachet` îl pune în `devDependencies`.

- **dependencies**: codul de care aplicația are nevoie când rulează (`react`, `@nestjs/core`, `@prisma/client`, `zod`);
- **devDependencies**: uneltele (`typescript`, `vite`, `oxlint`/`eslint`, `prettier`, `vitest`, `@types/*`).

**De ce contează.** Pe frontend, distincția e mai mult o convenție, pentru că Vite pune în bundle doar ce imporți efectiv. Pe **backend contează pe bune**: la deploy se instalează de obicei doar `dependencies` (`npm ci --omit=dev`). Un pachet folosit la runtime, dar pus greșit în `devDependencies`, înseamnă că serverul crapă în producție cu `Cannot find module`.

**Exemplu minim**

```bash
npm install @tanstack/react-query   # runtime → dependencies
npm install -D prettier             # tool → devDependencies
```

<details>
<summary>Întrebare de interviu: Care e diferența dintre dependencies și devDependencies?</summary>

`dependencies` sunt pachetele de care aplicația are nevoie ca să ruleze. `devDependencies` sunt cele necesare doar în dezvoltare: compilatorul TypeScript, bundler-ul, linter-ul, test runner-ul, tipurile. Diferența contează mai ales pe backend și la librăriile publicate: într-o instalare de producție, `devDependencies` se omit, deci tot ce se folosește la runtime trebuie să fie în `dependencies`.
</details>

## node_modules și package-lock.json

**Ce e.**

- **`node_modules/`** e folderul cu codul instalat: pachetele tale plus dependențele lor (dependențe _tranzitive_). E uriaș și **nu intră niciodată în git**, pentru că se reface oricând.
- **`package-lock.json`** fixează **versiunea exactă** a fiecărui pachet din tot arborele, plus un hash de integritate. **Intră în git**, ca toată lumea (tu, colegii, CI-ul, serverul de deploy) să instaleze exact același cod.

`npm install` instalează și poate actualiza lockfile-ul în limitele din `package.json`. `npm ci` (clean install) șterge `node_modules` și instalează **exact** ce scrie în lockfile. Dacă lockfile-ul nu corespunde cu `package.json`, pică. E comanda pentru CI și deploy.

**De ce contează.** Fără lockfile, două instalări la o săptămână distanță pot aduce versiuni diferite. Așa apare „la mine merge, pe server nu”.

**Exemplu minim**

```bash
du -sh node_modules        # how big is it?
npm ls react               # which version of react is installed, and who requires it
npm ls --all | wc -l       # how many packages are really in the tree
```

**Capcana.** Ștergi lockfile-ul „ca să rezolvi o problemă” și, fără să observi, actualizezi zeci de pachete. La conflicte de merge în lockfile: rezolvi `package.json`, apoi rulezi `npm install`, ca lockfile-ul să se regenereze corect. Nu-l editezi de mână.

<details>
<summary>Întrebare de interviu: Ce e package-lock.json și trebuie pus în git?</summary>

E lista cu versiunea exactă și hash-ul de integritate pentru fiecare pachet din arborele de dependențe, inclusiv cele indirecte. Da, trebuie pus în git, ca toți developerii, CI-ul și serverul să instaleze identic. În CI și la deploy se folosește `npm ci`, care respectă strict lockfile-ul. `node_modules` nu se pune în git, pentru că se poate reface oricând din cele două fișiere.
</details>

## Semver: ^ și ~

**Ce e.** Versiunile de pachete urmează **semver** (semantic versioning): `MAJOR.MINOR.PATCH`.

- **PATCH** (`6.0.3` → `6.0.4`): reparații, nimic nou;
- **MINOR** (`6.0` → `6.1`): funcționalități noi, compatibile cu codul vechi;
- **MAJOR** (`6` → `7`): schimbări care pot strica codul existent.

Prefixele din `package.json`:

| Scris | Acceptă | Exemplu |
|---|---|---|
| `"^19.3.1"` | orice `19.x.x` ≥ 19.3.1 (același major) | `19.4.0` da, `20.0.0` nu |
| `"~6.0.3"` | orice `6.0.x` ≥ 6.0.3 (același minor) | `6.0.9` da, `6.1.0` nu |
| `"6.0.3"` | doar exact versiunea asta | |

Pentru pachetele încă în `0.x`, `^0.4.2` acceptă doar `0.4.x`, pentru că în faza `0` și minorul poate strica lucruri.

**De ce contează.** În șablonul Vite de mai sus, `typescript` are `~`, nu `^`. Fiecare versiune minoră de TypeScript poate aduce verificări noi care îți „strică” build-ul, așa că șablonul acceptă doar patch-uri.

**Exemplu minim**

```bash
npm outdated                 # what has newer versions
npm update                   # update within the allowed ranges
npm install zod@latest       # jump to the latest version (maybe a new major)
```

**Capcana.** Semver e o promisiune, nu o garanție. Și un „minor” poate strica ceva. De asta există lockfile-ul, și de asta faci update-urile mari separat, pe branch-ul lor, cu testele rulate.

<details>
<summary>Întrebare de interviu: Ce înseamnă ^ în fața unei versiuni din package.json?</summary>

Acceptă orice versiune compatibilă cu aceeași versiune majoră: `^4.2.0` înseamnă orice ≥ 4.2.0 și < 5.0.0. Tilda (`~`) acceptă doar patch-uri în același minor. Versiunea efectiv instalată e fixată în lockfile. Range-ul contează doar când cineva actualizează dependențele.
</details>

## npm scripts și npx

**Ce e.**

- **Scripts**: comenzile din `package.json`, rulate cu `npm run dev`, `npm run build`. `npm test` și `npm start` merg și fără `run`. În timpul unui script, binarele din `node_modules/.bin` sunt pe PATH, așa că scriptul poate chema `vite` sau `nest` fără instalare globală.
- **`npx`** rulează binarul unui pachet fără să-l instalezi global (`npx prisma migrate dev`, `npx tsx script.ts`). Dacă pachetul e instalat local, folosește versiunea locală. Dacă nu, îl descarcă temporar. `npm create vite@latest` e prescurtarea pentru `npx create-vite@latest`.

**De ce contează.** Așa rulează toată echipa aceleași comenzi cu aceleași versiuni de unelte.

**Cum îl vezi în cod real.** Scripturile unui proiect NestJS nou, generat cu `nest new`:

```json
"scripts": {
  "build": "nest build",
  "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
  "start": "nest start",
  "start:dev": "nest start --watch",
  "start:prod": "node dist/main",
  "lint": "oxlint src/ test/",
  "test": "vitest run",
  "test:watch": "vitest",
  "test:e2e": "vitest run --config ./vitest.config.e2e.ts"
}
```

`npm run start:dev` pornește serverul cu restart automat la fiecare salvare, `npm test` rulează testele o dată. În proiecte NestJS mai vechi vei vedea `jest` în loc de `vitest` și `eslint` în loc de `oxlint`. Rolurile sunt aceleași.

**Capcana.** `npm install -g` pentru uneltele unui proiect. Fiecare om din echipă ajunge cu altă versiune globală. Le pui ca `devDependencies` și le rulezi prin scripts sau `npx`. Și nu rulezi niciodată `sudo npm install -g`: cu `fnm` nu ai nevoie de `sudo`.

<details>
<summary>Întrebare de interviu: Care e diferența dintre npm și npx?</summary>

`npm` instalează și administrează pachete și rulează scripturile din `package.json`. `npx` execută comanda unui pachet: dacă e instalat local, folosește versiunea locală, altfel îl descarcă temporar. E bun pentru unelte de o singură folosire, ca generatoarele de proiecte (`npx create-vite`), sau pentru CLI-urile proiectului fără instalare globală.
</details>

## Cum alegi un pachet

**Ce e.** Înainte de `npm install`, mai ales când pachetul ți l-a sugerat AI-ul, verifică pe npmjs.com și pe GitHub:

- [ ] **numele e exact cel corect** (vezi capcana de mai jos);
- [ ] **descărcări săptămânale**: mii sau milioane, nu zeci;
- [ ] **ultima versiune publicată**: luni, nu ani;
- [ ] **are repo pe GitHub** legat de pagina npm, cu issues la care se răspunde;
- [ ] **are tipuri TypeScript** (badge-ul „TS” pe npmjs.com) sau un pachet `@types/...`;
- [ ] **licență** permisivă (MIT, Apache-2.0, ISC);
- [ ] **puține dependențe** și dimensiune rezonabilă (bundlephobia.com îți arată cât adaugă în bundle).

Și întrebarea cea mai importantă: **chiar am nevoie de el?** Platforma are deja `fetch` (în loc de `axios`), `Intl.NumberFormat` (formatare de bani), `structuredClone` (în loc de `lodash.cloneDeep`), `crypto.randomUUID()`.

**Exemplu minim**

```bash
npm view zod                 # description, latest version, dependencies, maintainers
npm view zod versions --json # every published version
```

## Riscuri: supply chain și pachete halucinate

**Ce e.** Un pachet npm poate rula cod pe calculatorul tău la instalare (scripturi `postinstall`) și ajunge în aplicația ta. Asta e **supply chain risk**: ești atât de sigur cât e cea mai slabă dependență din arbore. Cazuri reale:

- **septembrie 2025, chalk și debug.** Un maintainer al unor pachete foarte populare (`chalk`, `debug` și altele, cu miliarde de descărcări pe săptămână în total) a fost păcălit cu un email de phishing. Atacatorii au publicat versiuni cu cod care fura criptomonede. Au fost retrase în câteva ore, dar au apucat să fie instalate.
- **septembrie 2025, Shai-Hulud.** Primul „vierme” din npm: un pachet compromis fura token-urile de npm ale developerului care îl instala și publica automat versiuni infectate ale pachetelor acelui developer. Au fost afectate sute de pachete.
- **typosquatting**: pachete cu nume aproape identice cu cele populare (`reacct`, `lodahs`).
- **slopsquatting**: modelele AI inventează uneori nume de pachete care nu există. Atacatorii observă numele inventate des și publică pachete malițioase exact cu acele nume.

**De ce contează.** Lucrezi cu AI, deci vei primi sugestii de pachete. Unele vor fi inventate.

**Cum te aperi**

1. Verifici că pachetul există, e cel corect și e popular **înainte** de `npm install`.
2. Lockfile în git + `npm ci` în CI și la deploy.
3. `npm audit` din când în când, plus Dependabot pe GitHub (vezi [09-git-github-echipa/05-conventii-si-ci.md](../09-git-github-echipa/05-conventii-si-ci.md)).
4. Cât mai puține dependențe.
5. Nu rulezi comenzi `npx ceva` din surse necunoscute.
6. Token-urile de npm și GitHub nu stau în fișiere din proiect.

**Capcana.** Claude îți spune „instalează `react-invoice-pdf-generator`”. Pachetul poate să nu existe, să fie abandonat sau să fie o capcană. Verifici pe npmjs.com, apoi instalezi.

<details>
<summary>Întrebare de interviu: Cum te protejezi de un pachet npm malițios?</summary>

Verific pachetul înainte să-l instalez: numele exact, popularitatea, cine îl întreține, când a fost actualizat. Țin lockfile-ul în git și folosesc `npm ci` în CI, ca să nu intre versiuni noi neverificate. Rulez `npm audit` și am Dependabot pentru vulnerabilități cunoscute. Limitez numărul de dependențe și prefer API-urile platformei. Cu AI am grijă în plus, pentru că poate sugera pachete inexistente, pe care atacatorii le înregistrează.
</details>

## Mai departe

- [npm Docs — package.json](https://docs.npmjs.com/cli/configuring-npm/package-json)
- [npm Docs — About semantic versioning](https://docs.npmjs.com/about-semantic-versioning)
- [npm Docs — npm ci](https://docs.npmjs.com/cli/commands/npm-ci)
