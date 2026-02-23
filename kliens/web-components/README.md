# Web komponensek

Fokozatosan elindultunk az egységbe zárás irányába, először egyszerűen csak egy osztályt létrehozva, ami paraméterként megkapta a táblázatot, amit szeretnénk felokosítani. Azonban ez még nem a vég! Van ennél jobb, okosabb eszköz is a kezünkben, hogy egységbe zárjunk funkcionalitást.
A `Web Components` igazából egy technológia (vagy eszközök gyűjteménye), amiknek a segítségével saját HTML elementeket tudunk létrehozni, ezeknek definiálni a viselkedést, elzárni a stílusát, így tovább. Alapvetően három fő technológiából áll:

1. Custom Elements
2. Shadow DOM
3. HTML Templates

Innen először megismerkedünk az elsővel...

## Custom Elements

Életünk első custom elementje a `sortable-table`. Ez egy olyan HTML element volt, aminek a működését mi magunk szabtuk meg, figyelve arra, hogy továbbra is összhangban maradjunk a progresszív fejlesztéssel, és csak okosítsuk a funkcionalitást, ha van JS. Egy ilyen custom element lérehozása a következőképpen nézett ki:

```js
class SortableTable extends HTMLElement {
  constructor() {
    super();

    // egyéb dolgok
  }

  // funkcionalitás, egyébb logika
}

// definiáljuk a custom elementet: megmondjuk, hogy milyen taggel akarunk rá referálni, illetve, hogy melyik class biztosítja számára a funkcionalitást:
customElements.define("sortable-table", SortableTable);
// itt megadhatunk egy harmadik, "options" paramétert is, erről majd később
```

A custom elementek rendelkeznek úgynevezett "lifecycle callback"-ekkel. (Akik esetleg jártasabbak Angularban, hasonló concept, mint ott a "lifecycle hook"-ok, pl. `ngOnInit`)
Innen amit szeretnék most kiemelni: `connectedCallback()`. Ez minden egyes alkalom meghívódik, amikor az elementünk hozzáadódik a dokumentumhoz. Ha pl. hozzáadok egy darab `sortable-table`-t HTML-hez, amikor az bekerül a dokumentumba, az ő SortableTable példányában ekkor fut le a `connectedCallback()`. És így tovább minden további hozzáadott esetén. Alapvetően az az ajánlott, hogy a custom element "setupját" ebben és ne a konstruktorban oldjuk meg. Ezalatt azt értem, hogy minden olyan műveletet, ami a DOM-ot értinti, vagy a `this`-re hivatkozik, érdemes itt elhelyezni, mert a konstruktorban még nem biztos, hogy minden készen áll. Mondhatni, párja a `disconnectedCallback()`, ami akkor fut le, amikor az element eltávolításra kerül a DOM-ból.

Nézzük a konkrét példa skeletonját:

```js
class SortableTable extends HTMLElement {
  constructor() {
    super();
  }

  connectedCallback() {
    // beveszünk elemeket
    // feliratkozunk eventekre
  }

  disconnectedCallback() {
    // leiratkozunk eventekről
  }
}

customElements.define("sortable-table", SortableTable);
```

HTML-en belül pedig:

```HTML
<sortable-table>
    <table>
        ...
    </table>
<sortable-table>
```

> ### 💡 FONTOS
>
> Fontos megérteni, hogyan viselkedik a `this` custom elementeken belül. Ilyenkor is létrejön egy példány az osztályból, és minden egyes `<custom-tag>` a saját instance-t kap, ami a működésért és az állapotért felel. Mivel az osztály `HTMLElement`-ből származik, a létrejövő példány nem egy "sima" JS objektum, hanem egy valódi DOM elem (HTMLElement/Element), tehát maga a `<sortable-table> node`. Ezért ha a konstruktorban logolom a `this`-t, DevTools-ban azt fogom látni, hogy `<sortable-table>…</sortable-table>`. Ugyanakkor ez is egy objektum, így instance property-ket (pl. this.data) nyugodtan hozzáadhatok, és a saját metódusaimat is ezen az instance-en keresztül érem el.

Ha belegondolunk, hogy mi történik, akkor ebben az implementációban mivel a HTMLElement-et bővítettük ki, a `table`-re vonatkozó default viselkedést, funcionalitást nem kaptuk meg. Ha végiggondoljátok a feladatot, tulajdonképpen itt csak egy alap táblázatot okosítottunk fel. Nem adtunk hozzá semmi extra UI elemet, mint például a char-counter-inputnál. Ezzel szemben létrehoztunk egy adott esetben fölöslegesnek gondolható wrapper taget.

Jó lenne, ha meg tudnánk tenni azt, hogy ilyenkor, amikor csak egy meglévő elemet akarunk felokosítani, ki tudjuk bővíteni az adott elem viselkedését és el tudjuk kerülni a wrappert. Erre valók a `custom built-in element`-ek:

1. Ilyenkor nem a `HTMLElement`-et bővítjuk ki, hanem a konkrét elemet, amivel dolgozni akarunk.
2. Amikor definiáljuk a custom elementet, megadunk egy harmadik paramétert is, amiben definiáljuk, hogy mit extendelünk.
3. HTML-ben nem lesz wrapper tag, az adott, kibővített elementet látjuk el egy `is="custom-name"` attribútummal.

Így a módosított kódunk:

```js
// JS szintjén most:
class SortableTable extends HTMLTableElement {
  cosntructor() {
    console.log(this);
    // ebben az esetben a this maga a table lesz, a korábbi implementációban nyilván ez a sortable-table volt
  }
}

// harmadik paraméter: extends: "table"
customElements.define("sortable-table", SortableTable, { extends: "table" });

// HTML szintjén:
<table is="sortable-table">....</table>;
```

Erre tökéletes példa volt az, amikor egy linket kellett felokosítanunl, ami kattintáskor nem navigál automatikusan a megoadott oldalra, hanem egy pop-upban megkérdezi, hogy biztosan szeretnénk-e ezt megtenni. Megint csak arról van szó, hogy egy meglévő elemet kell felokosítani. Nem akarunk új dolgot hozzáadni, semmi extrát csinálni. Csak az adott elem default viselkedésére van szükségünk, amit majd mi kibővítunk.

```js
class ConfirmLink extends HTMLAnchorElement {
  constructor() {
    super();
  }

  connectedCallback() {
    // feliratkozás a click eventre
  }

  disconnectedCallback() {
    // leiratkozás a click eventről
  }

  onClick = (e) => {
    // logika
  };
}

// itt pedig azt kell mondanunk, hogy extends: "a"
customElements.define("confirm-link", ConfirmLink, { extends: "a" });

// HTML-ben pedig:
<a is="confirm-link" href=""></a>;
```

> ### 💡 FONTOS
>
> Remélem, érezhető a különbség az előző órai, illetve az ezórai built-in element között. Sokadszor is hangúlyozva: célszerű ezt csinálni, ha csupán egy meglévő elementnek szeretnék egy plusz funkcionalitást adni, pl: legyen a táblázat rendezhető, a linknél kérdezze meg kattintásra, hogy biztos-e, stb.

## Shadow DOM

[Még több a Shadow DOM-ról (innen van a kép is)](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)

A webkomponensek témakör alapvetően három nagy részre bomlik tehát:

1. Custom Elements
2. Shadow DOM
3. Templates, Slots

A célunk most az, hogy modulárissá tegyük a web komponensünket úgy, hogy nem csak JS szintjén, de DOM szintjén is alkalmazunk rá valamilyen egységbe zárást. Ezt `Shadow DOM` segítségével tudjuk megoldani. Azt mondjuk, hogy a komponens egy saját DOM-mal tud rendelkezni, el van zárva a szülő DOM-jától, így van egy black-box a kezemben, ami a külső világtól függetlenül mindig ugyanúgy fog kinézni, bármikor be tudom rakni az oldalra. Ezáltal kapok egy magasabb szintű szeparációt.

- `Encapsulation`: a stílusok és a scriptek a Shadow DOM-ban nincsenek hatással az outside world-re
- `Scoped Styling`: a bent definiált stílusok elkülönülnek

Az ábra egész jól reprezentálja a koncepciót. A következőket érdemes megérteni/megjegyezni:

![image](https://github.com/user-attachments/assets/88c12877-f83c-4a59-bfcf-3e67dd02dde3)

- `Shadow host`: az a node a Light DOM-ban, amihez a Shadow DOM-ot "attach"-oljuk.
- `Shadow tree`: a DOM tree a Shadow DOM-ban
- `Shadow root`: a Shadow DOM root node-ja

Teljesen általánosan kód szintjén:

```js
// 1. bevesszük a hostot
const host = document.querySelector("#host");

// 2. hozzakapcsoljuk a Shadow DOM-ot
const shadow = host.attachShadow({ mode: "open" }); // FONTOS: a mode-ot hagyjuk MINDIG open-en!

// 3. legyártjuk az elemet
const span = document.createElement("span");
span.textContent = "HELLÓ A SHADOW DOM-BÓL";

// 4. befűzzük a shadow root alá
shadow.appendChild(span);
```

### Button szeparáció

Az alábbi példa jól demonstrálja a Shadow DOM lényegét, és annak működét.

Először is hozzunk létre egy egyszerű HTML fájlt, amiben helyezzünk el teljesen általános buttont, illetve előlegezzünk meg magunknak egy shadow-buttont is. Adjunk hozzá minimális stilizálást az oldalhoz.

```HTML
<style>
  button {
    background-color: red;
    color: white;
    padding: 10px 20px;
    width: 200px;
    height: 40px;
  }

  button:hover {
    background-color: brown;
  }
</style>

...
<button>LIGHT DOM BUTTON</button>
<shadow-button></shadow-button>
```

Ezen a ponton nyilván csak a "LIGHT DOM BUTTON" szöveget tartalmazó gomb lesz látható a megfelelő stílussal, a `shadow-button`-t még nem hoztuk létre.

> Itt ugye már mindenki látja, hogy ha így akarom használni HTML-ben, akkor ez NEM egy "custom built-in element" lesz

```js
class ShadowButton extends HTMLElement {
  constructor() {
    super();
  }
}

customElements.define("shadow-button", ShadowButton);
```

A következőkben három lépést fogunk követni:

1. Hozzácsatoljuk a Shadow DOM-ot a shadow-buttonhöz.
2. Létrehozunk benne egy buttont, amit hozzácsatolunk a shadow roothoz.
3. Definiálunk benne valamilyen, a globálistól eltérő stílust a gombra.

```js
class ShadowButton extends HTMLElement {
  constructor() {
    super();
  }

  connectedCallback() {
    // 1. lépés: Shadow DOM "bekötése"
    this.attachShadow({ mode: "open" });

    // 2. lépés: Létrehoztuk a buttont, majd hozzácsatoltuk a shadow roothoz
    const shadowButton = document.createElement("button");
    shadowButton.textContent = "SHADOW DOM BUTTON";

    // teljesen ugyanaz, mintha egy standard DOM-beli elemhez csatolnám hozzá
    this.shadowRoot.appendChild(shadowButton);
  }
}

customElements.define("shadow-button", ShadowButton);
```

Most ha ezen a ponton megállunk, még mielőtt bármi további stílust alkalmaznánk az így létrehozott gombunkra, akkor a következőt látjuk:

![image](https://github.com/user-attachments/assets/4e70c64f-6bf0-465e-816f-f79149df8a8d)

És ponotsan ez az elvárt viselkedés: a Light DOM-ra definiált stílusok nem "másznak be" a Shadow DOM-ba. Most adjunk valamilyen stílust a Shadow DOM-ban létrehozott gombhoz. Ezt a legegyszerűen úgy tehetjük meg, ha az előbbi kódot kiegészítjuk egy létrehozott style-taggel, benne a kívánt stílussal. Ekkor az alkalmazott stílus a teljes Shadow DOM-ban érvényes lesz:

```js
class ShadowButton extends HTMLElement {
  constructor() {
    super();
  }

  connectedCallback() {
    // 1. attach-oljuk a Shadow DOM-ot
    this.attachShadow({ mode: "open" });
    // 2. gomb létrehozása
    this.generateButton();
    // 3. stílus létrehozása
    this.generateStyle();
    // 4. befűzzük a shadow rootbe
    this.shadowRoot.append(this.styleTag, this.button);
  }

  generateButton() {
    this.button = document.createElement("button");
    this.button.innerHTML = "SHADOW DOM BUTTON";
  }

  generateStyle() {
    this.styleTag = document.createElement("style");
    this.styleTag.innerHTML = `
        button {
            background-color: blue;
            border-radius: 5px;
            border: 1px solid black;
        }
    `;
  }
}

customElements.define("shadow-button", ShadowButton);
```

Ekkora az oldal a következőképpen néz ki:

![image](https://github.com/user-attachments/assets/b7476d30-6521-4cb2-aeff-96a79e58d977)

Ha pedig a szerkezetét is megvizsgáljuk:

![image](https://github.com/user-attachments/assets/b026de9e-d7c2-483b-9053-943ad9fb82f1)

Láthatjuk, hogy a `shadow-button`-ön belül létrejött egy `shadow root`, ami alatt helyezkedik el a létrehozott style és button. Nyilván, hiszen pont ezt akartuk, amikor a shadow root-hoz ezeket appendeltük.

Ha esetleg szeretnénk elérni a a gombokat, akkor a következőket tapasztalhatjuk:

```js
// Így csupán csak a Light DOM-ban megtalálható buttonöket tudjuk elérni,
// így most ez pontosan 1 darab gombot fog tartalmani, azt, amelyiknek "LIGHT DOM BUTTON" a szövege.
const buttons = document.querySelectorAll("button");
buttons.forEach((b) => console.log(`BUTTONS: ${b.textContent}`));

// Ha a Shadow DOM-ban található gombot szeretnénk elérni, akkor azt a shadow rooton keresztül tudjuk megtenni.
// Ennek a hostja a shadow-button, így először bevesszük a shadow-buttont, majd ennek a shadow rootján
// keresztül egy selectorral ki tudjuk választani a benne található gombot.
const shadowButton = document
  .querySelector("shadow-button")
  .shadowRoot.querySelector("button");
console.log(`SHADOW BUTTON: ${shadowButton.textContent}`);
```

## Templates

[Templates, Slots - RÉSZLETESEN](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_templates_and_slots)

A template-ek segítségével újrahasznosítható HTML struktúrákat tudunk létrehozni, ezzel elkerülve azt, hogy ismételni kelljen nagyon sokszor önmagunkat. Ha van egy card például, amit többször szeretnénk megjeleníteni, készíthetünk számára egy template-et, így ezt kell csak mindig klózoznunk és befűznünk a megfelelő helyre.

Alapvetően template-et az azonos nevű tag segítségével hozunk létre. Az így létrehozott rész nem fog megejelnni az oldalon, amíg annak a contentje nem fűződött be a DOM-ba.

```html
<template>
  <div class="card">
    <h2>Default title</h2>
    <p>Default paragparh</p>
  </div>
</template>
```

Tehát csupán ezt hozzáadva a HTML-hez, nem fog semmi megjelenni. Ahhoz, hogy egy `template`-et megjelenítsünk, először nyilván el kell érnünk a DOM-ból, majd pedig a `content` property-jét klónoznunk kell. Erre gondolhattok úgy, mint egy gyári sablonra, amiből mindig újabb példányokat gyárthatunk, így elkerülve a duplikációt. A klónozott contentet pedig már csak be kell fűznünk a megfelelő helyre.

```js
const template = document.querySelector("template");
const body = document.querySelector("body");

// A paraméter itt, amit true-ként adunk meg arra vonatkozni, hogy a teljes subtree-t
// vagy csak az adott node-ot szeretnénk-e klónozni. True: subtree)
const templateContent = template.content.cloneNode(true);
body.appendChild(templateContent);
```
