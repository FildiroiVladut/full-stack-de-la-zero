# 02 · Cum funcționează web-ul

Înainte de React și NestJS trebuie să știi ce se întâmplă între browser și server: cine trimite ce, în ce format, unde rulează fiecare linie de cod și de ce unele lucruri nu au voie în browser. E capitolul care face diferența între „știu să generez cod cu AI” și „înțeleg ce a generat”. Conceptele de aici apar în fiecare proiect și în aproape orice interviu tehnic.

## La final poți să…

- explici drumul unui click de la buton până în baza de date și înapoi, fără notițe;
- citești un request și un response în DevTools și le reproduci cu `curl`;
- alegi metoda HTTP și status code-ul corect pentru un endpoint;
- spui de ce o cheie secretă nu are voie în frontend și unde trebuie mutată;
- explici ce e CORS și de ce nu e o măsură de securitate pentru API-ul tău.

## Fișierele, în ordine

- [ ] [01-client-server.md](01-client-server.md): client, server, URL, DNS, localhost și porturi, ce rulează unde (~40 min)
- [ ] [02-http.md](02-http.md): request/response, metode, status codes, headers, JSON, DevTools Network, `curl` (~50 min)
- [ ] [03-api-rest.md](03-api-rest.md): resurse, CRUD, path vs query, paginare, erori, idempotență (~40 min)
- [ ] [04-frontend-vs-backend.md](04-frontend-vs-backend.md): cine ce face, drumul complet al unui request, ce e public, SPA vs SSR (~45 min)
- [ ] [05-auth-cookies-cors.md](05-auth-cookies-cors.md): sesiuni vs JWT, cookies, unde ții token-ul, CORS, preflight (~50 min)

Total: ~4 ore, plus timpul în care te joci cu DevTools și `curl`.

## Unde îl folosești

- **P1 Shelf**: request-uri `GET` spre Open Library, URL-uri cu `URLSearchParams`, tab-ul Network.
- **P2 Invoicer**: tot capitolul. Primul tău API REST, prima eroare de CORS, prima diagramă click → baza de date.
- **P3 Booking**: JWT în headerul `Authorization`, `401` vs `403`, `409` la un slot ocupat.
- **P4 Kitchen AI**: cheia Claude API rămâne pe server, iar sesiunea stă în cookies cu Next.js.
- **[CP1](../checkpoints/CP1.md)**: întrebarea principală e drumul unui request de la click la baza de date.

## Verifică-te

1. Utilizatorul apasă „Șterge client”. Ce request pleacă, ce componente de pe server îl ating, în ce ordine, și ce status code se întoarce?
2. De ce merge un request cu `curl`, dar din browser dă eroare de CORS?
3. Ai o cheie de API de la un serviciu extern. Cum o folosești dintr-o aplicație React fără ca utilizatorii să o poată vedea?
4. Care e diferența dintre `401` și `403`? Dă un exemplu din P3.
5. De ce nu calculezi totalul unei facturi doar în frontend?
