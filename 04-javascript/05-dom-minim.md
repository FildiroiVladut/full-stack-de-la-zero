# DOM, minimul necesar

În proiecte nu vei manipula DOM-ul de mână, pentru că se ocupă React. Merită totuși să vezi o dată cum arată treaba făcută manual, ca să înțelegi ce problemă rezolvă React și ce se întâmplă sub el. Citești fișierul o dată, rulezi exemplul și treci mai departe.

Ca să rulezi exemplele: în WSL faci un folder cu un `index.html`, rulezi în el `npx vite` și deschizi în browserul din Windows adresa afișată (de obicei `http://localhost:5173`). Pentru DevTools apeși F12 în browser.

## DOM-ul

**Ce e.** Când încarcă un HTML, browserul construiește din el un arbore de obiecte: **DOM** (Document Object Model). Fiecare tag devine un nod pe care JavaScript îl poate citi și modifica. Ce vezi pe ecran e mereu DOM-ul curent, nu HTML-ul original.

**De ce contează.** React, Vue și orice alt framework de frontend modifică tot DOM-ul, doar că o fac în locul tău. Când inspectezi o pagină în DevTools (tab-ul Elements), vezi DOM-ul.

**Exemplu minim**
```html
<ul id="shelf"><li>Dune</li></ul>
<script type="module">
  const shelf = document.querySelector("#shelf");
  const item = document.createElement("li");
  item.textContent = "Neuromancer";
  shelf.append(item); // the page updates immediately
</script>
```

**Capcana.** `element.innerHTML = userInput` inserează HTML interpretat, nu text. Dacă `userInput` conține `<img src=x onerror="...">`, codul din el rulează. Pentru text folosești `textContent`. Detalii în [10-securitate-env/04-validare-si-atacuri.md](../10-securitate-env/04-validare-si-atacuri.md).

## Evenimente

**Ce e.** Browserul emite evenimente (click, input, submit, keydown), iar tu atașezi funcții care le ascultă cu `addEventListener`. Funcția primește un obiect `event` cu detalii.

**Exemplu minim**
```html
<form id="search">
  <input name="q" placeholder="Search books" />
  <button>Search</button>
</form>
<script type="module">
  document.querySelector("#search").addEventListener("submit", (event) => {
    event.preventDefault(); // stop the browser from reloading the page
    const query = new FormData(event.target).get("q");
    console.log("searching for", query);
  });
</script>
```

**Cum îl vezi în cod real.** În React scrii `onSubmit={handleSubmit}` direct pe element, iar `event.preventDefault()` apare la fel în fiecare formular.

**Capcana.** Fără `preventDefault()`, un `<form>` reîncarcă pagina la submit și pierzi tot state-ul. Bug-ul ăsta îl vei vedea în codul juniorilor și, ocazional, în codul generat de AI.

<details>
<summary>Întrebare de interviu: Ce e event delegation?</summary>

În loc să pui câte un listener pe fiecare element dintr-o listă, pui unul singur pe părinte. Evenimentele urcă prin arbore (_bubbling_), iar în listener verifici pe ce element s-a dat click, prin `event.target` sau `closest()`. Merge și pentru elemente adăugate mai târziu și folosește mai puțină memorie. React aplică intern aceeași idee: atașează listener-ele la rădăcina aplicației.
</details>

## Problema: sincronizarea manuală

**Ce e.** Când aceeași informație apare în mai multe locuri pe ecran, fiecare modificare a datelor trebuie reflectată manual peste tot.

**Exemplu minim** (o listă de lectură cu contor și mesaj de „listă goală”)
```js
const books = [];

function addBook(title) {
  books.push(title);
  const li = document.createElement("li");
  li.textContent = title;
  document.querySelector("#list").append(li);                    // place 1
  document.querySelector("#count").textContent = books.length;   // place 2
  document.querySelector("#empty").hidden = books.length > 0;    // place 3
}

function removeBook(title) {
  // now do all three again, in reverse... and don't forget any of them
}
```

Adaugi apoi filtrare, sortare, „marchează ca citită” și statistici. Fiecare funcție nouă trebuie să știe toate locurile din pagină care depind de date. Când uiți unul, ecranul arată altceva decât datele. Bug-ul ăsta se numește _UI out of sync_ și a fost problema principală a frontend-ului înainte de framework-uri.

## Soluția: UI-ul calculat din state

**Ce e.** În loc să modifici pagina bucată cu bucată, ții datele într-un singur loc (**state**) și scrii o funcție care desenează tot UI-ul din ele. La orice schimbare actualizezi state-ul și redesenezi.

**Exemplu minim**
```js
let state = { books: [] };

function render() {
  const { books } = state;
  document.querySelector("#list").innerHTML = "";
  for (const title of books) {
    const li = document.createElement("li");
    li.textContent = title;
    document.querySelector("#list").append(li);
  }
  document.querySelector("#count").textContent = books.length;
  document.querySelector("#empty").hidden = books.length > 0;
}

function addBook(title) {
  state = { ...state, books: [...state.books, title] };
  render(); // one place knows how the UI looks
}
```

Acum adăugarea, ștergerea și filtrarea doar schimbă state-ul. Funcția `render` e singurul loc care știe cum arată pagina.

**Problema care rămâne.** Să ștergi și să recreezi tot DOM-ul la fiecare schimbare e lent și pierde lucruri ca focusul dintr-un input sau poziția de scroll.

**Aici apare React.** Scrii componente care descriu cum arată UI-ul pentru un anumit state, ca funcția `render` de mai sus. React compară ce ai descris acum cu ce era înainte și modifică în DOM **doar diferențele**. Ideea de bază, `UI = f(state)`, e exact cea din exemplul de mai sus. Continuarea e în [06-react/01-cum-gandeste-react.md](../06-react/01-cum-gandeste-react.md).

<details>
<summary>Întrebare de interviu: De ce folosim React în loc de JavaScript simplu cu DOM-ul?</summary>

Cu DOM-ul manual, fiecare schimbare a datelor trebuie propagată de mână în toate locurile din pagină care depind de ele. Pe măsură ce aplicația crește, ecranul ajunge ușor să nu mai corespundă cu datele. React inversează modelul: descriu declarativ cum arată UI-ul pentru un state dat, iar React calculează ce s-a schimbat și actualizează eficient doar acele noduri din DOM. Pe lângă asta primesc componente reutilizabile și un ecosistem matur.
</details>

## Mai departe

- [MDN — Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
- [javascript.info — Document](https://javascript.info/document)
