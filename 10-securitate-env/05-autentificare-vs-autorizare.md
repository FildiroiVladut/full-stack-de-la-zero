# Autentificare vs autorizare

Cel mai frecvent bug de securitate din codul scris de juniori, și din codul generat de AI, nu e o parolă slabă. E un endpoint care verifică *că* ești logat, dar nu verifică *dacă* ai voie să vezi ce ceri. Fișierul ăsta e despre diferența asta.

## Autentificare vs autorizare

**Ce e.**

- **Autentificare (authn):** cine ești? Login cu email și parolă, magic link, Google. Rezultatul e un utilizator identificat, de obicei printr-un token.
- **Autorizare (authz):** ce ai voie să faci? Ai voie să vezi factura 123? Ai voie să ștergi o rezervare? Ai voie în panoul de admin?

Seamănă cu ecusonul de la intrarea în restaurant și cheia de la camera frigorifică. Ecusonul arată că ești angajat, dar nu orice angajat are cheia de la frigider.

**Status codes:**

| Cod | Înseamnă | Exemplu |
|---|---|---|
| **401 Unauthorized** | nu știu cine ești | lipsește token-ul sau e expirat |
| **403 Forbidden** | știu cine ești, dar nu ai voie | un `member` încearcă o rută de `admin` |
| **404 Not Found** | nu există sau nu-ți arăt că există | factura altui utilizator |

**Capcana.** Numele lui 401 („Unauthorized”) e istoric și induce în eroare. De fapt înseamnă „neautentificat”.

<details>
<summary>Întrebare de interviu: Care e diferența dintre autentificare și autorizare?</summary>

Autentificarea stabilește cine e utilizatorul: login, verificarea token-ului. Autorizarea stabilește ce are voie să facă acel utilizator cu o anumită resursă: dacă factura e a lui, dacă are rol de admin. Lipsa autentificării dă 401. Lipsa dreptului dă 403, sau 404 dacă nu vreau să confirm că resursa există. Autorizarea se verifică pe server, la fiecare cerere, pentru fiecare resursă.
</details>

## IDOR: cel mai frecvent bug de junior

**Ce e.** IDOR vine de la *Insecure Direct Object Reference*. Un endpoint primește ID-ul unui obiect și îl întoarce fără să verifice că obiectul îi aparține utilizatorului care cere. Orice utilizator logat poate schimba ID-ul din URL și vede datele altcuiva.

**Exemplu minim: VULNERABIL**

```ts
// invoices.controller.ts
@UseGuards(AuthGuard)          // checks the user is logged in... and that's all
@Get(':id')
findOne(@Param('id') id: string) {
  return this.prisma.invoice.findUnique({ where: { id } });
  // any logged-in user can read ANY invoice by changing the id
}
```

**Reparat**

```ts
// invoices.controller.ts
@UseGuards(AuthGuard)
@Get(':id')
findOne(@Param('id') id: string, @CurrentUser() user: AuthUser) {
  return this.invoices.findOneForUser(id, user.id);
}

// invoices.service.ts
async findOneForUser(id: string, userId: string) {
  const invoice = await this.prisma.invoice.findFirst({
    where: { id, userId },        // ownership is part of the query
  });
  if (!invoice) throw new NotFoundException(); // 404: don't reveal that it exists
  return invoice;
}
```

`@CurrentUser()` e un decorator scris de tine, care scoate utilizatorul din request după ce guard-ul a verificat token-ul. Vezi [07-backend-node-nest/06-guards-si-auth.md](../07-backend-node-nest/06-guards-si-auth.md).

**De ce contează.** Broken Access Control e pe primul loc în [OWASP Top 10](04-validare-si-atacuri.md#owasp-top-10-varianta-scurtă), iar IDOR e forma lui cea mai des întâlnită. E și prima întrebare de securitate pe care o pune un senior la review.

**Capcana.** „ID-urile sunt UUID-uri, nu le poate ghici nimeni.” ID-urile apar în URL-uri, în loguri, în răspunsuri de la API, în linkuri trimise pe chat. Faptul că un ID e greu de ghicit nu înseamnă autorizare.

<details>
<summary>Întrebare de interviu: Ce e IDOR și cum îl previi?</summary>

Insecure Direct Object Reference: un endpoint întoarce sau modifică o resursă după ID-ul primit, fără să verifice că utilizatorul curent are drept asupra ei. Îl previn punând verificarea de ownership chiar în interogare, de exemplu `where: { id, userId }`, pentru fiecare citire, modificare și ștergere, nu doar pentru citire. Listele le filtrez mereu după utilizator, iar dacă resursa nu e a lui răspund cu 404. Ca a doua linie de apărare pot folosi Row Level Security în baza de date.
</details>

## Verificarea de ownership, peste tot

**Ce e.** Regula din fișa de mai sus aplicată consecvent: **orice** operație pe o resursă care aparține cuiva verifică proprietarul.

**Cum îl vezi în cod real** (P3 Booking)

```ts
// list: always filtered by the current user
findMine(userId: string) {
  return this.prisma.booking.findMany({ where: { userId }, orderBy: { startsAt: 'asc' } });
}

// cancel: members can cancel their own bookings, admins can cancel any
async cancel(bookingId: string, user: AuthUser) {
  const booking = await this.prisma.booking.findUnique({ where: { id: bookingId } });
  if (!booking) throw new NotFoundException();
  if (booking.userId !== user.id && user.role !== 'admin') {
    throw new NotFoundException(); // or ForbiddenException, if your team prefers
  }
  return this.prisma.booking.delete({ where: { id: bookingId } });
}
```

**Capcana.** Verifici ownership-ul la `GET`, dar îl uiți la `PATCH` și `DELETE`. La review te uiți la **fiecare** metodă care primește un ID.

## Roluri

**Ce e.** Un rol (`admin`, `member`) e o etichetă care spune ce categorie de acțiuni are voie un utilizator să facă. Rolul se stochează **pe server** și se verifică **pe server**.

**Cum îl vezi în cod real.** În P3, rolul stă în tabelul `profiles`, pe care utilizatorul nu-l poate modifica. Un guard Nest îl verifică pe rutele protejate:

```ts
@Roles('admin')
@UseGuards(AuthGuard, RolesGuard)
@Post('resources')
createResource(@Body() dto: CreateResourceDto) { /* ... */ }
```

Implementarea guard-ului e în [07-backend-node-nest/06-guards-si-auth.md](../07-backend-node-nest/06-guards-si-auth.md).

**Capcana, triplă:**

1. **Butonul de admin ascuns în UI nu e autorizare.** E doar UX. Oricine trimite cererea direct cu `curl`.
2. **Rolul nu se ia niciodată din body, din localStorage sau din ceva controlat de client.**
3. **La Supabase, `user_metadata` poate fi modificat de utilizator.** Documentația spune explicit să nu-l folosești pentru autorizare. Rolurile stau într-un tabel al tău, cum e `profiles`, sau în `app_metadata`, pe care utilizatorul nu-l poate schimba.

## Parolele nu le stochezi tu

**Ce e.** Parolele nu se salvează niciodată în clar și nici criptate reversibil. Se salvează un **hash**: o amprentă dintr-o singură direcție, calculată cu algoritmi lenți, făcuți special pentru asta (bcrypt, argon2), plus un *salt* unic pentru fiecare utilizator. La login se calculează hash-ul parolei introduse și se compară cu cel salvat.

**De ce contează.** E o întrebare clasică de interviu. Iar în practică **nu implementezi tu autentificarea**. Folosești Supabase Auth, care rezolvă deja hash-ul, confirmarea emailului, resetarea parolei, rate limiting-ul la login și login-ul cu Google.

**Capcana.** „Îmi fac eu un tabel `users` cu parole, e simplu.” E simplu până la resetarea parolei, la sesiunile expirate și la protecția împotriva ghicirii. Codul de autentificare scris de mână e una dintre cele mai sigure căi spre o breșă.

<details>
<summary>Întrebare de interviu: Cum stochezi parolele utilizatorilor?</summary>

Nu le stochez în clar și nici criptate. Stochez un hash calculat cu un algoritm lent, dedicat parolelor, cum sunt bcrypt sau argon2, cu salt unic per utilizator, ca două parole identice să aibă hash-uri diferite și atacurile cu tabele precalculate să nu funcționeze. Hash-ul e dintr-o singură direcție, iar criptarea s-ar putea inversa. În practică prefer un serviciu de autentificare matur, cum e Supabase Auth, în locul unei implementări proprii.
</details>

## Unde ții token-ul în browser

**Ce e.** După login, browserul primește un token (un JWT, vezi [02-cum-functioneaza-webul/05-auth-cookies-cors.md](../02-cum-functioneaza-webul/05-auth-cookies-cors.md)) pe care îl trimite la fiecare cerere. Îl poți ține în două locuri principale:

| | localStorage | Cookie `httpOnly` |
|---|---|---|
| Cine îl poate citi | orice JavaScript de pe pagină | doar serverul. JS-ul nu-l vede |
| Risc principal | XSS: un script injectat fură token-ul | CSRF: îl rezolvi cu `SameSite` și cu metode POST |
| Cum pleacă la server | îl pui tu în header-ul `Authorization` | browserul îl atașează automat |
| În proiectele tale | P3: supabase-js îl ține implicit aici | P4: `@supabase/ssr` folosește cookie-uri |

**De ce contează.** Nu există variantă perfectă, ci doar compromisuri. Interviul vrea să vadă că le cunoști. În ambele cazuri, token-urile de acces expiră repede și sunt reîmprospătate automat cu un *refresh token*.

**Capcana.** Să ții token-ul în localStorage și să ai și un `dangerouslySetInnerHTML` cu conținut de la utilizatori. Asta e rețeta completă pentru furtul sesiunii.

## RLS: a doua linie de apărare

**Ce e.** Row Level Security înseamnă reguli definite direct în Postgres: „un utilizator vede doar rândurile unde `user_id = auth.uid()`”. Chiar dacă ai un bug în API, baza de date refuză.

**În proiectele tale.**

- **P2 și P3:** backend-ul Nest se conectează prin Prisma cu un rol care **ocolește** RLS, deci autorizarea se face în Nest (fișele de mai sus). Activezi totuși RLS pe toate tabelele. Altfel, oricine are URL-ul proiectului și cheia publishable poate citi tabelele direct prin API-ul Supabase.
- **P4:** Next.js folosește supabase-js cu sesiunea utilizatorului, deci politicile RLS fac o parte reală din autorizare.

Detaliile, cu exemple de politici, sunt în [08-baze-de-date-supabase/06-rls-auth-storage.md](../08-baze-de-date-supabase/06-rls-auth-storage.md).

**Capcana.** Un tabel creat printr-o migrare Prisma nu are RLS activat automat. Supabase te avertizează în Security Advisor. Nu ignora avertismentul.

## Mai departe

- [Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) și [IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html): OWASP
- [Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security) și [User management](https://supabase.com/docs/guides/auth/managing-user-data): Supabase
