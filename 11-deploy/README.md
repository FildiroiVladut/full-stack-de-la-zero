# 11 · Deploy

O aplicație care rulează doar pe `localhost` nu există pentru un recrutor. Capitolul ăsta te învață să-ți pui proiectele online, gratuit, cu deploy automat la fiecare push. Faci primul deploy încă din săptămâna 2, cu P1, și din momentul ăla fiecare proiect are un link live în README.

## La final poți să…

- explici diferența dintre build și run și ce fel de hosting cere fiecare proiect;
- pui un frontend Vite pe Vercel, cu routing care merge și la refresh;
- pui un API NestJS pe Render, conectat la Supabase și accesibil din frontend-ul de pe Vercel (CORS corect);
- configurezi variabilele de mediu pe fiecare mediu (local, preview, production) fără să scapi vreun secret;
- găsești în loguri cauza unei erori care apare doar în producție.

## Fișierele, în ordine

- [ ] [01 · Ce înseamnă deploy](01-ce-inseamna-deploy.md) — 30 min
- [ ] [02 · Vercel](02-vercel.md) — 45 min (+ deploy-ul lui P1)
- [ ] [03 · Render](03-render.md) — 60 min (+ deploy-ul API-ului din P2)
- [ ] [04 · Loguri și producție](04-loguri-si-productie.md) — 30 min

01 și 02 le citești în săptămâna 2, pentru P1. 03 și 04 în săptămânile 3–4, pentru P2.

## Unde îl folosești

- **[P1 Shelf](../proiecte/P1-shelf.md):** 01, 02. Un site static pe Vercel.
- **[P2 Invoicer](../proiecte/P2-invoicer.md) și [P3 Booking](../proiecte/P3-booking.md):** tot capitolul. Frontend pe Vercel, Nest pe Render, bază de date pe Supabase.
- **[P4 Kitchen AI](../proiecte/P4-kitchen-ai.md):** 02 (Next.js pe Vercel) și 04.
- **Checkpoint-uri:** [CP2](../checkpoints/CP2.md) (prima aplicație live) și [CP3](../checkpoints/CP3.md) (full-stack live).

## Verifică-te

1. De ce frontend-ul din P2 poate sta pe un CDN, iar API-ul Nest nu?
2. Ai schimbat o variabilă de mediu în Vercel, dar site-ul se comportă la fel. De ce?
3. Deschizi direct `https://shelf.vercel.app/books/OL123W` și primești 404, deși din aplicație pagina merge. Ce lipsește?
4. API-ul de pe Render merge cu curl, dar din frontend primești o eroare CORS. Unde te uiți și ce schimbi?
5. Primul request la API-ul tău durează aproape un minut, următoarele sunt instantanee. De ce, și cum previi un recrutor?
