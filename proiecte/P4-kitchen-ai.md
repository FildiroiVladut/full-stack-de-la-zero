# P4 · Kitchen AI (flagship)

> Factura de la furnizor intră, costurile rețetelor ies.

**Când:** săptămânile 7–8 (~40 h + polish) · **Tip:** produs AI full-stack · **Domeniu:** bucătărie profesională, adică al tău

## De ce e flagship

Un recruiter vede zeci de clone de ChatGPT pe săptămână. Proiectul ăsta rezolvă o problemă reală, pe care o știi din interior: prețurile de la furnizori se schimbă tot timpul, iar food cost-ul din fișele tehnice rămâne pe hârtie, neactualizat, până când marja dispare fără să observe nimeni. Poți vorbi despre el ca omul care a stat la pass, nu ca un student care a urmat un tutorial.

## Ce face

1. Urci factura furnizorului: un PDF sau o poză făcută cu telefonul.
2. Claude citește factura și extrage furnizorul, data și liniile (produs, cantitate, unitate, preț).
3. **Tu verifici:** fiecare linie e potrivită cu un ingredient din lista ta. Aplicația propune potrivirea, tu confirmi sau corectezi.
4. La confirmare, prețurile ingredientelor se actualizează, cu istoric.
5. Costul pe porție, food cost-ul și marja fiecărei rețete se recalculează.
6. Dashboard-ul îți arată ce rețete au depășit food cost-ul țintă după ultima factură, și cu cât.

## Ce înveți pe el

- **Next.js App Router:** Server vs Client Components, Server Actions și Route Handlers.
- **Integrarea unui LLM pe server:** cheia API nu ajunge niciodată în browser.
- **Input multimodal și structured output:** PDF și poze la intrare, date structurate validate cu zod la ieșire.
- **Human-in-the-loop:** AI-ul propune, omul confirmă. Așa se construiesc produsele AI serioase.
- **Costuri, latență, erori și refuzuri** la un API de AI.
- **Evals:** cum măsori dacă extragerea e bună, în loc să ghicești.
- **Supabase Auth cu `@supabase/ssr`, Storage și RLS.** Aici frontend-ul vorbește direct cu Supabase, deci RLS-ul e obligatoriu, nu opțional.

**Capitole:** [06/08 Next.js](../06-react/08-nextjs.md), [08/06](../08-baze-de-date-supabase/06-rls-auth-storage.md), [10/02](../10-securitate-env/02-ce-ajunge-in-browser.md), [12](../12-lucrul-cu-ai/), [13](../13-interviuri/).

## Arhitectura

```
             ┌─────────────────────────── Vercel ────────────────────────────┐
 Browser ──> │ Next.js                                                       │
             │  Client Components (UI)                                        │
             │  Server Components / Server Actions ──> Claude API             │
             │                     │        ANTHROPIC_API_KEY (doar server)   │
             └─────────────────────┼─────────────────────────────────────────┘
                                   v
                            Supabase: Auth · Postgres (RLS) · Storage (facturi)
```

## Modelul de date

- **`ingredients`:** `user_id`, `name`, `unit` (kg, l, buc), `current_price_cents` (pe unitate).
- **`ingredient_prices`:** istoricul prețului (`ingredient_id`, `price_cents`, `supplier`, `invoice_id`, `recorded_at`).
- **`recipes`:** `user_id`, `name`, `portions` (randament), `selling_price_cents`, `target_food_cost_pct`.
- **`recipe_items`:** `recipe_id`, `ingredient_id`, `quantity`, `unit`, `waste_pct`.
- **`invoices`:** `user_id`, `supplier`, `invoice_date`, `file_path` (în Storage), `status` (`uploaded` | `extracted` | `confirmed`).
- **`invoice_lines`:** `invoice_id`, `raw_name`, `quantity`, `unit`, `unit_price_cents`, `matched_ingredient_id`.

Fiecare tabel are RLS, cu politici pe `auth.uid() = user_id`. Fiecare bucătar vede doar datele lui.

P4 are **proiectul lui Supabase**, separat de cel comun al lui P2 și P3. Tabelele stau în `public`, pentru că aici frontend-ul vorbește direct cu Supabase, prin cheia publishable, iar RLS-ul e cel care protejează datele.

## Funcționalități

**Obligatorii**

1. Login cu Supabase Auth.
2. Ingrediente cu unitate și preț curent, plus istoricul prețului.
3. Rețete (fișe tehnice): ingrediente, cantități, randament, pierderi %, preț de vânzare. Din ele rezultă costul pe porție, food cost-ul % și marja.
4. Upload de factură (PDF sau poză de pe telefon) în Supabase Storage.
5. Extragere cu Claude: furnizor, dată, linii. Ieșirea e structurată și validată cu zod.
6. Ecranul de review: fiecare linie e potrivită cu un ingredient existent (cu sugestie automată) sau creează unul nou. Omul confirmă.
7. La confirmare: prețuri actualizate cu istoric și rețete recalculate.
8. Dashboard cu rețetele care au depășit food cost-ul țintă.
9. **Evals:**
   - 8–10 facturi reale, anonimizate, în `eval/`, fiecare cu rezultatul așteptat scris de mână;
   - `npm run eval` rulează extragerea pe toate și raportează cât de bine a mers (linii găsite, prețuri corecte).
10. **Folosit de 2–3 bucătari reali** pe facturile lor. Ce au spus intră în README.

**Dacă ai timp**

- Conversii de unități (kg ↔ g, bax ↔ bucăți).
- „Întreabă rețetarul”: RAG cu pgvector peste fișele tehnice.
- Export al fișei tehnice ca PDF.

## Claude API: ce trebuie să știi înainte

**Costuri și cont**

- **Abonamentul Pro NU include API-ul.** API-ul se plătește separat, la consum, din contul de pe [platform.claude.com](https://platform.claude.com).
- **Pune credit mic și setează o limită de cheltuieli din prima zi.** Câțiva dolari ajung pentru tot proiectul, dacă nu rulezi bucle scăpate de sub control.
- **O factură de 1–2 pagini costă câțiva cenți** cu un model mare și sub un cent cu unul mic. Verifică prețurile curente pe pagina de pricing din documentație.

**Unde stă cheia**

- Local, cheia stă în `.env.local`, fără prefixul `NEXT_PUBLIC_`. Pe Vercel o pui în env-ul proiectului.
- SDK-ul oficial e `@anthropic-ai/sdk` și îl apelezi **doar din cod de server** (Server Action sau Route Handler).

**Cum trimiți factura și ce primești înapoi**

- PDF-ul și pozele se trimit în mesaj ca blocuri de tip `document` (pentru PDF) și `image`.
- Pentru date structurate folosești `client.messages.parse()` cu `output_config.format` și `zodOutputFormat(schema)` din `@anthropic-ai/sdk/helpers/zod`. Primești obiectul deja validat pe schema ta.

**Ce model folosești**

- Pornești cu Claude Opus 5.5 (`claude-opus-5-5`).
- Cu setul de evals compari apoi și Claude Haiku 5.5 (`claude-haiku-5-5`), care e mult mai ieftin. Dacă scorul pe facturile tale e la fel de bun, treci pe el.
- Asta e o decizie de inginer pe care o poți povesti la interviu: „am măsurat, nu am ghicit”.

**Ce poate merge prost**

Tratezi erorile (rate limit, timeout, răspuns refuzat) și îi arăți utilizatorului un mesaj clar. Niciodată un ecran alb.

Când scrie codul de integrare, cere-i lui Claude Code să verifice API-ul curent în documentația oficială. SDK-ul se schimbă des.

## Feature-ul de mână

**Calculul de cost:** cost pe porție (cu pierderi), food cost % și marjă, ca funcții pure cu teste. E exercițiul tău de food cost din ziua 3, de data asta într-un produs adevărat.

## Break & fix

Pe un branch, Claude mută accidental citirea cheii API într-un Client Component (sau îi pune prefixul `NEXT_PUBLIC_`). Tu o găsești în bundle-ul din browser, cu DevTools → Sources. A doua variantă: o politică RLS lipsă, după care un user vede datele altuia. O prinzi logându-te cu două conturi.

## Ordinea sugerată a PR-urilor

1. Next.js + Supabase Auth (`@supabase/ssr`) + layout + deploy pe Vercel.
2. Schema + RLS + seed (ingrediente și rețete de test).
3. CRUD pentru ingrediente și rețete, cu calculul de cost (de mână).
4. Upload în Storage.
5. Extragerea cu Claude, pe server, cu schema zod.
6. Ecranul de review și potrivire.
7. Confirmare → prețuri → recalculare → dashboard.
8. Evals.
9. Utilizatori reali, feedback, polish, README cu GIF demo.

## Promptul de start (în plan mode)

```
Citește spec-ul din README.md și CLAUDE.md. Construim Kitchen AI cu Next.js (App Router), Supabase (Auth, Postgres, Storage) și Claude API.
Propune un plan care acoperă:
- ce e Server Component, ce e Client Component și unde rulează apelul către Claude
- schema și politicile RLS pentru fiecare tabel
- schema zod pentru extragerea unei facturi și cum tratăm liniile pe care modelul nu le-a putut citi
- fluxul de review: ce vede omul și ce poate corecta
- cum arată setul de evals și ce măsurăm
- ordinea PR-urilor
Verifică în documentația oficială API-ul curent pentru PDF input și structured outputs înainte să-l propui.
```

## Ce trebuie să poți explica la final (→ [CP5](../checkpoints/CP5.md))

- De ce cheia API stă doar pe server și cum ai verificat că nu a ajuns în bundle.
- Server vs Client Components în proiectul tău, cu exemple concrete.
- Cum trimiți o factură la Claude și cum primești date structurate înapoi.
- Ce faci când modelul greșește: validarea zod, review-ul uman, mesajele de eroare.
- Ce măsoară evals-urile, ce scor ai obținut și ce model ai ales pe baza lor.
- Cât costă procesarea unei facturi.
