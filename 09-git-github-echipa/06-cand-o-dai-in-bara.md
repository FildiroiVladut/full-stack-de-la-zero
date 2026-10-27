# Când o dai în bară

Vestea bună e că aproape nimic din ce a ajuns măcar o dată într-un commit nu se pierde în Git. Primul pas e mereu același: calm, `git status`, și nu rulezi comenzi luate la întâmplare de pe internet sau sugerate de AI fără să le înțelegi.

Fișele de aici sunt scenarii: **situația**, **soluția** și **ce s-a întâmplat de fapt**.

## Am stricat un fișier și vreau versiunea din ultimul commit

**Situația.** Ai modificat un fișier, nu mai merge nimic și vrei să revii la cum era.

**Soluția**

```bash
git restore src/invoices.service.ts                      # back to the last commit
git restore --source=HEAD~2 src/invoices.service.ts      # back to 2 commits ago
```

**Ce s-a întâmplat de fapt.** Git ia fișierul din snapshot-ul cerut și îl scrie peste cel din working directory. Ce nu era într-un commit dispare definitiv.

## Vreau să anulez ultimul commit

**Situația.** Ai făcut commit și îți dai seama că e greșit, incomplet sau pe branch-ul greșit.

**Soluția depinde de un singur lucru: ai dat deja push?**

**Încă n-ai dat push.** Rescrii liniștit, istoria e doar la tine:

```bash
git commit --amend -m "better message"   # fix the message or add a forgotten file (git add it first)
git reset --soft HEAD~1                  # undo the commit, keep the changes staged
```

**Ai dat push pe un branch comun sau pe `main`.** Nu rescrii istoria. Creezi un commit nou care face exact inversul:

```bash
git revert 3f9c2e1      # new commit that undoes 3f9c2e1
git push
```

**Cele trei feluri de `reset`:**

| Comandă | Commit-ul | Modificările |
|---|---|---|
| `git reset --soft HEAD~1` | anulat | rămân în staging |
| `git reset HEAD~1` (`--mixed`) | anulat | rămân în fișiere, nestaged |
| `git reset --hard HEAD~1` | anulat | **șterse** |

**Capcana.** `git reset --hard` e singura comandă din tabel care distruge muncă. Înainte s-o rulezi, `git status` trebuie să fie curat, sau trebuie să știi exact ce pierzi.

<details>
<summary>Întrebare de interviu: Care e diferența dintre git reset și git revert?</summary>

`reset` mută branch-ul înapoi la un commit anterior și practic rescrie istoria. E sigur doar pe commit-uri pe care nu le-am publicat. `revert` adaugă un commit nou care anulează efectele unui commit vechi și păstrează istoria intactă, deci e varianta corectă pe `main` sau pe orice branch comun.
</details>

## Am făcut commit pe main în loc de branch

**Situația.** Ai uitat `git switch -c` și ai făcut două commit-uri direct pe `main`. Încă n-ai dat push.

**Soluția**

```bash
git switch -c feat/my-work       # new branch here: it keeps your commits
git switch main
git reset --hard origin/main     # move local main back to where GitHub has it
git switch feat/my-work          # continue working here
```

**Ce s-a întâmplat de fapt.** Branch-ul nou e o etichetă pusă pe ultimul tău commit, deci commit-urile sunt în siguranță. Apoi muți eticheta `main` înapoi unde e pe GitHub. Aici `reset --hard` e sigur, pentru că nimic nu se pierde: commit-urile sunt pe `feat/my-work`.

## Detached HEAD

**Situația.** Git îți spune `You are in 'detached HEAD' state`.

**Ce s-a întâmplat de fapt.** HEAD arată direct spre un commit, nu spre un branch. Asta se întâmplă de obicei după `git checkout <hash>` sau după ce ai deschis un tag. Poți să te uiți prin cod fără probleme. Doar că, dacă faci commit-uri acolo, ele nu aparțin niciunui branch.

**Soluția**

```bash
git switch main                  # just looking? go back
git switch -c rescue/my-commits  # made commits? give them a branch first
```

## Reflog: plasa de siguranță

**Situația.** Ai făcut un reset, un rebase sau ai șters un branch și ți-au „dispărut” commit-urile.

**Soluția**

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~3
# 9f8e7d6 HEAD@{1}: commit: feat(search): add debounce   <- this one
# ...
git branch recovered 9f8e7d6     # put a branch on the lost commit
```

**Ce s-a întâmplat de fapt.** Reflog-ul ține local fiecare mutare a lui HEAD: commit, reset, switch, rebase. Implicit le păstrează cam 90 de zile. Commit-urile „pierdute” încă sunt acolo, doar că nicio etichetă nu mai arată spre ele.

**Capcana.** Reflog-ul există doar pe calculatorul tău. Nu ajută la ce n-a fost niciodată într-un commit și nici pentru colegi.

## Am făcut commit cu un secret

**Situația.** Un `.env` sau o cheie API a intrat într-un commit, poate chiar a ajuns pe GitHub.

**Soluția, în ordinea asta:**

1. **Rotești cheia imediat**, de la provider (Supabase, Anthropic etc.). Asta e partea care contează. Pașii exacți sunt în [10-securitate-env/03-secrete-scurse.md](../10-securitate-env/03-secrete-scurse.md).
2. Scoți fișierul din Git, dar îl păstrezi pe disc:

   ```bash
   git rm --cached .env
   echo ".env" >> .gitignore
   git commit -m "chore: stop tracking .env"
   ```

3. Abia apoi, dacă vrei, cureți și istoria. Unealta recomandată e `git filter-repo`, urmată de force push. O faci doar după ce cheia veche e deja moartă.

**Ce s-a întâmplat de fapt.** Un commit nou care șterge fișierul nu șterge și istoria: cheia rămâne în commit-ul vechi. Chiar dacă rescrii istoria, cineva poate să fi clonat repo-ul sau un bot l-a scanat între timp. Doar rotirea cheii face scurgerea inofensivă.

**Ajutor de la GitHub.** Pe repo-urile publice, *push protection* blochează din start push-urile care conțin chei recunoscute (Anthropic, Supabase și multe altele). Dacă ți se blochează un push, nu ocoli blocajul. Scoate cheia din commit.

## Force push: doar cu --force-with-lease și doar pe branch-ul tău

**Situația.** Ai făcut rebase sau amend pe un branch deja urcat, iar `git push` e refuzat.

**Soluția**

```bash
git push --force-with-lease
```

**Ce s-a întâmplat de fapt.** Ai rescris istoria locală, iar GitHub refuză un push care ar pierde commit-uri. `--force` suprascrie orice, inclusiv commit-urile urcate între timp de un coleg. `--force-with-lease` refuză dacă pe remote a apărut ceva ce tu n-ai văzut încă.

**Capcana.** Force push pe `main`. Ruleset-ul din [05](05-conventii-si-ci.md) îl blochează, și e bine că o face.

## Când chiar nu știi ce să faci

1. **Oprește-te.** Nu mai rula comenzi.
2. Fă o copie de siguranță a folderului: `cp -r ~/code/proiect ~/code/proiect-backup`.
3. Strânge contextul: `git status` și `git log --oneline --graph --all -15`.
4. Întreabă-l pe Claude, explicit fără acțiuni: _„Uite output-ul de la `git status` și `git log`. Explică-mi în ce stare e repo-ul și ce opțiuni am. Nu rula nicio comandă care modifică ceva.”_
5. Dacă e pe un repo comun, scrie-i colegului sau lui Bogdan **înainte** de orice force push.

## Mai departe

- [Pro Git — Undoing Things](https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things)
- [git-reflog](https://git-scm.com/docs/git-reflog) și [git-revert](https://git-scm.com/docs/git-revert): documentația oficială
- [Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository): GitHub Docs
