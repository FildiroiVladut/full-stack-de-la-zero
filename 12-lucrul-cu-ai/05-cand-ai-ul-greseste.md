# Când AI-ul greșește

O să greșească, des și cu mare siguranță în glas. Lecția e să observi din timp, să verifici la sursă și să nu te lași tras într-o buclă de „fix-uri” care nu duc nicăieri.

## Cum greșește un LLM

**Ce e.** Un model de limbaj produce textul cel mai **plauzibil**, nu neapărat pe cel **adevărat**. De aici vin câteva tipare previzibile:

- **Cunoștințe învechite.** A văzut cod din mai mulți ani și amestecă versiuni de biblioteci.
- **Nu știe versiunile tale** până nu citește `package.json`.
- **Inventează cu încredere:** o opțiune de configurare, o funcție dintr-o bibliotecă, un nume de pachet.
- **Îți dă dreptate prea ușor.** Dacă insiști „sigur e de la X?”, poate schimba răspunsul doar pentru că ai insistat.
- **Tratează simptomul.** Face eroarea să dispară, nu neapărat cauza.

**De ce contează.** Tonul e la fel de sigur și când are dreptate, și când inventează. Siguranța din răspuns nu-ți spune nimic despre corectitudine. Te ajută doar verificarea.

**Capcana.** Să-l întrebi pe el dacă a greșit („ești sigur?”). Verifică în altă parte: rulând codul sau în documentație.

## Semnele că o ia pe arătură

- Aceeași eroare revine după al doilea „fix”.
- Modifică fișiere care n-au legătură cu task-ul.
- Diff-ul crește la fiecare încercare, în loc să scadă.
- Propune să dezactivezi verificări: `// @ts-ignore`, `as any`, `eslint-disable`, `--force`, `--legacy-peer-deps`, testul șters sau marcat `skip`.
- Adaugă un pachet nou ca să rezolve o problemă mică.
- Scrie „acum ar trebui să meargă” fără să fi rulat nimic.
- Nu mai înțelegi ce face și de ce.

## Verifică la sursă

**Ce e.** Când ceva sună ciudat, mergi la sursă, în ordinea asta:

1. **Mesajul de eroare**, citit complet, de sus. Primul fișier din stack trace care e al tău e de obicei locul problemei.
2. **Versiunea din `package.json`**, apoi **documentația oficială** pentru versiunea aia (react.dev, docs.nestjs.com, supabase.com/docs, prisma.io/docs).
3. **Tipurile.** În VS Code, `F12` pe o funcție dintr-o bibliotecă te duce în fișierul `.d.ts`, unde vezi exact ce parametri primește. Tipurile nu mint.
4. **Issue-urile de pe GitHub** ale bibliotecii, pentru erori ciudate.
5. Abia apoi, din nou, AI-ul, dar cu informația găsită:

```text
Conform documentației pentru [biblioteca] versiunea [x] (link), opțiunea corectă e [...].
Refă bucata asta folosind-o.
```

**De ce contează.** „Am verificat în documentația pentru versiunea pe care o folosim” e o frază de senior. „Așa a zis AI-ul” nu e.

## Bucla care nu se termină

**Ce e.** Claude greșește, îl corectezi, greșește altfel, îl corectezi iar. Contextul se umple de încercări eșuate, iar următoarele răspunsuri sunt influențate de ele.

**Cum ieși**

1. **Regula celor două corecturi.** După a doua corectură nereușită pe aceeași problemă: `Esc Esc` (rewind) până înainte de primul fix greșit, sau `/clear`.
2. **Prompt nou, mai precis**, care include ce ai aflat: „Eroarea X apare la linia Y. Am verificat deja că Z nu e cauza. Am nevoie de...”
3. **Restrânge problema.** Nu „nu merge salvarea”, ci „`POST /invoices` întoarce 400 cu mesajul ăsta, iar body-ul trimis e ăsta”.
4. **Cere ipoteze, nu cod**: promptul „Ipoteze, nu fix-uri” din [kit](04-kit-de-prompturi.md#depanare).
5. **Întâi testul care pică**, apoi fix-ul. Așa ai o verificare obiectivă, nu impresia lui Claude că „acum merge”.
6. **Git e plasa ta.** `git stash` sau `git restore .` te aduc înapoi la ultimul commit bun. Pentru asta faci commit des.

**Exemplu minim**

```text
Stop. Ne învârtim în cerc. Nu mai modifica nimic.
Rezumă în 5 rânduri: ce încercăm să reparăm, ce am încercat, ce a ieșit la fiecare încercare.
```

Rezumatul îl folosești ca punct de plecare într-o sesiune nouă, după `/clear`.

## Când te oprești și citești tu

- Aceeași eroare apare a treia oară.
- Fix-ul propus nu-l poți explica.
- Soluția trece prin dezactivarea unei verificări.
- Problema e la granița dintre două sisteme (frontend–backend, backend–DB, local–producție). Acolo AI-ul vede cel mai puțin și ghicește cel mai mult.
- E vorba de securitate sau de date reale (migrații, ștergeri).

În cazurile astea îți iei 20 de minute: citești eroarea, documentația și codul. Adesea găsești singur. Dacă nu, ai de unde să pui o întrebare bună, lui Claude sau lui Bogdan.

## Capcana: fix-urile care ascund simptomul

**Ce e.** Soluții care fac eroarea să dispară fără să rezolve cauza.

**Cum le recunoști**

```ts
// Hides the type error. The bug stays, now invisible
// @ts-ignore
const total = invoice.lines.reduce((sum, l) => sum + l.amount, 0);

// "Fixes" the crash, but the user never finds out the save failed
try {
  await saveInvoice(data);
} catch {}

// The test was failing, so the expectation was changed instead of the code
expect(calculateTotal(lines)).toBe(1189); // was 1190
```

Alte exemple: un timeout mărit „până trece”, un `retry` în buclă peste o eroare care nu e temporară, `npm install --force` peste un conflict de versiuni.

**Ce faci.** Întrebi: „Asta rezolvă cauza sau doar ascunde eroarea? Care e cauza?” Uneori dezactivarea unei verificări e legitimă, dar atunci vine cu un comentariu care explică de ce, și o decizi tu, nu din reflex.

<details>
<summary>Întrebare de interviu: Povestește o situație în care un AI ți-a dat o soluție greșită. Cum ți-ai dat seama?</summary>

Răspunde cu un caz real din proiectele tale, în structura: context → semnal → verificare → rezolvare → lecție. De exemplu: „În Invoicer, Claude a reparat un test roșu schimbând valoarea așteptată din 1190 în 1189. Am observat în diff că modificase testul, nu codul. Am calculat manual totalul, am văzut că rotunjirea TVA-ului era greșită și am reparat calculul, cu bani ținuți în cenți. De atunci verific mereu dacă un fix atinge codul sau doar testul.” Intervievatorul nu vrea să audă că AI-ul tău nu greșește. Vrea să vadă că tu prinzi greșelile.
</details>

## Mai departe

- [Best practices: avoid common failure patterns — Claude Code docs](https://code.claude.com/docs/en/best-practices)
- [Checkpointing — Claude Code docs](https://code.claude.com/docs/en/checkpointing)
