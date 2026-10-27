# DTO, validare, erori

Regula numărul unu în backend: **nu ai încredere în nimic din ce vine de la client.** Validarea din frontend e pentru confortul user-ului, iar validarea din backend e cea care contează pentru securitate. Aici vezi cum o face Nest și cum răspunzi corect când ceva nu e în regulă.

## DTO (Data Transfer Object)

**Ce e.** O clasă care descrie forma exactă a datelor care intră într-un endpoint. Cu decoratori din `class-validator`, aceeași clasă spune și regulile: câmp obligatoriu, tip, interval permis.

**De ce contează.** E contractul API-ului tău. Cine citește `CreateInvoiceDto` știe exact ce trebuie trimis, fără să caute prin service.

**Exemplu minim** (`npm i class-validator class-transformer`):

```ts
import { Type } from 'class-transformer';
import { ArrayMinSize, IsInt, IsISO8601, IsString, IsUUID, Max, Min, MinLength, ValidateNested } from 'class-validator';

export class CreateInvoiceItemDto {
  @IsString() @MinLength(1)
  description: string;

  @IsInt() @Min(1)
  quantity: number;

  @IsInt() @Min(0)
  unitPriceCents: number; // money as integer cents, never floats
}

export class CreateInvoiceDto {
  @IsUUID()
  clientId: string;

  @IsISO8601()
  dueDate: string;

  @IsInt() @Min(0) @Max(10_000)
  vatRateBps: number; // basis points: 2100 = 21%, integer like the money

  @ValidateNested({ each: true })
  @Type(() => CreateInvoiceItemDto) // tells the transformer which class each item is
  @ArrayMinSize(1)
  items: CreateInvoiceItemDto[];
}
```

**Despre `!`.** Dacă `tsconfig.json` are `strict: true`, TypeScript se plânge că `clientId` nu e inițializat. Scrii atunci `clientId!: string`. Semnul `!` înseamnă „promit că valoarea vine din afară”, adică o pune ValidationPipe. Vei vedea ambele variante în proiecte.

**Ce să observi.** Nu există câmp `totalCents`. Totalul îl calculează serverul din linii. Dacă l-ai accepta din body, oricine ar putea trimite o factură de 1000 de lei cu total 1 leu.

**Capcana.** La array-uri de obiecte, fără `@ValidateNested` + `@Type`, obiectele din interior **nu sunt validate deloc**. Trec orice. E o greșeală pe care și AI-ul o face des, așa că verific-o la fiecare DTO cu obiecte imbricate.

## ValidationPipe global

**Ce e.** Un pipe e o clasă care transformă sau validează datele înainte să ajungă în handler. `ValidationPipe` ia body-ul, îl transformă în instanță de DTO și rulează toate regulile. Dacă ceva pică, răspunde automat cu 400 și handler-ul nici nu mai rulează.

**Setarea** (o dată, în `main.ts`):

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,            // strips properties that have no decorator in the DTO
    forbidNonWhitelisted: true, // ...or better: rejects the request with 400
    transform: true,            // body becomes a real DTO instance; ":id" params get converted to declared types
  }),
);
```

**De ce `whitelist` contează.** Fără el, un client poate trimite `{ ..., "status": "PAID" }` la creare, iar dacă service-ul face `prisma.invoice.create({ data: dto })`, factura apare plătită. Atacul se numește *mass assignment*. `whitelist` taie tot ce nu e declarat explicit.

**Cum arată eroarea:**

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": ["items must contain at least 1 elements", "vatRateBps must not be greater than 10000"]
}
```

Frontend-ul poate afișa direct mesajele din `message`.

<details>
<summary>Întrebare de interviu: De ce validezi și pe backend dacă formularul din frontend validează deja?</summary>

Pentru că frontend-ul e sub controlul user-ului. Oricine poate trimite request-uri direct cu curl sau Postman, ocolind formularul complet. Validarea din frontend e pentru experiență (feedback imediat), iar cea din backend e pentru corectitudine și securitate. În Nest o fac cu DTO-uri și `ValidationPipe` global, cu `whitelist` ca să nu accept câmpuri pe care nu le-am declarat.
</details>

## Excepțiile HTTP built-in

**Ce e.** Nest are clase de excepție pentru fiecare status uzual. Le arunci din service, iar Nest le transformă în răspunsul HTTP corect.

| Situația | Excepția | Status |
|---|---|---|
| Body invalid (o face ValidationPipe pentru tine) | `BadRequestException` | 400 |
| Lipsește token-ul sau e invalid | `UnauthorizedException` | 401 |
| Ești logat, dar n-ai voie | `ForbiddenException` | 403 |
| Resursa nu există | `NotFoundException` | 404 |
| Conflict cu starea curentă (factură deja plătită, slot ocupat) | `ConflictException` | 409 |
| Eroare neprevăzută | orice altă excepție | 500 |

**Exemplu din P2:**

```ts
async send(id: string) {
  const invoice = await this.findOne(id); // throws 404 if missing
  if (invoice.status !== 'DRAFT') {
    throw new ConflictException('Only draft invoices can be sent');
  }
  return this.prisma.invoice.update({ where: { id }, data: { status: 'SENT' } });
}
```

**Capcana.** 401 vs 403 e o întrebare clasică. 401 înseamnă „nu știu cine ești”, iar 403 înseamnă „știu cine ești, dar n-ai voie”. Altă capcană: orice eroare neprinsă devine 500 cu mesajul generic `Internal server error`. E bine că nu scapă detalii interne spre client, dar înseamnă că detaliile le găsești doar în log-urile serverului.

## Exception filters (pe scurt)

**Ce e.** Locul care prinde excepțiile și construiește răspunsul. Nest are unul implicit, care face tabelul de mai sus. Îți scrii unul propriu doar când vrei alt format de răspuns sau vrei să traduci erori din alte librării.

**Un caz real.** Prisma aruncă o eroare cu codul `P2002` când încalci o constrângere `UNIQUE`, de exemplu un număr de factură duplicat. Fără filtru, user-ul primește 500. Un filtru mic o poate transforma în 409:

```ts
@Catch(Prisma.PrismaClientKnownRequestError)
export class PrismaExceptionFilter implements ExceptionFilter {
  catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();
    if (exception.code === 'P2002') {
      return response.status(409).json({ statusCode: 409, message: 'Already exists' });
    }
    return response.status(500).json({ statusCode: 500, message: 'Internal server error' });
  }
}
```

Nu-l scrie din prima. Când ți se întâmplă un 500 pe un duplicat, cere-i lui Claude un filtru ca ăsta și citește-l cu harta asta în minte.

## Documentația API cu Swagger

**Ce e.** `@nestjs/swagger` generează automat o pagină de documentație interactivă (OpenAPI) din controllers și DTO-uri. Poți testa endpoint-urile direct din browser.

**Setarea** (`npm i @nestjs/swagger`, apoi în `main.ts`):

```ts
const config = new DocumentBuilder()
  .setTitle('Invoicer API')
  .setVersion('1.0')
  .addBearerAuth() // adds the "Authorize" button for JWTs (useful in P3)
  .build();
const documentFactory = () => SwaggerModule.createDocument(app, config);
SwaggerModule.setup('docs', app, documentFactory); // -> http://localhost:3000/docs
```

**De ce contează.** E primul lucru pe care îl deschide un recrutor tehnic care se uită la backend-ul tău. Un link `/docs` în README-ul proiectului arată că știi cum se livrează un API.

**Capcana.** Fără `@ApiProperty()` pe câmpurile DTO-urilor (sau fără plugin-ul CLI al Swagger activat în `nest-cli.json`), pagina arată body-uri goale. Dacă `/docs` arată `{}` la `POST`, asta e cauza.

<details>
<summary>Întrebare de interviu: Ce coduri de status ai returna pentru: creare reușită, resursă inexistentă, user nelogat, user fără drepturi, conflict?</summary>

201 Created pentru creare, 404 Not Found pentru resursă inexistentă, 401 Unauthorized când lipsește autentificarea sau token-ul e invalid, 403 Forbidden când user-ul e autentificat dar n-are drepturi, și 409 Conflict când operația contrazice starea curentă, de exemplu o rezervare pe un interval ocupat. Pentru input invalid, 400 Bad Request.
</details>

## Mai departe

- [NestJS — Validation](https://docs.nestjs.com/techniques/validation)
- [NestJS — Exception filters](https://docs.nestjs.com/exception-filters)
- [NestJS — OpenAPI (Swagger)](https://docs.nestjs.com/openapi/introduction)
