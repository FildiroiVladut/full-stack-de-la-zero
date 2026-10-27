# npm vs Bun, pe propria piele

Fișier hands-on. Creezi același proiect de două ori, o dată cu npm și o dată cu Bun, cronometrezi, compari și înțelegi ce face fiecare. Ai nevoie de Node (prin fnm) și de Bun instalate (vezi [01-setup/03-node-npm-bun.md](../01-setup/03-node-npm-bun.md)). Durează cam o oră.

## Package manager, bundler, runtime: trei cuvinte care se încurcă

**Ce e.** O să auzi „packager” folosit pentru lucruri diferite. Pune-le în ordine:

| Rol | Ce face | Exemple |
|---|---|---|
| **runtime** | rulează codul JavaScript | Node, Bun, Deno, browserul |
| **package manager** | descarcă și administrează dependențele (`node_modules`, lockfile) | npm, Bun (`bun install`), pnpm, Yarn |
| **bundler** | împachetează codul tău într-un set mic de fișiere pentru browser | Vite (cu Rolldown), webpack, esbuild, Bun (`bun build`) |
| **type checker** | verifică tipurile TypeScript | `tsc` |
| **test runner** | rulează testele | Vitest, Jest, Bun (`bun test`) |

Când cineva zice „packager”, de obicei se referă la un **package manager**, uneori la un **bundler**. Întreabă-l la care se referă.

**De ce contează.** npm e **doar** package manager. Bun e **runtime + package manager + bundler + test runner** într-un singur program. De aici vine toată confuzia „npm vs Bun”: compari un singur rol cu patru.

<details>
<summary>Întrebare de interviu: Care e diferența dintre npm și Bun?</summary>

npm e package manager-ul care vine cu Node: instalează pachete și rulează scripturi, iar codul îl rulează Node. Bun e un runtime separat, alternativ la Node, care are încorporat un package manager mult mai rapid, plus bundler și test runner. Le poți combina: Bun doar ca package manager, cu codul rulat tot de Node. Diferența practică e viteza de instalare. Diferența strategică e compatibilitatea: Node + npm sunt standardul pe care îl suportă toate framework-urile și hosting-urile.
</details>

## Pașii

### 1. Pregătire

```bash
node --version && npm --version && bun --version   # all three must print a version
mkdir -p ~/code/playground && cd ~/code/playground
```

**Verifică.** Ești în home-ul Linux (`pwd` arată `/home/<user>/code/playground`), nu pe `/mnt/c/...`.

### 2. Creează proiectul cu npm

```bash
npm create vite@latest shelf-npm -- --template react-ts
```

Dublul `--` e necesar: tot ce urmează după el ajunge la `create-vite`, nu la npm. Dacă te întreabă dacă vrei să instaleze dependențele și să pornească acum, răspunde **No**. Vrem să cronometrăm noi instalarea.

### 3. Creează același proiect cu Bun

```bash
bun create vite shelf-bun --template react-ts
```

La Bun nu e nevoie de `--`. Dacă te întreabă de instalare, tot **No**.

**Verifică.** `ls shelf-npm shelf-bun` arată aceleași fișiere în ambele foldere. Șablonul e identic, s-a schimbat doar programul care l-a descărcat.

### 4. Instalare „la rece” (cache gol)

Ambele unelte țin un cache global de pachete. Ca să compari corect, îl golești întâi:

```bash
npm cache clean --force
bun pm cache rm

cd ~/code/playground/shelf-npm && time npm install
cd ~/code/playground/shelf-bun && time bun install
```

Notează rândul `real` din output-ul lui `time`. Ăla e timpul efectiv.

### 5. Instalare „la cald” (cache plin, `node_modules` șters)

Așa arată reinstalarea în viața de zi cu zi:

```bash
cd ~/code/playground/shelf-npm && rm -rf node_modules && time npm install
cd ~/code/playground/shelf-bun && rm -rf node_modules && time bun install
```

Completează tabelul în jurnal:

| | npm | bun |
|---|---|---|
| instalare la rece | … s | … s |
| instalare la cald | … s | … s |
| lockfile | `package-lock.json`, … linii | `bun.lock`, … linii |

Numărul de linii îl afli cu `wc -l package-lock.json` și `wc -l bun.lock`.

### 6. Compară lockfile-urile

```bash
head -30 ~/code/playground/shelf-npm/package-lock.json
head -30 ~/code/playground/shelf-bun/bun.lock
```

Ce observi:

- ambele fixează versiunea exactă a fiecărui pachet din arbore;
- `package-lock.json` e JSON, `bun.lock` e un format text asemănător, mai compact;
- **un proiect folosește un singur package manager.** Dacă ai ambele lockfile-uri în repo, CI-ul și hosting-ul nu mai știu pe care să-l creadă. Vercel, de exemplu, alege package manager-ul după lockfile-ul pe care îl găsește.

### 7. Pornește serverul de dezvoltare

```bash
cd ~/code/playground/shelf-npm && npm run dev    # Ctrl+C to stop
cd ~/code/playground/shelf-bun && bun run dev    # Ctrl+C to stop
```

Deschide `http://localhost:5173` în browserul din Windows. E aceeași aplicație.

Surpriza: **în ambele cazuri, Vite rulează cu Node.** `bun run dev` doar pornește scriptul. CLI-ul `vite` declară că vrea Node, iar Bun respectă asta. Ca să rulezi Vite chiar cu runtime-ul Bun:

```bash
bun run --bun dev
```

Concluzia: **să folosești Bun ca package manager nu înseamnă că folosești Bun ca runtime.**

### 8. Bun ca runtime și ca test runner

```bash
cd ~/code/playground && mkdir bun-extras && cd bun-extras
```

Creează `money.ts`:

```ts
// money.ts — amounts are integer cents to avoid floating point errors
export function sumCents(lines: number[]): number {
  return lines.reduce((total, cents) => total + cents, 0);
}

console.log(sumCents([1999, 1])); // 2000
```

Și `money.test.ts`:

```ts
import { expect, test } from "bun:test";
import { sumCents } from "./money.ts";

test("sums invoice lines in cents", () => {
  expect(sumCents([1999, 1])).toBe(2000);
});
```

Rulează:

```bash
bun money.ts     # Bun runs TypeScript directly
node money.ts    # recent Node versions also run simple .ts files (types are stripped, not checked)
bun test         # finds *.test.ts files and runs them
```

**Verifică.** `bun test` afișează testul trecut, cu verde. Dacă `node money.ts` afișează un warning despre „module type”, e inofensiv: Node îți sugerează să pui `"type": "module"` într-un `package.json`.

### 9. Curățenie

```bash
rm -rf ~/code/playground/shelf-npm/node_modules ~/code/playground/shelf-bun/node_modules
```

Folderul `playground` îl poți păstra pentru experimente.

## Ce s-a întâmplat de fapt

- **Același registry, aceleași pachete.** `bun install` descarcă din npm aceleași versiuni și construiește un `node_modules` compatibil. Codul tău nu știe cu ce a fost instalat.
- **De ce e Bun mai rapid.** E scris în cod nativ, descarcă în paralel agresiv și folosește un cache global din care „copiază” fișierele aproape instant. npm e scris în JavaScript și are ani de compatibilitate de cărat după el.
- **Securitate.** `bun install` nu rulează implicit scripturile `postinstall` ale dependențelor, cu excepția unei liste de pachete populare și de încredere. npm le rulează. Asta face Bun mai prudent împotriva pachetelor malițioase (vezi [02-npm-si-pachete.md](02-npm-si-pachete.md)).
- **Migrare.** Dacă rulezi `bun install` într-un proiect care are doar `package-lock.json`, Bun îl convertește automat în `bun.lock`.

## De ce rămânem pe npm în ghid

1. **Joburile.** Majoritatea repo-urilor din companii folosesc npm sau pnpm. Trebuie să le stăpânești.
2. **NestJS.** E dezvoltat și testat pe Node + npm. Documentația, erorile de pe Stack Overflow și ce generează Claude presupun asta.
3. **Hosting-ul.** Render și Vercel merg perfect cu npm fără nicio configurare.
4. **Mai puține surprize.** Când înveți, vrei ca o eroare să aibă o singură cauză posibilă, nu două.

Bun e perfect pentru scripturi, experimente și proiecte personale. Dacă îl folosești într-un proiect, îl folosești până la capăt (lockfile, CI, deploy), nu amestecat cu npm.

## pnpm, pe scurt

**Ce e.** Al treilea package manager pe care îl vei întâlni des, mai ales în companii cu monorepo-uri (mai multe aplicații într-un singur repo). Ține fiecare versiune de pachet o singură dată pe disc și o leagă în proiecte. E rapid, economic cu spațiul și **strict**: nu te lasă să imporți un pachet pe care nu l-ai declarat în `package.json`, ceea ce prinde bug-uri ascunse. Comenzile sunt aproape identice: `pnpm install`, `pnpm add zod`, `pnpm dev`.

**Capcana.** Ajungi într-un repo cu `pnpm-lock.yaml` și rulezi `npm install`. Creezi un al doilea lockfile și instalezi alte versiuni decât colegii. **Uită-te mereu ce lockfile are proiectul și folosește package manager-ul lui.**

<details>
<summary>Întrebare de interviu: Intri într-un proiect nou. Cum știi ce package manager să folosești?</summary>

Mă uit după lockfile: `package-lock.json` înseamnă npm, `pnpm-lock.yaml` înseamnă pnpm, `yarn.lock` înseamnă Yarn, `bun.lock` înseamnă Bun. Mă mai uit în README și după câmpul `packageManager` din `package.json`. Folosesc exact acel package manager, ca să nu creez un lockfile paralel și să instalez alte versiuni decât restul echipei.
</details>

## Mai departe

- [Vite — Scaffolding your first Vite project](https://vite.dev/guide/#scaffolding-your-first-vite-project)
- [Bun — bun install](https://bun.com/docs/cli/install)
- [Bun — Lockfile](https://bun.com/docs/install/lockfile)
