# Styling

Fișier scurt. Nu ai nevoie să devii designer. Ai nevoie să poți citi stilurile generate de Claude, să știi de ce arată codul așa și să recunoști trei-patru lucruri pe care le verifică orice reviewer.

## Tailwind CSS

**Ce e.** Un framework de CSS bazat pe clase utilitare: în loc să scrii o regulă `.card { padding: 1rem; border-radius: 8px }` într-un fișier separat, pui direct pe element clase mici precum `p-4 rounded-lg border`. La build, Tailwind generează doar CSS-ul pentru clasele pe care le folosești efectiv.

**De ce contează.** E cea mai răspândită soluție de styling în proiectele React noi și aproape întotdeauna alegerea implicită a lui Claude. Echipele l-au adoptat pentru că:

- nu mai inventezi nume de clase;
- stilul stă lângă markup;
- spațierile și culorile vin dintr-o scală comună, deci interfața rămâne consistentă;
- nu se mai adună CSS mort.

**Exemplu minim.** Setup-ul curent cu Vite (verificat pe tailwindcss.com):

```bash
npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

```tsx
<button className="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 disabled:opacity-50">
  Save invoice
</button>
```

**Cum îl vezi în cod real.** Clasele lungi se citesc pe grupuri: layout (`flex`, `grid`, `gap-4`), spațiere (`p-4`, `mt-2`), tipografie (`text-sm`, `font-semibold`), culori (`bg-…`, `text-…`), stări (`hover:`, `disabled:`, `focus-visible:`), responsive (`md:`, `lg:`, adică „de la lățimea asta în sus”). În tutoriale mai vechi vei vedea un fișier `tailwind.config.js` și directive `@tailwind base;`. E setup-ul versiunilor anterioare.

**Capcana.** Clase construite dinamic din string-uri: `` `bg-${color}-500` ``. Tailwind caută clasele ca text întreg în cod, iar o clasă compusă la runtime nu există la build, deci nu primește CSS. Scrie clasele complete: `status === "paid" ? "bg-green-500" : "bg-red-500"`.

<details>
<summary>Întrebare de interviu: Ce avantaje și dezavantaje are Tailwind?</summary>

Avantaje: nu mai inventez nume de clase, stilul stă lângă componentă, am o scală comună de spațieri și culori care ține UI-ul consistent, iar CSS-ul final conține doar ce folosesc. Dezavantaje: markup-ul devine aglomerat cu clase lungi, iar la început trebuie învățat vocabularul. În React, aglomerarea se rezolvă extrăgând componente: un `<Button>` refolosit, nu aceleași 12 clase copiate peste tot.
</details>

## shadcn/ui

**Ce e.** O colecție de componente gata făcute (butoane, dialoguri, tabele, formulare), construite cu Tailwind peste primitive accesibile. Diferența față de o bibliotecă obișnuită: nu o instalezi ca dependență. Comanda CLI **copiază codul sursă al componentei în proiectul tău** (`src/components/ui/`), unde îl poți modifica.

**De ce contează.** E foarte populară în 2026 și Claude o propune des pentru UI-uri care arată profesionist rapid. Trebuie să știi că `components/ui/button.tsx` e cod din proiectul tău, nu un pachet din `node_modules`.

**Exemplu minim**

```bash
npx shadcn@latest init        # once per project (see ui.shadcn.com for the Vite setup)
npx shadcn@latest add button dialog
```

```tsx
import { Button } from "@/components/ui/button";

<Button variant="destructive" onClick={handleCancel}>Cancel booking</Button>
```

**Cum îl vezi în cod real.** Importuri din `@/components/ui/...` (`@` e un alias spre `src/`, configurat în `vite.config.ts` și `tsconfig`) și o funcție `cn(...)` care combină clasele condiționat.

**Capcana.** Să modifici componentele din `components/ui/` fără să știi că le-ai modificat pentru tot proiectul. Sau să ceri lui Claude „instalează shadcn” la jumătatea unui proiect care are deja alt sistem de stiluri. Alege la început și notează alegerea în `CLAUDE.md`.

<details>
<summary>Întrebare de interviu: Prin ce diferă shadcn/ui de o bibliotecă de componente clasică?</summary>

Nu e o dependență pe care o actualizezi din npm. CLI-ul copiază codul componentei în proiectul meu, deci îl dețin și îl pot modifica liber. Avantajul e controlul total și lipsa de lock-in. Dezavantajul e că actualizările nu vin automat: dacă vreau versiunea nouă a unei componente, o regenerez și îmi reaplic modificările.
</details>

## CSS Modules

**Ce e.** Fișiere CSS normale, numite `Componenta.module.css`, pe care le imporți în componentă. Build-ul face numele claselor unice automat, deci `.title` dintr-o componentă nu se ciocnește cu `.title` din alta. Vite le suportă fără nicio configurare.

**De ce contează.** E alternativa cea mai frecventă la Tailwind când o echipă preferă CSS clasic. O vei întâlni în proiecte existente.

**Exemplu minim**

```css
/* BookCard.module.css */
.card { padding: 1rem; border: 1px solid #ddd; border-radius: 8px; }
.title { font-weight: 600; }
```

```tsx
import styles from "./BookCard.module.css";

export function BookCard({ title }: { title: string }) {
  return (
    <article className={styles.card}>
      <h3 className={styles.title}>{title}</h3>
    </article>
  );
}
```

**Cum îl vezi în cod real.** `className={styles.ceva}`. În browser, clasa apare ca `_card_x7f3a` sau similar. Asta e garanția de izolare.

**Capcana.** Să amesteci trei sisteme de stil în același proiect (Tailwind, CSS Modules și stiluri inline) pentru că fiecare cerere către Claude a mers pe altă cale. Un proiect, un sistem principal.

<details>
<summary>Întrebare de interviu: Cum eviți conflictele de nume între clasele CSS ale componentelor?</summary>

Folosesc un sistem cu izolare: CSS Modules, care face numele claselor unice la build, sau clase utilitare (Tailwind), unde nu definesc clase proprii. Ambele elimină problema CSS-ului global, unde o clasă `.title` dintr-o parte a aplicației strică alt ecran.
</details>

## Ce verifică un reviewer la UI

**Ce e.** Patru lucruri de bază care fac diferența între „merge” și „e făcut cum trebuie”. Nu țin de un framework anume.

**De ce contează.** Un recrutor sau un client deschide proiectul pe telefon. Un interviewer apasă Tab prin formular. Astea se văd în primele 30 de secunde.

**Exemplu minim**

```tsx
// ✅ semantic, accessible, responsive
<form className="grid gap-4 md:grid-cols-2">
  <label htmlFor="email">Email</label>
  <input id="email" type="email" required />
  <button type="submit">Book</button>
</form>

// ❌ div soup: no keyboard, no screen reader, no Enter-to-submit
<div onClick={book} className="btn">Book</div>
```

**Cum îl vezi în cod real.** Checklist-ul de review:

1. **Responsive:** arată bine la 375px lățime (DevTools → Toggle device toolbar).
2. **Semantic:** `<button>` pentru acțiuni, `<a>`/`<Link>` pentru navigare, `<label>` pentru fiecare input.
3. **Tastatură:** poți folosi tot formularul doar cu Tab, Enter și Escape, iar focus-ul se vede.
4. **Stări:** hover, disabled și loading sunt vizibile.

**Capcana.** `<div onClick>` în loc de `<button>`. Arată la fel, dar nu poate fi folosit de pe tastatură și e invizibil pentru cititoarele de ecran.

<details>
<summary>Întrebare de interviu: De ce contează să folosești `<button>` în loc de un `<div>` cu onClick?</summary>

`<button>` primește focus din tastatură, se activează cu Enter și Space, e anunțat corect de cititoarele de ecran și trimite formularul în care se află. Un `<div>` nu are nimic din toate astea, iar ca să-l aduci la același nivel ai nevoie de `role`, `tabIndex` și handlere de tastatură. HTML-ul semantic aduce accesibilitate gratis.
</details>

## Mai departe

- [Tailwind CSS: Installing with Vite](https://tailwindcss.com/docs/installation/using-vite)
- [shadcn/ui: Vite](https://ui.shadcn.com/docs/installation/vite)
- [Vite: CSS Modules](https://vite.dev/guide/features.html#css-modules)
