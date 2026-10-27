# CP3 · Full-stack și primele aplicări

**Când:** finalul săptămânii 4 · **Acoperă:** [07](../07-backend-node-nest/) (01–05), [08](../08-baze-de-date-supabase/) (01–05), [10](../10-securitate-env/), [11](../11-deploy/) (03–04), [P2 Invoicer](../proiecte/P2-invoicer.md)

## Poți explica fără notițe

- [ ] Drumul complet al lui *Save invoice*: componentă → `fetch` → controller → DTO și ValidationPipe → service → Prisma → Postgres, și înapoi până în UI.
- [ ] Module, controller, service și dependency injection în Nest, pe codul tău.
- [ ] Ce e un DTO și ce face `whitelist` în ValidationPipe.
- [ ] Schema ta: tabelele, cheile primare și străine, relația factură–linii.
- [ ] Un `JOIN` și un `GROUP BY`, scrise de mână pe datele tale, în SQL Editor.
- [ ] Ce e o tranzacție și unde ai folosit-o.
- [ ] Ce e o migrație și ce rulează la deploy.
- [ ] De ce banii sunt întregi și cum faci rotunjirea la TVA.
- [ ] De ce totalul se calculează pe server (*never trust the client*).
- [ ] Ce variabile de mediu ai pe Render și pe Vercel, și de ce `DATABASE_URL` nu are voie în frontend.
- [ ] Cheia publishable vs cheia secret din Supabase, și de ce ai activat RLS pe tabele.
- [ ] Ce ai configurat la CORS și de ce.
- [ ] Ce faci în primele 10 minute dacă ai urcat un secret pe GitHub.

## Quiz cu Claude

Scor: \_\_ / 10 · De recitit: \_\_\_

## Livrabile

- [ ] Invoicer live: web _link_ · API `/docs` _link_ · repo _link_
- [ ] [Definition of done](../proiecte/README.md#definition-of-done-pentru-toate) bifat, inclusiv diagrama arhitecturii în README.
- [ ] Testele unitare pentru `computeInvoiceTotals`, scrise de tine.
- [ ] **Profil pregătit pentru aplicări:**
  - [ ] README de profil pe GitHub
  - [ ] Shelf și Invoicer pinned
  - [ ] CV în engleză
  - [ ] LinkedIn actualizat
- [ ] Primele 5 aplicări trimise, trecute în tabelul din [jurnal.md](../jurnal.md).
- [ ] Un mock interview cu Claude ([13-interviuri/07-mock-interviews.md](../13-interviuri/07-mock-interviews.md)).

## Call cu Bogdan

Call mai lung, de vreo 60 de minute: review pe Invoicer, apoi un mock interview, behavioral și pe proiect.

_Notițe:_
