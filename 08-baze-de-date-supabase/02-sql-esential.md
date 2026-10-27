# SQL esențial

SQL e limbajul cu care vorbești cu orice bază de date relațională. Prisma îl scrie pentru tine în 90% din cazuri, dar la debugging, la rapoarte și la interviu ai nevoie de el direct. Aici e setul pe care îl folosește un developer full-stack în fiecare săptămână.

Ai nevoie de contul Supabase din [03](03-cont-supabase.md). Exersezi în **SQL Editor** din dashboard.

## Pregătirea: un sandbox separat

Ca să nu atingi tabelele aplicației (pe care le gestionează Prisma prin migrații), creezi o schemă separată, `sandbox`. O **schemă** Postgres e ca un folder de tabele în aceeași bază de date. Rulează o dată, în SQL Editor:

```sql
create schema if not exists sandbox;
set search_path to sandbox;

create table clients (
  id bigint generated always as identity primary key,
  name text not null,
  email text not null unique,
  created_at timestamptz not null default now()
);

create table invoices (
  id bigint generated always as identity primary key,
  number text not null unique,
  client_id bigint not null references clients(id),
  status text not null default 'DRAFT' check (status in ('DRAFT', 'SENT', 'PAID')),
  issue_date date not null default current_date,
  due_date date not null,
  total_cents integer not null check (total_cents >= 0)
);

create table payments (
  id bigint generated always as identity primary key,
  invoice_id bigint not null references invoices(id),
  amount_cents integer not null check (amount_cents > 0),
  paid_at timestamptz not null default now()
);

insert into clients (name, email) values
  ('Acme SRL', 'office@acme.ro'),
  ('Beta Studio', 'hello@beta.ro'),
  ('Gamma Tech', 'billing@gamma.tech');

insert into invoices (number, client_id, status, issue_date, due_date, total_cents) values
  ('INV-2026-0001', 1, 'PAID',  '2026-09-01', '2026-09-15', 121000),
  ('INV-2026-0002', 1, 'SENT',  '2026-10-01', '2026-10-15',  60500),
  ('INV-2026-0003', 2, 'SENT',  '2026-10-10', '2026-11-10', 242000),
  ('INV-2026-0004', 2, 'DRAFT', '2026-10-20', '2026-11-20',  36300);

insert into payments (invoice_id, amount_cents, paid_at) values
  (1, 121000, '2026-09-12'),
  (3, 100000, '2026-10-20');
```

**Verifică.** În Table Editor, din selectorul de schemă (sus-stânga), alege `sandbox` și vezi cele trei tabele cu date.

**Regula pentru tot fișierul.** Fiecare rulare în SQL Editor e o sesiune nouă, deci începe fiecare query cu `set search_path to sandbox;`, ca în exemple. La final, `drop schema sandbox cascade;` șterge tot.

## SELECT, WHERE, ORDER BY, LIMIT

**Ce e.** Citești rânduri: ce coloane (`SELECT`), din ce tabel (`FROM`), care rânduri (`WHERE`), în ce ordine (`ORDER BY`) și câte (`LIMIT`).

**Exemplu minim** — facturile restante, cea mai veche prima:

```sql
set search_path to sandbox;

select number, due_date, total_cents / 100.0 as total_lei
from invoices
where status = 'SENT' and due_date < current_date
order by due_date asc
limit 10;
```

**Ce să observi.**

- `as total_lei` redenumește coloana în rezultat.
- `/ 100.0` face împărțire cu zecimale. `/ 100` ar fi împărțire întreagă, iar `60500 / 100` ar da `605` și ai pierde banii.
- String-urile se scriu cu ghilimele **simple**. Ghilimelele duble sunt pentru nume de coloane.

**Capcana.** `NULL` nu e egal cu nimic, nici măcar cu el însuși. `where vat_number = null` nu găsește nimic. Corect e `where vat_number is null`.

## JOIN

**Ce e.** Combini rânduri din mai multe tabele, după o condiție, de obicei foreign key = primary key.

- **`inner join`** (sau doar `join`): doar rândurile care au corespondent în ambele tabele.
- **`left join`**: toate rândurile din tabelul din stânga, plus corespondentul din dreapta, dacă există (altfel `NULL`).

**Exemplu** — fiecare factură cu numele clientului:

```sql
set search_path to sandbox;

select i.number, c.name as client, i.status, i.total_cents
from invoices i
join clients c on c.id = i.client_id
order by i.issue_date;
```

**Diferența contează** — toți clienții și câte facturi au, inclusiv cei fără facturi:

```sql
set search_path to sandbox;

select c.name, count(i.id) as invoice_count
from clients c
left join invoices i on i.client_id = c.id
group by c.name;
```

Cu `left join`, Gamma Tech apare cu 0. Cu `join` simplu, Gamma Tech dispare din rezultat, fără nicio eroare.

**Capcana.** `count(*)` vs `count(i.id)`: într-un `left join`, `count(*)` numără și rândul cu `NULL` și îi dă lui Gamma Tech 1 factură. `count(coloană)` ignoră `NULL`-urile.

<details>
<summary>Întrebare de interviu: Care e diferența dintre INNER JOIN și LEFT JOIN?</summary>

INNER JOIN returnează doar rândurile care au potrivire în ambele tabele. LEFT JOIN returnează toate rândurile din tabelul din stânga și, unde există, datele potrivite din dreapta. Unde nu există potrivire, coloanele din dreapta sunt NULL. Exemplu: „toți clienții cu numărul lor de facturi” cere LEFT JOIN, altfel clienții fără facturi dispar din raport.
</details>

## GROUP BY și agregări

**Ce e.** Strângi rândurile în grupuri și calculezi ceva pe fiecare grup: `count`, `sum`, `avg`, `min`, `max`. `HAVING` filtrează grupurile, după agregare. `WHERE` filtrează rândurile, înainte de agregare.

**Exemplu** — cât a facturat și cât a încasat fiecare client:

```sql
set search_path to sandbox;

select
  c.name,
  sum(i.total_cents) / 100.0 as invoiced_lei,
  coalesce(sum(p.amount_cents), 0) / 100.0 as collected_lei
from clients c
join invoices i on i.client_id = c.id
left join payments p on p.invoice_id = i.id
where i.status <> 'DRAFT'
group by c.name
having sum(i.total_cents) > 50000
order by invoiced_lei desc;
```

**Ce să observi.** `coalesce(x, 0)` înlocuiește `NULL` cu 0 (`sum` peste nimic dă `NULL`, nu 0).

**Capcana.** Query-ul de mai sus are o problemă subtilă: o factură cu **două** plăți apare de două ori după `join payments`, iar `sum(i.total_cents)` o numără dublu. Agregările peste mai multe JOIN-uri 1-n se umflă. Soluția e să agregi plățile separat, într-un subquery, și apoi să faci JOIN pe rezultat. E exact genul de bug care trece neobservat în codul generat de AI, pentru că rezultatul „arată plauzibil”.

<details>
<summary>Întrebare de interviu: În ce ordine execută baza de date un SELECT?</summary>

Logic, nu în ordinea în care e scris: FROM (și JOIN-urile), WHERE, GROUP BY, HAVING, SELECT, ORDER BY, LIMIT. De aceea nu pot folosi în WHERE un alias definit în SELECT. În schimb, în ORDER BY pot, pentru că rulează după SELECT. Și de aceea filtrul pe rânduri merge în WHERE, iar filtrul pe rezultatul agregărilor în HAVING.
</details>

## INSERT, UPDATE, DELETE

**Ce e.** Scrii date. `returning` îți dă înapoi rândurile afectate, util ca să vezi ce ai făcut.

```sql
set search_path to sandbox;

insert into clients (name, email) values ('Delta Foods', 'office@delta.ro') returning id, name;

update invoices set status = 'SENT' where number = 'INV-2026-0004' returning number, status;

delete from payments where id = 2 returning *;
```

**Capcana, cea mai scumpă din SQL.** `update invoices set status = 'PAID';` fără `WHERE` modifică **toate** facturile. `delete from clients;` le șterge pe toate (sau cade din cauza foreign keys, dacă ai noroc). Obiceiul seniorilor: scrii întâi `select * from ... where ...`, verifici că sunt exact rândurile vizate, apoi transformi în `update` sau `delete` cu același `WHERE`.

## Constrângeri

**Ce e.** Reguli pe care baza de date le impune la fiecare scriere, indiferent din ce aplicație vine:

- **`not null`:** valoarea e obligatorie;
- **`unique`:** nu se repetă (`email`, `number`);
- **`check (...)`:** o condiție oarecare (`total_cents >= 0`, `status in (...)`);
- **`references`:** foreign key.

**Încearcă să le încalci:**

```sql
set search_path to sandbox;

insert into clients (name, email) values ('Copy', 'office@acme.ro');          -- unique violation
insert into invoices (number, client_id, due_date, total_cents)
  values ('X-1', 999, '2026-12-01', 100);                                     -- foreign key violation
insert into payments (invoice_id, amount_cents) values (1, -50);              -- check violation
```

Citește mesajele de eroare. Sunt aceleași pe care le vei vedea în log-urile Nest când Prisma le primește înapoi (`P2002` e unique, `P2003` e foreign key).

## Tranzacții

**Ce e.** Un grup de operații care se execută toate sau niciuna. Între `begin` și `commit`, schimbările nu sunt vizibile altora. `rollback` le anulează. Conceptul și varianta Prisma sunt în [07/05](../07-backend-node-nest/05-baza-de-date-cu-prisma.md).

```sql
set search_path to sandbox;

begin;
  insert into payments (invoice_id, amount_cents) values (3, 142000);
  update invoices set status = 'PAID' where id = 3;
rollback; -- undo both

select number, status from invoices where id = 3; -- still SENT, and the payment is gone
```

SQL Editor afișează rezultatul ultimei instrucțiuni, de aceea `select`-ul e la final. Rulează o dată cu `rollback` și verifică că nu s-a schimbat nimic. Apoi înlocuiește `rollback` cu `commit` și rulează din nou: factura devine `PAID`. Acum factura 3 are două plăți, deci rulează din nou query-ul cu `GROUP BY` de mai sus și vezi capcana cu dublarea totalului.

**Capcana.** În producție, un `UPDATE` periculos rulat de mână se face într-o tranzacție: `begin`, `update ... returning *`, te uiți la rezultat, apoi `commit` sau `rollback`. Așa ai o plasă de siguranță.

## Mai departe

- [PostgreSQL Tutorial — The SQL language](https://www.postgresql.org/docs/current/tutorial-sql.html)
- [Supabase — SQL Editor](https://supabase.com/docs/guides/database/overview)
