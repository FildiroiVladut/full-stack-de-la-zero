# De ce TypeScript

Ce e TypeScript, ce face pentru tine, ce nu face, și cum îți pregătești un folder unde rulezi exemplele din tot capitolul.

## Setup: folderul de joacă

O singură dată, în WSL:

```bash
mkdir -p ~/code/ts-playground && cd ~/code/ts-playground
npm init -y
npm pkg set type=module      # treat .ts/.js files as ES modules
npm install -D typescript tsx
npx tsc --init               # creates tsconfig.json with strict mode on
code .
```

**Verifică.** Creează `hello.ts` cu `const n: number = 42; console.log(n);` și rulează `npx tsx hello.ts`. Trebuie să vezi `42`.

**Ce s-a întâmplat de fapt.** Ai creat un proiect npm (`package.json`), ai instalat compilatorul TypeScript (`tsc`) și `tsx`, un utilitar care rulează direct fișiere `.ts`, și ai generat `tsconfig.json`, unde stă configurația compilatorului. `type=module` contează: fără el, `tsx` tratează fișierele ca CommonJS, iar `await` la nivelul fișierului dă eroare.

Ai două comenzi pe care le vei folosi mereu:
- `npx tsx fisier.ts` **rulează** codul. Doar șterge tipurile, nu le verifică.
- `npx tsc --noEmit` **verifică** tipurile din tot proiectul, fără să genereze fișiere.

Variantele recente de Node rulează și singure fișiere `.ts` simple (`node fisier.ts`), tot fără verificare. `tsx` acceptă mai multe construcții, de aceea îl folosim pe el.

## Ce e TypeScript

**Ce e.** JavaScript plus un sistem de tipuri. Scrii `.ts`, compilatorul verifică tipurile, apoi le șterge și rămâne JavaScript obișnuit, singurul lucru pe care îl pot rula browserul și Node.

**De ce contează.** Erorile pe care le-ai găsi la runtime (`undefined is not a function`, câmp scris greșit, argument de tip greșit) le găsești în editor, cât scrii. Pe un proiect cu sute de fișiere, sau cu cod generat de AI, ăsta e principalul tău mecanism de siguranță.

**Exemplu minim**
```ts
function vatCents(subtotalCents: number, rate: number): number {
  return Math.round(subtotalCents * rate);
}

vatCents(7690, 0.21);   // ok
vatCents("7690", 0.21); // error: Argument of type 'string' is not assignable to parameter of type 'number'
```

**Capcana.** Să crezi că TypeScript e alt limbaj. Sintaxa, runtime-ul și bibliotecile sunt aceleași ca în JavaScript, iar tot ce știi din zilele 1–7 rămâne valabil. TypeScript doar adaugă adnotări și verificare.

## Tipurile există doar la compilare

**Ce e.** După compilare, din tipuri nu mai rămâne nimic. În JavaScript-ul care rulează nu există `number`, `Book` sau `interface`. Diferența față de C e mare: acolo tipul influențează memoria și execuția, pe când aici e doar o verificare făcută înainte de rulare.

**De ce contează.** E cea mai importantă idee din capitol. TypeScript verifică doar codul **tău**, nu și datele care vin din afară: răspunsuri de API, formulare, `localStorage`, fișiere, variabile de mediu.

**Exemplu minim**
```ts
type Book = { title: string; year: number };

const res = await fetch("https://openlibrary.org/search.json?q=dune&limit=1");
const data = (await res.json()) as { docs: Book[] }; // "trust me" — no check happens
console.log(data.docs[0]?.year.toFixed(0)); // compiles fine, crashes at runtime
// Open Library has no `year` field; it's `first_publish_year`
```

**Cum îl vezi în cod real.** Acolo unde datele intră în aplicație, cod bine scris le validează la runtime (cu zod, sau cu class-validator în Nest) și abia după aceea le tratează ca tipuri sigure. Vezi [05-validare-cu-zod.md](05-validare-cu-zod.md).

**Capcana.** `as Tip` nu convertește și nu verifică nimic. Doar spune compilatorului „crede-mă”. Fiecare `as` din codul generat de AI e un loc unde merită să te uiți de două ori.

<details>
<summary>Întrebare de interviu: TypeScript te protejează de date greșite venite de la un API?</summary>

Nu, pentru că tipurile sunt șterse la compilare și nu există la runtime. TypeScript verifică doar consistența codului meu. Dacă API-ul trimite alt format decât cel declarat, codul compilează și pică la rulare. De aceea validez datele externe la granița aplicației, cu o bibliotecă precum zod, și derivez tipul din schemă, ca validarea și tipul să nu se desincronizeze.
</details>

## tsconfig și strict mode

**Ce e.** `tsconfig.json` spune compilatorului cât de sever să fie și ce JavaScript să genereze. Cea mai importantă opțiune e `"strict": true`, care activează un set întreg de verificări. Cea mai valoroasă dintre ele e `strictNullChecks`: `null` și `undefined` nu mai pot fi folosite în locul oricărui alt tip.

**Exemplu minim**
```ts
const books = [{ title: "Dune" }];
const found = books.find((b) => b.title === "Neuromancer"); // type: { title: string } | undefined
console.log(found.title);  // error with strict: 'found' is possibly 'undefined'
console.log(found?.title); // ok
```

**Cum îl vezi în cod real.** Toate proiectele noi generate de Vite, Next.js și Nest pornesc cu `strict` activat. `tsc --init` îți mai activează și `noUncheckedIndexedAccess`, care face ca `arr[0]` să fie tratat ca posibil `undefined`. E enervant la început, dar prinde bug-uri reale.

**Capcana.** Să dezactivezi `strict`, sau să pui `// @ts-ignore`, ca să scapi de erori. E echivalentul scoaterii bateriei din detectorul de fum. Dacă vezi asta într-un PR, întrebi de ce.

## Ce primești în editor

**Ce e.** VS Code folosește TypeScript și pentru fișierele `.js`, dar într-un proiect TS primești mult mai mult:
- **hover** pe orice variabilă îți arată tipul ei;
- **autocomplete** cu câmpurile reale ale obiectului;
- **F12 / Ctrl+click** te duce la definiție, inclusiv în bibliotecile din `node_modules`;
- **F2** redenumește un simbol în tot proiectul, corect;
- **subliniere roșie** înainte să rulezi ceva.

**De ce contează.** Așa citești repede un proiect generat de Claude: treci cu mouse-ul peste lucrurile pe care nu le înțelegi și sari la definiții. Tipurile sunt documentație care nu se învechește.

**Capcana.** Să ignori sublinierile roșii pentru că „merge oricum”. `tsx` rulează codul chiar dacă are erori de tip, dar build-ul de producție (`tsc`, Vite cu verificare, CI) nu va trece.

<details>
<summary>Întrebare de interviu: De ce a trecut industria la TypeScript?</summary>

Pentru că pe proiecte mari și cu echipe mari erorile de tip devin cea mai frecventă clasă de bug-uri, iar TypeScript le prinde înainte de rulare. Pe lângă asta, editorul devine mult mai util (autocomplete, navigare, refactorizare sigură), iar tipurile servesc ca documentație care rămâne la zi. Costul e un pas de build și puțin mai mult cod, dar e mic față de ce câștigi.
</details>

## Mai departe

- [TypeScript Handbook — The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)
- [TypeScript for JavaScript Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [TSConfig Reference — strict](https://www.typescriptlang.org/tsconfig/#strict)
