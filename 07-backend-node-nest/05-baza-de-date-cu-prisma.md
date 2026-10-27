# Baza de date cu Prisma

Nest primește request-ul și îl validează, dar datele trebuie să ajungă undeva. În P2 ajung în Postgres-ul din Supabase, iar Nest vorbește cu el prin Prisma. Înainte de fișierul ăsta, citește [08-baze-de-date-supabase/01](../08-baze-de-date-supabase/01-model-relational.md) (tabele, chei, relații). Conexiunea cu Supabase e explicată în [08/04](../08-baze-de-date-supabase/04-conectare-si-chei.md).

## ORM vs SQL brut vs query builder

**Ce e.** Trei feluri de a vorbi cu o bază de date din cod:

| Varianta | Cum arată | Exemple |
|---|---|---|
| SQL brut | scrii string-uri SQL, primești rânduri | `pg`, `postgres.js` |
| Query builder | construiești SQL-ul din funcții tipizate, aproape 1:1 cu SQL | Drizzle, Kysely |
| ORM | descrii modele, lucrezi cu obiecte, ORM-ul scrie SQL-ul | Prisma, TypeORM |

**De ce Prisma.** E cel mai răspândit ORM în proiectele TypeScript din anunțurile de job. Schema e un fișier ușor de citit, migrațiile sunt incluse, iar clientul generat e complet tipizat: dacă scrii `invoice.totl`, editorul te oprește.

**Capcana.** Un ORM nu te scutește de SQL. Când o pagină se încarcă greu, problema e aproape mereu în query-urile generate. De asta există [08/02 · SQL esențial](../08-baze-de-date-supabase/02-sql-esential.md). Iar la interviu te întreabă de JOIN-uri, nu de Prisma.

<details>
<summary>Întrebare de interviu: Avantaje și dezavantaje ale unui ORM?</summary>

Avantaje: productivitate, tipuri generate din schemă, migrații gestionate și protecție implicită la SQL injection, pentru că parametrii sunt trimiși separat de query. Dezavantaje: un strat de abstracție în plus, query-uri generate uneori ineficiente (problema N+1, de exemplu) și cazuri complexe (rapoarte, agregări) unde SQL-ul scris de mână e mai clar și mai rapid. În practică folosesc ORM-ul pentru CRUD și cobor la SQL brut parametrizat acolo unde e nevoie.
</details>

## Setarea Prisma în Nest (Prisma 7)

Pașii urmează [rețeta oficială Nest + Prisma](https://docs.nestjs.com/recipes/prisma). Claude îi va face pentru tine, dar trebuie să știi ce face fiecare.

**1. Instalează, cu versiunea fixată:**

```bash
npm i -D prisma@7 dotenv
npm i @prisma/client@7 @prisma/adapter-pg
```

De ce `@7`: la momentul scrierii, `npm install prisma` fără versiune instalează Prisma 8, care e încă release candidate și are alt API. `prisma` și `@prisma/client` trebuie să fie mereu pe aceeași versiune majoră. Dacă vezi în codul generat de Claude `contract.prisma` sau `db.orm...`, e Prisma 8, și nu e ce vrei acum.

**2. Inițializează:**

```bash
npx prisma init --output ../src/generated/prisma
```

Creează:

- `prisma/schema.prisma`, cu modelele;
- un fișier de config în rădăcina proiectului: `prisma.config.ts` (în versiunile recente de 7.x se poate numi `prisma7.config.ts`, deci folosește ce a generat comanda);
- `.env`, cu `DATABASE_URL`.

Comanda adaugă în `.gitignore` atât `.env`, cât și folderul clientului generat. Verifică totuși că au ajuns acolo.

**3. Config-ul** (CLI-ul Prisma citește de aici URL-ul pentru migrații):

```ts
import 'dotenv/config';
import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  migrations: { path: 'prisma/migrations', seed: 'tsx prisma/seed.ts' },
  datasource: { url: env('DATABASE_URL') },
});
```

**4. `DATABASE_URL` din Supabase.** Folosește connection string-ul de **session pooler** (portul 5432 pe host-ul `pooler.supabase.com`), din butonul **Connect** al proiectului. Merge și pentru server, și pentru migrații, și de pe orice rețea. Motivul e explicat în [08/04](../08-baze-de-date-supabase/04-conectare-si-chei.md).

**4b. Schema proprie.** P2 și P3 împart același proiect Supabase, fiecare cu schema lui Postgres: `invoicer` pentru P2, `booking` pentru P3 (vezi [08/03](../08-baze-de-date-supabase/03-cont-supabase.md#planul-două-proiecte-supabase-în-total) și [08/01](../08-baze-de-date-supabase/01-model-relational.md#schemă-postgres)). Pentru P2 setezi schema în **două** locuri:

```bash
# .env — the CLI (migrate) reads the schema from the URL
DATABASE_URL="postgresql://postgres.<ref>:<password>@<pooler-host>:5432/postgres?schema=invoicer"
```

```ts
// PrismaService — the driver adapter ignores ?schema= in the URL, so pass it explicitly
const adapter = new PrismaPg(
  { connectionString: config.getOrThrow<string>('DATABASE_URL') },
  { schema: 'invoicer' },
);
```

Dacă pui schema doar în URL, migrațiile creează tabelele în `invoicer`, dar aplicația caută în `public` și primești `relation "invoices" does not exist`. Dacă `migrate dev` se plânge că schema nu există, o creezi o dată în SQL Editor: `create schema invoicer;`. (Alternativa e funcția multi-schema din Prisma, cu `schemas = [...]` în `datasource` și `@@schema("invoicer")` pe fiecare model. Ai nevoie de ea doar dacă un singur proiect folosește mai multe scheme.)

**5. `PrismaService`**, clientul pe care îl injectezi peste tot:

```ts
// src/prisma/prisma.service.ts
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '../generated/prisma/client.js';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  constructor(config: ConfigService) {
    // Prisma 7 talks to Postgres through a driver adapter (here: node-postgres)
    const adapter = new PrismaPg(
      { connectionString: config.getOrThrow<string>('DATABASE_URL') },
      { schema: 'invoicer' }, // P2's own Postgres schema, see step 4b
    );
    super({ adapter });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Plus un `PrismaModule` care îl declară în `providers` și `exports`, importat în `AppModule`. Și `app.enableShutdownHooks()` în `main.ts`, ca deconectarea să ruleze și când Render oprește serverul.

**Ce s-a schimbat în Prisma 7.** O să găsești pe net multe tutoriale vechi. Diferențele principale:

- URL-ul nu mai stă în `schema.prisma`, ci în fișierul de config;
- clientul se generează într-un folder din proiect, nu în `node_modules`, și se importă de acolo;
- conexiunea trece obligatoriu printr-un driver adapter (`PrismaPg`);
- `directUrl` nu mai există.

Dacă Claude scrie `url = env("DATABASE_URL")` în `datasource`, sau importă din `@prisma/client`, a amestecat cu versiunea 6.

**Capcana.** `prisma migrate dev` nu mai rulează automat `prisma generate`. După fiecare schimbare de schemă rulezi și `npx prisma generate`, altfel TypeScript nu vede câmpurile noi. Și pentru că folderul generat e în `.gitignore`, pe serverul de deploy clientul trebuie generat la build (vezi [11-deploy/03](../11-deploy/03-render.md)).

## Schema din P2

**Ce e.** Fișierul `schema.prisma` descrie tabelele ca modele. Din el se generează și migrațiile SQL, și clientul TypeScript.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

enum InvoiceStatus {
  DRAFT
  SENT
  PAID
}

model Client {
  id        String    @id @default(uuid()) @db.Uuid
  name      String
  email     String
  vatNumber String?   @map("vat_number")
  createdAt DateTime  @default(now()) @map("created_at")
  invoices  Invoice[]

  @@map("clients")
}

model Invoice {
  id            String        @id @default(uuid()) @db.Uuid
  number        String        @unique
  clientId      String        @map("client_id") @db.Uuid
  client        Client        @relation(fields: [clientId], references: [id])
  status        InvoiceStatus @default(DRAFT)
  issueDate     DateTime      @default(now()) @map("issue_date") @db.Date
  dueDate       DateTime      @map("due_date") @db.Date
  vatRateBps    Int           @map("vat_rate_bps")
  subtotalCents Int           @map("subtotal_cents")
  vatCents      Int           @map("vat_cents")
  totalCents    Int           @map("total_cents")
  items         InvoiceItem[]
  payments      Payment[]
  createdAt     DateTime      @default(now()) @map("created_at")

  @@index([clientId])
  @@map("invoices")
}

model InvoiceItem {
  id             String  @id @default(uuid()) @db.Uuid
  invoiceId      String  @map("invoice_id") @db.Uuid
  invoice        Invoice @relation(fields: [invoiceId], references: [id], onDelete: Cascade)
  description    String
  quantity       Int
  unitPriceCents Int     @map("unit_price_cents")
  lineTotalCents Int     @map("line_total_cents")

  @@index([invoiceId])
  @@map("invoice_items")
}

model Payment {
  id          String   @id @default(uuid()) @db.Uuid
  invoiceId   String   @map("invoice_id") @db.Uuid
  invoice     Invoice  @relation(fields: [invoiceId], references: [id])
  amountCents Int      @map("amount_cents")
  paidAt      DateTime @default(now()) @map("paid_at")

  @@index([invoiceId])
  @@map("payments")
}
```

**Ce să observi.**

- **`@@map` și `@map`.** În cod lucrezi cu `Invoice.clientId` (camelCase), dar în baza de date tabelul e `invoices`, iar coloana `client_id` (snake_case, convenția Postgres). Fără ele, în SQL ai scrie `"Invoice"."clientId"` cu ghilimele peste tot.
- **Banii sunt `Int`, în bani (cenți).** `0.1 + 0.2 !== 0.3` în JavaScript. Floats pentru bani sunt o greșeală clasică de junior.
- **TVA-ul e tot întreg, în puncte de bază:** `vatRateBps = 2100` înseamnă 21%. Și cota stă pe fiecare factură, nu e hardcodată, pentru că s-a schimbat (de la 19% la 21% în 2025), iar facturile vechi trebuie să-și păstreze cota de atunci.
- **`@@index([clientId])`.** Postgres nu creează automat index pe foreign keys. Fără el, „toate facturile clientului X” scanează tot tabelul.
- **„Overdue” nu e un status salvat.** Se calculează (`status = SENT` și `dueDate` în trecut). Ce se poate calcula nu se stochează, pentru că altfel ajunge să nu mai fie sincronizat.

## CRUD și relații

**Exemplu minim** — operațiile de bază, așa cum apar în service-uri:

```ts
// list with filter, newest first, including the client's name
const invoices = await this.prisma.invoice.findMany({
  where: { status: 'SENT' },
  orderBy: { issueDate: 'desc' },
  include: { client: { select: { name: true } } },
});

const invoice = await this.prisma.invoice.findUnique({ where: { id }, include: { items: true } });

await this.prisma.client.update({ where: { id }, data: { email: 'new@acme.ro' } });

await this.prisma.client.delete({ where: { id } }); // fails if the client still has invoices (FK)
```

**Capcana: problema N+1.** Codul de mai jos face 1 query pentru facturi și apoi încă **N** query-uri, câte unul pentru fiecare client:

```ts
const invoices = await this.prisma.invoice.findMany();
for (const inv of invoices) {
  inv.client = await this.prisma.client.findUnique({ where: { id: inv.clientId } }); // N extra queries!
}
```

Cu `include: { client: true }`, Prisma le aduce eficient. AI-ul scrie uneori bucle de felul ăsta. Caută `await` în interiorul unui `for` care face query-uri.

<details>
<summary>Întrebare de interviu: Ce e problema N+1?</summary>

Când încarci o listă de N înregistrări și apoi, pentru fiecare, faci încă un query ca să aduci date legate. Rezultă 1 + N query-uri în loc de unul sau două. La 10 rânduri nu se simte, la 1000 pagina devine foarte lentă. Soluția e să încarci relațiile dintr-o dată: cu un JOIN în SQL, sau cu `include` în Prisma, care face un singur query suplimentar cu `WHERE id IN (...)`.
</details>

## Totaluri calculate pe server și tranzacții

**Ce e.** O tranzacție grupează mai multe operații pe baza de date astfel încât **ori se fac toate, ori niciuna**. Dacă a doua cade, prima e anulată (rollback).

**Factura cu linii, într-un singur pas.** Un nested create e atomic, adică factura și liniile se salvează împreună sau deloc:

```ts
async create(dto: CreateInvoiceDto) {
  const items = dto.items.map((item) => ({
    ...item,
    lineTotalCents: item.quantity * item.unitPriceCents,
  }));
  const subtotalCents = items.reduce((sum, item) => sum + item.lineTotalCents, 0);
  const vatCents = Math.round((subtotalCents * dto.vatRateBps) / 10_000); // 2100 bps = 21%

  return this.prisma.invoice.create({
    data: {
      number: await this.nextNumber(),
      clientId: dto.clientId,
      dueDate: new Date(dto.dueDate),
      vatRateBps: dto.vatRateBps,
      subtotalCents,
      vatCents,
      totalCents: subtotalCents + vatCents,
      items: { create: items }, // inserted in the same transaction as the invoice
    },
    include: { items: true },
  });
}
```

**Plata care închide factura: două scrieri care trebuie să meargă împreună.**

```ts
async addPayment(invoiceId: string, amountCents: number) {
  return this.prisma.$transaction(async (tx) => {
    const invoice = await tx.invoice.findUniqueOrThrow({ where: { id: invoiceId }, include: { payments: true } });
    const paidSoFar = invoice.payments.reduce((sum, p) => sum + p.amountCents, 0);

    const payment = await tx.payment.create({ data: { invoiceId, amountCents } });
    if (paidSoFar + amountCents >= invoice.totalCents) {
      await tx.invoice.update({ where: { id: invoiceId }, data: { status: 'PAID' } });
    }
    return payment; // if anything above throws, the payment is rolled back too
  });
}
```

**Ce să observi.** În interiorul tranzacției folosești `tx`, nu `this.prisma`. Un query făcut cu `this.prisma` în interior rulează *în afara* tranzacției. E o greșeală subtilă, pe care n-o prinde nici TypeScript.

**Capcana: numerotarea facturilor.** Un `nextNumber()` naiv face `count() + 1`. Două request-uri simultane primesc același număr. Plasa de siguranță e `@unique` pe `number`: baza de date refuză duplicatul (eroarea Prisma `P2002`, transformată în 409 de filtrul din [03](03-dto-validare-erori.md)). Regula generală: **invarianții importanți se garantează în baza de date** (unique, foreign key, check), nu doar în cod.

<details>
<summary>Întrebare de interviu: Ce e o tranzacție și când ai nevoie de ea?</summary>

O tranzacție execută un grup de operații ca pe una singură: fie reușesc toate (commit), fie nu rămâne niciuna (rollback). E proprietatea A din ACID, atomicitatea. Am nevoie de ea când mai multe scrieri trebuie să fie consistente între ele. De exemplu, înregistrez o plată și marchez factura ca plătită: dacă a doua scriere cade, nu vreau să rămână plata fără statusul actualizat. În Prisma folosesc `$transaction` sau nested writes, care sunt atomice implicit.
</details>

## SQL brut, când ai nevoie

Pentru rapoarte (totaluri pe lună, top clienți), SQL-ul e mai clar decât ORM-ul. Prisma îl permite, în siguranță, cu tagged templates:

```ts
const year = 2026;
const rows = await this.prisma.$queryRaw<{ month: Date; total_cents: bigint }[]>`
  SELECT date_trunc('month', issue_date) AS month, SUM(total_cents) AS total_cents
  FROM invoicer.invoices
  WHERE extract(year FROM issue_date) = ${year}
  GROUP BY 1 ORDER BY 1`;
```

În SQL brut scrii numele complet, cu schema (`invoicer.invoices`), ca să nu depinzi de schema implicită a conexiunii.

`${year}` nu e lipit în string. E trimis ca parametru separat, deci e protejat de SQL injection. Varianta `$queryRawUnsafe` cu string concatenat **nu** e protejată. Vezi [10-securitate-env/04](../10-securitate-env/04-validare-si-atacuri.md).

**Capcana.** `SUM` în Postgres întoarce un `bigint`, care ajunge în JS ca `BigInt`. `JSON.stringify` nu știe să-l serializeze și aruncă eroare. Convertește-l cu `Number(...)` înainte să-l returnezi.

## Mai departe

- [NestJS — Prisma recipe](https://docs.nestjs.com/recipes/prisma)
- [Prisma 7 — Relation queries](https://www.prisma.io/docs/orm/v7/prisma-client/queries/relation-queries)
- [Prisma 7 — Transactions](https://www.prisma.io/docs/orm/v7/prisma-client/queries/transactions)

Atenție: paginile de documentație Prisma fără `/v7/` în URL descriu acum Prisma 8. Dacă un exemplu de pe site nu seamănă cu codul tău, verifică versiunea din URL.
