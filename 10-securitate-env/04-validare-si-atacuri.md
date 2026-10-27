# Validare și atacuri

Atacurile clasice de web au aceeași rădăcină: aplicația a avut încredere într-un input pe care nu trebuia să-l creadă. Fișierul ăsta îți arată principiul și cele câteva atacuri pe care trebuie să le poți explica la orice interviu, plus felul în care le eviți în stack-ul tău.

## Never trust the client

**Ce e.** Orice vine din browser poate fi modificat: formularul, query string-ul, header-ele, body-ul JSON. Oricine îți poate chema API-ul direct, cu `curl`, ocolind complet interfața.

**De ce contează.** Validarea din frontend e **UX**: utilizatorul vede imediat că a greșit ceva. Validarea din backend e **securitate**. Ai nevoie de amândouă, dar doar a doua te protejează.

**Exemplu minim** (formularul React nu te mai ajută cu nimic aici)

```bash
curl -X POST http://localhost:3000/invoices \
  -H "Content-Type: application/json" \
  -d '{"clientId":"c1","vatRateBps":2100,"items":[{"description":"Design","quantity":-5,"unitPriceCents":10000}]}'
```

**Cum îl vezi în cod real** (P2: totalul se calculează **pe server**, niciodată nu-l primești de la client)

```ts
// create-invoice.dto.ts — validated automatically by ValidationPipe
export class InvoiceItemDto {
  @IsString() @MaxLength(200) description!: string;
  @IsInt() @Min(1) quantity!: number;
  @IsInt() @Min(0) unitPriceCents!: number;   // money as integer cents
}

export class CreateInvoiceDto {
  @IsUUID() clientId!: string;
  @IsInt() @Min(0) @Max(10000) vatRateBps!: number; // basis points: 2100 = 21%
  @ValidateNested({ each: true }) @Type(() => InvoiceItemDto) @ArrayMinSize(1)
  items!: InvoiceItemDto[];
}

// invoices.service.ts
const subtotalCents = dto.items.reduce((sum, l) => sum + l.quantity * l.unitPriceCents, 0);
const vatCents = Math.round((subtotalCents * dto.vatRateBps) / 10_000);
// the client never sends `total`; even if it did, we would ignore it
```

**Capcana.** Codul generat de AI primește uneori `total` din body „pentru că frontend-ul îl are deja calculat”. Un utilizator își trimite atunci o factură de 10.000 de lei cu totalul 1 leu. Orice cifră cu impact (preț, total, rol, discount) se calculează sau se citește pe server.

## Mass assignment

**Ce e.** Iei tot body-ul cererii și îl scrii direct în baza de date. Atacatorul adaugă câmpuri pe care nu le-ai prevăzut.

```ts
// WRONG: the client sends { "name": "Ana", "role": "admin" }
await prisma.profile.update({ where: { id: userId }, data: req.body });
```

**Soluția.** Un DTO explicit, plus `ValidationPipe` configurat să arunce tot ce nu e declarat:

```ts
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,              // strip properties without decorators
    forbidNonWhitelisted: true,   // or reject the request entirely
    transform: true,
  }),
);
```

**Capcana.** `data: { ...dto }` e sigur doar dacă DTO-ul e strict și `whitelist` e activ. Altfel e tot un `req.body` deghizat.

## SQL injection

**Ce e.** Input-ul utilizatorului e lipit direct în textul unei interogări SQL, așa că devine cod SQL.

```ts
// WRONG: name = "x' OR '1'='1" returns every client
const sql = "SELECT * FROM clients WHERE name = '" + name + "'";
```

**Soluția: interogări parametrizate.** Valoarea se trimite separat de textul SQL și e tratată mereu ca dată, niciodată ca SQL. Prisma face asta automat:

```ts
// safe: Prisma Client methods are parameterized
await prisma.client.findMany({ where: { name } });

// safe: $queryRaw as a tagged template escapes every ${...}
await prisma.$queryRaw`SELECT * FROM clients WHERE name = ${name}`;

// VULNERABLE: $queryRawUnsafe with string concatenation
await prisma.$queryRawUnsafe(`SELECT * FROM clients WHERE name = '${name}'`);
```

**Capcana.** Placeholder-ele (`${...}` în `$queryRaw`, `$1` în SQL) funcționează doar pentru **valori**, nu pentru nume de tabele sau coloane. Dacă ai nevoie de o coloană dinamică pentru sortare, o alegi dintr-o listă fixă (`['createdAt', 'total']`), nu o iei direct din input.

<details>
<summary>Întrebare de interviu: Ce e SQL injection și cum îl previi?</summary>

Când input-ul utilizatorului e concatenat în textul unei interogări, atacatorul poate închide string-ul și adăuga SQL propriu, ca să citească sau să șteargă date. Îl previn prin interogări parametrizate, unde valorile se trimit separat de SQL. ORM-urile ca Prisma fac asta implicit. La SQL scris de mână folosesc `$queryRaw` cu template literal, nu concatenare, iar numele de coloane dinamice le iau dintr-o listă albă.
</details>

## XSS (cross-site scripting)

**Ce e.** Atacatorul reușește să pună cod JavaScript într-o pagină pe care o deschid alți utilizatori. Codul rulează în browserul lor, cu sesiunea lor, și poate fura token-uri sau poate face acțiuni în numele lor.

**Ce face React pentru tine.** Tot ce afișezi cu `{}` e tratat ca **text**, nu ca HTML:

```tsx
<p>{book.description}</p>   // "<script>...</script>" shows up as text, harmless
```

**Unde te poți arde:**

```tsx
// DANGEROUS: renders raw HTML
<div dangerouslySetInnerHTML={{ __html: book.description }} />

// DANGEROUS: a user-provided link could be "javascript:..."
<a href={client.website}>Website</a>
```

**Soluția.**

- Eviți `dangerouslySetInnerHTML`. Dacă chiar trebuie să afișezi HTML venit din afară, îl cureți cu o bibliotecă dedicată (de exemplu DOMPurify).
- Linkurile introduse de utilizatori le validezi pe server: acceptă doar `https://` și `http://`.

**Capcana, specifică P4.** Și textul generat de AI e input din afară. Dacă afișezi răspunsul lui Claude ca HTML sau markdown randat, folosești o componentă de markdown care nu execută HTML brut.

## CSRF, pe scurt

**Ce e.** Un site străin face browserul utilizatorului să trimită o cerere spre aplicația ta. Browserul atașează automat cookie-urile de sesiune, deci cererea pare legitimă. Atacul funcționează doar când autentificarea se bazează pe **cookie-uri trimise automat**.

**În proiectele tale.**

- **P3 (React + Nest):** token-ul pleacă explicit, în header-ul `Authorization: Bearer ...`. Un site străin nu-l poate adăuga, așa că CSRF-ul clasic nu se aplică.
- **P4 (Next.js + `@supabase/ssr`):** sesiunea e în cookie-uri, deci CSRF contează. Te ajută faptul că browserele trimit implicit cookie-urile cu `SameSite=Lax` și că Server Actions din Next.js au protecții încorporate: acceptă doar POST și verifică originea cererii. Regula ta: orice modificare de date se face prin POST, PUT, PATCH sau DELETE, niciodată prin GET.

## Rate limiting

**Ce e.** Limitezi câte cereri poate face un client într-un interval de timp.

**De ce contează.** Te apără de încercări de ghicire a parolelor și de abuz. La P4, mai ales, te apără și de **costuri**: fiecare cerere spre Claude costă bani.

**Cum îl vezi în cod real** (NestJS, `@nestjs/throttler`)

```ts
// app.module.ts
@Module({
  imports: [
    ThrottlerModule.forRoot({
      throttlers: [{ ttl: 60_000, limit: 100 }], // 100 requests / minute / client
    }),
  ],
  providers: [{ provide: APP_GUARD, useClass: ThrottlerGuard }],
})
export class AppModule {}

// a stricter limit on one expensive route
@Throttle({ default: { limit: 5, ttl: 60_000 } })
@Post('extract')
extract() { /* ... */ }
```

**Capcana.** `ttl` e în **milisecunde**. Un `ttl: 60` înseamnă 60 ms, adică practic nicio limită.

## Upload de fișiere

**Ce e.** În P4, utilizatorul urcă facturi (PDF sau poză). Orice fișier urcat e input din afară.

**Regulile.**

- Verifici **pe server** tipul (MIME-ul și extensia, dintr-o listă albă: `application/pdf`, `image/jpeg`, `image/png`) și **mărimea** (de exemplu maxim 10 MB).
- Nu folosești numele original al fișierului ca nume de stocare. Generezi unul (`${userId}/${crypto.randomUUID()}.pdf`).
- Pui fișierele într-un bucket **privat** în Supabase Storage, cu politici care lasă fiecare utilizator să-și vadă doar fișierele lui ([08-baze-de-date-supabase/06-rls-auth-storage.md](../08-baze-de-date-supabase/06-rls-auth-storage.md)).

**Capcana.** Să verifici tipul doar în `<input accept=".pdf">`. E un simplu filtru în dialogul de alegere a fișierului. Cu `curl` urci orice.

## OWASP Top 10, varianta scurtă

**Ce e.** OWASP e o organizație non-profit care publică periodic lista celor mai frecvente categorii de riscuri în aplicațiile web. Versiunea curentă e **2025**. La interviu nu trebuie s-o știi pe de rost, dar trebuie să recunoști categoriile și să știi unde apar la tine.

| # | Categorie | Unde o întâlnești tu |
|---|---|---|
| A01 | Broken Access Control | IDOR, roluri verificate doar în UI ([05](05-autentificare-vs-autorizare.md)) |
| A02 | Security Misconfiguration | RLS lăsat dezactivat, CORS `*`, erori cu stack trace în producție |
| A03 | Software Supply Chain Failures | dependențe vechi sau compromise, pachete inventate de AI |
| A04 | Cryptographic Failures | parole stocate în clar, date sensibile trimise fără HTTPS |
| A05 | Injection | SQL injection, XSS |
| A06 | Insecure Design | logica de business fără verificări, de exemplu double booking-ul din P3 |
| A07 | Authentication Failures | parole slabe, sesiuni care nu expiră. Folosești Supabase Auth |
| A08 | Software or Data Integrity Failures | cod sau date acceptate fără verificare |
| A09 | Security Logging & Alerting Failures | nu știi că ai fost atacat pentru că nu loghezi nimic |
| A10 | Mishandling of Exceptional Conditions | erori tratate prost care lasă aplicația „deschisă” |

**Capcana.** A03 te privește direct când lucrezi cu AI. Claude poate sugera, rar, un pachet npm care nu există sau unul cu un nume foarte apropiat de cel real. Înainte de `npm install pachet-nou`, verifici pe npmjs.com că pachetul există, e popular și e întreținut ([03-ecosistem-js/02-npm-si-pachete.md](../03-ecosistem-js/02-npm-si-pachete.md)).

## Mai departe

- [OWASP Top 10:2025](https://owasp.org/Top10/2025/)
- [Rate limiting](https://docs.nestjs.com/security/rate-limiting) și [Validation](https://docs.nestjs.com/techniques/validation): NestJS
- [Raw queries și SQL injection](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries): Prisma
