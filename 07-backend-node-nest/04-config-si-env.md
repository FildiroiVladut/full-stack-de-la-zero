# Config și env

Parola bazei de date, URL-ul Supabase și portul nu stau în cod. Stau în variabile de mediu (environment variables), diferite pe laptop și în producție. Aici vezi cum le citește Nest și de ce aplicația trebuie să refuze să pornească dacă lipsește una. Partea generală despre env vars e în [10-securitate-env/01](../10-securitate-env/01-env-vars.md).

## Variabile de mediu și `.env`

**Ce e.** Variabilele de mediu sunt perechi `NUME=valoare` pe care sistemul de operare le dă procesului la pornire. În Node le citești din `process.env.NUME`. Local, ca să nu le exporți de mână, le scrii într-un fișier `.env` în rădăcina proiectului:

```bash
# .env — NEVER committed
PORT=3000
DATABASE_URL="postgresql://postgres.abcdefgh:parola@aws-1-eu-central-1.pooler.supabase.com:5432/postgres"
FRONTEND_URL=http://localhost:5173
```

Și un `.env.example` cu aceleași chei, dar fără valori reale, care **intră** în git:

```bash
# .env.example — committed, documents what the app needs
PORT=3000
DATABASE_URL=
FRONTEND_URL=
```

**De ce contează.** Același cod rulează local, pe Render și în CI, cu valori diferite. Și secretele nu ajung niciodată în git.

**Capcana.** Verifică `.gitignore` înainte de primul commit: trebuie să conțină `.env`. Dacă un `.env` cu parole a ajuns pe GitHub, ștergerea fișierului nu ajută, pentru că rămâne în istorie. Schimbi parola. Vezi [10-securitate-env/03](../10-securitate-env/03-secrete-scurse.md).

## `@nestjs/config`

**Ce e.** Modulul oficial Nest care încarcă `.env` și îți dă un `ConfigService` injectabil, în loc de `process.env` presărat prin cod.

**Setarea** (`npm i @nestjs/config`):

```ts
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true, validate }), // isGlobal: inject ConfigService anywhere
    PrismaModule,
    InvoicesModule,
  ],
})
export class AppModule {}
```

**Folosirea:**

```ts
@Injectable()
export class MailService {
  constructor(private readonly config: ConfigService) {}

  private get apiKey() {
    return this.config.getOrThrow<string>('MAIL_API_KEY'); // throws if missing, type is string (not string | undefined)
  }
}
```

**Capcana.** `config.get('X')` returnează `undefined` în tăcere dacă variabila lipsește, iar eroarea apare mult mai târziu, într-un loc fără legătură. Folosește `getOrThrow` pentru tot ce e obligatoriu.

## Validarea env-ului la pornire

**Ce e.** O funcție care verifică toate variabilele *o singură dată, la start*. Dacă lipsește ceva sau are format greșit, aplicația nu pornește și îți spune exact ce lipsește.

**De ce contează.** E diferența dintre:

- **Fără validare:** deploy-ul reușește, aplicația pornește, iar prima factură creată de un client cade cu o eroare criptică de conexiune, la 2 noaptea.
- **Cu validare:** deploy-ul pică imediat pe Render cu `DATABASE_URL should not be empty`, și repari în 30 de secunde.

**Fail fast** e un principiu pe care îl vei auzi des: o problemă de configurare trebuie să oprească totul cât mai devreme, nu să se ascundă până la primul user.

**Exemplu** (`src/config/env.validation.ts`, varianta din documentația Nest, cu `class-validator`, pe care îl ai deja):

```ts
import { plainToInstance } from 'class-transformer';
import { IsInt, IsNotEmpty, IsString, IsUrl, Max, Min, validateSync } from 'class-validator';

class EnvironmentVariables {
  @IsInt() @Min(0) @Max(65535)
  PORT: number = 3000;

  @IsString() @IsNotEmpty()
  DATABASE_URL: string;

  @IsUrl({ require_tld: false }) // allows http://localhost:5173
  FRONTEND_URL: string;
}

export function validate(config: Record<string, unknown>) {
  const validated = plainToInstance(EnvironmentVariables, config, { enableImplicitConversion: true });
  const errors = validateSync(validated, { skipMissingProperties: false });
  if (errors.length > 0) {
    throw new Error(errors.toString()); // the app refuses to start
  }
  return validated;
}
```

Documentația Nest recomandă pentru proiectele noi și librării de tip Standard Schema, cum e zod (îl știi din [05-typescript/05](../05-typescript/05-validare-cu-zod.md)). Oricare merge. Important e să existe validarea.

**Capcana.** `enableImplicitConversion` contează: toate variabilele de mediu sunt string-uri. Fără conversie, `PORT=3000` e `"3000"` și pică la `@IsInt()`.

<details>
<summary>Întrebare de interviu: Cum gestionezi configurarea și secretele într-o aplicație Node?</summary>

Secretele și tot ce diferă între medii stau în variabile de mediu, niciodată în cod. Local le țin într-un `.env` ignorat de git, cu un `.env.example` comis ca documentație. În producție le setez din dashboard-ul platformei (Render, Vercel). În Nest folosesc `@nestjs/config` cu o funcție de validare, ca aplicația să nu pornească dacă lipsește ceva: fail fast. În cod accesez valorile prin `ConfigService.getOrThrow`, nu prin `process.env` direct.
</details>

## CORS, configurat din env

**Ce e.** Frontend-ul de pe `localhost:5173` (sau de pe Vercel) cheamă API-ul de pe `localhost:3000` (sau de pe Render). Sunt origini diferite, iar browserul blochează răspunsul dacă serverul nu spune explicit că are încredere în originea aia. Conceptul e explicat în [02-cum-functioneaza-webul/05](../02-cum-functioneaza-webul/05-auth-cookies-cors.md).

**Setarea în `main.ts`:**

```ts
const config = app.get(ConfigService);
app.enableCors({
  origin: config.getOrThrow<string>('FRONTEND_URL'), // e.g. https://invoicer.vercel.app in production
});
await app.listen(config.get<number>('PORT') ?? 3000);
```

**Capcana.** `origin: '*'` „rezolvă” eroarea de CORS și e exact ce va propune AI-ul când îi arăți eroarea. Pentru un API cu date private e greșit. Pui URL-ul exact al frontend-ului, din env, ca să fie diferit local și în producție. Și reține: CORS nu protejează API-ul de curl sau Postman. Protejează doar user-ii din browser.

## Mai departe

- [NestJS — Configuration](https://docs.nestjs.com/techniques/configuration)
- [NestJS — CORS](https://docs.nestjs.com/security/cors)
