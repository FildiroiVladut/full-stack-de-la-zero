# RLS, Auth, Storage

Trei piese din Supabase care țin de același lucru: **cine are voie la ce**. Row Level Security decide ce rânduri vede fiecare, Auth îți spune cine e user-ul, iar Storage ține fișierele, cu aceleași reguli. Le folosești în P3 (Auth) și P4 (toate trei).

## Row Level Security (RLS)

**Ce e.** O funcție a Postgres prin care fiecare tabel poate avea **politici**: reguli care decid, rând cu rând, ce poate citi sau scrie un anumit user. Când RLS e activat pe un tabel, implicit nimeni nu vede nimic prin Data API. Vezi doar ce permit politicile.

**De ce contează în Supabase.** Data API (folosit de `supabase-js`) e o ușă directă spre tabele, iar cheia publishable cu care intri e publică (vezi [04](04-conectare-si-chei.md)). Singurul lucru dintre „oricine de pe internet” și datele tale sunt politicile RLS. De aceea, în P4, unde frontend-ul vorbește direct cu baza de date, RLS e securitatea aplicației.

**Exemplu minim** — în P4, fiecare bucătar își vede și își modifică doar rețetele lui:

```sql
alter table public.recipes enable row level security;

create policy "Users read own recipes" on public.recipes
  for select to authenticated
  using ( (select auth.uid()) = user_id );

create policy "Users create own recipes" on public.recipes
  for insert to authenticated
  with check ( (select auth.uid()) = user_id );

create policy "Users update own recipes" on public.recipes
  for update to authenticated
  using ( (select auth.uid()) = user_id )        -- which existing rows you may touch
  with check ( (select auth.uid()) = user_id );  -- what the row may look like after the update

create index on public.recipes (user_id); -- every policy filters by user_id
```

**Ce să observi.**

- `auth.uid()` e id-ul user-ului logat, luat din JWT-ul trimis de `supabase-js`. Scris ca `(select auth.uid())`, e calculat o singură dată pe query, nu pe fiecare rând, și e mult mai rapid pe tabele mari.
- `to authenticated` spune pentru cine e politica: user-ii logați. Singur, `to authenticated` nu restrânge nimic, ci înseamnă „orice user logat vede tot”. Condiția de ownership din `using` face restricția.
- **`using`** filtrează rândurile existente (ce poți citi, modifica, șterge). **`with check`** validează rândul nou sau modificat. Fără `with check` la update, un user și-ar putea muta rețeta pe `user_id`-ul altcuiva.

**Capcana.** Un `UPDATE` are nevoie și de o politică de `SELECT`: Postgres trebuie întâi să „vadă” rândul. Fără ea, update-ul nu dă nicio eroare, doar afectează 0 rânduri. Dacă „salvarea nu face nimic, dar nici nu dă eroare”, verifică politicile.

<details>
<summary>Întrebare de interviu: Ce e Row Level Security?</summary>

E un mecanism Postgres prin care atașezi unui tabel politici care decid, pentru fiecare rând, dacă user-ul curent îl poate citi, insera, modifica sau șterge. În Supabase, user-ul vine din JWT-ul trimis de client, iar politicile folosesc `auth.uid()` ca să compare cu coloana de ownership. Avantajul e că regula stă în baza de date, deci se aplică indiferent de unde vine request-ul. Dacă RLS e activat și nu există politici, nimeni nu vede nimic prin API.
</details>

## RLS când doar backend-ul accesează baza de date

**Situația din P2 și P3.** Frontend-ul nu vorbește cu Supabase pentru date, ci doar cu Nest. Nest intră prin connection string ca utilizatorul `postgres`, iar pe Supabase acest rol **ocolește RLS** (la fel ca cheia secret). Deci RLS nu-ți încurcă backend-ul.

**De ce activezi totuși RLS.** Tabelele din schema `public` pot fi accesibile prin Data API oricui are URL-ul proiectului și cheia publishable, și în P3 cheia asta e chiar în frontend, pentru login. Dacă un tabel e expus prin Data API și n-are RLS, oricine îl poate citi și modifica fără să treacă prin Nest.

Supabase a schimbat în 2026 regula implicită: pe proiectele noi, tabelele noi din `public` **nu mai sunt expuse automat** prin Data API, iar de la 30 octombrie 2026 regula se aplică tuturor proiectelor. E mai sigur, dar ce e expus depinde de setările proiectului, iar setările se pot schimba. Regula profesionistă:

- **în `public`, RLS activat pe fiecare tabel, obligatoriu.** E cazul P4, unde chiar folosești Data API;
- **în schemele proprii** (`invoicer` pentru P2, `booking` pentru P3, vezi [03](03-cont-supabase.md#planul-două-proiecte-supabase-în-total)), tabelele nu sunt expuse implicit prin Data API. Activezi totuși RLS, ca apărare în plus: dacă cineva adaugă vreodată schema la cele expuse, datele rămân închise.

**Cum.** Activezi RLS **fără politici**, printr-o migrație scrisă de mână (pașii în [05](05-migratii.md#regula-de-aur-schema-se-schimbă-doar-prin-migrații)):

```sql
alter table invoices enable row level security;
-- no policies: Data API sees nothing, Nest (as postgres) works normally
```

**Verifică.** În dashboard, **Advisors → Security Advisor** nu mai arată avertismentul „RLS disabled” pentru tabelele tale. Uită-te acolo după fiecare migrație.

**Capcana.** Când un feature „nu merge” din cauza RLS, AI-ul propune uneori `disable row level security` sau o politică `using (true)`. Asta deschide tabelul pentru toată lumea. Soluția corectă e o politică precisă, sau mutarea operației pe server, unde rulează cu drepturi controlate.

## Supabase Auth

**Ce e.** Serviciul care gestionează conturile: înregistrare, login, resetare de parolă, login cu Google, sesiuni. User-ii stau în tabelul `auth.users`, într-o schemă gestionată de Supabase, pe care nu o modifici direct.

**Cum arată o sesiune.** La login, Supabase îi dă frontend-ului două token-uri:

- **access token:** un JWT cu durată scurtă (implicit o oră), care conține id-ul user-ului (`sub`), `role` și `email`. Ăsta se trimite la fiecare request;
- **refresh token:** cu el, `supabase-js` cere automat un access token nou înainte să expire.

**Fluxul din P3** (frontend React → Nest):

```ts
// frontend: login once
await supabase.auth.signInWithPassword({ email, password });

// frontend: every API call sends the current access token
const { data: { session } } = await supabase.auth.getSession();
const res = await fetch(`${API_URL}/me/bookings`, {
  headers: { Authorization: `Bearer ${session?.access_token}` },
});
```

În Nest, guard-ul verifică semnătura token-ului cu cheile publice ale proiectului (JWKS). Codul e în [07/06](../07-backend-node-nest/06-guards-si-auth.md).

**Profilul și rolul.** `auth.users` ține doar identitatea. Ce ține de aplicație (nume afișat, rol `admin`/`member`) stă în tabelul tău `profiles`, cu `id` egal cu id-ul user-ului din Auth. Varianta simplă pentru P3: Nest face `upsert` pe profil la primul request autentificat, cu rol implicit `member`. Pe primul admin îl setezi de mână în Table Editor. (Alternativa e un trigger SQL pe `auth.users`, pe care îl vei vedea în documentația Supabase. Merge, dar e mai mult SQL de înțeles.)

**Capcana.** **Nu pune rolul în `user_metadata`.** User-ul își poate modifica singur metadata din frontend (`supabase.auth.updateUser({ data: { role: 'admin' } })`), deci s-ar face singur admin. Rolurile stau în tabelul tău, sau în `app_metadata`, care poate fi modificat doar de pe server. A doua capcană: `getSession()` pe frontend îți dă token-ul ca să-l trimiți, nu ca să ai încredere în el. Pe server, încrederea vine doar din verificarea semnăturii.

**De știut pentru dezvoltare.** Serviciul de email implicit al Supabase (confirmări, resetări de parolă) are limite mici de trimitere și e gândit pentru teste, nu pentru producție. Dacă nu-ți vin emailurile de confirmare când testezi, asta e cauza, nu codul tău.

<details>
<summary>Întrebare de interviu: Unde ții rolurile user-ilor și de ce?</summary>

În baza de date, într-un tabel de profiluri controlat de aplicație, sau în metadata pe care doar serverul o poate modifica, cum e `app_metadata` în Supabase. Niciodată într-un loc pe care user-ul îl poate scrie singur, cum e `user_metadata`, pentru că și-ar putea acorda singur drepturi. Pe server citesc rolul din sursa de încredere la fiecare request protejat, într-un guard. Dacă îl pun în JWT, țin cont că se actualizează abia la următorul refresh de token.
</details>

## Storage

**Ce e.** Locul pentru fișiere: poze, PDF-uri, facturile scanate din P4. Fișierele stau în **bucket-uri**, adică foldere de nivel superior. Accesul e controlat tot prin politici RLS, pe tabelul `storage.objects`.

**Două feluri de bucket:**

- **public:** oricine are URL-ul vede fișierul. Bun pentru poze de produs, avataruri;
- **privat:** accesul trece prin politici, iar pentru afișare generezi un **signed URL**, un link temporar care expiră. Facturile din P4 merg într-un bucket privat, pentru că sunt documente financiare.

**Exemplu din P4** — fiecare user urcă și citește fișiere doar din folderul lui (`invoices/<user_id>/...`):

```sql
create policy "Users upload own invoices" on storage.objects
  for insert to authenticated
  with check ( bucket_id = 'invoices' and (storage.foldername(name))[1] = (select auth.uid())::text );

create policy "Users read own invoices" on storage.objects
  for select to authenticated
  using ( bucket_id = 'invoices' and (storage.foldername(name))[1] = (select auth.uid())::text );
```

```ts
// upload from the browser, into the user's own folder
const path = `${user.id}/${crypto.randomUUID()}.pdf`;
const { error } = await supabase.storage.from('invoices').upload(path, file);

// temporary link valid for 60 seconds, to show or download the file
const { data } = await supabase.storage.from('invoices').createSignedUrl(path, 60);
```

**Capcana.** Ca să **înlocuiești** un fișier existent (upload cu `upsert: true`), ai nevoie de politici pentru `insert`, `select` **și** `update`. Cu doar `insert`, upload-ul nou merge, dar înlocuirea pică în tăcere. Și ține cont de limita planului Free: 50 MB per fișier.

## Mai departe

- [Supabase — Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supabase — Auth](https://supabase.com/docs/guides/auth)
- [Supabase — Storage access control](https://supabase.com/docs/guides/storage/security/access-control)
