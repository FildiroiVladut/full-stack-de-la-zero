# Tipurile de bază

Tipurile pe care le vei vedea în aproape orice fișier: primitive, obiecte, array-uri, `type` vs `interface`, câmpuri opționale, literal types și union types. Exemplele vin din P1 Shelf.

## Primitive, array-uri și inferența

**Ce e.** Tipurile primitive sunt cele din JavaScript, scrise cu literă mică: `string`, `number`, `boolean`, `null`, `undefined` (plus `bigint` și `symbol`, rare). Array-urile se scriu `string[]` sau `Array<string>`, care e același lucru.

**De ce contează.** De cele mai multe ori **nu** scrii tipul: TypeScript îl deduce singur din valoare (_type inference_). Adnotările le pui la granițe: parametrii funcțiilor, structurile de date și ce exporți.

**Exemplu minim**
```ts
const title = "Dune";        // inferred: string (actually the literal "Dune", since it's const)
let pages = 608;             // inferred: number
const tags = ["sf", "classic"]; // inferred: string[]

function pagesPerDay(totalPages: number, days: number) { // parameters: annotate
  return Math.ceil(totalPages / days);                    // return type: inferred as number
}
```

**Capcana.** Codul generat de AI pune uneori tipuri peste tot (`const x: number = 5`). Nu e greșit, dar e zgomot. În code review vei auzi „let inference do its job”.

## Obiecte: `type` și `interface`

**Ce e.** Două moduri de a descrie forma unui obiect. Pentru obiecte simple sunt aproape interschimbabile.

**Exemplu minim**
```ts
type Book = {
  key: string;
  title: string;
  authorNames?: string[];      // optional: may be missing
  firstPublishYear?: number;
  readonly addedAt: string;    // can't be reassigned after creation
};

interface Shelf {
  id: string;
  name: string;
  books: Book[];
}
```

**Când pe care.**
- `type` poate descrie orice: obiecte, union-uri (`"a" | "b"`), tupluri, funcții. E alegerea implicită în multe proiecte React.
- `interface` descrie doar obiecte, poate fi extinsă cu `extends` și poate fi redeschisă (două declarații cu același nume se combină). Nest și multe biblioteci o folosesc des.
- Regula practică: alege una ca default și fii consecvent în proiect. Folosește `type` când ai nevoie de union.

**Capcana.** `readonly` e o regulă doar pentru compilator. La runtime obiectul poate fi modificat fără probleme, iar `readonly` nu face nimic recursiv: un array `readonly` poate conține obiecte modificabile.

<details>
<summary>Întrebare de interviu: Type vs interface?</summary>

Pentru forma unui obiect sunt aproape echivalente. `type` e mai general: poate fi union, tuplu, tip de funcție sau tip derivat cu utility types. `interface` se poate extinde cu `extends` și se poate redeclara, iar declarațiile se combină (_declaration merging_), ceea ce e util pentru a extinde tipuri din biblioteci. În practică folosesc convenția proiectului și aleg `type` când am nevoie de union-uri.
</details>

## Câmpuri opționale și `null`

**Ce e.** `authorNames?: string[]` înseamnă că acel câmp poate lipsi, deci are tipul `string[] | undefined`. `vatNumber: string | null` înseamnă că acel câmp există mereu, dar poate fi `null`. Diferența contează când modelezi date din baza de date, unde coloanele nullable devin `| null`.

**Exemplu minim**
```ts
type Client = {
  name: string;
  vatNumber: string | null;  // always present in the DB row, may be null (individual client)
  notes?: string;            // may not be sent at all
};

function displayVat(client: Client) {
  return client.vatNumber ?? "Individual";  // TypeScript forces you to handle null
}
```

**Cum îl vezi în cod real.** Prisma generează tipuri cu `| null` pentru coloanele nullable. Formularele și payload-urile de update folosesc câmpuri opționale.

**Capcana.** Să tratezi `null` și `undefined` ca pe același lucru peste tot. JSON nu are `undefined`: un câmp `undefined` dispare la `JSON.stringify`, iar `null` rămâne. Dacă vrei să-i spui unui API „șterge valoarea asta”, trimiți `null`.

## Literal types și union types

**Ce e.** Un tip poate fi o **valoare exactă**: `"read"` e un tip care acceptă doar string-ul `"read"`. Combinate cu `|` (union), dau o listă închisă de valori permise.

**Exemplu minim**
```ts
type ShelfStatus = "want-to-read" | "reading" | "read";

let status: ShelfStatus = "reading"; // ok
status = "finished";                 // error: '"finished"' is not assignable to type 'ShelfStatus'

const sizes = ["S", "M", "L"] as const;   // readonly tuple of exact values
type CoverSize = (typeof sizes)[number];  // "S" | "M" | "L"
const coverUrl = (coverId: number, size: CoverSize = "M") =>
  `https://covers.openlibrary.org/b/id/${coverId}-${size}.jpg`;
```

**De ce contează.** Union-urile de literali sunt felul în care modelezi statusuri, roluri și categorii: `"admin" | "member"` în P3, `"draft" | "sent" | "paid"` în P2. Editorul îți completează valorile, iar o greșeală de scriere devine eroare de compilare.

**Capcana.** `as const` contează. Fără el, `["S", "M", "L"]` e doar `string[]` și pierzi valorile exacte.

## De ce union de string-uri și nu `enum`

**Ce e.** TypeScript are și `enum`, pe care îl vei vedea în cod mai vechi și uneori în Nest:

```ts
enum Role { Admin = "admin", Member = "member" } // generates real JavaScript code
type RoleUnion = "admin" | "member";               // disappears entirely after compilation
```

**De ce preferă multe echipe union-ul.**
- `enum` e una dintre puținele construcții TypeScript care **generează cod JavaScript**, deci nu e doar un tip. Instrumentele care doar șterg tipurile, cum e rularea directă cu `node fisier.ts`, nu-l suportă.
- Union-ul funcționează direct cu string-urile din JSON și din baza de date, fără conversii.
- Când ai nevoie și de lista valorilor la runtime (pentru un `<select>`, de exemplu), folosești tiparul `as const` de mai sus sau o schemă zod (`z.enum([...])`).

**Capcana.** Prisma generează enum-uri pentru coloanele de tip enum din schema bazei de date. Acolo sunt OK: le folosești, doar nu le crea tu fără motiv.

## Tipuri pentru răspunsuri de API

**Ce e.** Când consumi un API, descrii cu un tip forma răspunsului, ca restul codului să lucreze cu câmpuri cunoscute.

**Exemplu minim** (Open Library, P1)
```ts
// mirrors the API exactly — field names as the API sends them
type SearchDoc = {
  key: string;                  // "/works/OL893414W"
  title: string;
  author_name?: string[];
  first_publish_year?: number;
  cover_i?: number;
};

type SearchResponse = {
  numFound: number;
  docs: SearchDoc[];
};
```

**Cum îl vezi în cod real.** Două niveluri:
1. tipul care oglindește API-ul, cu numele lui (`author_name`, `cover_i`);
2. tipul **tău**, pe care îl folosește UI-ul (`Book` cu `authorNames`, `coverUrl`), plus o funcție mică care transformă din unul în celălalt.

Așa, dacă API-ul se schimbă, modifici într-un singur loc.

**Capcana.** Să scrii tipul API-ului „din memorie” sau să-l lași pe AI să-l ghicească. Deschide o dată URL-ul în browser sau cu `curl`, uită-te la răspunsul real și marchează ca opțional (`?`) tot ce nu apare garantat. Iar un tip scris de mână tot nu verifică nimic la runtime, așa că pentru siguranță reală validezi cu zod ([05-validare-cu-zod.md](05-validare-cu-zod.md)).

<details>
<summary>Întrebare de interviu: Cum tipezi datele primite de la un API extern?</summary>

Mă uit la răspunsul real și scriu un tip care îl oglindește, cu câmpurile incerte marcate ca opționale. Separat definesc modelul meu intern, cu nume și forme convenabile pentru UI, și o funcție de mapare între ele. Pentru că tipurile nu există la runtime, validez răspunsul la graniță cu o schemă zod și derivez tipul din schemă cu `z.infer`, ca să am o singură sursă de adevăr.
</details>

## Mai departe

- [TypeScript Handbook — Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Handbook — Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
