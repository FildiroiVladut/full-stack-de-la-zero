# CP4 · Conturi, teste și lucru în echipă

**Când:** finalul săptămânii 6 · **Acoperă:** [07](../07-backend-node-nest/) (06–07), [08/06](../08-baze-de-date-supabase/06-rls-auth-storage.md), [09](../09-git-github-echipa/) (02–06), [P3 Booking](../proiecte/P3-booking.md)

## Poți explica fără notițe

- [ ] Authentication vs authorization, pe exemple din codul tău.
- [ ] Drumul login-ului: Supabase Auth → JWT → header `Authorization` → guard în Nest → user și rol.
- [ ] Ce conține un JWT, cine îl semnează, cum se verifică, și de ce nu pui date secrete în el.
- [ ] Ce e IDOR și unde te-ai protejat de el în Booking.
- [ ] Cum garantezi că nu există două rezervări suprapuse, chiar la request-uri simultane, și de ce un `if` în cod nu ajunge.
- [ ] UTC vs ora locală: cum stochezi și cum afișezi.
- [ ] Ce verifică testele tale unit și e2e, ce ai pus pe mock și ce nu.
- [ ] Ce rulează CI-ul, când rulează, și ce se întâmplă dacă pică.
- [ ] Fluxul issue → branch → PR → review → merge.
- [ ] Merge vs rebase vs squash.
- [ ] Cum ai rezolvat conflictul de merge.
- [ ] Ce faci dacă ai făcut commit pe `main` din greșeală.

## Quiz cu Claude

Scor: \_\_ / 10 · De recitit: \_\_\_

## Livrabile

- [ ] Booking live: _link_ · repo: _link_
- [ ] [Definition of done](../proiecte/README.md#definition-of-done-pentru-toate) bifat.
- [ ] CI verde, iar `main` e protejat (merge doar prin PR).
- [ ] Cel puțin 3 PR-uri cu review de la Bogdan, la care ai răspuns.
- [ ] Un conflict de merge rezolvat, cu linkul spre PR.
- [ ] Test e2e care dovedește că suprapunerea întoarce 409.
- [ ] Aplicări: 5–10 pe săptămână, toate în jurnal.
- [ ] Un mock interview tehnic cu Claude.

## Call cu Bogdan

_Notițe:_
