# 06 · React

React e biblioteca de UI pe care o cer cele mai multe anunțuri de frontend și full-stack. P1, P2 și P3 au frontend în React + Vite, iar P4 e în Next.js, care e construit peste React. Scopul capitolului nu e să memorezi API-uri: Claude le scrie mai repede decât tine. Scopul e să ai modelul mental corect, ca să poți citi codul generat, să vezi când e greșit și să-l conduci spre ce vrei.

## La final poți să…

- explici ce declanșează un re-render și de ce asta nu e o problemă în sine;
- citești un arbore de componente și să spui unde stau datele (state) și pe unde circulă (props);
- recunoști un `useEffect` inutil sau greșit în codul generat de AI;
- legi un frontend de un API cu TanStack Query, cu toate cele patru stări: loading, error, empty, success;
- explici diferența dintre un SPA făcut cu Vite și o aplicație Next.js (Server vs Client Components).

## Fișierele, în ordine

- [ ] [01 · Cum gândește React](01-cum-gandeste-react.md) — 45 min
- [ ] [02 · Props și state](02-props-si-state.md) — 60 min
- [ ] [03 · Liste, condiții, formulare](03-liste-conditii-formulare.md) — 45 min
- [ ] [04 · Efecte](04-efecte.md) — 60 min
- [ ] [05 · Hooks și compoziție](05-hooks-si-compozitie.md) — 60 min
- [ ] [06 · Routing și date](06-routing-si-date.md) — 60 min
- [ ] [07 · Styling](07-styling.md) — 20 min
- [ ] [08 · Next.js](08-nextjs.md) — 60 min (în săptămâna 7, înainte de P4)

Fișierele 01–07 le citești în săptămâna 2, în paralel cu P1. Fișierul 08 îl lași pentru P4.

## Unde îl folosești

- **[P1 Shelf](../proiecte/P1-shelf.md):** tot, de la 01 la 07. E primul tău proiect React.
- **[P2 Invoicer](../proiecte/P2-invoicer.md) și [P3 Booking](../proiecte/P3-booking.md):** frontend-ul, mai ales 03 (formulare) și 06 (TanStack Query peste API-ul tău Nest).
- **[P4 Kitchen AI](../proiecte/P4-kitchen-ai.md):** 08, Next.js.
- **Checkpoint-uri:** [CP2](../checkpoints/CP2.md) (React) și [CP5](../checkpoints/CP5.md) (Next.js).

## Verifică-te

1. Ce se întâmplă, pas cu pas, de la click până pe ecran, când apeși un buton care face `setCount(count + 1)`?
2. De ce e o idee proastă să folosești indexul ca `key` într-o listă de rezervări pe care o poți sorta și din care poți șterge?
3. Ai un câmp `fullName` în state, ținut la zi cu un `useEffect` din `firstName` și `lastName`. Ce e greșit și cum repari?
4. Ce probleme rezolvă TanStack Query pe care un `fetch` pus într-un `useEffect` nu le rezolvă?
5. În Next.js, de ce o componentă cu `useState` are nevoie de `"use client"`? Și ce valoare are în ea o variabilă de mediu fără prefixul `NEXT_PUBLIC_`?
