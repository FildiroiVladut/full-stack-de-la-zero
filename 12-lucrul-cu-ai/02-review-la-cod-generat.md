# Review la cod generat

Claude se oprește când codul **pare** gata. Dacă e și corect, verifici tu. Aici ai checklist-ul pe care îl treci peste fiecare diff înainte de commit, cu greșelile tipice ale codului generat de AI.

## De ce faci review la codul AI-ului

**Ce e.** Review înseamnă să citești diff-ul ca și cum l-ar fi scris un coleg nou: deștept și rapid, dar care nu cunoaște proiectul și nu răspunde de bug-uri. Răspunzi tu.

**De ce contează.** Codul generat arată impecabil: nume bune, structură curată, comentarii. Tocmai de aceea e periculos. Interviurile din 2026 testează explicit dacă știi să validezi ce produce un AI, iar la job PR-ul are numele tău pe el.

**Capcana.** Să citești doar explicația lui Claude („am adăugat validare și teste”) în loc de cod. Explicația e o afirmație. Diff-ul e dovada.

## Cum faci review, practic

```bash
git diff --stat           # which files changed and by how much: is the size reasonable?
git diff                  # the actual changes
```

Sau panoul Source Control din VS Code (`Ctrl+Shift+G`), unde vezi fiecare fișier lângă original.

Ordinea în care citești: **întâi fișierele de date și tipuri** (schema Prisma, DTO-uri, tipuri), apoi logica (services, hooks), apoi UI-ul. Dacă datele sunt greșite, restul nu mai contează.

Ca a doua opinie, poți rula `/code-review` în Claude Code: un subagent cu context proaspăt caută bug-uri în diff-ul curent. Ajută, dar nu decide în locul tău.

## Checklist-ul

Treci prin el la fiecare PR. După câteva proiecte îl faci din reflex.

### 1. Rulează?

- Aplicația pornește, build-ul trece, testele trec, typecheck-ul trece (`npm run typecheck` sau `npx tsc --noEmit`), lint-ul e curat.
- Ai văzut feature-ul funcționând **cu ochii tăi**, nu doar „testele sunt verzi”.

### 2. Face ce scrie în spec, inclusiv la margini?

Cazurile limită pe care AI-ul le uită cel mai des: listă goală, `null`/`undefined`, `0`, numere negative, string gol, duplicate, diacritice și caractere speciale, fusuri orare, două click-uri rapide pe același buton, utilizator fără drepturi.

### 3. Erorile sunt tratate sau ascunse?

```ts
// Red flag: the error disappears, the user sees nothing, you find out in a month
try {
  await api.markPaid(invoiceId);
} catch (e) {
  console.log(e);
}

// Red flag: fetch does NOT throw on 404 or 500
const res = await fetch(`/api/invoices/${id}`);
const invoice = await res.json(); // could be an error body

// OK
if (!res.ok) throw new Error(`Failed to load invoice ${id}: ${res.status}`);
```

### 4. Securitate

- Niciun secret în cod. Nicio cheie secretă în variabile `VITE_*` sau `NEXT_PUBLIC_*`, pentru că ajung în browser ([10-securitate-env](../10-securitate-env/02-ce-ajunge-in-browser.md)).
- Validarea e pe **backend**, nu doar în formular.
- Totalurile și prețurile se calculează pe server, nu se iau de la client.
- Verificarea de ownership: userul 7 nu poate citi factura userului 8 schimbând ID-ul din URL ([10/05](../10-securitate-env/05-autentificare-vs-autorizare.md)).
- Fără SQL construit din string-uri, fără `dangerouslySetInnerHTML` cu input de la utilizator.

### 5. Complexitate inutilă

AI-ul supra-construiește din reflex: un `InvoiceServiceFactory` pentru o singură implementare, o abstracție generică „pentru viitor”, parametri pe care nu-i trimite nimeni, trei straturi unde ajungea o funcție. Întrebarea de pus: **„Care e varianta cea mai simplă care respectă spec-ul?”** Regula din template-ul de `CLAUDE.md`: nicio abstracție până nu se repetă ceva de trei ori.

### 6. Dependențe noi: există și merită?

- AI-ul poate inventa nume de pachete plauzibile. **Slopsquatting** e atacul în care cineva publică pe npm, cu cod malițios, exact pachetele pe care modelele le halucinează des. Rulezi `npm install` pe un nume inventat, și pachetul instalat chiar există.
- Înainte de orice pachet nou: `npm view <pachet>` sau pagina de pe npmjs.com. Te uiți la descărcările săptămânale, la data ultimei versiuni, la link-ul de repo și la autor.
- Întrebarea mai bună: **„Chiar ne trebuie?”** Adesea există deja în proiect (zod, TanStack Query) sau e o funcție de 10 linii.

### 7. API-uri depășite

Modelul a fost antrenat pe cod din mai mulți ani, așa că poate amesteca versiuni. Semne: class components în React, `useEffect` pentru fetch când proiectul are TanStack Query, `pages/` în loc de `app/` în Next.js, opțiuni de configurare care nu mai există, import-uri care nu se potrivesc cu versiunea din `package.json`. Când ai dubii: versiunea din `package.json`, apoi documentația oficială pentru versiunea aia.

### 8. Cod mort, duplicat, uitat

`console.log`-uri de debug, cod comentat, fișiere create și nefolosite, același helper scris de două ori în locuri diferite, `TODO`-uri lăsate fără explicație.

### 9. Teste care nu testează nimic

```ts
// Passes no matter what the code does
it('marks invoice as paid', async () => {
  const service = { markPaid: vi.fn().mockResolvedValue({ status: 'paid' }) };
  expect(await service.markPaid('inv_1')).toEqual({ status: 'paid' });
});
```

Testul de mai sus face mock exact la lucrul pe care pretinde că-l testează. Alte semne: `expect(true).toBe(true)`, snapshot-uri uriașe aprobate fără să fie citite și, cel mai grav, **teste modificate ca să treacă pe codul greșit**. Când Claude „repară” un test roșu, verifică dacă a schimbat codul sau doar așteptarea din test.

### 10. Diff-ul are o mărime rezonabilă?

Peste ~300–400 de linii, review-ul devine superficial, al tău și al lui Bogdan. Cere împărțirea: „Împarte schimbările astea în 3 commit-uri logice” sau „Fă doar partea de backend în PR-ul ăsta”.

### 11. Se potrivește cu restul codului?

Aceleași convenții de nume, aceeași structură de foldere, aceleași biblioteci pentru aceleași probleme. Un fișier care arată „altfel” decât restul e de obicei semn că modelul a adus un tipar din altă parte.

## Capcana cea mare: review „în diagonală”

**Ce e.** După al zecelea diff bun, începi să aprobi fără să citești. Exact atunci trece bug-ul.

**Cum te aperi.** Pași mici, ca diff-urile să rămână citibile. Checklist-ul ăsta, ținut la vedere. Și regula din metodă: **dacă nu-l poți explica, nu-l comiți.**

<details>
<summary>Întrebare de interviu: Ce urmărești când faci code review?</summary>

Întâi corectitudinea: face ce trebuie, inclusiv la cazurile limită, și erorile sunt tratate, nu ascunse. Apoi securitatea: validare pe server, autorizare, niciun secret în cod. Apoi simplitatea și consistența cu restul proiectului: abstracții inutile, cod duplicat, dependențe noi nejustificate. Mă uit și dacă testele chiar verifică comportamentul. La final, lizibilitatea: dacă un coleg nou înțelege codul fără explicații. Comentariile le scriu ca întrebări sau sugestii concrete, nu ca judecăți.
</details>

## Mai departe

- [Best practices: give Claude a way to verify its work — Claude Code docs](https://code.claude.com/docs/en/best-practices)
- [09-git-github-echipa/04-code-review.md](../09-git-github-echipa/04-code-review.md): review-ul din perspectiva echipei
