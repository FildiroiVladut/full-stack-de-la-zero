# Cum arată o fișă de concept

Ca o fișă tehnică din bucătărie: același format la fiecare concept, ca să știi mereu unde găsești ce-ți trebuie. O fișă se citește în 5–10 minute.

## Formatul

```markdown
## Numele conceptului

**Ce e.** Două-trei propoziții, fără jargon neexplicat.

**De ce contează.** Unde o să-l întâlnești și ce se strică dacă nu-l înțelegi.

**Exemplu minim**
(cel mult 15 linii de cod, rulabile)

**Cum îl vezi în cod real.** Un fragment cum l-ar scrie Claude într-un proiect
adevărat, plus 1–3 lucruri de observat în el.

**Capcana.** Greșeala tipică, a juniorilor sau a AI-ului, și cum o recunoști.

<details>
<summary>Întrebare de interviu: ...?</summary>

Răspunsul scurt, cum l-ai spune cu voce tare.
</details>
```

## Cum le folosești

1. **Citește „Ce e” și „De ce contează”.** Dacă nu sunt clare, întreabă-l pe Claude: _„Explică-mi [conceptul] pe scurt. Știu C și JavaScript de bază.”_
2. **Rulează exemplul minim.** Copiază-l într-un fișier (`node fisier.js` sau `npx tsx fisier.ts`) și schimbă ceva în el ca să vezi ce se întâmplă.
3. **La „Cum îl vezi în cod real”, uită-te după tipar.** Pe ăsta o să-l recunoști în codul generat de Claude în proiecte.
4. **Citește capcana de două ori.** E partea pe care o întreabă seniorii la review și la interviu.
5. **Ascunde răspunsul de interviu și spune-l tu cu voce tare.** Abia apoi deschide-l. Dacă al tău e mai slab, reformulează-l până sună natural.

Nu toate fișele au toate secțiunile. Fișierele de setup și deploy sunt pași concreți (comandă, ce trebuie să vezi, ce s-a întâmplat de fapt). Ce nu lipsește niciodată e explicația _de ce_.

## Convenții în tot ghidul

- **Termenii tehnici rămân în engleză:** request, deploy, branch, state, hook. Așa îi auzi la job și la interviu. La prima apariție sunt explicați.
- **Codul e scris în engleză**, cu nume de variabile și comentarii în engleză. Așa se scrie în orice echipă, chiar și în România. Explicațiile sunt în română.
- **Exemplele vin din proiectele tale:** cărți (P1), facturi (P2), rezervări (P3), rețete (P4). Când ajungi la proiect, conceptele îți sunt deja cunoscute.
- **Ce e marcat ca _opțional_ poți sări** fără să pierzi firul.
