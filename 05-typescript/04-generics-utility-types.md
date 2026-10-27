# Generics și utility types

Generics sunt partea din TypeScript care sperie cel mai tare la prima vedere, din cauza parantezelor unghiulare. Nu trebuie să scrii biblioteci generice, ci să le **citești** fără panică, să scrii una-două funcții simple și să folosești utility types ca să derivi tipuri unele din altele.

## Generics: tipuri cu parametru

**Ce e.** Un generic e un tip sau o funcție care primește **un alt tip ca parametru**. Îl folosești de mult fără să-l numești așa: `Array<string>` e „array de string-uri”, `Promise<Invoice>` e „promise care se rezolvă cu o factură”. `T` e doar un nume de variabilă pentru tip, ca `x` pentru valori.

**De ce contează.** Codul din biblioteci e plin de generics: `useState<T>`, `useQuery<T>`, `Repository<T>`, `z.infer<typeof schema>`. Dacă le citești ca „X de T”, devin lizibile.

**Exemplu minim**
```ts
function firstOrNull<T>(items: T[]): T | null {
  return items[0] ?? null;
}

const n = firstOrNull([1, 2, 3]);   // T = number  → n: number | null
const s = firstOrNull(["a", "b"]);  // T = string  → s: string | null
```

Fără generic, ai fi avut de ales între `any` (pierzi tipul) și câte o funcție pentru fiecare tip. Analogia din C++ e directă: e același concept ca template-urile, doar că există numai la compilare.

**Capcana.** Să adaugi generics unde nu e nevoie. Dacă `T` apare o singură dată în semnătură, de obicei nu ai nevoie de el.

## Generics pe care le scrii în practică

**Ce e.** În proiecte vei scrie generics în două locuri: helper-e pentru request-uri și hook-uri reutilizabile.

**Exemplu minim** (un helper de fetch)
```ts
async function getJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<T>;  // still a "trust me" — see the zod file
}

const data = await getJson<SearchResponse>("https://openlibrary.org/search.json?q=dune");
```

**Cum îl vezi în cod real** (P1: un hook care ține orice valoare în `localStorage`)
```ts
import { useEffect, useState } from "react";

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored !== null ? (JSON.parse(stored) as T) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

// usage: T is SavedBook[], so `books` and `setBooks` are fully typed
const [books, setBooks] = useLocalStorage<SavedBook[]>("shelf.books", []);
```
- `as const` la final face ca return-ul să fie un tuplu `[T, setter]`, ca la `useState`, nu un array de tipuri amestecate.
- `JSON.parse(...) as T` e tot o promisiune făcută compilatorului. Dacă cineva a modificat `localStorage` de mână, primești date greșite.

<details>
<summary>Întrebare de interviu: Ce sunt generics și când le folosești?</summary>

Generics sunt parametri de tip: scriu o funcție sau un tip o dată și îl las să lucreze cu orice tip, păstrând informația de tip. Exemplul clasic e `Array<T>` sau o funcție `firstOrNull<T>(items: T[]): T | null`. Le folosesc pentru helper-e reutilizabile, cum ar fi un wrapper peste `fetch`, un hook ca `useLocalStorage<T>` sau un tip `ApiResponse<T>`, ca să nu recurg la `any` și să nu duplic cod pentru fiecare tip.
</details>

## Constrângeri: `extends` și `keyof`

**Ce e.** `keyof T` e union-ul numelor de câmpuri ale lui `T`. `K extends keyof T` înseamnă că `K` trebuie să fie unul dintre câmpurile lui `T`. `T[K]` e tipul câmpului respectiv.

**Exemplu minim**
```ts
type Client = { id: number; name: string; email: string };

type ClientKey = keyof Client; // "id" | "name" | "email"

function getField<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const client: Client = { id: 1, name: "Acme", email: "office@acme.ro" };
const name = getField(client, "name");  // string
getField(client, "phone");              // error: '"phone"' is not assignable to '"id" | "name" | "email"'
```

**Cum îl vezi în cod real.** Rar scris de tine, des în semnăturile bibliotecilor: componente de tabel care primesc numele coloanelor, funcții de sortare după câmp, `orderBy` în Prisma.

## Utility types: Partial, Pick, Omit, Record

**Ce e.** Tipuri generice incluse în TypeScript, care construiesc un tip nou dintr-unul existent:

| Utility | Ce face | Exemplu |
|---|---|---|
| `Partial<T>` | toate câmpurile devin opționale | payload de update |
| `Required<T>` | toate câmpurile devin obligatorii | |
| `Pick<T, "a" \| "b">` | păstrează doar câmpurile date | preview într-o listă |
| `Omit<T, "a">` | scoate câmpurile date | input fără `id` |
| `Record<K, V>` | obiect cu cheile `K` și valori `V` | dicționare, etichete |
| `Readonly<T>` | toate câmpurile devin `readonly` | |

**Exemplu minim** (P2: DTO-urile pentru clienți)
```ts
type Client = {
  id: number;
  name: string;
  email: string;
  vatNumber: string | null;
  createdAt: string;
};

type CreateClientDto = Omit<Client, "id" | "createdAt">; // the server sets those
type UpdateClientDto = Partial<CreateClientDto>;         // send only what changed
type ClientPreview = Pick<Client, "id" | "name">;        // for a dropdown

type InvoiceStatus = "draft" | "sent" | "paid";
const statusLabels: Record<InvoiceStatus, string> = {
  draft: "Draft",
  sent: "Sent",
  paid: "Paid",
}; // forget one status → compile error
```

**De ce contează.** O singură sursă de adevăr. Când adaugi un câmp în `Client`, toate tipurile derivate se actualizează singure, iar `Record` te anunță dacă ai uitat o etichetă pentru un status nou.

**Cum îl vezi în cod real.** DTO (_Data Transfer Object_) e forma datelor care circulă printr-un request. În Nest, DTO-urile sunt clase cu decoratori de validare, iar pachetul `@nestjs/swagger` (sau `@nestjs/mapped-types`) are echivalente pentru clase: `PartialType`, `PickType`, `OmitType`. E aceeași idee. Vezi [07-backend-node-nest/03-dto-validare-erori.md](../07-backend-node-nest/03-dto-validare-erori.md).

**Capcana.** Să copiezi tipul de mână („`UpdateClient` e la fel ca `Client`, dar cu `?`”). Peste o lună cele două tipuri nu mai sunt sincronizate și nimeni nu observă.

<details>
<summary>Întrebare de interviu: Ce utility types folosești des?</summary>

`Partial` pentru payload-uri de update, unde trimit doar câmpurile schimbate. `Omit` pentru input-uri de creare, unde scot câmpurile generate de server, ca `id` și `createdAt`. `Pick` pentru vederi reduse ale unui model. `Record` pentru dicționare cu chei cunoscute, de exemplu etichete pentru fiecare status. Avantajul e că tipurile derivate rămân sincronizate automat cu modelul de bază.
</details>

## `typeof` la nivel de tip

**Ce e.** Într-o poziție de tip, `typeof valoare` îți dă tipul unei valori existente. E inversul lucrului obișnuit: scrii întâi valoarea și extragi tipul din ea.

**Exemplu minim**
```ts
const defaultSettings = { currency: "RON", vatRate: 0.21, locale: "ro-RO" };
type Settings = typeof defaultSettings; // { currency: string; vatRate: number; locale: string }
```

**Cum îl vezi în cod real.** În `z.infer<typeof ClientSchema>` (fișierul următor) și în tiparul `(typeof sizes)[number]` din [02-tipuri-de-baza.md](02-tipuri-de-baza.md). Merită să recunoști cele două sensuri ale lui `typeof`: în cod JavaScript întoarce un string la runtime (`"number"`), iar într-o poziție de tip e operatorul TypeScript.

## Mai departe

- [TypeScript Handbook — Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript — Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
