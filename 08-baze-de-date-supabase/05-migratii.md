# Migrații

Codul îl versionezi cu git. Schema bazei de date (tabele, coloane, indecși) o versionezi cu migrații. Fără ele, nimeni nu știe ce structură are baza de date din producție și cum s-a ajuns acolo.

## Ce e o migrație

**Ce e.** Un fișier SQL care descrie **o schimbare** de schemă: „creează tabelul `payments`”, „adaugă coloana `vat_number`”, „pune index pe `client_id`”. Migrațiile se aplică în ordine, iar baza de date ține un tabel cu cele deja aplicate (la Prisma, `_prisma_migrations`), ca să știe de unde continuă.

```
prisma/migrations/
├── 20261103120000_init/migration.sql              CREATE TABLE clients, invoices, ...
├── 20261105093000_add_payments/migration.sql      CREATE TABLE payments ...
└── 20261110170000_add_client_vat_number/migration.sql   ALTER TABLE clients ADD COLUMN ...
```

**De ce contează.** O bază de date goală plus toate migrațiile în ordine dă exact schema curentă. Asta înseamnă că un coleg nou, un mediu de test sau serverul de producție ajung toate la aceeași structură, automat. Iar istoria e în git, cu autor și PR, ca orice cod.

**Analogie.** E caietul de modificări al unei rețete: nu rescrii fișa tehnică de la zero de fiecare dată, ci notezi „15 noiembrie: +10 g sare, fără usturoi”. Oricine poate reface versiunea de azi pornind de la prima variantă.

<details>
<summary>Întrebare de interviu: Ce sunt migrațiile de bază de date și de ce le folosim?</summary>

Sunt fișiere versionate care descriu, în ordine, fiecare schimbare de schemă. Le folosim ca schema să fie reproductibilă (orice mediu ajunge la aceeași structură aplicând migrațiile), ca schimbările să fie revizuite în PR-uri, ca deploy-ul să aplice automat ce lipsește și ca istoria să fie trasabilă. Alternativa, modificări manuale în producție, duce la medii diferite între ele și la schimbări pe care nu le mai poate explica nimeni.
</details>

## Fluxul cu Prisma

**În dezvoltare** (pe laptop):

```bash
# 1. edit prisma/schema.prisma (e.g. add a field)
# 2. create + apply the migration
npx prisma migrate dev --name add_client_vat_number
# 3. regenerate the typed client (not automatic in Prisma 7)
npx prisma generate
```

`migrate dev` compară schema din `schema.prisma` cu baza de date, scrie SQL-ul necesar într-un folder nou din `prisma/migrations/` și îl aplică.

**Citește SQL-ul generat.** Deschide `migration.sql` înainte de commit. E cel mai simplu review de bază de date pe care îl poți face, și acolo vezi dacă o schimbare inocentă înseamnă de fapt `DROP COLUMN`.

**În producție** (la deploy, pe Render):

```bash
npx prisma migrate deploy
```

`migrate deploy` doar aplică migrațiile care lipsesc. Nu generează nimic, nu compară, nu resetează. Pe Render îl pui în comanda de build (vezi [11-deploy/03](../11-deploy/03-render.md)).

**Alte comenzi utile:**

- `npx prisma migrate status`: ce migrații sunt aplicate și ce lipsește;
- `npx prisma migrate reset`: șterge **tot** și reaplică migrațiile de la zero, plus seed. Doar local, niciodată pe o bază cu date reale;
- `npx prisma migrate dev --create-only`: scrie migrația fără s-o aplice, ca s-o poți edita (vezi mai jos, la RLS).

**Capcana.** Nu modifica niciodată o migrație care a fost deja aplicată și comisă. Prisma ține un checksum al fiecărui fișier și detectează schimbarea. Dacă ai greșit ceva, faci o migrație **nouă** care corectează.

## Seed: date de pornire

**Ce e.** Un script care populează baza de date cu date de test (clienți și facturi de exemplu). E util pentru dezvoltare, pentru demo-ul din portofoliu și pentru testele e2e.

**Setarea.** În fișierul de config Prisma ai deja `migrations: { seed: 'tsx prisma/seed.ts' }` (vezi [07/05](../07-backend-node-nest/05-baza-de-date-cu-prisma.md)). Scriptul:

```ts
// prisma/seed.ts
import 'dotenv/config';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '../src/generated/prisma/client.js';

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL! }, { schema: 'invoicer' });
const prisma = new PrismaClient({ adapter });

const acme = await prisma.client.upsert({
  where: { id: '00000000-0000-0000-0000-000000000001' },
  update: {},
  create: { id: '00000000-0000-0000-0000-000000000001', name: 'Acme SRL', email: 'office@acme.ro' },
});
console.log('Seeded client', acme.name);
await prisma.$disconnect();
```

Rulezi cu `npx prisma db seed`. În Prisma 7, seed-ul rulează **doar** când îl chemi explicit.

**Capcana.** Folosește `upsert` cu id-uri fixe, nu `create`, ca scriptul să poată fi rulat de mai multe ori fără duplicate. Un seed care merge doar o dată te va încurca la prima rulare repetată.

## Regula de aur: schema se schimbă doar prin migrații

**Ce e.** Odată ce ai migrații, **nu mai modifici structura de mână**: nici din Table Editor, nici cu `ALTER TABLE` în SQL Editor.

**Ce se întâmplă dacă o faci.** Baza de date nu mai corespunde cu istoria din `prisma/migrations`, iar asta se numește **drift**. La următorul `migrate dev`, Prisma observă diferența și îți propune să resetezi baza de date, adică să ștergi toate datele.

**Cum faci schimbări pe care Prisma nu le știe exprima.** Există lucruri care nu se pot scrie în `schema.prisma`: RLS, politici, constrângeri de excludere, trigger-e. Le pui tot în migrații, scrise de mână:

```bash
npx prisma migrate dev --create-only --name enable_rls
# edit the generated migration.sql, then:
npx prisma migrate dev
```

```sql
-- prisma/migrations/2026..._enable_rls/migration.sql
alter table invoicer.invoices enable row level security;
alter table invoicer.invoice_items enable row level security;
alter table invoicer.clients enable row level security;
alter table invoicer.payments enable row level security;
```

De ce ai nevoie exact de asta e explicat în [06](06-rls-auth-storage.md).

<details>
<summary>Întrebare de interviu: Ce faci dacă ai nevoie să redenumești o coloană într-o tabelă din producție cu date?</summary>

O fac printr-o migrație, nu manual. Verific SQL-ul generat, pentru că unele ORM-uri interpretează o redenumire ca DROP plus ADD, iar asta pierde datele. Corect e `ALTER TABLE ... RENAME COLUMN`, pe care îl scriu de mână în migrație dacă e nevoie. Dacă aplicația rulează continuu, fac schimbarea în pași compatibili: adaug coloana nouă, scriu în ambele, mut datele, schimb citirile, apoi șterg coloana veche. Și o testez întâi pe o copie a bazei de date.
</details>

## Un singur DB pentru local și producție

În proiectele de portofoliu, laptopul și serverul de pe Render folosesc de obicei **aceeași** bază de date Supabase. Pentru învățat e acceptabil, dar ține minte ce înseamnă: un `migrate reset` „local” șterge datele demo-ului live. La un job există baze separate (local, staging, producție), iar migrațiile trec prin toate, în ordinea asta. Varianta profesionistă pentru local e Supabase CLI cu o bază locală în Docker. Nu ai nevoie de ea acum, dar e bine să știi că există și ce problemă rezolvă.

## Alternativa: migrațiile Supabase CLI

Supabase are propriul sistem de migrații: fișiere SQL scrise de tine (`supabase migration new <nume>`), aplicate cu CLI-ul. E alegerea firească în proiectele unde **nu** folosești un ORM, de exemplu când frontend-ul vorbește direct cu Supabase, cum e P4. Principiul e identic: fișiere SQL versionate, aplicate în ordine, niciodată modificări de mână. Ce nu faci niciodată e să amesteci ambele sisteme pe aceleași tabele.

## Mai departe

- [Prisma 7 — Prisma Migrate](https://www.prisma.io/docs/orm/v7/prisma-migrate/getting-started)
- [Supabase — Database migrations](https://supabase.com/docs/guides/deployment/database-migrations)
