# Teste

Cu AI, codul se scrie repede. Testele sunt felul în care știi că ce s-a scris încă merge după următoarea modificare, a ta sau a lui Claude. În P3 scrii teste pentru regula cea mai importantă: **nu poți rezerva un interval deja ocupat.**

## Piramida testelor

**Ce e.** Trei niveluri, de la multe și rapide la puține și lente:

```
        /\        e2e: whole app via HTTP (few, slower)
       /  \
      /----\      integration: several pieces together, e.g. service + real DB
     /      \
    /--------\    unit: one class/function, dependencies mocked (many, milliseconds)
```

**De ce contează.** Unit testele îți spun exact *ce* s-a stricat. E2e testele îți spun că aplicația *chiar* merge cap-coadă. Ai nevoie de amândouă, în proporții diferite.

**Ce merită testat** (pe un proiect mic nu testezi tot):

- regulile de business: suprapunerea rezervărilor, calculul totalului unei facturi;
- cazurile de eroare: 404, 403, 409;
- orice bug reparat (scrii întâi testul care îl reproduce).

**Ce nu merită testat:** că Nest știe să ruteze, că Prisma știe să facă `findMany`, getter-e banale.

**Test runner-ul.** Proiectele create acum cu `nest new` sunt ESM și vin cu **Vitest**. Pe **Jest** îl vei întâlni în proiectele Nest mai vechi (CommonJS). API-ul e aproape identic: `describe`, `it`, `expect`, iar `vi.fn()` în Vitest e `jest.fn()` în Jest. Rulezi cu `npm test`. În CI, la fiecare PR, rulează pe rând `npm run lint`, `npm run typecheck` (`tsc --noEmit`) și `npm test`. Vezi [09/05](../09-git-github-echipa/05-conventii-si-ci.md).

## Unit test pe un service, cu Prisma fals

**Ce e.** Testezi `BookingsService` singur. În loc de baza de date reală, îi dai un obiect fals (mock) care răspunde ce vrei tu. Asta e posibil datorită dependency injection-ului din [02](02-nest-arhitectura.md).

**Service-ul testat:**

```ts
// bookings.service.ts (fragment)
async create(userId: string, dto: CreateBookingDto) {
  const overlapping = await this.prisma.booking.findFirst({
    where: {
      resourceId: dto.resourceId,
      startsAt: { lt: new Date(dto.endsAt) }, // existing starts before new ends
      endsAt: { gt: new Date(dto.startsAt) }, // existing ends after new starts
    },
  });
  if (overlapping) throw new ConflictException('This slot is already booked');
  return this.prisma.booking.create({ data: { ...dto, userId } });
}
```

**Testul:**

```ts
// bookings.service.spec.ts
import { Test } from '@nestjs/testing';
import { ConflictException } from '@nestjs/common';
import { beforeEach, describe, expect, it, vi } from 'vitest';
import { BookingsService } from './bookings.service.js';
import { PrismaService } from '../prisma/prisma.service.js';

describe('BookingsService.create', () => {
  let service: BookingsService;
  const prisma = { booking: { findFirst: vi.fn(), create: vi.fn() } };

  beforeEach(async () => {
    vi.resetAllMocks();
    const moduleRef = await Test.createTestingModule({
      providers: [BookingsService, { provide: PrismaService, useValue: prisma }], // fake instead of real DB
    }).compile();
    service = moduleRef.get(BookingsService);
  });

  const dto = { resourceId: 'court-1', startsAt: '2026-11-10T10:00:00Z', endsAt: '2026-11-10T11:00:00Z' };

  it('rejects an overlapping booking', async () => {
    prisma.booking.findFirst.mockResolvedValue({ id: 'existing' });
    await expect(service.create('user-1', dto)).rejects.toThrow(ConflictException);
    expect(prisma.booking.create).not.toHaveBeenCalled();
  });

  it('creates the booking when the slot is free', async () => {
    prisma.booking.findFirst.mockResolvedValue(null);
    prisma.booking.create.mockResolvedValue({ id: 'new', ...dto });
    await expect(service.create('user-1', dto)).resolves.toMatchObject({ id: 'new' });
  });
});
```

**Ce să observi.** Fiecare test are trei părți: **arrange** (pregătești mock-urile), **act** (chemi metoda) și **assert** (verifici rezultatul). Le vei vedea separate în orice test bine scris.

**Capcana.** Un unit test cu mock nu verifică query-ul în sine. Dacă condiția de suprapunere din `where` e greșită (de exemplu `lte` în loc de `lt`), testul de mai sus trece oricum, pentru că mock-ul răspunde ce i-ai zis. De asta ai nevoie și de testele de mai jos.

<details>
<summary>Întrebare de interviu: Ce e un mock și când îl folosești?</summary>

Un mock e un obiect fals care înlocuiește o dependență reală (baza de date, un API extern) și răspunde controlat. Îl folosesc în unit teste ca să izolez logica pe care o testez, să fac testele rapide și deterministe și să pot simula cazuri greu de reprodus, cum ar fi o eroare de la un serviciu extern. Dezavantajul: mock-ul nu verifică integrarea reală, de aceea completez cu teste de integrare sau e2e pentru fluxurile importante.
</details>

## Testul e2e cu supertest

**Ce e.** Pornești aplicația Nest întreagă, în memorie, și trimiți request-uri HTTP reale cu `supertest`. Verifici status code-uri și body-uri, exact cum le-ar vedea frontend-ul.

**Exemplu** (`test/bookings.e2e-spec.ts`):

```ts
import { Test } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import request from 'supertest';
import { afterAll, beforeAll, describe, it } from 'vitest';
import { AppModule } from '../src/app.module.js';

describe('Bookings (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({ imports: [AppModule] }).compile();
    app = moduleRef.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
    await app.init();
  });

  afterAll(() => app.close());

  it('returns 401 without a token', () => {
    return request(app.getHttpServer()).get('/me/bookings').expect(401);
  });

  it('returns 400 for an invalid body', () => {
    return request(app.getHttpServer())
      .post('/bookings')
      .set('Authorization', `Bearer ${process.env.TEST_USER_JWT}`)
      .send({ resourceId: 'court-1' }) // missing startsAt/endsAt
      .expect(400);
  });
});
```

**Capcana.** `main.ts` **nu** rulează în teste. Ce ai setat acolo (ValidationPipe global, CORS) trebuie repetat în setup-ul testului, altfel testele trec pe o aplicație configurată diferit de cea reală. AI-ul uită des asta. A doua capcană: e2e testele care scriu în baza de date trebuie să ruleze pe o bază separată, niciodată pe cea cu date reale.

## Regula de no double booking, testată în stratul corect

Unit testul verifică *logica* service-ului. Dar două request-uri simultane pot trece amândouă de `findFirst` înainte ca vreunul să facă `create`, iar rezultatul e o rezervare dublă. Asta se numește race condition. Garanția reală vine din **baza de date**, printr-o constrângere: un `UNIQUE (resource_id, starts_at)` dacă sloturile sunt fixe, de o oră, sau o constrângere de excludere pe intervale în Postgres dacă sunt libere. Detaliile sunt în brief-ul [P3](../proiecte/P3-booking.md).

Deci:

- **unit test:** service-ul aruncă 409 când găsește suprapunere;
- **e2e sau integration test:** al doilea `POST` pe același slot primește 409;
- **constrângerea din DB:** prinde ce scapă de amândouă.

<details>
<summary>Întrebare de interviu: Cum previi rezervările duble într-un sistem de booking?</summary>

În cod verific dacă există o rezervare care se suprapune (existentul începe înainte să se termine noul și se termină după ce începe noul) și răspund cu 409. Dar verificarea din cod nu e suficientă la request-uri simultane, din cauza race condition-urilor. Garanția trebuie pusă în baza de date: o constrângere unique pe resursă și slot, sau în Postgres o constrângere de excludere pe intervale de timp. Codul tratează eroarea constrângerii și o transformă tot în 409. Și am teste pe ambele niveluri.
</details>

## O notă despre testele de frontend

În proiectele React (Vite) se folosesc **Vitest** și **React Testing Library**. Principiul e să testezi ce vede și face user-ul („apas butonul, apare mesajul de eroare”), nu detaliile interne ale componentei. Pentru proiectele tale, prioritatea e la backend. În frontend ajunge un test pentru un formular important sau pentru o funcție de calcul (de exemplu statisticile din P1).

## Mai departe

- [NestJS — Testing](https://docs.nestjs.com/fundamentals/testing)
- [Vitest — Getting started](https://vitest.dev/guide/)
- [Testing Library — Guiding principles](https://testing-library.com/docs/guiding-principles)
