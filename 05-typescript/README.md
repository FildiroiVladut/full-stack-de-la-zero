# 05 · TypeScript

În 2026, aproape orice proiect JavaScript profesionist e scris în TypeScript, iar în anunțurile de job apare ca cerință, nu ca bonus. Pentru tine e cel mai mare upgrade față de JavaScript-ul pe care îl știai: vii din C, deci ideea de tipuri îți e familiară, diferența fiind că aici tipurile există doar la compilare. Capitolul acoperă cât TypeScript îți trebuie ca să citești codul generat de Claude, să înțelegi erorile compilatorului și să nu cazi în capcanele clasice. Nu e o referință completă a limbajului.

## La final poți să

- explici ce verifică TypeScript și ce nu (tipurile dispar la runtime);
- citești și scrii tipuri pentru obiecte, răspunsuri de API, funcții și componente;
- folosești union types și narrowing ca să modelezi stări (de exemplu statusul unei facturi);
- recunoști generics și utility types (`Partial`, `Pick`, `Omit`, `Record`) în codul altora;
- validezi datele la granițele aplicației cu zod și derivi tipurile din aceeași schemă.

## Fișierele, în ordine

- [ ] [01 · De ce TypeScript](01-de-ce-typescript.md) — 40 min (include setup-ul de joacă)
- [ ] [02 · Tipurile de bază](02-tipuri-de-baza.md) — 60 min
- [ ] [03 · Narrowing și funcții](03-narrowing-si-functii.md) — 60 min
- [ ] [04 · Generics și utility types](04-generics-utility-types.md) — 45 min
- [ ] [05 · Validare cu zod](05-validare-cu-zod.md) — 45 min

Total: cam 4 ore.

## Unde îl folosești

- **Peste tot.** P1–P4 sunt scrise integral în TypeScript strict.
- **P1 Shelf:** tipuri pentru răspunsurile Open Library, un hook generic `useLocalStorage`, zod pe răspunsul API-ului.
- **P2 Invoicer:** discriminated union pentru statusul facturii, DTO-uri derivate cu `Omit`/`Partial`.
- **P4 Kitchen AI:** zod pentru output-ul structurat primit de la Claude.
- **[CP2](../checkpoints/CP2.md):** explici tipurile din P1 și de ce ai validat răspunsul API-ului.

## Verifică-te

1. Dacă backend-ul trimite `price` ca string în loc de number, te salvează TypeScript? De ce?
2. Când folosești `unknown` și când `any`? Care e riscul la `any`?
3. Cum modelezi o factură care, doar când e plătită, are `paidAt`?
4. Ce face `Omit<Client, "id">` și unde l-ai folosi?
5. Ce diferență e între `schema.parse()` și `schema.safeParse()` în zod?
