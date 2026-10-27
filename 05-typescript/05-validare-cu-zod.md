# Validare cu zod

TypeScript verifică doar codul tău. Datele care intră din afară (răspunsuri de API, formulare, `localStorage`, variabile de mediu, output-ul unui LLM) pot avea orice formă. zod e biblioteca prin care le verifici la runtime, iar tipul TypeScript îl obții din aceeași schemă. Exemplele merg în folderul de joacă după `npm install zod`.

## Unde validezi: la granițe

**Ce e.** O _graniță_ e orice loc prin care date necontrolate de codul tău intră în aplicație. În interior ai încredere în tipuri, iar la graniță verifici.

```
   outside world                  │  your code (types are trusted)
   ───────────────────────────────┼────────────────────────────────
   API response (Open Library)  ──┤ validate → typed Book[]
   form input                   ──┤ validate → typed ClientInput
   process.env                  ──┤ validate → typed Env
   localStorage                 ──┤ validate → typed SavedBook[]
   LLM output (Claude, P4)      ──┤ validate → typed InvoiceLine[]
```

**De ce contează.** E diferența dintre o eroare clară la intrare („câmpul `price` lipsește”) și un `undefined` care traversează trei fișiere și dă crash undeva fără legătură. E una dintre primele întrebări pe care le pune un senior la review: „unde validezi asta?”.

**Capcana.** Validarea doar în frontend. Formularul validează pentru o experiență bună pentru utilizator. Backend-ul validează **din nou**, pentru siguranță, pentru că oricine poate trimite request-uri direct la API, ocolind formularul. Vezi [10-securitate-env/04-validare-si-atacuri.md](../10-securitate-env/04-validare-si-atacuri.md).

## Schema și tipul dintr-o singură sursă

**Ce e.** Descrii forma datelor o singură dată, ca schemă zod, și extragi tipul TypeScript din ea cu `z.infer`. Nu mai scrii separat tipul și validarea, care s-ar desincroniza.

**Exemplu minim**
```ts
import * as z from "zod";

const SavedBookSchema = z.object({
  key: z.string(),
  title: z.string(),
  status: z.enum(["want-to-read", "reading", "read"]),
  rating: z.number().int().min(1).max(5).optional(),
});

type SavedBook = z.infer<typeof SavedBookSchema>;
// { key: string; title: string; status: "want-to-read" | "reading" | "read"; rating?: number }
```

**Cum îl vezi în cod real.** Schemele stau de obicei într-un fișier `schemas.ts` lângă codul care le folosește, cu tipurile exportate imediat sub ele. În unele proiecte vei vedea același nume pentru schemă și tip (`const Book = z.object(...)` și `type Book = z.infer<typeof Book>`). TypeScript permite asta, pentru că valorile și tipurile trăiesc în spații de nume separate.

**Capcana.** În tutoriale vechi vei vedea `import { z } from "zod"` și `z.string().email()`. În zod 4, documentația folosește `import * as z from "zod"`, iar formele de tipul `z.string().email()` sunt marcate ca deprecated în favoarea lui `z.email()`, `z.url()` și altele asemenea. Și AI-ul generează uneori varianta veche.

## `parse` vs `safeParse`

**Ce e.** Două feluri de a valida:
- `schema.parse(data)` întoarce datele validate (tipate) sau **aruncă** un `ZodError`;
- `schema.safeParse(data)` nu aruncă niciodată. Întoarce `{ success: true, data }` sau `{ success: false, error }`, care e un discriminated union.

**Exemplu minim** (P1: răspunsul de la Open Library)
```ts
import * as z from "zod";

const SearchResponseSchema = z.object({
  numFound: z.number(),
  docs: z.array(
    z.object({
      key: z.string(),
      title: z.string(),
      author_name: z.array(z.string()).optional(),
      first_publish_year: z.number().int().optional(),
      cover_i: z.number().optional(),
    }),
  ),
});
type SearchResponse = z.infer<typeof SearchResponseSchema>;

async function searchBooks(query: string): Promise<SearchResponse> {
  const res = await fetch(
    `https://openlibrary.org/search.json?q=${encodeURIComponent(query)}&limit=10`,
  );
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return SearchResponseSchema.parse(await res.json()); // throws if the shape is wrong
}
```

**Când pe care.** `parse` când o formă greșită e o eroare reală, care trebuie să urce (răspunsul API-ului, variabilele de mediu la pornire). `safeParse` când eșecul e un caz normal pe care îl tratezi (input de la utilizator, un `localStorage` care poate fi corupt).

<details>
<summary>Întrebare de interviu: Parse vs safeParse?</summary>

`parse` întoarce datele validate și tipate sau aruncă un `ZodError`, așa că îl folosesc când datele invalide înseamnă o eroare care trebuie să oprească fluxul. `safeParse` nu aruncă: întoarce un obiect cu `success: true` și `data`, sau `success: false` și `error`, iar TypeScript restrânge tipul după `success`. Îl folosesc pentru input de la utilizator, unde eroarea de validare e un rezultat normal pe care vreau să-l afișez.
</details>

## Formulare: transformări și mesaje de eroare

**Ce e.** zod poate și transforma datele în timpul validării (curăță spațiile, transformă string în number) și produce mesaje de eroare pe câmpuri.

**Exemplu minim**
```ts
import * as z from "zod";

const ClientFormSchema = z.object({
  name: z.string().trim().min(2, { error: "Name is too short" }),
  email: z.email({ error: "Invalid email" }),
  vatRate: z.coerce.number().min(0).max(1), // form inputs are strings → coerce to number
});

const result = ClientFormSchema.safeParse({ name: "A", email: "nope", vatRate: "0.21" });
if (!result.success) {
  console.log(z.flattenError(result.error).fieldErrors);
  // { name: ["Name is too short"], email: ["Invalid email"] }
} else {
  console.log(result.data); // { name: "...", email: "...", vatRate: 0.21 } — number, trimmed
}
```

**Cum îl vezi în cod real.** În formularele React, zod se leagă de obicei de `react-hook-form` printr-un „resolver”, ca erorile să apară automat sub câmpuri. Pentru log-uri și debugging există `z.prettifyError(error)`, care dă un text lizibil.

**Capcana.** `z.coerce.number()` transformă orice prin `Number(...)`, deci un string gol devine `0`. Pentru câmpuri obligatorii verifică explicit și cazul ăsta.

## Variabilele de mediu

**Ce e.** `process.env` e un obiect cu string-uri sau `undefined`, adică exact genul de date necontrolate. Le validezi **o singură dată, la pornirea aplicației**, ca o configurație greșită să oprească imediat aplicația, cu un mesaj clar.

**Exemplu minim**
```ts
import * as z from "zod";

const EnvSchema = z.object({
  DATABASE_URL: z.url(),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

export const env = EnvSchema.parse(process.env); // crash at startup, not at 3 AM on the first request
```

**De ce contează.** Fără asta, o variabilă lipsă pe server (pe Render, de exemplu) se descoperă abia când un utilizator ajunge pe ruta care o folosește. Mai multe despre env vars în [10-securitate-env/01-env-vars.md](../10-securitate-env/01-env-vars.md). În Nest, `@nestjs/config` are mecanismul lui de validare, cu aceeași idee: [07-backend-node-nest/04-config-si-env.md](../07-backend-node-nest/04-config-si-env.md).

## Output-ul unui LLM e tot o graniță

**Ce e.** În P4, Claude citește o factură și întoarce liniile ei într-o structură. Chiar dacă îi ceri un format exact, tratezi răspunsul ca date externe: îl validezi cu aceeași schemă pe care o folosește restul aplicației.

**Exemplu minim**
```ts
import * as z from "zod";

const InvoiceLineSchema = z.object({
  product: z.string().min(1),
  quantity: z.number().positive(),
  unit: z.enum(["kg", "l", "pcs"]),
  unitPriceCents: z.number().int().nonnegative(),
});
const ExtractedInvoiceSchema = z.object({
  supplier: z.string(),
  lines: z.array(InvoiceLineSchema),
});
type ExtractedInvoice = z.infer<typeof ExtractedInvoiceSchema>;
```

**Cum îl vezi în cod real.** Aceeași schemă descrie formatul cerut modelului, validează răspunsul și dă tipul pentru UI-ul în care omul confirmă liniile. Detaliile despre structured output sunt în brief-ul [P4](../proiecte/P4-kitchen-ai.md).

**Capcana.** Să presupui că modelul respectă mereu formatul pentru că „așa i-ai cerut în prompt”. Validarea e ieftină, iar o linie de factură cu prețul greșit ajunge direct în costurile rețetelor.

<details>
<summary>Întrebare de interviu: De ce ai nevoie de zod dacă ai deja TypeScript?</summary>

TypeScript verifică doar la compilare și nu știe nimic despre datele care intră la runtime. zod validează datele reale în momentul în care intră în aplicație: răspunsuri de API, formulare, variabile de mediu, output de LLM. Din aceeași schemă obțin și tipul cu `z.infer`, așa că validarea și tipul nu se pot desincroniza. Combinația îmi dă siguranță la compilare în interior și siguranță la runtime la granițe.
</details>

## Mai departe

- [zod — Basic usage](https://zod.dev/basics)
- [zod — Defining schemas](https://zod.dev/api)
- [zod — Formatting errors](https://zod.dev/error-formatting)
