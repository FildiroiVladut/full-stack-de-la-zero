# 03 · Ecosistemul JavaScript

JavaScript-ul de azi nu înseamnă doar limbajul. Înseamnă runtime-uri, package manager-e, bundlere, linter-e și framework-uri, fiecare cu rolul lui. Când Claude îți generează un proiect, primești dintr-odată zeci de fișiere de configurare și sute de pachete. Capitolul ăsta îți dă harta, ca să știi ce e fiecare bucată, de ce e acolo și ce poate merge prost. Tot aici înveți diferența dintre npm și Bun, pe propriul laptop.

## La final poți să…

- explici diferența dintre engine, runtime, package manager și bundler, și unde stau Node, npm, Bun și Vite;
- citești un `package.json` și spui stack-ul, comenzile și ce intră în producție;
- explici ce e lockfile-ul, de ce intră în git și când folosești `npm ci`;
- spui de ce Vite nu verifică tipurile și ce înseamnă prefixul `VITE_`;
- verifici un pachet înainte să-l instalezi, mai ales unul sugerat de AI.

## Fișierele, în ordine

- [ ] [01-runtime-node-bun.md](01-runtime-node-bun.md): engine vs runtime, ce adaugă Node, LTS, Bun și Deno (~30 min)
- [ ] [02-npm-si-pachete.md](02-npm-si-pachete.md): pachete, `package.json`, lockfile, semver, scripts, `npx`, riscuri de supply chain (~50 min)
- [ ] [03-npm-vs-bun.md](03-npm-vs-bun.md): **hands-on**, același proiect cu npm și cu Bun, cronometrat și comparat (~60 min)
- [ ] [04-vite-si-build.md](04-vite-si-build.md): bundling, transpilare, dev server, HMR, build, variabile de mediu (~40 min)
- [ ] [05-harta-ecosistemului.md](05-harta-ecosistemului.md): library vs framework, harta uneltelor, linter vs formatter (~40 min)

Total: ~4 ore.

## Unde îl folosești

- **P1 Shelf**: primul proiect Vite, primul `npm install`, varianta cu Bun pentru comparație, primul build și deploy.
- **P2 Invoicer și P3 Booking**: două `package.json`-uri (frontend și backend), `dependencies` vs `devDependencies` contează la deploy-ul backend-ului, `VITE_API_URL`.
- **P4 Kitchen AI**: Next.js și diferența dintre variabilele publice și cele secrete.
- **[CP1](../checkpoints/CP1.md)**: diferența dintre npm și Bun, ce e un package manager și ce e un bundler.

## Verifică-te

1. Care e diferența dintre npm și Bun? Dar dintre a folosi Bun ca package manager și a-l folosi ca runtime?
2. De ce intră `package-lock.json` în git, iar `node_modules` nu?
3. Aplicația pornește cu `npm run dev`, dar build-ul de pe Vercel pică. Care sunt două cauze posibile?
4. Ce înseamnă `"^19.3.1"` și ce înseamnă `"~6.0.3"`?
5. Claude îți sugerează un pachet pe care nu-l cunoști. Ce verifici înainte de `npm install`?
