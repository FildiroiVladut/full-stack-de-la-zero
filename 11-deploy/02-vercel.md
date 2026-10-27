# Vercel

Vercel e platforma pe care stau frontend-urile tale: P1 complet, frontend-ul din P2 și P3 și toată aplicația Next.js din P4. Planul **Hobby** e gratuit și e gândit pentru proiecte personale, necomerciale. Pentru portofoliu e exact ce-ți trebuie.

Pașii de mai jos sunt pentru un SPA Vite (P1). La final ai diferențele pentru Next.js.

## Înainte să începi

Verifică trei lucruri, local, în WSL:

```bash
npm run build      # must finish without errors
npm run preview    # serves dist/ locally, like production would
git status         # everything committed and pushed to GitHub
```

Dacă `npm run build` pică local, va pica și pe Vercel. Repară întâi local.

## Pașii

### 1. Contul

Intră pe [vercel.com](https://vercel.com) din browserul din Windows și fă-ți cont cu **Continue with GitHub**, cu același cont pe care sunt repo-urile. Alege planul Hobby.

**Verifică:** ajungi în dashboard și vezi butonul de proiect nou.

### 2. Importă repo-ul

În dashboard: **Add New… → Project** (sau butonul **New Project**). Vercel îți arată repo-urile din GitHub. La primul import îi dai voie să le vadă: poți alege doar anumite repo-uri.

Alege repo-ul (de exemplu `shelf`). Pe ecranul de configurare:

- **Framework Preset:** Vite (detectat automat);
- **Root Directory:** rădăcina repo-ului (lasă implicit);
- **Build Command / Output Directory:** implicite (`npm run build`, `dist`). Nu le schimba.

**Verifică:** Vercel a recunoscut singur Vite. Dacă scrie „Other”, probabil ai ales folderul greșit.

### 3. Variabilele de mediu (dacă proiectul are)

P1 nu are nevoie de niciuna (Open Library nu cere cheie). La P2 și P3 adaugi aici, înainte de primul deploy:

```
VITE_API_URL = https://invoicer-api.onrender.com
```

Fiecare variabilă se aplică pe unul sau mai multe medii: **Production**, **Preview**, **Development**. Pentru URL-ul API-ului bifează Production și Preview.

> Tot ce începe cu `VITE_` ajunge în JavaScript-ul trimis browserului și poate fi citit de oricine. URL-uri și chei *publishable*, da. Chei secrete, niciodată. Vezi [02-ce-ajunge-in-browser](../10-securitate-env/02-ce-ajunge-in-browser.md).

### 4. Rewrite pentru client-side routing

Un SPA are un singur `index.html`. Fără configurare, un refresh pe `/books/OL123W` dă 404 pe Vercel (vezi [06-routing-si-date](../06-react/06-routing-si-date.md)). Creezi în rădăcina proiectului fișierul `vercel.json`:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

Commit, push.

**Verifică:** după deploy, deschizi direct o rută internă (`/shelf`) și dai refresh. Trebuie să vezi pagina, nu un 404.

### 5. Deploy

Apeși **Deploy**. Durează de obicei sub un minut pentru un proiect Vite. Primești un URL de forma `shelf-xxxx.vercel.app`. Din Settings poți schimba numele proiectului, ca URL-ul să arate bine în CV.

**Verifică:** deschide URL-ul pe telefon. Așa îl vede un recrutor.

### 6. Preview deployments pe fiecare PR

De acum, la fiecare push pe un branch care nu e `main`, Vercel face un deploy separat, de preview. Când deschizi un PR, botul Vercel lasă pe el un comentariu cu linkul.

```
git switch -c feature/reading-stats
# ...commits...
git push -u origin feature/reading-stats
gh pr create --fill
# → on the PR page: "Vercel — Preview: https://shelf-git-feature-reading-stats-....vercel.app"
```

**Verifică:** Bogdan poate deschide linkul de preview și vede schimbarea live, înainte de merge.

### 7. Rollback, dacă ai stricat producția

Pe pagina proiectului, la tile-ul deployment-ului de producție, ai **Instant Rollback**. Pe Hobby te poți întoarce la deployment-ul de producție imediat anterior. Repari apoi printr-un commit nou. Atenție: după un rollback, push-urile noi nu mai ajung automat în producție până nu apeși **Undo Rollback**.

## Ce s-a întâmplat de fapt

```
git push → GitHub notifies Vercel
  → Vercel clones the repo in a clean container
  → npm install (from package-lock.json)
  → npm run build (VITE_* values are baked into the JS here)
  → uploads dist/ to its CDN
  → assigns a URL to this immutable deployment
```

Fiecare deployment e **imutabil**: are URL-ul lui și nu se mai schimbă. „Producția” e doar un pointer spre deployment-ul curent, iar de aici vine rollback-ul instant.

O consecință importantă: **schimbarea unei variabile de mediu nu afectează deployment-urile existente**. Valorile `VITE_*` au fost copiate în cod la build. După ce schimbi o variabilă, faci redeploy (din tab-ul Deployments, sau cu un commit nou).

## Next.js pe Vercel (P4)

Vercel e compania din spatele Next.js, deci deploy-ul e fără configurare: același import, framework preset **Next.js**, fără `vercel.json` pentru rewrite. Diferențele care contează:

- **Partea de server rulează ca funcții.** Server Components, Server Actions și Route Handlers rulează ca Vercel Functions, iar pe Hobby o funcție are o durată maximă de câteva minute. Un apel la Claude pentru o factură încape lejer, dar nu poți face procesări de zeci de minute.
- **Variabilele fără `NEXT_PUBLIC_` rămân pe server.** `ANTHROPIC_API_KEY` și `SUPABASE_SECRET_KEY` le adaugi ca variabile de tip **Secret**: după salvare nu mai pot fi citite din dashboard, doar înlocuite.
- **Logurile de runtime.** Pe Hobby, Vercel păstrează logurile funcțiilor doar pentru o perioadă scurtă (o oră, la momentul scrierii). Pentru debugging uită-te imediat după ce reproduci problema. Detalii în [04-loguri-si-productie](04-loguri-si-productie.md).

## Vercel CLI (opțional)

Nu ai nevoie de el pentru fluxul de mai sus. E util când vrei variabilele de mediu din Vercel și pe laptop:

```bash
npm i -g vercel
vercel login
vercel link        # connects the local folder to the Vercel project
vercel env pull    # writes the Development env vars into a local .env file
```

## Capcane

- **Ai schimbat o variabilă și nu s-a întâmplat nimic:** n-ai făcut redeploy.
- **404 la refresh pe o rută internă:** lipsește `vercel.json` cu rewrite.
- **Ecran alb în producție, local merge:** deschide consola din browser. De obicei e un `VITE_API_URL` lipsă (deci `undefined/invoices`) sau o eroare CORS.
- **Build-ul pică doar pe Vercel:** importuri cu litere mari/mici greșite sau o dependență instalată global pe laptop, care lipsește din `package.json`. Citește logul de build de sus în jos până la prima eroare.

<details>
<summary>Întrebare de interviu: De ce o variabilă `VITE_*` schimbată în dashboard nu apare în site până la un nou deploy?</summary>

Pentru că Vite înlocuiește `import.meta.env.VITE_*` cu valoarea literală în momentul build-ului. Valoarea e copiată în fișierele JavaScript generate. Deployment-ul existent conține fișierele vechi, cu valoarea veche. Ca să intre valoarea nouă, trebuie un build nou. Tot din acest motiv, orice variabilă de build ajunge publică în browser.
</details>

## Mai departe

- [Vite on Vercel](https://vercel.com/docs/frameworks/frontend/vite)
- [Environment variables](https://vercel.com/docs/environment-variables)
- [Deploying Git repositories](https://vercel.com/docs/git)
