# 12 · Lucrul cu AI

Să scrii prompturi e partea ușoară. Greul e să conduci un agent de cod: să-i dai direcția, să verifici ce a făcut și să înțelegi rezultatul suficient de bine încât să-ți asumi codul. Capitolul transformă bucla în 7 pași din [00-start/01-metoda.md](../00-start/01-metoda.md) în gesturi concrete cu Claude Code: ce scrii, în ce mod lucrezi, când dai `/clear`, ce verifici înainte de commit și ce faci când AI-ul o ia pe arătură.

Instalarea, modurile și scurtăturile le ai în [01-setup/05-claude-code.md](../01-setup/05-claude-code.md). Aici e felul de lucru.

## La final poți să…

- duci un feature de la spec la PR cu Claude Code, pe pași mici, fără să pierzi controlul;
- faci review la un diff generat de AI cu un checklist concret și să prinzi problemele tipice;
- scrii un `CLAUDE.md` scurt și util și să-ți ții contextul curat;
- folosești un set de prompturi gata făcute ca să înțelegi, să verifici și să exersezi;
- recunoști când AI-ul greșește și să ieși dintr-o buclă de „fix-uri” care nu duc nicăieri.

## Fișierele, în ordine

- [ ] [01 · Workflow-ul](01-workflow.md) — 30 min
- [ ] [02 · Review la cod generat](02-review-la-cod-generat.md) — 30 min
- [ ] [03 · CLAUDE.md și contextul](03-claude-md-si-context.md) — 20 min
- [ ] [04 · Kit de prompturi](04-kit-de-prompturi.md) — 15 min de citit, apoi îl ții deschis
- [ ] [05 · Când AI-ul greșește](05-cand-ai-ul-greseste.md) — 20 min

## Unde îl folosești

Îl citești în săptămâna 2, înainte de [P1 Shelf](../proiecte/P1-shelf.md), și te întorci la el la fiecare proiect. Checklist-ul de review îl folosești la fiecare PR, iar kit-ul de prompturi la fiecare code walk. La [CP2](../checkpoints/CP2.md) trebuie să poți arăta, pe proiectul tău, cum a arătat bucla de la spec la PR.

## Verifică-te

1. De ce e mai bine să dai `/clear` și să rescrii prompt-ul decât să-l corectezi pe Claude a treia oară în aceeași sesiune?
2. Numește trei lucruri pe care le verifici într-un diff generat de AI înainte de commit.
3. Ce pui într-un `CLAUDE.md` și ce nu pui niciodată acolo?
4. Ce e „slopsquatting” și cum te aperi?
5. Claude îți propune `// @ts-ignore` ca să treacă build-ul. Ce faci?
