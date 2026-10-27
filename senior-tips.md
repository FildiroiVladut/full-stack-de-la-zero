# Senior tips

Lucruri pe care seniorii le știu din experiență, de obicei pentru că au plătit pentru ele. Recitește lista la începutul fiecărui proiect. Când înveți una pe pielea ta, adaug-o.

## Cod

1. **Codul se citește de zece ori mai des decât se scrie.** Scrie-l pentru cel care îl citește peste șase luni, adică de obicei pentru tine.
2. **Simplu și plictisitor bate deștept.** Dacă un one-liner are nevoie de un comentariu ca să fie înțeles, scrie-l pe trei rânduri.
3. **Numele bune sunt jumătate din documentație.** `unpaidInvoices` spune tot; `data2` nu spune nimic.
4. **Nu abstractiza la prima repetiție.** Abia la a treia.
5. **Banii sunt numere întregi** (bani, cenți), niciodată float. `0.1 + 0.2 !== 0.3`. → [04-javascript/06](04-javascript/06-js-in-cod-real.md)
6. **Datele se stochează în UTC și se afișează în ora locală.**
7. **Validează la granițe și ai încredere în interior.** Granițele sunt inputul de la user, răspunsurile de la API-uri și variabilele de mediu. → [05-typescript/05](05-typescript/05-validare-cu-zod.md)
8. **Ce poate fi calculat nu se stochează.** O valoare derivată ținută separat ajunge, mai devreme sau mai târziu, să nu mai corespundă.
9. **Șterge codul mort.** Git îl ține minte pentru tine.

## Debug

10. **Citește mesajul de eroare. Tot, de sus în jos.** Prima linie din stack trace care e din codul tău e de obicei vinovata.
11. **Reproduci înainte să repari.** Un bug pe care nu-l poți reproduce nu l-ai reparat, doar l-ai ascuns.
12. **Schimbă un singur lucru pe rând.**
13. **`console.log` e în regulă. Debugger-ul e mai bun.** Network tab-ul din DevTools e cel mai subestimat instrument din frontend.
14. **„Merge la mine” înseamnă aproape mereu:** alt env, altă versiune, alte date.
15. **Explică problema cu voce tare** (rubber duck), chiar și lui Claude, fără să-i ceri soluția. Multe bug-uri se rezolvă din explicație.
16. **Când ceri ajutor, spune trei lucruri:** ce ai vrut să se întâmple, ce s-a întâmplat (eroarea exactă, copiată) și ce ai încercat deja.

## Frontend

17. **Tot ce ajunge în browser e public,** inclusiv variabilele `VITE_` și `NEXT_PUBLIC_`. → [10-securitate-env/02](10-securitate-env/02-ce-ajunge-in-browser.md)
18. **Validarea din frontend e UX. Validarea din backend e securitate.**
19. **Orice ecran cu date are patru stări:** loading, eroare, gol, cu date. Juniorii o fac doar pe ultima.
20. **Nu ține în state ce poți calcula din alt state.**
21. **Majoritatea `useEffect`-urilor scrise de juniori (și de AI) nu trebuiau să existe.** → [06-react/04](06-react/04-efecte.md)

## Backend și baze de date

22. ***Never trust the client.*** Prețul, totalul, rolul și id-ul userului vin de pe server, nu din body-ul request-ului.
23. **Verifică ownership-ul la fiecare request.** „E logat” nu înseamnă „are voie la factura asta” (IDOR). → [10-securitate-env/05](10-securitate-env/05-autentificare-vs-autorizare.md)
24. **Regulile importante le impune baza de date:** `UNIQUE`, `CHECK`, `FOREIGN KEY`, constrângeri. Un `if` în cod nu te apără de două request-uri care vin simultan.
25. **`DELETE` și `UPDATE` fără `WHERE` au stricat mai multe weekenduri decât orice altceva.**
26. **Fiecare schimbare de schemă e o migrație în git.** Nimic nu se modifică de mână în producție. → [08-baze-de-date-supabase/05](08-baze-de-date-supabase/05-migratii.md)
27. **Aplicația trebuie să cadă la pornire dacă lipsește o variabilă de mediu,** nu la primul request al unui client.
28. **Paginare de la început** pe orice listă care poate crește.
29. **Index pe coloanele după care filtrezi des.** Nu pe toate: fiecare index încetinește scrierile.

## Securitate

30. **Ai urcat un secret pe GitHub? Îl rotești imediat.** Ștersul commit-ului nu ajunge, pentru că boții scanează repo-urile publice în câteva minute. → [10-securitate-env/03](10-securitate-env/03-secrete-scurse.md)
31. **`.env` intră în `.gitignore` din primul commit.** `.env.example` are toate cheile, cu valori false.
32. **CORS nu e securitate pentru API-ul tău.** Protejează utilizatorul din browser, nu serverul.
33. **Nu-ți scrie singur sistemul de autentificare.** Folosește unul testat de alții.
34. **Înainte să instalezi un pachet sugerat de AI, verifică-l pe npmjs.com:** există, e întreținut, are descărcări. AI-ul mai inventează nume de pachete, iar atacatorii le înregistrează exact pe alea.

## Git și echipă

35. **Commit-uri mici, cu mesaje care spun de ce, nu doar ce.**
36. **PR-urile mici primesc review adevărat.** Cele de 2000 de linii primesc „LGTM”.
37. **Fă-ți singur review la PR înainte să-l ceri altcuiva.** Prinzi jumătate din probleme.
38. **Nu rescrii istoria pe branch-uri pe care lucrează și alții.** Dacă trebuie, `--force-with-lease`, niciodată `--force`.
39. **Lockfile-ul intră în git. `node_modules` și `dist` nu.**
40. **Comentariile din review sunt despre cod, nu despre tine.** Mulțumești, întrebi ce nu înțelegi, repari.
41. **„Nu știu, verific și revin” e un răspuns de senior.** Să inventezi un răspuns e ce fac juniorii.

## AI

42. **AI-ul scrie codul repede, dar tu răspunzi pentru el.** Ce nu poți explica nu intră în commit.
43. **Cel mai bun prompt e un spec clar.** Dacă nu știi ce vrei, AI-ul decide în locul tău.
44. **Pași mici, verificați pe rând.** „Fă-mi toată aplicația” produce cod pe care nu-l înțelege nimeni.
45. **Când AI-ul se învârte în cerc pe un bug, oprește-l.** Dai `/clear` și descrii problema de la zero, sau citești tu documentația.
46. **AI-ul a învățat și din cod vechi.** Verifică dacă API-ul pe care îl folosește e cel de acum.
47. **Testele sunt plasa de siguranță când lași AI-ul să modifice cod.**

## Carieră

48. **Deploy din prima zi.** Un proiect care nu e online nu există pentru un recruiter.
49. **README-ul e vitrina:** screenshot, link live, ce face, cum rulezi, ce decizii ai luat.
50. **Aplici înainte să te simți pregătit.** Anunțurile sunt liste de dorințe, nu cerințe minime.
51. **Fiecare interviu e antrenament.** Notezi întrebările la care te-ai blocat și cauți răspunsul în aceeași zi.
52. **Mise en place:** înainte să scrii cod, citești tot ticket-ul și îți pregătești mediul. Ca înainte de serviciu.
