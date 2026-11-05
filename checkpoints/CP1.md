# CP1 · Fundamentele

**Când:** finalul săptămânii 1 · **Acoperă:** [00](../00-start/), [01](../01-setup/), [02](../02-cum-functioneaza-webul/), [03](../03-ecosistem-js/), [04](../04-javascript/), [09/01](../09-git-github-echipa/01-cum-gandeste-git.md)

## Poți explica fără notițe

- [x] Ce se întâmplă, pe scurt, de când scrii un URL în browser până vezi pagina.
- [x] Diferența dintre frontend și backend, și de ce secretele nu au voie în frontend.
- [x] Ce conține un request HTTP (metodă, URL, headers, body) și ce înseamnă 200, 201, 400, 401, 403, 404, 500.
- [x] Cum ar arăta un API REST pentru facturi: rutele și metodele.
- [x] Ce e CORS și ce NU e.
- [x] Runtime vs package manager: Node vs npm vs Bun.
- [x] Ce face `npm install`, ce e `package-lock.json` și de ce intră în git.
- [x] `dependencies` vs `devDependencies`, și ce înseamnă `^1.4.2`.
- [x] Ce face Vite la `npm run build`.
- [x] În ce ordine se execută un `console.log`, un `setTimeout(..., 0)` și un `Promise.then`, și de ce.
- [x] `async`/`await` cu `fetch`, și de ce `fetch` nu aruncă eroare la un 404.
- [x] Named export vs default export.
- [x] Ce e un commit, ce e staging area, ce e un branch.

## Quiz cu Claude

Scor: **8.3 / 10** (2026-11-03, a doua încercare) · De recitit: [04/02 Event loop și promises](../04-javascript/02-event-loop-promises.md), [04/03 async/await și fetch](../04-javascript/03-async-await-fetch.md), [03/02 npm și pachete](../03-ecosistem-js/02-npm-si-pachete.md), [09/01 Cum gândește git](../09-git-github-echipa/01-cum-gandeste-git.md)

Slab încă: `Promise.all` vs `forEach` + `async`, rolul `package-lock.json` la `npm install`, commit-ul salvează staging-ul (nu tot proiectul).

Prima încercare: 6.75 / 10 (2026-11-02).

## Livrabile

- [x] **Mediul complet:**
  - [x] WSL cu Ubuntu și Warp
  - [x] git cu SSH și `gh`
  - [x] Node LTS prin fnm și Bun
  - [x] VS Code cu extensia WSL
  - [x] Claude Code logat
- [x] Repo-ul ăsta publicat pe GitHub-ul tău, cu Bogdan colaborator.
- [x] `js-de-la-zero` clonat în WSL.
- [x] Bug-ul din `ziua7/05-inventar.js` reparat (vezi [04-javascript/01-recap-zilele-1-7.md](../04-javascript/01-recap-zilele-1-7.md)), cu un commit care spune ce era greșit. → [9c9d27a](https://github.com/FildiroiVladut/js-de-la-zero/commit/9c9d27a)
- [ ] Un `ziua8/` în `js-de-la-zero`, scris **de mână, fără AI**: un script care caută o carte pe Open Library cu `fetch` + `async/await`, tratează erorile (inclusiv `res.ok`) și afișează primele 5 rezultate.
- [x] [jurnal.md](../jurnal.md) completat zilnic.

## Call cu Bogdan

_Notițe:_
