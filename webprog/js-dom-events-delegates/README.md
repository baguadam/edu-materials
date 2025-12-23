# 2-3. gyakorlat

Előző gyakorlat elején megismerkedtünk a `DOM`-mal (`Document Object Model`), amely a weboldalak szerkezetét és tartalmát reprezentálja egy fa struktúrában. Megtanultuk, hogyan lehet JavaScript segítségével hozzáférni és módosítani a `DOM` elemeit, például szövegeket, attribútumokat és stílusokat. A `DOM`-ra gondolhattok úgy, mintha a HTML-dokumentum fává alakulna, ahol minden elem egy csomópont. A `DOM` gyökere a `document`, ezen felül a következő fontosabb csomópontokkal találkozhatunk:

- `Element`: HTML elemeket reprezentál, például `<div>`, `<p>`, `<table>`, stb.
- `Text`: Az elemek közötti szöveges tartalmat képviseli.
- `Attribute`: Az elemek attribútumait, például `id`, `class`, `src`, stb.

## DOM elemek elérése

A `DOM` tulajdonképpen egy programozható interface-t ad a kezünkbe a felület felé. Ez a gyakorlatban azt jelenti, hogy JS kódot írva hozzá tundunk férni az oldal elemeihez, változtatni, módosítani tudjuk azokat. Ennek az első lépése, hogy elérjük az elemeket a kódunkban.

Minden oldalhoz kapunk egy globális `document` objektumot, ami - ahogy azt megbeszéltük - a `DOM` gyökere. Már ezen keresztül is el tudunk érni bizonyos dolgokat közvetlenül:

```js
document.title; // az oldal címe
document.body; // a body elem
```

És nyilván mivel ez a gyökér, ami a kezünkben van, rajta keresztül tudunk mélyebbre menni, megkeresni egyéb elemeket is. Erre több módszerünk is van, a legfontosabbak:

```js
// a getElementById() metódus - nevéből adódóan - ID-n keresztüli elérésre korlátozódik, talán ma
// már kevésbé használatos, hiszen a querySelector is tudja ezt, de érdemes ismerni. Nyilván ez is
// csupán egyetlen elemet ad vissza, ha az elem nem létezik, null a visszatérési érték
document.getElementById("myElement");

// a querySelector() metódus egy CSS szelektort vár, és az első olyan elemet adja vissza, ami megfelel a
// szelektornak. Általánosabb, mint a getElementById, hiszen bármilyen szelektort használhatunk
// (id, class, tag, attribútum, stb.), de szintén csak egyetlen elemet ad vissza, ha nincs találat, null a
// visszatérési érték
document.querySelector(".myClass");

// a querySelectorAll() metódus szintén egy CSS szelektort vár, de itt nem csak az első találatot kapjuk
// vissza, hanem egy NodeList-et, ami egy tömbszerű objektum, és az összes találatot tartalmazza. Ha nincs
// találat, egy üres NodeList-et kapunk. Fontos - és újra hangsúlyozandó -, hogy nem TÖMB a visszatérési érték!
document.querySelectorAll("div.myClass"); // az összes "myClass" class-szal rendelkező div
```

Nemcsak a `document`-en keresztül érhetünk el elemeket, hanem bármelyik `Element`-en is. Ez azt jelenti, hogy ha már van egy elemünk, akkor azon keresztül is tudunk további elemeket keresni, például a gyerekeit, unokáit, stb.

```HTML
<div id="container">
    <p class="text">Hello World</p>
    <p class="text">Hello Universe</p>
</div>
```

```js
const container = document.querySelector("#container");
const paragraphs = container.querySelector(".text");
```

## DOM elemek manipulálása

Okés, kezünkben vannak az elemek, de mit kezdjünk velük? Ezen a ponton gyakorlatilag tetszőlegesen tudjuk "manipulálni" az elemeket. Módosíthatjuk a szövegüket, az attribútumaikat, a stílusukat, létrehozhatunk új elemeket, törölhetünk elemeket. Nézzünk ezekre (`fancy kifejezés`: a teljesség igénye nélkül) néhány példát!

### Szöveg módosítása

Használhatjuk az `innerText` és a `textContent` property-ket, ha nyitó-záró tagek között lévő szöveget akarunk módosítani. A kettő között annyi a különbség, hogy az `innerText` figyelembe veszi a CSS stílusokat (pl. ha egy elem `display: none`, akkor az `innerText` nem fogja visszaadni a benne lévő szöveget, tehát csak azt a szöveget adja vissza, ami látható), míg a `textContent` nem.

```HTML
<p><span hidden>Mr. Invisible </span>Mr. Incredible</p>
```

```js
const p = document.querySelector("p");
console.log(p.innerText); // "Mr. Incredible"
console.log(p.textContent); // "Mr. Invisible Mr. Incredible"

// nyilván nemcsak lekérdezni, hanem módosítani is tudjuk a szöveget, tehát getter-setter egyben:
p.innerText = "Ellastigirl";
```

> ### 💡 innerText vs innerHTML
>
> Fontos különbséget tenni a kettő között. Az `innerText` csak a szöveget módosítja, míg az `innerHTML` a HTML tartalmat. Tehát ha HTML tageket is szeretnénk hozzáadni vagy módosítani, akkor az `innerHTML`-t kell használni. Természetesen ebből adódóan az innerHTML a megadott `string`et HTML-ként értelmezi, tehát ha `<b>Bold</b>`-ot adunk meg, akkor a szöveg "Bold" lesz félkövérrel. Fontos viszont, hogy mindkettő "felülüti" a korábbi tartalmat!

```HTML
<p><span>Nagyon egyszerű!</span></p>
```

```js
const p = document.querySelector("p");
p.innerText = "<b>Bold</b>"; // a szöveg pontosan ez lesz: <b>Bold</b>, <p>"<b>Bold</b>"</p> kerül a DOM-ba
p.innerHTML = "<b>Bold</b>"; // a szöveg most félkövér Bold lesz, mert <p><b>Bold</b></p> kerül a DOM-ba
```

### Attribútumok kezelése

Ezen a téren négy fontos metódus van a kezünkben: `getAttribute()`, `setAttribute()`, `hasAttribute()`, `removeAttribute()`, nyilván ők nevüknek megfelelően működnek.

```HTML
<img id="kerkez" src="kerkez-liverpool.jpg" alt="Kerkez Milos Liverpool mezben" />
```

```js
const img = document.querySelector("#kerkez");
console.log(img.hasAttribute("src")); // true
console.log(img.hasAttribute("title")); // false

const src = img.getAttribute("src"); // "kerkez-liverpool.jpg"

img.setAttribute("alt", "Kerkez Milos Liverpool mezben, gólörömmel"); // módosítja az alt attribútumot
img.setAttribute("title", "Kerkez Milos"); // létrehozza a title attribútumot

img.removeAttribute("src"); // eltávolítja a src attribútumot
```

### Data attribútumok (`data-*`)

HTML-ben bármilyen "extra" adatot el tudunk tárolni a `data-*` attribútumok segítségével. Ezeket a `DOM`-ban a `dataset` property-n keresztül érhetjük el, ami egy objektum, ahol a kulcsok a `data-` utáni részek `camelCase` formában.

```HTML
<div id="kerkez" data-position="left" data-team="liverpool" data-team-league="premier-league">Kerkez Milos</div>
```

```js
const kerkez = document.querySelector("#kerkez");
console.log(kerkez.dataset.position); // "left"
console.log(kerkez.dataset.team); // "liverpool"
console.log(kerkez.dataset.teamLeague); // "premier-league"

kerkez.dataset.position = "right"; // módosítja a data-position értékét
kerkez.dataset.age = "22"; // létrehozza a data-age attribútum
```

### Stílus módosítása

Az elemek stílusát a `style` property-n keresztül tudjuk módosítani. Itt a CSS tulajdonságokat `camelCase` formában kell megadni, ezen felül ez a rész új/extra dolgok nem tartalmaz.

```HTML
<div id="kerkez" style="color: red; background-color: yellow;">Kerkez Milos</div>
```

```js
const kerkez = document.querySelector("#kerkez");
kerkez.style.color = "blue"; // kék szöveg
kerkez.style.fontSize = "24px"; // 24px betűméret
kerkez.style.border = "2px solid black"; // fekete keret
kerkez.style.backgroundColor = ""; // eltávolítja a háttérszínt
```

Viszont látjátok, hogy ez nem a legszebb megoldás, nagyon repetatív ebben a formában. Ha csak egy-egy stílust akarunk módosítani vagy hozzáadni, akkor nyilván ez a legegyszerűbb, de ha több stílust is módosítani akarunk, akkor érdemesebb egy `CSS` osztályt létrehozni, és azt hozzáadni/eltávolítani az elemről a `classList` property-n keresztül.

```HTML
<div id="kerkez" class="kerkez">Kerkez Milos</div>
```

```CSS
.kerkez {
  color: red;
  background-color: yellow;
}
```

```js
const kerkez = document.querySelector("#kerkez");
kerkez.classList.add("kerkez"); // hozzáadja a "kerkez" osztályt
kerkez.classList.remove("kerkez"); // eltávolítja a "kerkez" osztályt

// Induljunk ki abból, hogy feltételtől függően akarunk stílusosztályt hozzáadni vagy eltávolítani:
const isInStartingLineup = true;
if (isInStartingLineup) {
  kerkez.classList.add("starting-lineup");
} else {
  kerkez.classList.remove("starting-lineup");
}

// Erre van ennél sokkal elegánsabb és egyszerűbb megoldás is: toggle:
kerkez.classList.toggle("starting-lineup", isInStartingLineup);
// itt az első paraméter a hozzáadandó/eltávolítandó osztály, a második pedig egy boolean,
// ha true, akkor hozzáadja, ha false, akkor eltávolítja
```

### Property-k módosítása

A DOM elemek `property`-kkel is rendelkeznek, amik bár gyakran tükrözik az attribútumokat, de nem mindig ugyanazok. Például egy `input` elem esetén a `value` attribútum beállítása az alapértelmezett értéket határozza meg, míg a `value` property a jelenlegi értéket. Néhány tipikus példa a property-k használatára:

```HTML
<input id="username" type="text" value="defaultUser" />
<input id="subscribe" type="checkbox" checked />
```

```js
const usernameInput = document.querySelector("#username");
const subscribeInput = document.querySelector("#subscribe");

console.log(usernameInput.value); // "defaultUser"
console.log(subscribeInput.checked); // true

usernameInput.value = "newUser"; // módosítja a jelenlegi értéket
subscribeInput.checked = false; // eltávolítja a pipát
```

### Új elemek létrehozása és beszúrása

Viszonylag alapvető feladat, hogy dinamikusan szeretnék új elemeket létrehozni, azokat beszúrni a kódba. Alap JavaScript segítségével ez például egy módja lehet egy `SPA (Single Page Application)` létrehozásának. Ehhez tudnunk kell legenerálni az elemet, ellátni azt a megfelelő tulajdonságokkal, értékekkel, majd beszúrni a `DOM`-ba.

Az elem létrehozására a `document.createElement()` metódust használhatjuk, amely egy új `Element`-et hoz létre a megadott tag névvel. Ezen a ponton a létrehozott `Element` ugyanúgy viselkedik és kezelhető, mint bármelyik másik `Element`, amit például `querySelector` használatával értünk el, tehát tudunk neki attribútumokat adni, stílusokat beállítani, szöveget hozzáadni, stb.

```js
const playerContainer = document.createElement("div"); // létrehozunk egy div elemet
playerContainer.textContent = "Játékos Konténer";
playerContainer.classList.add("player-container"); // hozzáadunk egy class-t
playerContainer.style.border = "1px solid black"; // hozzáadunk egy stílust
// ... és bármilyen más korábban látott műveletet elvégezhetünk rajta
```

Ezen a ponton a létrehozott elem azonban még nincs a `DOM`-ban, tehát az oldalon sem látszik értelemszerűen. Ahhoz, hogy megjelenjen, be kell szúrnunk valahova. Erre több módszer is rendelekzésünkre áll, a legfontosabbak:

- `appendChild()`: Egy elemet hozzáad egy szülő elemhez, a szülő elem utolsó gyerekeként.
- `append()`: Hasonló az `appendChild()`-hoz, de több elemet is hozzá tud adni egyszerre.
- `insertBefore()`: Egy elemet egy adott szülő elemhez ad hozzá, egy megadott referencia elem elé.
- `replaceChild()`: Egy szülő elem egyik gyerekét egy új elemmel helyettesíti.

```HTML
<div id="contaier"></div>
```

```js
const container = document.querySelector("#container");
container.appendChild(playerContainer); // hozzáadjuk a playerContainer-t a container-hez, mint utolsó gyerek
```

> ### 💡 appendChild() vs innerHTML
>
> Ismét fontos különbséget tenni a kettő között. Az `appendChild()` egy létező elemet ad hozzá a `DOM`-hoz (tehát például egy `createElement()`-tel létrehozottat). A hozzáad alatt azt értjük, hogy megtartja a korábbi struktúrát, és csak hozzáfűzi az újat mint utolsó gyerek. Hasonló viselkedést érünk el a fent felsorolt metódusokkal is, nyilván a nevüknek és a viselkedésüknek megfelelően. Ezzel szemben, ha az `innerHTML`-t használjuk/módosítjuk, nem egy ténylegesen létező `Node`-ot adunk hozzá, hanem egy `string`-et, amit a böngésző HTML-ként értelmez. Ez azt jelenti, hogy ha van egy meglévő tartalmunk, és az `innerHTML`-t használjuk, akkor a korábbi tartalom felülíródik az újjal. Ez azt jelenti, hogy a korábbi gyerek elemek elvesznek, és csak az új tartalom marad meg.

## Események és eseménykezelés

Az eseményekre tekinhetünk úgy, mintha "jelek" lennének, amiket a böngésző küld, amikor a felhasználó interakcióba lép az oldallal. Ez lehet például egy `kattintás`, egy `gomb lenyomása`, `egér mozgatása`, `input megváltozása`, stb. Az események segítségével dinamikusan tudunk reagálni a felhasználói interakciókra, és ennek megfelelően módosítani az oldal tartalmát vagy viselkedését. Ezekre JavaScriptben `eseménykezelők (event handlers)` segítségével tudunk reagálni.

Régebbi módszer volt az eseménykezelők HTML-ben történő megadása (ez ma már NEM ajánlott), például:

```HTML
<button onclick="alert('Hello World!')">Kattints rám!</button>
```

Ezzel szemben a modernebb és ajánlott módszer az `addEventListener()` metódus használata. Ennek egyik nagyon előnye, hogy több eseménykezelőt is hozzá tudunk adni ugyanahhoz az eseményhez, illetve pontosabban szabályozhatjuk az eseménykezelők viselkedését. Például:

```HTML
<button id="substitute-btn">Játékos lecserlélése</button>
```

```js
const button = document.querySelector("#substitute-btn");

// Hozzáadhatom az eseménykezelőt úgy, hogy létrehozok egy névvel ellátott arrow function-t, majd azt adom át paraméterként,
// azonban ebben az esetben fontos, hogy a függvényt NE hívjam meg, tehát ne legyenek utána zárójelek!
const handleClick = () => {
  alert("Játékos lecserélve!");
};

button.addEventListener("click", handleClick);
// ha azt írnám, hogy button.addEventListener("click", handleClick()), nem lenne helyes megközelítés.

// Ugyanígy működik név nélküli függvény esetén is:
button.addEventListener("click", () => {
  alert("Játékos lecserélve!");
});
```

Az eseménykezelőknek átadható egy esemény objektum is, ami további információkat tartalmaz az eseményről, például az eseményt kiváltó elemet, az esemény típusát, stb.

```js
const handleClick = (event) => {
  alert("Játékos lecserélve!");
  console.log(event); // Az esemény objektum kiíratása a konzolra
};

button.addEventListener("click", handleClick); // automatikusan átadja az esemény objektumot
```

Mi történik akkor, ha szeretnék az eseménykezelőnek átadni további paramétereket is? Azt mondtuk, nem jó szintaxis, ha a függvény után zárójeleket teszünk (tehát meghívjuk). Akkor mégis mi a megoldás? Ilyenkor egy "wrapper" függvényt kell létrehoznunk, ami meghívja a tényleges eseménykezelőt a szükséges paraméterekkel együtt.

```js
const handleClick = (event, additionalParam) => {
  alert("Játékos lecserélve!");
  console.log(event);
  console.log(additionalParam);
};

// fontos, hogy ezen a ponton a wrapper functionnal el kell "kérnie" az esemény objektumot is, majd azt továbbadni
// az eseménykezelőként használt függvénynek, ahogy tesszük azt itt is.
button.addEventListener("click", (event) =>
  handleClick(event, "További információ")
);
```

### Event Bubbling és Delegálás

> ### 💡 EVENT BUBBLING
>
> Amikor egy esemény, például egy `kattintás` bekövetkezik egy elementen, az esemény "buborékol" fel a DOM fa szerkezetén keresztül, elérve a szülő elemeket. Ez azt jelenti, hogy ha egy gyermek elemre kattintunk, az esemény először a gyermek elemre hat, majd tovább terjed a szülő elemekre egészen a `document` gyökér elemig.
>
> Ez a viselkedés lehetővé teszi, hogy egyetlen eseménykezelőt alkalmazzunk egy szülő elemre, amely kezeli az összes gyermek elem eseményeit. Ez különösen hasznos dinamikusan létrehozott elemek esetén, ahol nem tudjuk előre, hány gyermek elem lesz jelen a DOM-ban.

Nézzük az alábbi struktúrát:

```HTML
<div id="parent">
  <button class="child">Gyermek 1</button>
  <button class="child">Gyermek 2</button>
  <button class="child">Gyermek 3</button>
</div>
```

Ekkor a `bubbling` viselkedése a következőképpen néz ki valamelyik gyermek gombra kattintva: `button.child` -> `div#parent` -> `body` -> `html` -> `document`. Tehát először a forrás elemen (adott esetben a gombon) történik meg az eseménykezelés, majd feljebb a szülő elemen (ha több szülő van, akkor mindegyiken sorban feljebb), egészen a `document`-ig.

> ### 💡 DELEGATION
>
> Az esemény delegálás egy olyan technika, amely kihasználja az esemény buborékolását a DOM-ban. Ahelyett, hogy minden egyes gyermek elemhez külön eseménykezelőt adnánk hozzá, egyetlen eseménykezelőt helyezünk el a szülő elemre. Amikor egy gyermek elemre kattintanak, az esemény "buborékol" fel a szülő elemhez, ahol az eseménykezelő meghívódik.
>
> Röviden tehát: a `bubbling` lehetővé teszi, hogy egy szülőn kezeljük a gyerekek eseményeit. Ez különösen hasznos dinamikusan létrehozott elemek esetén, ahol nem tudjuk előre, hány gyermek elem lesz jelen a DOM-ban.

Az alap gondolat tehát az, hogy a szülőre tesszük az eseménykezelőt. Képzeljétek el azt az esetet, amikor van egy táblázatunk, ami tartalmaz nagyon sok sort és nagyon sok oszlopot. Azt szeretném elérni, hogy a cellákra kattintva történjen valami. Egyrészt, ha minden cellára eseménykezelőt teszek az sok kódot jelenteni, nehéz lenne karbantartani, költséges lehet a teljesítmény szempontjából, arról nem is beszélve, hogy mi történik akkor, ha hozzáadunk újabb sorokat/oszlopokat a táblázathoz. Ekkor jön képbe a delegálás, amikor a táblázat szülő elemére (például a `<table>` vagy `<tbody>` elemre) helyezünk egy eseménykezelőt, és azon keresztül kezeljük a cellák eseményeit. Tegyük fel, hogy a `table` elemre tettünk egy `click` eseménykezelőt. Ekkor viszont bárhol kattintunk a táblázaton belül, az eseménykezelő meghívódik, nem csak a cellákra kattintva. Ezt úgy tudjuk kezelni, hogy az esemény objektum `target` property-jét használjuk, ami megmutatja, hogy pontosan melyik elemre történt a kattintás.

```HTML
<table>
  <thead>
    <tr>
      <th>Csapat</th>
      <th>Győzelmek</th>
      <th>Vereségek</th>
      <th>Döntetlenek</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="team">Liverpool</td>
      <td>20</td>
      <td>5</td>
      <td>3</td>
    </tr>
    <tr>
      <td class="team">Manchester City</td>
      <td>18</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <td class="team">Chelsea</td>
      <td>15</td>
      <td>10</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
```

```js
const table = document.querySelector("table");

table.addEventListener("click", (event) => {
  console.log(event.target); // kiírja, hogy melyik elemre kattintottunk (ezt nevezzük a forrás elemnek)
});
```

Azt, hogy a `target` elem megegyzik-e egy adott szelektorral, a `matches()` metódussal tudjuk ellenőrizni. Ez egy boolean értéket ad vissza, tehát igaz vagy hamis lesz a visszatérési érték attól függően, hogy a `target` elem megfelel-e a szelektornak.

```js
// ha azt szeretném, hogy minden <td> elemre kattintva történjen valami, akkor így nézne ki:
table.addEventListener("click", (event) => {
  if (event.target.matches("td")) {
    console.log("Egy cellára kattintottál!");
  }
});

// ha csak a "team" class-szal rendelkező cellákra szeretnék reagálni:
table.addEventListener("click", (event) => {
  if (event.target.matches("td.team")) {
    console.log(`Egy "csapat" cellára kattintottál!`);
  }
});
```

Nézzünk egy példát dinamikusan létrehozott elemekkel is:

```HTML
<div id="team-container"></div>
<button id="add-team-btn">Új csapat hozzáadása</button>
```

```js
const teamContainer = document.querySelector("#team-container");
const addTeamBtn = document.querySelector("#add-team-btn");

addTeamBtn.addEventListener("click", () => {
  const newTeam = document.createElement("div");
  newTeam.classList.add("team");
  newTeam.textContent = `Csapat ${teamContainer.children.length + 1}`; // nem túl kreatív név
  teamContainer.appendChild(newTeam);
});

// Tehát nem szükséges minden egyes "team" elemre külön eseménykezelőt tenni, elég a szülőre tenni egyet, majd
// azon keresztül nézzünk, hogy a "team" class-szal rendelkező elemre kattintottak-e. Ezen a ponton lehetne akár
// sokkal több elem is a container divben, más class-szal, stb., de csak a "team" class-szal rendelkezőkre fogunk reagálni.
teamContainer.addEventListener("click", (event) => {
  if (event.target.matches(".team")) {
    alert(`Kattintottál a ${event.target.textContent} elemre!`);
  }
});
```
