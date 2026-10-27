# Variabile de mediu

Același cod rulează pe laptopul tău, în CI și în producție, dar cu altă bază de date, alte chei și alte URL-uri. Variabilele de mediu sunt mecanismul prin care codul primește aceste valori din afară, fără să le aibă scrise în el.

## Ce e o variabilă de mediu

**Ce e.** O pereche `NUME=valoare` pe care sistemul de operare (sau platforma de hosting) o dă unui proces când îl pornește. Procesul o citește la rulare.

**De ce contează.** Secretele și configurările care diferă de la un mediu la altul nu stau în cod. Codul rămâne același, doar valorile se schimbă.

**Exemplu minim** (în terminalul din WSL)

```bash
echo $HOME                                  # a variable the system already has
GREETING="salut" node -e 'console.log(process.env.GREETING)'
# salut
```

**Capcana.** Variabilele de mediu sunt mereu **string-uri**. `PORT=3000` ajunge în cod ca `"3000"`, iar `DEBUG=false` ajunge ca `"false"`, adică un string care e truthy. Le convertești explicit sau le validezi (vezi ultima fișă).

## .env și .env.example

**Ce e.**

- `.env` e un fișier local cu variabilele tale reale, câte una pe linie. Uneltele (Nest, Vite, Next.js, Prisma) îl citesc automat și le pun în mediu.
- `.env.example` e același fișier, dar cu valori goale sau false. El intră în Git și le arată colegilor ce variabile trebuie să-și seteze.

**Exemplu minim** (backend-ul din P2)

```bash
# .env.example  (committed, no real values)
DATABASE_URL=
DIRECT_URL=
PORT=3000
FRONTEND_URL=http://localhost:5173
```

```bash
# .env  (NOT committed, real values)
DATABASE_URL=postgresql://postgres.abcd:secret@aws-0-eu-central-1.pooler.supabase.com:5432/postgres
DIRECT_URL=postgresql://postgres.abcd:secret@aws-0-eu-central-1.pooler.supabase.com:5432/postgres
PORT=3000
FRONTEND_URL=http://localhost:5173
```

Când cineva clonează repo-ul, din README află că rulează `cp .env.example .env` și completează valorile.

**Cum îl vezi în cod real.** Orice proiect serios are `.env.example` în root și o secțiune *Environment variables* în README. Pentru proiectele tale e deja în [templates/README-proiect.md](../templates/README-proiect.md).

**Capcana.** Să adaugi o variabilă nouă în `.env` și să uiți de `.env.example`. Colegul (sau tu, peste o lună, pe alt laptop) pornește aplicația și primește o eroare criptică.

## Medii diferite

**Ce e.** Aplicația ta rulează în mai multe medii (environments), fiecare cu valorile lui:

| Mediu | Unde | De unde vin variabilele |
|---|---|---|
| local (development) | laptopul tău | `.env` |
| CI | GitHub Actions | *secrets* și *variables* din setările repo-ului |
| preview | un deploy pentru fiecare PR (Vercel) | setările proiectului pe Vercel |
| production | Vercel, Render | dashboard-ul platformei |

**De ce contează.** În producție nu există fișier `.env`. Variabilele le setezi în dashboard-ul platformei: vezi [11-deploy/02-vercel.md](../11-deploy/02-vercel.md) și [11-deploy/03-render.md](../11-deploy/03-render.md).

**Capcana.** Aceeași cheie și aceeași bază de date pentru development și producție. Un test local îți șterge datele reale, iar o cheie scăpată de pe laptop îți deschide producția. Ideal ai chei separate pe mediu.

## Cum le citește fiecare unealtă

**Ce e.** Ideea e aceeași peste tot, doar sintaxa diferă.

**Node.js simplu**

```ts
// node --env-file=.env server.js   (Node can load a .env file natively)
const port = Number(process.env.PORT ?? 3000);
```

**NestJS** (cu `@nestjs/config`, detalii în [07-backend-node-nest/04-config-si-env.md](../07-backend-node-nest/04-config-si-env.md))

```ts
@Injectable()
export class InvoicesService {
  constructor(private readonly config: ConfigService) {}

  frontendUrl() {
    return this.config.getOrThrow<string>('FRONTEND_URL'); // throws if missing
  }
}
```

**Vite** (frontend-ul din P1–P3)

```ts
// only variables prefixed with VITE_ are exposed, and they are PUBLIC
const apiUrl = import.meta.env.VITE_API_URL;
```

**Next.js** (P4)

```ts
// server-only code (route handler, server action, server component):
const key = process.env.ANTHROPIC_API_KEY;     // stays on the server

// anywhere, including the browser — inlined into the JS bundle at build time:
const url = process.env.NEXT_PUBLIC_SUPABASE_URL;
```

**Capcana.** În Vite și Next.js, variabilele publice (`VITE_*`, `NEXT_PUBLIC_*`) sunt **înlocuite în cod la build**. Dacă le schimbi pe Vercel, trebuie să faci redeploy, altfel aplicația rămâne cu valoarea veche. Iar local, după ce modifici `.env`, repornești dev server-ul.

## Validezi variabilele la pornire

**Ce e.** La pornire, aplicația verifică dacă are toate variabilele de care are nevoie, în formatul corect. Dacă nu le are, se oprește imediat cu un mesaj clar.

**De ce contează.** Principiul se cheamă *fail fast*. E mult mai bine ca serverul să refuze să pornească cu „DATABASE_URL lipsește” decât să pornească și să cadă la prima cerere, la 2 noaptea, cu `Cannot read properties of undefined`.

**Exemplu minim** (cu zod, într-un proiect Node)

```ts
import { z } from 'zod';

const EnvSchema = z.object({
  DATABASE_URL: z.url(),
  PORT: z.coerce.number().default(3000),   // "3000" -> 3000
  FRONTEND_URL: z.url(),
});

export const env = EnvSchema.parse(process.env); // throws with a clear message
```

**Cum îl vezi în cod real.** În NestJS, `ConfigModule.forRoot({ validate })` face același lucru ([07-backend-node-nest/04-config-si-env.md](../07-backend-node-nest/04-config-si-env.md)). În Next.js, un fișier `env.ts` ca cel de mai sus, importat de codul de server.

**Capcana.** Codul de genul `process.env.API_KEY || ''` ascunde problema: aplicația pornește cu o cheie goală și pică abia mai târziu, într-un loc care n-are nicio legătură cu cauza.

<details>
<summary>Întrebare de interviu: Cum gestionezi configurarea și secretele într-o aplicație Node?</summary>

Prin variabile de mediu: local dintr-un `.env` ignorat de Git, în producție din dashboard-ul platformei sau dintr-un secret manager. În repo pun un `.env.example` cu toate cheile, fără valori. La pornire validez variabilele cu o schemă, ca aplicația să pice imediat cu un mesaj clar dacă lipsește ceva. Folosesc chei separate pe medii, iar în frontend nu pun niciodată un secret, pentru că acolo totul e public.
</details>

## Mai departe

- [Env Variables and Modes](https://vite.dev/guide/env-and-mode): Vite
- [Environment Variables](https://nextjs.org/docs/app/guides/environment-variables): Next.js
- [Configuration](https://docs.nestjs.com/techniques/configuration): NestJS
