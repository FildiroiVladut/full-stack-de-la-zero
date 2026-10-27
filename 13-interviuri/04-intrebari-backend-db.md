# Întrebări backend, baze de date, securitate, git

43 de întrebări pentru partea de full-stack a interviului: HTTP și REST, Node și NestJS, SQL și Postgres, autentificare și securitate, git și lucrul în echipă, deploy. Mulți candidați de junior pe frontend se blochează exact aici. Dacă tu răspunzi bine, ieși din grămadă.

Citește întrebarea, răspunde cu voce tare, abia apoi deschide răspunsul.

## HTTP și REST

Detalii: [02-cum-functioneaza-webul/](../02-cum-functioneaza-webul/)

<details>
<summary><b>1. What happens when you type a URL and press Enter?</b></summary>

The browser parses the URL and resolves the domain name to an IP address through DNS. It opens a TCP connection to the server and, for HTTPS, does a TLS handshake to set up encryption. Then it sends an HTTP request, for example `GET /` with headers. The server, or a CDN in front of it, responds with a status code, headers and a body, usually HTML. The browser parses the HTML, requests the CSS, JavaScript and images it references, builds the page and runs the scripts, which may make more requests to an API.

_De ce se întreabă:_ e o întrebare deschisă. Nu contează să știi toate detaliile, ci să le spui în ordinea corectă.
</details>

<details>
<summary><b>2. Explain the HTTP methods. Which are safe and which are idempotent?</b></summary>

`GET` reads a resource, `POST` creates one or triggers an action, `PUT` replaces a resource, `PATCH` partially updates it and `DELETE` removes it. Safe methods don't change anything on the server: `GET`, `HEAD`, `OPTIONS`. Idempotent means that repeating the same request has the same effect as sending it once: `GET`, `PUT` and `DELETE` are idempotent, while `POST` is not, since two POSTs create two invoices. This matters for retries: you can safely retry an idempotent request after a timeout.
</details>

<details>
<summary><b>3. Which status codes do you use, and when?</b></summary>

`200 OK` for a successful read or update, `201 Created` after creating something, `204 No Content` for a successful delete. `400 Bad Request` for invalid input, `401 Unauthorized` when the user isn't authenticated, `403 Forbidden` when they are authenticated but not allowed, `404 Not Found`, `409 Conflict` for things like a duplicate or a slot that's already booked, and `422` for semantically invalid data in some APIs. `500` is an unexpected server error, and clients should never see a stack trace with it.

_Capcana:_ confuzia dintre 401 și 403 e clasică.
</details>

<details>
<summary><b>4. What makes an API RESTful? Design the endpoints for invoices and their lines.</b></summary>

REST models the API around resources identified by URLs, uses HTTP methods for the operations, keeps requests stateless (each request carries everything needed, like the auth token) and uses status codes meaningfully. For invoices: `GET /invoices` with filters like `?status=overdue&page=2`, `GET /invoices/:id`, `POST /invoices`, `PATCH /invoices/:id`, `DELETE /invoices/:id`. For lines: `GET /invoices/:id/lines` and `POST /invoices/:id/lines`. Actions that don't fit CRUD well can be sub-resources, like `POST /invoices/:id/send`.
</details>

<details>
<summary><b>5. Path params vs query params vs request body?</b></summary>

Path params identify a specific resource: `/invoices/42`. Query params filter, sort or paginate a collection: `/invoices?status=paid&sort=-dueDate`. The body carries the data you're sending, usually JSON in `POST`, `PUT` and `PATCH`. Sensitive data should never go in the URL, because URLs end up in logs and browser history.
</details>

<details>
<summary><b>6. How do you paginate a list endpoint?</b></summary>

The simplest way is offset pagination: `?page=3&limit=20`, implemented with `LIMIT` and `OFFSET`. It's easy, but it gets slow on large offsets and can skip or repeat items when data changes between pages. Cursor (keyset) pagination returns a cursor, for example the last item's `created_at` and `id`, and the next page asks for items after it. It's stable and fast with an index. I'd start with offset for an admin table and use cursors for long feeds or infinite scroll. Either way, I return metadata like `nextCursor` or the total count.
</details>

<details>
<summary><b>7. What is CORS? Does it protect your API?</b></summary>

CORS is a browser mechanism. By default, JavaScript on one origin can't read responses from another origin. The server can allow specific origins with headers like `Access-Control-Allow-Origin`. For non-simple requests, the browser first sends a preflight `OPTIONS` request to ask permission. It doesn't protect your API: curl, Postman or another server can call it regardless. CORS protects users' browsers from malicious sites reading data with their credentials. API protection comes from authentication, authorization and rate limiting.

_De ce se întreabă:_ aproape toți juniorii cred că CORS e o măsură de securitate pentru API.
</details>

<details>
<summary><b>8. Sessions vs JWT? Where should a token be stored in the browser?</b></summary>

With sessions, the server stores session data and the browser holds only a session ID in a cookie. It's easy to revoke, but the server has to look the session up on every request. A JWT is a signed, self-contained token: the server verifies the signature without a lookup, but revoking it before it expires is hard, so access tokens are short-lived and paired with refresh tokens. For storage, an `httpOnly`, `Secure`, `SameSite` cookie can't be read by JavaScript, so an XSS attack can't steal it, but you have to think about CSRF. `localStorage` is readable by any script on the page. Many SPAs, including the Supabase client by default, keep tokens in `localStorage` and rely on preventing XSS.
</details>

## Node.js și NestJS

Detalii: [07-backend-node-nest/](../07-backend-node-nest/)

<details>
<summary><b>9. What is Node.js? If it's single-threaded, how does it handle many requests at once?</b></summary>

Node.js runs JavaScript outside the browser, on the V8 engine, with APIs for files, the network and processes. Your JavaScript runs on one main thread, but I/O is non-blocking. While a request waits for the database or another API, Node hands that work to the operating system or libuv's thread pool and serves other requests. When the result is ready, the callback goes back on the event loop. That's why Node handles many concurrent I/O-bound requests well.
</details>

<details>
<summary><b>10. What happens if a request handler runs a heavy CPU loop?</b></summary>

It blocks the event loop, so every other request waits, not just that one. The server looks frozen until the loop finishes. Solutions: move the heavy work to a worker thread; push it to a background job queue and return immediately; run it in a separate service; or make the algorithm cheaper. Examples are generating big PDFs, processing images or hashing passwords with very high cost settings.
</details>

<details>
<summary><b>11. Why NestJS instead of plain Express?</b></summary>

Express is minimal: routing and middleware, and you decide the structure. NestJS is a framework that runs on top of Express or Fastify and gives you a consistent architecture: modules, controllers, providers with dependency injection, decorators, validation pipes, guards, and testing utilities. In a team, that means every Nest codebase looks familiar and responsibilities are clearly separated. The cost is more concepts and boilerplate up front.
</details>

<details>
<summary><b>12. Explain modules, controllers and providers in NestJS.</b></summary>

A module groups related code, for example an `InvoicesModule`, and declares what it provides and what it exports to other modules. A controller handles HTTP: it maps routes to methods, reads params and the body, and returns responses. It should stay thin. Providers, usually services, hold the business logic and data access, and Nest injects them where they're needed. So the controller receives the request, calls `invoicesService.create(dto)` and returns the result.
</details>

<details>
<summary><b>13. What is dependency injection, and why is it useful?</b></summary>

Instead of a class creating its own dependencies with `new`, it declares them, usually in the constructor, and a container provides them. In Nest, `constructor(private readonly prisma: PrismaService) {}` is enough: Nest creates one `PrismaService` and injects it. The benefits are loose coupling and testing. In a unit test, I can inject a fake `PrismaService` and test the business logic without a real database.
</details>

<details>
<summary><b>14. What is the request lifecycle in NestJS?</b></summary>

An incoming request goes through middleware first, then guards, which decide whether the request is allowed, for example whether the user is authenticated and has the right role. Then interceptors run their "before" logic, then pipes validate and transform the input, then the controller method runs and calls services. On the way out, interceptors can transform the response. If anything throws, exception filters turn the error into an HTTP response.
</details>

<details>
<summary><b>15. How do you validate request bodies in NestJS?</b></summary>

I define a DTO class with class-validator decorators, like `@IsString()`, `@IsInt()`, `@Min(1)` and `@IsEmail()`, and enable a global `ValidationPipe` with `whitelist: true`, which strips properties not in the DTO; `forbidNonWhitelisted: true`, which rejects them instead; and `transform: true`, which converts the payload into DTO instances and primitive types. Invalid requests get a `400` with clear messages before reaching my code. Business rules that need the database, like "this client exists", go in the service.
</details>

<details>
<summary><b>16. How do you handle errors and return the right status codes?</b></summary>

In services I throw Nest's built-in HTTP exceptions, such as `NotFoundException`, `BadRequestException`, `ConflictException` and `ForbiddenException`, and Nest turns them into the right status code and JSON body. Database errors I can predict, like a unique-constraint violation, I map to a clear `409`. Anything unexpected becomes a `500` with a generic message for the client, while the details go to the logs. I never leak stack traces or SQL errors to clients.
</details>

<details>
<summary><b>17. How do you manage configuration and environment variables?</b></summary>

Configuration comes from environment variables: a `.env` file locally, which is never committed, and the hosting platform's settings in production. A committed `.env.example` documents which variables are needed. In Nest I use `@nestjs/config` and validate the variables at startup, so if `DATABASE_URL` is missing, the app fails immediately with a clear error instead of crashing at the first request. Secrets never get hardcoded or logged.
</details>

## SQL și Postgres

Detalii: [08-baze-de-date-supabase/](../08-baze-de-date-supabase/), [07-backend-node-nest/05-baza-de-date-cu-prisma.md](../07-backend-node-nest/05-baza-de-date-cu-prisma.md)

<details>
<summary><b>18. Primary key vs foreign key?</b></summary>

A primary key uniquely identifies each row in a table: it's unique and never null, for example `invoices.id`. A foreign key is a column that references the primary key of another table, like `invoices.client_id` pointing to `clients.id`. The database enforces it, so you can't create an invoice for a client that doesn't exist, and you choose what happens on delete: block it, cascade, or set null.
</details>

<details>
<summary><b>19. Explain INNER JOIN vs LEFT JOIN with an example.</b></summary>

An `INNER JOIN` returns only rows that have a match in both tables. Clients joined with invoices gives only clients who have at least one invoice. A `LEFT JOIN` returns every row from the left table, plus the matches from the right, with `NULL`s where there's no match. Clients `LEFT JOIN` invoices gives every client, including those with no invoices. A typical use: "list all clients with their invoice count, including zero" needs a `LEFT JOIN` with `COUNT(invoices.id)`.
</details>

<details>
<summary><b>20. Write a query: the total unpaid amount per client, highest first.</b></summary>

```sql
SELECT c.id, c.name, SUM(i.total_cents) AS unpaid_cents
FROM clients c
JOIN invoices i ON i.client_id = c.id
WHERE i.status IN ('sent', 'overdue')
GROUP BY c.id, c.name
ORDER BY unpaid_cents DESC;
```

`WHERE` filters rows before grouping. To filter the groups themselves, for example "only clients owing more than 1,000 lei", add `HAVING SUM(i.total_cents) > 100000` after `GROUP BY`.

_De ce se întreabă:_ JOIN + GROUP BY + diferența dintre WHERE și HAVING acoperă 80% din SQL-ul de interviu.
</details>

<details>
<summary><b>21. What is an index? When wouldn't you add one?</b></summary>

An index is a separate data structure, by default a B-tree in Postgres, that lets the database find rows by a column's value without scanning the whole table. Think of the index at the back of a book. It speeds up `WHERE`, `JOIN` and `ORDER BY` on those columns. The cost is slower writes, because every insert or update also updates the index, plus extra storage. So I don't index columns that are rarely queried, or tiny tables. Postgres automatically indexes primary keys and unique constraints, but not foreign keys, so I usually add indexes on foreign keys I join or filter on.
</details>

<details>
<summary><b>22. What is a transaction? Give an example where you need one.</b></summary>

A transaction groups several statements so they either all succeed or all fail, with no half-done state. It's the "A" in ACID: atomicity, consistency, isolation, durability. Example: creating an invoice inserts a row in `invoices` and several rows in `invoice_lines`. If inserting a line fails, the invoice shouldn't exist without its lines. In Prisma, I'd use a nested create or `prisma.$transaction`.
</details>

<details>
<summary><b>23. How do you model a many-to-many relationship?</b></summary>

With a junction table. A recipe has many ingredients and an ingredient appears in many recipes, so I create `recipe_ingredients` with `recipe_id` and `ingredient_id` as foreign keys, often together as the primary key. The relationship's own data, like `quantity` and `unit`, also lives in that table.
</details>

<details>
<summary><b>24. What is normalization? When would you deliberately denormalize?</b></summary>

Normalization means storing each fact in one place, so you avoid duplication and the update anomalies it causes, like changing a client's name in one row but not another. In practice that means separate tables linked by foreign keys. I denormalize deliberately for read performance, or to keep a historical snapshot. A good example: an invoice line stores the product description and unit price at the time of invoicing, instead of pointing to the current price, because an issued invoice must never change when the price list does.
</details>

<details>
<summary><b>25. What is the N+1 query problem?</b></summary>

You run one query to load a list, then one extra query per item to load related data. For 50 invoices, that's 51 queries instead of one or two. It often hides behind ORMs, where looping and accessing a relation looks innocent. The fix is to load the relations together, with a JOIN, Prisma's `include`, or a single `WHERE id IN (...)` query, and then match them in code.
</details>

<details>
<summary><b>26. What is SQL injection, and how do you prevent it?</b></summary>

It happens when user input is concatenated into a SQL string, so input like `' OR 1=1 --` changes the query itself. The prevention is parameterized queries: the SQL and the values are sent separately, so values are never interpreted as SQL. ORMs like Prisma parameterize their query API, and Prisma's tagged `$queryRaw` template is parameterized too. The danger is `$queryRawUnsafe` or any string building with user input.
</details>

<details>
<summary><b>27. What are the pros and cons of an ORM like Prisma?</b></summary>

Pros: type-safe queries that match your schema, migrations, less boilerplate, and productivity in common CRUD work. Cons: it hides the SQL, so it's easy to write inefficient queries, N+1 included; complex reports and queries can be awkward; and it's another layer to learn and debug. I use the ORM for most work and drop to raw SQL for complex reporting queries. Knowing SQL is still required, because you need it to understand what the ORM generates.
</details>

<details>
<summary><b>28. What is a database migration? Why not just change the schema by hand?</b></summary>

A migration is a versioned file describing a schema change, like creating a table or adding a column. It's committed to git and applied in order in every environment. That makes changes reproducible, reviewable in pull requests and reversible. If you edit the production schema by hand, environments drift apart, nobody knows the real schema, and the next deploy breaks.
</details>

<details>
<summary><b>29. How would you prevent double booking in a reservation system?</b></summary>

Checking "is the slot free?" in code and then inserting has a race condition: two requests can both see the slot as free and both insert. The fix belongs in the database. For fixed slots, a unique constraint on `(resource_id, starts_at)`. For arbitrary time ranges, Postgres has exclusion constraints: `EXCLUDE USING gist (resource_id WITH =, tstzrange(starts_at, ends_at) WITH &&)`, which needs the `btree_gist` extension. The API catches the constraint violation and returns a `409 Conflict`.

_De ce se întreabă:_ e o întrebare de gândire, nu de memorie. În Booking ai rezolvat-o exact așa.
</details>

## Autentificare și securitate

Detalii: [10-securitate-env/](../10-securitate-env/)

<details>
<summary><b>30. Authentication vs authorization?</b></summary>

Authentication answers "who are you?": logging in and verifying a token. Authorization answers "what are you allowed to do?": can this user see this invoice, is this user an admin. A request can be authenticated and still forbidden. Authentication failures return `401`, authorization failures `403`, or sometimes `404` so you don't reveal that the resource exists.
</details>

<details>
<summary><b>31. What is a JWT? Can the client read what's inside?</b></summary>

A JSON Web Token has three base64url-encoded parts: a header, a payload with claims like the user ID, role and expiry, and a signature. Anyone can decode and read the payload, because it's signed, not encrypted, so you never put secrets in it. The signature proves the token was issued by the auth server and hasn't been modified. On every request the server verifies the signature, the expiry, and ideally the issuer and audience.
</details>

<details>
<summary><b>32. How should passwords be stored?</b></summary>

Never in plain text and never with reversible encryption. They should be stored with a slow, salted hashing algorithm made for passwords, such as Argon2id or bcrypt. The salt makes precomputed tables useless, and the slowness makes brute force expensive. Better still, I don't build this myself: I use a proven auth provider like Supabase Auth, which also handles password resets, email verification and token refresh correctly.
</details>

<details>
<summary><b>33. Why must secret keys never be in frontend code? What about Supabase's publishable key?</b></summary>

Everything shipped to the browser, from the JavaScript bundle to env variables prefixed with `VITE_` or `NEXT_PUBLIC_`, can be read by anyone in DevTools. A secret key there is a public key. Secrets stay on the server, and the frontend calls your backend, which uses them. Supabase's publishable key is designed to be public: it identifies the project, and access is controlled by Row Level Security policies in the database. The secret key bypasses RLS, so it lives only on the server.
</details>

<details>
<summary><b>34. What is IDOR, and how do you prevent it?</b></summary>

IDOR, insecure direct object reference, is when an API returns or changes an object just because you know its ID, without checking that it's yours. `GET /invoices/124` shows someone else's invoice because the server only checked that you're logged in. Prevention: always scope queries by the current user, for example `where: { id, userId }`, or check ownership and role explicitly. Row Level Security adds a second layer. It's one of the most common real-world API vulnerabilities.

_De ce se întreabă:_ e greșeala tipică de junior, și o face des și AI-ul.
</details>

<details>
<summary><b>35. What is XSS? How does React help, and where can it still happen?</b></summary>

Cross-site scripting means an attacker gets their script to run in other users' browsers, usually through content they submitted that the page renders as HTML. That script can steal tokens or act as the user. React escapes values rendered in JSX by default, so `{comment}` shows text, not HTML. XSS can still happen through `dangerouslySetInnerHTML`, through links like `href={userUrl}` with a `javascript:` URL, or through third-party scripts. Sanitize HTML you must render, with a library like DOMPurify, and validate URLs.
</details>

<details>
<summary><b>36. Why do you need rate limiting?</b></summary>

Rate limiting caps how many requests a client, by IP or user, can make in a time window. It protects login endpoints from brute force, protects the API from abuse and accidental floods, and controls costs, which matters a lot for endpoints that call a paid AI API. In Nest there's `@nestjs/throttler`. Exceeding the limit returns `429 Too Many Requests`.
</details>

## Git și lucrul în echipă

Detalii: [09-git-github-echipa/](../09-git-github-echipa/)

<details>
<summary><b>37. Merge vs rebase?</b></summary>

Merge combines two branches with a merge commit and preserves the history exactly as it happened. Rebase replays your commits on top of the target branch, which gives a linear history but rewrites your commits with new hashes. The rule is to not rebase branches other people are working on. Rebasing my own feature branch onto the latest `main` before opening a PR is fine, followed by `git push --force-with-lease`. Many teams squash-merge PRs so `main` has one clean commit per feature.
</details>

<details>
<summary><b>38. Describe your pull request workflow.</b></summary>

I start from an issue that describes the task. I create a branch from the latest `main`, commit in small logical steps, and push. I open a pull request with a description of what changed and why, how to test it, and screenshots for UI changes. CI runs lint, type checks and tests. A reviewer comments; I address the comments with new commits or reply when I disagree. Once it's approved and green, it's squash-merged, the branch is deleted, and the deploy happens automatically. In my projects, my mentor reviewed my PRs like a tech lead would.
</details>

<details>
<summary><b>39. How do you resolve a merge conflict?</b></summary>

Git marks the conflicting sections in the file with both versions. First I understand what each side was trying to do, and if it's someone else's change and it isn't obvious, I ask them. Then I edit the file to the correct combined result, not just one side, remove the markers, run the app and tests, and commit, or continue the rebase. Pulling `main` often and keeping PRs small makes conflicts rarer and smaller.
</details>

<details>
<summary><b>40. You committed a secret and pushed it. What now?</b></summary>

I assume it's compromised, because bots scan public repositories within minutes. First I revoke or rotate the key at the provider, which is the step that actually fixes the problem. Then I remove it from the code, move it to an environment variable, and check that `.env` is in `.gitignore`. Rewriting git history, with a tool like `git filter-repo`, only reduces future exposure; it doesn't make the key safe again. And I tell the team, because hiding it makes it worse.
</details>

## Deploy

Detalii: [11-deploy/](../11-deploy/)

<details>
<summary><b>41. What's the difference between development and production builds and environments?</b></summary>

A development build favors speed of iteration: a dev server with hot reload, source maps, verbose errors and no minification. A production build is optimized: minified, tree-shaken bundles, with `NODE_ENV=production`. Environments are separate copies of the app, such as local, maybe staging, and production, each with its own environment variables, its own database and its own keys. You never test against the production database.
</details>

<details>
<summary><b>42. Your app works locally but fails in production. How do you debug it?</b></summary>

I start with the logs on the hosting platform and the browser's console and Network tab, to see the actual error. Then I check the usual suspects:

- missing or different environment variables;
- a hardcoded `localhost` URL;
- CORS not allowing the production origin;
- migrations not applied to the production database;
- a different Node version;
- file name case, since Linux servers are case-sensitive and Windows isn't, so `./Invoice` vs `./invoice` works locally and fails in production.

If needed, I run the production build locally to reproduce it.

_De ce se întreabă:_ arată că ai livrat ceva real. Spune o poveste concretă din proiectele tale dacă ai una.
</details>

<details>
<summary><b>43. What is CI/CD?</b></summary>

Continuous integration means that every push or pull request automatically runs checks, such as lint, type checking, tests and a build, so problems show up before merge. Continuous delivery or deployment means that once `main` passes, the app is deployed automatically. In my projects, GitHub Actions runs the checks on every PR, and Vercel and Render deploy automatically from `main`.
</details>

## Mai departe

- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [NestJS — Documentation](https://docs.nestjs.com/)
- [PostgreSQL — Tutorial](https://www.postgresql.org/docs/current/tutorial.html)
