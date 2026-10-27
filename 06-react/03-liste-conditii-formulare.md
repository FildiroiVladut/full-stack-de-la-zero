# Liste, condiții, formulare

Trei lucruri pe care le vei scrie, sau citi, în fiecare ecran: liste de date, bucăți de UI afișate doar uneori și formulare. Sunt simple, dar au capcane pe care le întreabă toată lumea.

## Liste cu `map` și `key`

**Ce e.** Afișezi o listă transformând array-ul de date într-un array de elemente JSX cu `map`. Fiecare element primește o `key` unică și stabilă, ca React să știe care element e care între două render-uri.

**De ce contează.** Pe baza lui `key`, React decide ce elemente păstrează, ce mută și ce șterge. O cheie proastă înseamnă state-uri mutate pe elementul greșit, adică bug-uri foarte greu de explicat.

**Exemplu minim**

```tsx
type Booking = { id: string; room: string; startsAt: string };

export function BookingList({ bookings }: { bookings: Booking[] }) {
  return (
    <ul>
      {bookings.map((b) => (
        <li key={b.id}>
          {b.room}: {new Date(b.startsAt).toLocaleString("ro-RO")}
        </li>
      ))}
    </ul>
  );
}
```

**Cum îl vezi în cod real.** Cheia e aproape mereu id-ul din baza de date (`invoice.id`, `booking.id`). Pentru elemente create în frontend care nu au încă id, Claude generează unul cu `crypto.randomUUID()` *la creare*, nu la render.

**Capcana.** `key={index}`. Merge cât timp lista nu se reordonează și nu pierde elemente din mijloc. Dacă ștergi linia 2 dintr-o factură cu inputuri, linia 3 primește indexul 2 și moștenește state-ul liniei șterse: textul din input rămâne la elementul greșit. La fel de rău e `key={Math.random()}`: React recreează totul la fiecare render.

<details>
<summary>Întrebare de interviu: De ce are nevoie React de `key` în liste și de ce nu e bine indexul?</summary>

`key` îi spune lui React ce element din lista nouă corespunde cărui element din lista veche, ca să păstreze state-ul și DOM-ul corect la reordonare, inserare sau ștergere. Indexul se schimbă când lista se modifică, așa că React asociază greșit elementele: state-ul unui rând ajunge pe alt rând, iar inputurile își pierd valorile. Folosesc un id stabil, de obicei cel din baza de date.
</details>

## Randare condițională

**Ce e.** Afișezi sau ascunzi bucăți de UI în funcție de date. În JSX nu ai `if`, așa că folosești trei forme: `return` timpuriu, ternar `a ? b : c` și `&&`. `null` înseamnă „nu afișa nimic”.

**De ce contează.** Orice ecran real are cazuri: loading, eroare, gol, utilizator nelogat, buton dezactivat. Felul în care sunt scrise condițiile decide cât de ușor se citește componenta.

**Exemplu minim**

```tsx
export function InvoiceActions({ status }: { status: "draft" | "sent" | "paid" }) {
  if (status === "paid") return null; // early return

  return (
    <div>
      {status === "draft" ? <button>Send</button> : <button>Mark as paid</button>}
      {status === "sent" && <span>Waiting for payment</span>}
    </div>
  );
}
```

**Cum îl vezi în cod real.** Codul bun pune cazurile speciale la începutul componentei, cu `return` timpuriu (loading, eroare, gol), iar cazul principal la final. Ternarele imbricate pe trei niveluri sunt un semnal că trebuie extrasă o componentă.

**Capcana.** `{count && <Badge />}` când `count` e `0`. `0 && ...` are valoarea `0`, iar React afișează cifra `0` pe ecran. Scrie `{count > 0 && <Badge />}`. E o întrebare clasică de interviu și o greșeală frecventă în codul generat.

<details>
<summary>Întrebare de interviu: De ce `{items.length && <List />}` afișează uneori „0”?</summary>

Pentru că `&&` întoarce primul operand falsy. Dacă `items.length` e `0`, expresia are valoarea `0`, iar React afișează numerele, inclusiv `0`. (`false`, `null` și `undefined` nu sunt afișate.) Corect e o condiție booleană explicită, `items.length > 0 && ...`, sau un ternar.
</details>

## Cele patru stări ale unei liste

**Ce e.** Orice listă care vine de la server are patru stări: **loading** (încă nu știm), **error** (n-a mers), **empty** (a mers, dar nu e nimic) și **success** (avem date). Toate patru trebuie afișate explicit.

**De ce contează.** E primul lucru pe care îl verifică un senior la review pe un ecran nou și primul lucru care lipsește din proiectele de portofoliu ale juniorilor. Un ecran alb când API-ul e jos arată neterminat.

**Exemplu minim**

```tsx
type Props = { isPending: boolean; error: Error | null; invoices: Invoice[] | undefined };

export function InvoicesView({ isPending, error, invoices }: Props) {
  if (isPending) return <p>Loading invoices…</p>;
  if (error) return <p role="alert">Could not load invoices: {error.message}</p>;
  if (!invoices || invoices.length === 0) return <p>No invoices yet. Create your first one.</p>;

  return <InvoiceTable invoices={invoices} />;
}
```

**Cum îl vezi în cod real.** Cu TanStack Query, stările vin gata făcute (`isPending`, `isError`, `data`), vezi [06-routing-si-date](06-routing-si-date.md). Starea „empty” e de obicei un mic ecran cu un buton de acțiune („Create your first invoice”), nu doar un text.

**Capcana.** AI-ul uită aproape mereu starea *empty* și uneori pe cea de *error*. Când ceri un ecran nou, cere explicit: *„tratează loading, error și empty state”*.

<details>
<summary>Întrebare de interviu: Ce stări tratezi când afișezi date de la un API?</summary>

Loading, error, empty și success. La loading afișez un indicator sau un schelet. La error, un mesaj clar și, unde are sens, un buton de retry. La empty, un mesaj care spune ce poate face utilizatorul mai departe. Abia apoi vin datele. Să tratezi doar cazul fericit e cea mai frecventă scăpare în UI.
</details>

## Formulare: submit controlat

**Ce e.** Un `<form>` cu handler pe `onSubmit` și buton `type="submit"`. În handler oprești reîncărcarea paginii cu `e.preventDefault()`, validezi, trimiți și dezactivezi butonul cât timp request-ul e în curs.

**De ce contează.** Pune formularul într-un `<form>` real, nu doar câteva inputuri cu un `onClick` pe buton. Așa merge Enter pentru trimitere, merg cititoarele de ecran și merge validarea nativă a browserului.

**Exemplu minim**

```tsx
import { useState } from "react";

export function NewClientForm({ onCreate }: { onCreate: (name: string) => Promise<void> }) {
  const [name, setName] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await onCreate(name);
      setName("");
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="client-name">Client name</label>
      <input id="client-name" value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit" disabled={isSubmitting}>Save</button>
    </form>
  );
}
```

**Cum îl vezi în cod real.** Cu TanStack Query, `isSubmitting` vine din `mutation.isPending`, iar trimiterea e `mutation.mutate(data)`. Forma rămâne aceeași.

**Capcana.** Butonul care nu se dezactivează: utilizatorul apasă de trei ori și pleacă trei facturi identice. Și `<label>` fără `htmlFor`/`id`, care strică accesibilitatea.

<details>
<summary>Întrebare de interviu: De ce apelezi `e.preventDefault()` într-un submit handler?</summary>

Comportamentul implicit al unui `<form>` e să trimită datele și să reîncarce pagina. Într-un SPA vreau să trimit eu datele cu `fetch` și să rămân pe pagină, fără să pierd state-ul aplicației. `preventDefault` oprește acel comportament implicit.
</details>

## Validare cu zod în formular

**Ce e.** Descrii forma datelor valide o singură dată, cu o schemă zod, și o folosești la submit. Dacă datele nu trec, `z.flattenError` îți dă mesajele pe fiecare câmp. Ai văzut zod în [05-validare-cu-zod](../05-typescript/05-validare-cu-zod.md).

**De ce contează.** Validarea în frontend e pentru **experiența utilizatorului**: mesaje imediate, fără un drum la server. Nu e securitate. Backend-ul validează oricum, pentru că oricine poate trimite request-uri fără să treacă prin formularul tău (vezi [04-validare-si-atacuri](../10-securitate-env/04-validare-si-atacuri.md)).

**Exemplu minim**

```tsx
import * as z from "zod";

const clientSchema = z.object({
  name: z.string().trim().min(2, "Name is too short"),
  email: z.email("Invalid email"),
});

function validate(data: unknown) {
  const result = clientSchema.safeParse(data);
  if (result.success) return { data: result.data, errors: null };
  return { data: null, errors: z.flattenError(result.error).fieldErrors };
}
// errors?.email?.[0] → "Invalid email"
```

**Cum îl vezi în cod real.** Schema stă într-un fișier separat (`schemas.ts`) și e refolosită: la formular, la tipul TypeScript (`z.infer<typeof clientSchema>`) și uneori la validarea răspunsului de la API.

**Capcana.** Să consideri că e suficient să validezi în frontend. Nu e. Și să validezi abia la submit, fără să-i arăți utilizatorului *care* câmp e greșit: un singur mesaj „Invalid form” la final e frustrant.

<details>
<summary>Întrebare de interviu: Dacă validezi în frontend, mai e nevoie de validare în backend?</summary>

Da, întotdeauna. Validarea din frontend e pentru UX: feedback imediat, mai puține request-uri inutile. Orice client poate ocoli frontend-ul și trimite direct request-uri la API, cu curl, Postman sau un script, deci backend-ul trebuie să valideze și să respingă datele invalide. Ideal, regulile sunt definite o singură dată și refolosite.
</details>

## react-hook-form (ce vei vedea în proiecte)

**Ce e.** O bibliotecă pentru formulare care ține valorile, erorile și starea de trimitere fără să re-randeze tot formularul la fiecare tastă. Se combină cu zod printr-un „resolver”. E cea mai răspândită soluție de formulare din ecosistemul React.

**De ce contează.** Când îi ceri lui Claude un formular cu mai mult de 3–4 câmpuri, foarte probabil îl scrie cu react-hook-form. Trebuie să-l poți citi, chiar dacă nu-l scrii tu de la zero.

**Exemplu minim**

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import * as z from "zod";

const schema = z.object({ title: z.string().min(1, "Required"), pages: z.coerce.number().int().positive() });
type FormValues = z.infer<typeof schema>;

export function AddBookForm({ onSave }: { onSave: (v: FormValues) => void }) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(onSave)}>
      <input {...register("title")} />
      {errors.title && <p>{errors.title.message}</p>}
      <input type="number" {...register("pages")} />
      <button disabled={isSubmitting}>Save</button>
    </form>
  );
}
```

**Cum îl vezi în cod real.** Trei lucruri de recunoscut:

- `register("field")` leagă un input de formular;
- `handleSubmit(fn)` validează întâi și apelează `fn` doar cu date valide;
- `formState.errors` conține mesajele pe câmp.

**Capcana.** Inputurile de tip number întorc string-uri din DOM. Fără `z.coerce.number()` (sau `valueAsNumber`), primești `"12"` în loc de `12` și calculele ies greșite.

<details>
<summary>Întrebare de interviu: De ce ai folosi o bibliotecă de formulare în loc de useState pentru fiecare câmp?</summary>

Într-un formular mare, câte un `useState` pe câmp înseamnă mult cod repetitiv și un re-render la fiecare tastă. O bibliotecă precum react-hook-form gestionează valorile, erorile, câmpurile atinse și starea de trimitere, și se integrează cu o schemă de validare (zod). Primesc mai puțin cod, performanță mai bună și un tipar consistent în toată aplicația.
</details>

## Mai departe

- [Rendering Lists](https://react.dev/learn/rendering-lists)
- [Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [react-hook-form: Get Started](https://react-hook-form.com/get-started)
