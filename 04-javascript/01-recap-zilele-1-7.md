# Recap: zilele 1–7

O pagină cu tot ce ai făcut până acum, ca să ai o singură foaie de recitit înainte de interviu. Linkurile duc la codul tău. Partea a doua conține capcanele clasice, cele pe care le întreabă toată lumea.

## Ce ai acoperit, pe zile

**[Ziua 1 · variabile și tipuri](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua1)**
- `const` e default-ul, iar `let` îl folosești doar când reasignezi. `const` blochează legătura numelui cu valoarea, nu conținutul: poți face `push` într-un array `const`.
- Ai 7 tipuri primitive (`string`, `number`, `boolean`, `undefined`, `null`, `bigint`, `symbol`); restul sunt obiecte. `typeof null === "object"` e un bug istoric.
- Template literals: `` `Masa ${table}` ``.

**[Ziua 2 · comparații, logică, text](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua2)**
- `===` compară fără conversii, `==` convertește mai întâi tipurile. Folosește `===`.
- `??` alege valoarea din dreapta doar când stânga e `null` sau `undefined`. `?.` se oprește fără eroare dacă dă de `null`.
- String-urile sunt imutabile: `text.trim()` întoarce un string nou. Asta ai observat singur în `06-text.js`.
- `Number("12.40 lei")` dă `NaN`, pe când `parseFloat("12.40 lei")` dă `12.4`. `NaN !== NaN`, așa că verifici cu `Number.isNaN`.

**[Ziua 3 · funcții](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua3)**
- O arrow function fără acolade face return implicit; cu acolade trebuie să scrii `return`.
- Callback-ul e o funcție dată ca argument altei funcții, care o apelează mai târziu.
- O funcție pură dă același rezultat la aceleași argumente și nu modifică nimic din afara ei. Funcțiile tale de food cost sunt pure, `addToTotal` nu e.
- Scope: `let`/`const` trăiesc în bloc, `var` în funcție.

**[Ziua 4 · obiecte și referințe](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua4)**
- Obiectele și array-urile se transmit prin referință: două variabile pot arăta spre același obiect.
- Spread (`{ ...obj }`) face copie superficială (shallow), `structuredClone` face copie completă (deep).
- Destructuring: `const { name, price } = product`.

**[Ziua 5 · metodele de array](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua5)**
- `map` transformă, `filter` păstrează doar o parte, `find` întoarce primul element găsit (sau `undefined`), `reduce` strânge totul într-o valoare, `some`/`every` răspund cu da sau nu.
- Le înlănțuiești: `filter → sort → map`.
- `sort` modifică array-ul original, de aceea ai scris `[...arr].sort(...)`.

**[Ziua 6 · closures și this](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua6)**
- Un closure e o funcție care își amintește variabilele din locul unde a fost creată. Exemple la tine: contorul, generatorul de ID-uri.
- `this` depinde de cum e apelată funcția, nu de unde e scrisă. Excepție: arrow function-urile iau `this` din jur.

**[Ziua 7 · clase și erori](https://github.com/FildiroiVladut/js-de-la-zero/tree/main/ziua7)**
- `class`, `constructor`, `extends`/`super`, câmpuri private `#`, getteri.
- `throw new Error(...)` oprește execuția până la cel mai apropiat `catch`. `finally` rulează în orice caz.

## Capcanele clasice de interviu

### `==` vs `===`

```js
1 == "1";          // true  — "1" is converted to a number
null == undefined; // true
[] == false;       // true  — both become 0
1 === "1";         // false — different types, no conversion
```

Regula din echipe e simplă: folosești mereu `===`. Singura excepție pe care o vei vedea în cod e `x == null`, care prinde `null` și `undefined` deodată.

<details>
<summary>Întrebare de interviu: Care e diferența dintre == și ===?</summary>

`===` compară valoarea și tipul, fără conversii. `==` aplică întâi coerciția de tip, adică încearcă să aducă ambele valori la același tip, și abia apoi compară. De aici rezultate ca `[] == false` → `true`. În cod folosesc `===`, ca să nu depind de regulile de coerciție.
</details>

### Truthy și falsy

Valorile falsy sunt `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined` și `NaN`. Orice altceva e truthy, inclusiv `"0"`, `[]` și `{}`.

```js
const stock = 0;
if (!stock) console.log("No stock info"); // wrong: 0 is a real value
if (stock == null) console.log("No stock info"); // correct: only null/undefined
```

Capcana apare în cod real la cantități, prețuri și indexuri: `0` e o valoare validă, dar `if (!x)` o tratează ca „lipsă”.

<details>
<summary>Întrebare de interviu: Ce valori sunt falsy în JavaScript?</summary>

`false`, `0`, `-0`, `0n`, șirul gol, `null`, `undefined` și `NaN`. Restul sunt truthy, inclusiv `"0"`, array-ul gol și obiectul gol. În practică, când `0` sau `""` sunt valori valide, verific explicit cu `== null` sau folosesc `??` în loc de `||`.
</details>

### `var`/`let`/`const`, hoisting și TDZ

```js
console.log(a); // undefined — var is hoisted and initialized with undefined
var a = 1;

console.log(b); // ReferenceError — let is in the TDZ until this line
let b = 2;

greet(); // works — function declarations are hoisted entirely
function greet() { console.log("hi"); }
```

Hoisting înseamnă că declarațiile sunt „ridicate” la începutul scope-ului înainte să ruleze codul. `let` și `const` sunt și ele ridicate, dar până la linia lor stau în _temporal dead zone_ (TDZ): dacă le accesezi acolo, primești eroare.

<details>
<summary>Întrebare de interviu: Ce e hoisting și ce e TDZ?</summary>

Înainte să ruleze codul, JavaScript înregistrează toate declarațiile din scope. `var` e inițializat imediat cu `undefined`, deci îl poți citi înainte de linia lui, chiar dacă valoarea nu e cea așteptată. `let` și `const` sunt înregistrate, dar rămân neinițializate până la linia declarației; intervalul acela e TDZ și orice acces aruncă `ReferenceError`. Declarațiile de funcție sunt ridicate cu totul, așa că le poți apela înainte de definiție.
</details>

### Closures în bucle

E exact fișierul tău [`ziua6/03-capcana-var.js`](https://github.com/FildiroiVladut/js-de-la-zero/blob/main/ziua6/03-capcana-var.js). Cu `var` există o singură variabilă pentru toată bucla, așa că toate callback-urile afișează `4`. Cu `let` se creează o variabilă nouă la fiecare iterație, deci afișează `1, 2, 3`. Explicația ta din comentariu e exact răspunsul de interviu.

### `this` în metodă vs arrow function

```js
const shelf = {
  name: "To read",
  show() { console.log(this.name); },
  showArrow: () => console.log(this?.name), // arrow: `this` comes from outside the object
};
shelf.show();      // "To read"
shelf.showArrow(); // undefined
const fn = shelf.show;
fn();              // TypeError in modules — `this` is undefined, the method lost its object
```

Arrow function-urile sunt bune pentru callback-uri în interiorul metodelor, cum ai făcut cu `setTimeout` în `garaj.deschide`. Ca metode de obiect nu sunt potrivite.

<details>
<summary>Întrebare de interviu: Cum se stabilește valoarea lui this?</summary>

La o funcție normală, `this` se stabilește la apel: în `obj.metoda()` e `obj`, iar într-un apel simplu `fn()` e `undefined` în strict mode, deci și în module. Arrow function-urile nu au `this` propriu, îl iau din scope-ul în care au fost scrise. De aceea un callback arrow din interiorul unei metode vede `this`-ul metodei, dar o metodă scrisă ca arrow nu vede obiectul.
</details>

### Referință vs copie, shallow vs deep

```js
const invoice = { client: { name: "Acme" }, total: 100 };
const shallow = { ...invoice };
shallow.client.name = "Other";     // also changes invoice.client.name
const deep = structuredClone(invoice);
deep.client.name = "Deep";         // invoice stays untouched
```

`structuredClone` nu copiază funcțiile (aruncă eroare), iar instanțele de clasă devin obiecte simple: își pierd metodele. Pentru date simple (JSON) e perfect.

<details>
<summary>Întrebare de interviu: Ce diferență e între copia shallow și cea deep?</summary>

Copia shallow (spread, `Object.assign`, `slice`) copiază doar primul nivel. Obiectele din interior rămân comune, deci dacă modifici unul dintr-o copie, modificarea se vede și în original. Copia deep copiază recursiv tot arborele. În JavaScript modern folosesc `structuredClone`, cu limita că nu copiază funcții și nu păstrează prototipul claselor.
</details>

### `sort()` fără comparator

În `ziua5/08-sort.js` ai lăsat comentariul `// n am idee` la `[10, 9, 1, 100].sort()`. Rezultatul e `[1, 10, 100, 9]`: fără comparator, `sort` transformă elementele în string-uri și le compară caracter cu caracter. Pentru numere scrii `(a, b) => a - b`.

În plus, `sort` modifică array-ul original. Alternativa modernă e `toSorted`, care întoarce un array nou: `prices.toSorted((a, b) => a - b)`.

### Metode care modifică vs metode care întorc array nou

| Modifică originalul | Întorc un array nou |
|---|---|
| `push`, `pop`, `shift`, `unshift` | `concat`, spread `[...a, x]` |
| `splice` | `slice`, `toSpliced` |
| `sort`, `reverse` | `toSorted`, `toReversed` |
| `arr[i] = x` | `with(i, x)`, `map` |

Contează enorm în React: state-ul nu se modifică pe loc, se înlocuiește cu o valoare nouă. Dacă faci `push` în state, React nu observă schimbarea. Detalii în [06-react/02-props-si-state.md](../06-react/02-props-si-state.md).

## Review la codul tău

Din [`ziua7/05-inventar.js`](https://github.com/FildiroiVladut/js-de-la-zero/blob/main/ziua7/05-inventar.js):

```js
adauga(nume, cantitate, pret, minim) {
  this.#produse.push({ nume, cantitate, pret, minim });
}

stocSubMinim() {
  return [...this.#produse]
    .filter((produs) => produs.cantitate <= 10)
    .map((produs) => produs.nume);
}
```

Un senior ți-ar lăsa trei comentarii la review:

1. **Pragul e scris direct în cod (`10`), deși fiecare produs își are propriul `minim`.** Numele funcției promite „sub minim”, iar `adauga` salvează câmpul `minim`, dar `filter` nu-l folosește. Mai interesant e că, cu datele tale (Pix 50/min 20, Caiet 10/min 15, Marker 5/min 10), rezultatul iese corect din coincidență: Caiet și Marker sunt sub 10 *și* sub minimul lor. Adaugă un produs cu cantitatea 12 și minimul 30 și o să vezi că lipsește din rezultat. Testul care trece pe date norocoase e cel mai periculos test.
2. **`[...this.#produse]` nu ajută la nimic aici.** `filter` întoarce deja un array nou și nu atinge originalul. Copia era necesară înainte de `sort`, care modifică pe loc. Ai luat obiceiul de la `sort` și l-ai aplicat peste tot. Nu strică, dar arată că operația nu e înțeleasă până la capăt, și un reviewer observă asta.
3. **`<=` sau `<`?** Dacă stocul e exact egal cu minimul, e „sub minim”? Asta e o decizie de business. O iei explicit și o pui în numele funcției sau într-un test.

```js
stocSubMinim() {
  return this.#produse
    .filter((produs) => produs.cantitate < produs.minim)
    .map((produs) => produs.nume);
}
```

**De ce prinde asta un review și nu un set de exerciții:** exercițiul verifică doar că output-ul e cel așteptat, iar output-ul era corect. Reviewer-ul citește *intenția* (numele funcției, datele salvate) și o compară cu *implementarea*. Exact asta faci și tu de acum înainte cu codul scris de Claude: nu te întrebi doar „merge?”, ci și „face ce zice că face?”. Mai multe în [12-lucrul-cu-ai/02-review-la-cod-generat.md](../12-lucrul-cu-ai/02-review-la-cod-generat.md).

Încă o observație: de acum scrii cod cu nume în engleză (`products`, `stockBelowMinimum`), pentru că așa se lucrează în orice echipă.

## Mai departe

- [javascript.info — The JavaScript language](https://javascript.info/js) (referință când vrei detalii)
- [MDN — Array.prototype.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- [MDN — Equality comparisons and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness)
