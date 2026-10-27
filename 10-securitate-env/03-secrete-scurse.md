# Secrete scurse

O să se întâmple, ție sau unui coleg: o cheie ajunge pe GitHub, într-un screenshot, într-un log sau în bundle-ul de frontend. Diferența dintre un junior și un senior nu e că seniorul n-a greșit niciodată. E că știe exact ce face în primele 10 minute.

## Ce înseamnă „scurs”

**Ce e.** Un secret e scurs din momentul în care a ajuns undeva unde nu mai ai control asupra lui:

- un repo public, chiar și pentru un minut;
- bundle-ul de frontend (vezi [02](02-ce-ajunge-in-browser.md));
- un screenshot, un mesaj pe chat, un issue sau un comentariu pe PR;
- logurile aplicației, de exemplu un `console.log(process.env)` uitat.

**De ce contează.** Există boți care scanează continuu repo-urile publice după chei. O cheie urcată poate fi folosită în câteva minute: credite consumate, date citite sau șterse.

**Capcana.** „Am șters commit-ul după 30 de secunde, nu l-a văzut nimeni.” Pornești mereu de la presupunerea că l-a văzut cineva.

## Primele 10 minute

**Ce e.** Ordinea contează. Mai întâi faci cheia veche inutilizabilă, abia apoi faci curățenie.

1. **Rotești cheia la provider.** Creezi una nouă și o dezactivezi sau o ștergi pe cea veche.
   - **Supabase, cheie secret:** Settings → **API Keys** → creezi o cheie secret nouă → o pui peste tot unde era folosită → verifici că totul merge → **abia apoi** ștergi cheia veche. Ștergerea nu se poate anula, de aceea întâi înlocuiești.
   - **Supabase, parola bazei de date** (dacă a scăpat un `DATABASE_URL`): o resetezi din setările bazei de date și actualizezi connection string-urile.
   - **Anthropic:** în Console ștergi cheia și creezi alta.
2. **Pui cheia nouă acolo unde rulează aplicația:** `.env` local, Render, Vercel. Apoi faci redeploy.
3. **Verifici ce s-a întâmplat între timp:** consumul din Console-ul Anthropic, logurile din Supabase, orice activitate ciudată.
4. **Scoți secretul din cod și, opțional, din istorie:** vezi [09-git-github-echipa/06-cand-o-dai-in-bara.md](../09-git-github-echipa/06-cand-o-dai-in-bara.md).
5. **Spui.** Echipei, clientului, lui Bogdan. O greșeală ascunsă e mult mai gravă decât una raportată în 10 minute.

<details>
<summary>Întrebare de interviu: Ce faci dacă ai urcat din greșeală o cheie API pe GitHub?</summary>

Primul lucru e să revoc sau să rotesc cheia la provider, pentru că din momentul ăla o consider compromisă, indiferent cât de repede șterg commit-ul. Pun cheia nouă în toate mediile și fac redeploy, apoi verific în logurile și în billing-ul providerului dacă a folosit-o cineva. Abia după aceea scot fișierul din repo, îl adaug în `.gitignore` și, dacă e cazul, curăț istoria cu `git filter-repo`. Și anunț echipa.
</details>

## De ce ștersul commit-ului nu ajunge

**Ce e.** Un commit nou care șterge `.env` lasă cheia în commit-ul vechi, vizibilă în istorie. Chiar dacă rescrii istoria, cheia poate exista deja în:

- clonele colegilor sau ale CI-ului;
- fork-urile repo-ului;
- cache-urile și scanerele care au trecut între timp pe acolo.

**De ce contează.** Singurul lucru care face scurgerea inofensivă e **rotirea cheii**. Curățenia din istorie e igienă, nu remediere. Chiar și ghidul GitHub despre ștergerea datelor sensibile începe cu „revoke and/or rotate that secret”.

## GitHub secret scanning și push protection

**Ce e.** Două plase de siguranță pe care GitHub le oferă gratuit pe repo-urile publice:

- **Secret scanning.** Rulează automat pe repo-urile publice. Dacă găsește cheia unui *partener*, anunță providerul, iar acesta o poate revoca singur. Anthropic și Supabase sunt pe lista partenerilor.
- **Push protection pentru utilizatori.** E activată implicit și îți blochează push-ul spre un repo public dacă detectează în el un secret cunoscut.

**Cum arată.** Dai `git push`, iar GitHub îl refuză cu un mesaj care spune ce secret a găsit, în ce fișier și în ce commit.

**Ce faci.** Scoți secretul din commit, de exemplu cu `git reset --soft HEAD~1`, apoi îl scoți din fișier și faci commit din nou. **Nu** ocolești blocajul cu „e doar o cheie de test”.

**Capcana.** Să crezi că GitHub te prinde mereu. Nu orice secret are un format recognoscibil. Un connection string cu parolă sau un token intern pot trece nedetectate. Plasa de siguranță nu ține locul obiceiurilor bune.

## Cum previi

**Ce e.** Câteva obiceiuri care fac scurgerile aproape imposibile:

- **`.gitignore` corect de la primul commit,** cu `.env` și `.env.*`, plus `!.env.example`. Template-urile din proiectele tale îl au deja.
- **`git status` și `git diff --staged` înainte de fiecare commit.** Te uiți la ce intră, nu doar la ce crezi că intră.
- **Nicio cheie scrisă în cod „temporar”.** Temporarul ajunge pe GitHub.
- **Chei separate pe medii.** Cheia de pe laptop nu e cea din producție, deci o scurgere locală nu atinge producția.
- **Limite de cheltuieli** la serviciile plătite. În Console-ul Anthropic setezi o limită lunară mică pentru proiectele de învățare.
- **Chei cu drepturi minime.** Dacă un serviciu îți dă chei doar pentru citire sau limitate la un proiect, le folosești pe acelea.

**Cu Claude Code.** Claude poate citi fișierele din proiect, inclusiv `.env`. Local nu e o problemă, dar:

- nu-i lipi chei în prompt;
- nu-l lăsa să afișeze conținutul `.env`-ului în output;
- poți bloca citirea fișierului cu o regulă de permisiuni (vezi [01-setup/05-claude-code.md](../01-setup/05-claude-code.md)).

## Mai departe

- [About secret scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) și [About push protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection): GitHub Docs
- [Understanding API keys](https://supabase.com/docs/guides/api/api-keys): Supabase, inclusiv pașii de rotire
- [Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
