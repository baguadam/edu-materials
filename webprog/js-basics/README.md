# 1. gyakorlat

A félév első felében a `JavaScript` programozási nyelvvel ismerkedünk meg. Maga a nyelv `dinamikusan típusos (dynamically typed)` és `interpretáló (interpreted)`. Az előbbi azt jelenti, hogy nem mi deklaráljuk a változók típusát manuálisan, hanem az _futási időben_ dől el. Ebből következik, amit gyakorlaton is láttunk: egy változónak változhat a típusa. Az interpretálás annyit jelent, hogy nem kell nekünk manuális fordítani a kódot a környezet (a mi esetünkben a böngésző) futtatja azt (a valóságban történik gépikódra fordítás a _JIT (Just-In-Time) Compiler_ segítségével, de ezzel nem kell foglalkoznunk).

## Házi

Ha még nincs a Canvasen megfelelő feladat a házira, és elkészültél vele, nyugodtan dobd át Teamsen üzenetben.

**Határidő**: következő gyakorlat kezdete.

> ### 📚 FELADAT
>
> `TASKS.md` 8. feladata. Elég csupán a `.js` fájlt elküldeni.

## Hasznos billentyűkombinációk VS Code-ban

- **Több kurzor**: `Ctrl + Alt + ↑ / ↓`
- **Sor fel/le mozgatása**: `Alt + ↑ / ↓`
- **Sor fel/le másolása**: `Shift + Alt + ↑ / ↓`
- **Adott blokk kommentté tétele/ennek visszavonása**: `Ctrl + k + c` / `Ctrl + k + u`
- **Szimbólum átnevezése**: `F2`

## Változók létrehozása (let vs const vs var)

JavaScriptben ez a három kulcsó áll rendelkezésünkre, ha változókat szeretnénk létrehozni. A hasonlóságokat, különbségeket a táblázat demonstrálja:
| Kulcsszó | `var` | `let` | `const` |
|---------------------|-----------------------------------------|------------------------------------|------------------------------------|
| Scope | Fuction-scoped: mindenhol látható a függvényen belül, ahol létrehoztuk, még akkor is, ha egy belső scope-ban jött létre (például egy `if`-en belül) | Block-scoped: csak abban a blokkban létezik, amiben létrehoztuk | Block-scoped - szintén |
| Újradeklarálás | ✅ Megengedett bárhol | ❌ Ugyanabban a scope-ban NEM | ❌ Ugyanabban a scope-ban NEM |
| Érték változtatása | ✅ Megengedett | ✅ Megengedett | ❌ NEM megengedett (ezért konstans nyilván) |
| Használata | Alapvetően legacy kódokban, modern JS-ben NEM | Ha a változó értéke változhat | Ha a változó értéke nem fog változni |

```js
function example() {
  if (true) {
    var x = 10;
    let y = 20;
    const z = 30;
  }
  console.log(x); // kiírja, hogy 10, hiszen x a függvényen belül létezik
  console.log(y); // ReferenceError, itt már nem létezik y
  console.log(z); // ReferenceError, itt már nem létezik z
}
testLetConst();
```

## Tömbök

A tömb az egyik legfontosabb adatszerkezet, amivel találkozunk most az első alkalmakon, és amit aktívan használni is fogunk.

```js
const fruits = ["alma", "körte", "szilva"]; // legegyszerűbb módja a tömbök létrehozásának
const mixed = [1, "alma", true]; // a dinamikus típusokból adódóan vegyes típusokat is tárolhatunk a tömbökben

// üres tömb létrehozása, majd feltöltése értékekkel:
const empty = [];
empty[0] = "nagyon";
empty[1] = "üres";
console.log(empty);

// a tömb tudja magáról, hogy hány elemet tárolunk benne, méretét egyszerűen lekérdezhetjük:
console.log(empty.length);

// elem beszúrása a végére
empty.push("minden");

// elem kivétele a tömb végéről
console.log(empty.pop());

// elem kivétele a tömb elejéről
console.log(empty.shift());

// tartalmaz-e adott elemet?
console.log(empty.includes("nagyon"));
```

Alapvetően `mátrix` adatszerkezet nem létezik a nyelvben, ha hasonló viselkedést szeretnénk elérni, azt \_tömbök tömbje_ként tudjuk megtenni. Nézzünk erre egy-egy példát, illetve azt, hogyan tudunk iterálni a tömb elemein foreach-típusú ciklusokkal:

```js
const matrix = [
  [1, 2, 3], // matrix[0]
  [3, 4, 5], // matrix[1]
  [6, 7, 8], // matrix[2]
];

// ha az tömb elemein szeretnénk végigmenni úgy, hogy nincs szükség az indexekre: for of
for (let elem of matrix) {
  console.log(elem); // itt most az egy es elemek: [1, 2, 3] és [3, 4, 5] és [4, 5, 6]
}

// ha a tömb elemeinek indexére van szükségünk: for in
for (let i in matrix) {
  console.log(i);
}
```

### Spread Operator (...)

Az egyik legfontosabb modern nyelvi elem JS-ben. Lényege az, hogy egy `iterálható` konténert (pl tömb, object) elemeire "spreadel/expandel". Gyakorlatilag olyan ez, mintha végigmenne az összes elemen és megteszi azt helyetted, hogy manuálisan minden egyes elemet "leír". Emiatt szoktam rá azt használni, hogy "belehányjuk" az elemeket... Értékeljük, miben különbözik, ha `push` method használatával adok hozzá új elemet, illetve miben, ha `spread operator`t használok, majd a végére beszúrom az értékemet.

```js
const nums = [1, 2, 3];
const copyWithAdditions = [-2, ...nums, 10]; // minthat azt írnám: [-2, 1, 2, 3, 10]

// ha a következőt teszem:
const newNums = [1, 2];
const copy = [...newNums];
console.log(copy === nums); // false lesz, hiszen egy új tömb jön létre, aminek más a memóriacíme!!!
```

Hol lehet hasznos? Sok helyen, látunk majd még rá példákot, most csak hogy egyet mutassak: [Math.max](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max). Ha megnézitek a dokumentációt, konkrét elemeket vár, amik közül eldönti, hogy melyik a legnagyobb. Hogyan tudnánk egy tömb elemeiről eldönteni, hogy melyik a maximum. Egyszerű! Használjuk a `spread operator`t:

```js
console.log(Math.max(1, -2, 6)); // 6

// tömbre:
const nums = [1, -2, 6];
console.log(Math.max(...nums)); // "belehányom" a tömb elemeit mint paramétereket
```

## Objects

> ### 💡 FONTOS
>
> Adott esetben az `objects` alatt nem egy osztály példányát értjük, hanem `key-value párok` gyűjteményét. Igazából egy collection ez.

Viszonylag egyszerűen tudunk létrehozni objecteket a `{}` segítségével:

```js
const person = {
  name: "Matyi",
  age: 24,
  hasStrava: true,
  greet: () function {
    console.log("Hello, az én nevem: ", this.name);
  }
};

// értékek kinyerése:
const name = person.name // "Matyi"
const age = person.age // 24;

// haladóbb módszer: destrukturálás szintaxisa
// mintha azt mondanám, hogy kérem a person objectből ezt a két értéket:
const { age, name } = person; // ekvivalens a fentivel, ahol egyenként szedem ki belőle az értékeket
```

Objectek esetén is tudjuk használni a már ismert `spread operatort`. Az alapelv hasonló, az objectet iterálhatók, így képesek vagyunk új copy-kat létrehozni a segítségével. Ezen felül ami nagy előny itt, hogy nemcsak le tudunk másolni egy objectet, de képesek vagyunk a segítségével annak meglévő `property`-jeit módosítani vagy akár újakat hozzáadni:

```js
const band = {
  name: "30Y",
  age: 25,
};

// spreadeljük a meglévő objectet, adunk neki egy új key-value párt
const updated = { ...band, singer: "Beck Zoli" };

// spreadeljük a meglévő objectet, felülírjuk egy meglévő key értékét
const newBand = { ...band, name: "hiperkarma" };
```

## Függvények létrehozása: tradicionális és arrow function

Alapvetően kétféle szignatúrát tudunk használni JS-ben, ha függvényeket szeretnénk létrehozni: `fuction keyword`és `arrow functions`. Ezekre példák:

```js
// Tradiciális függvény a function kulcsszóval:
function greet(name) {
  return "Hello, " + name;
}

console.log(greet("Pista")); // Hello, Pista

// Függvény létrehozása arrow functionként:
const greet = (name) => {
  return "Hello, " + name;
};

console.log(greet("Sanyi")); // Hello, Sanyi

// Ha a függvény törzse csak egy kifejezés elhagyhatjuk a {}-et és a returnt
// DE HA van {}, akkor mindenképp kell return, ha értéket akarok visszaadni.
// Itt most működne, hogy
const greetExpression = (name) => "Hello, " + name;
```

Mik a hasonlóságok és különbségek a kettő között?
| Tulajdonság | Hagyományos függvény (`function`) | Arrow function (`=>`) |
|-------------------------|-----------------------------------------|--------------------------------------------|
| Szintaxis | `function nev(param) { ... }` | `const nev = (param) => { ... }` |
| Hoisting | Hoisting történik (előbb is hívható, minthogy a kódban definiálva lenne) | Nincs hoisting (előbb definiálni kell, hogy hívni tudjuk) |
| `this` kötés | Dinamikus, a hívótól függ | Lexikális, a környezettől örökli |
| Név | Lehet névvel ellátott vagy névtelen | Mindig névtelen (változóhoz rendjük, minthacsak egy teljesen általános változót hoznánk létre) |
| Hossz / kódterjedelem | Bővebb szintaxis általában | Rövidebb, tömörebb (pl. egy soros return) |

Nézzünk egy példát arra, hol viselkedik máshogyan a `this` kötés:

```js
const person = {
  name: "Barna",
  age: 23,
  greetTraditional: function () {
    console.log("Hello, ", this.name);
  },
  greetArrow: () => {
    console.log("Hello, ", this.name);
  },
};

person.greetTraditional(); // Hello, Barna - a "this" kucslszó a hívó objektumra referál (person)
person.greetArrow(); // Hello, -  a körülvevő scope-ból örökli a "this"-t,
```

## Tömbfüggvények - Array Functions

Mielőtt a részletekbe megyünk, két fontos fogalom:

> ### 💡 ARRAY FUNCTIONS
>
> A `tömbfüggvények` a JS nyelv által beépített, tömbökön értelmezett metódusok, segíségükkel fel tudjuk dolgozni, át tudjuk alakítani, ki tudjuk szűrni, különböző műveleteket tudunk végezni a tömb elemeivel anélkül, hogy manuálisan `for-ciklus`okat kellene írnunk. Legtöbbjük `callback function`t kap paraméterül, ami definiálja, mit szeretnénk csinálni minden egyes elemmel.
>
> [TOVÁBBIAK ITT](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

> ### 💡 CALLBACK FUNCTION
>
> A `callback` egy egyszerű függvény, amit paraméterül átadunk egy másik függvénynek. Ekkor a külső függvény visszahívja (`calls back`) a kapott függvényt a megfelelő időben.

### `forEach`

Minden egyes elemre végrehajtunk valamit. Nem módosítjuk a tömböt, nem hozunk létre új tömböt, egyszerűen mintha végigiterálnánk rajta, majd például kiíratnánk az összes elemet.

```js
const users = [
  { name: "Barna", online: true },
  { name: "Eszter", online: false },
  { name: "Matyi", online: true },
];

// itt ugye most (user) => {} szignatúrájú callbacket adtam át paraméterül.
users.forEach((user) => {
  if (user.online) {
    console.log(`Szia! Van egy új üzeneted!`);
  } else {
    console.log(`MIÉRT NEM VAGY ELÉRHETŐ?`);
  }
});

// csinálhattam volna azt, hogy külön definiálom a callbacként használandő függvényt, majd átadom paraméterül:
const cb = (user) => {
  // ...
};
users.forEach(cb);
```

### `map`

A tömb minden elemét `transzformálja`, valamilyen műveletet végez rajtuk, így létrehozva egy új tömböt, ami a módosított elemeket fogja tartalmazni. Amire érdemes itt figyelni, mivel egy `transformer` callbacket használunk, mindenképp vissza kell adnia valamilyen értéket (`return`). Mit jelent ez?

```js
// ha csak egy expressionöm van, pl
(n) => n * 2; // ez egy jó callback, automatikus return történik

// ha összetettebb logikát írok és {}-t használok, manuális return kell
n => {
    // számolások-számolások-számolások
    const new = n * 2;
    return new;
}

// NEM JÓ:
n => {
    // valami kalkuláció, de nincs return
}

// példa:
const nums = [1, 2, 3];
const doubled = nums.map(n => n * 2);

// még egy példa:
const users = [
  { name: "Barna", age: 23 },
  { name: "Eszter", age: 20 },
  { name: "Matyi", age: 24 },
];

const userLabels = users.map(user => `${user.name} (${user.age} yrs)`);
```

### `filter`

Kiszűri (adott esetben jobb szó lenne a megtartja) azokat az elemeket, amikre teljesül a feltétel. Hasonló a helyzet, a callbacknek itt egy `boolean` értékkel kell visszatérnie.

```js
const nums = [1, 2, 3, 4, 5];
const evens = nums.filter((n) => n % 2 === 0);

// összetettebb példa:
const friends = [
  { name: "Barna", age: 23, favSport: "football", hasStrava: true },
  { name: "Matyi", age: 24, favSport: null, hasStrava: false },
  { name: "Eszter", age: 20, favSport: "running", hasStrava: true },
  { name: "Ádám", age: 23, favSport: "football", hasStrava: true },
];

const runningClub = friends.filter((friend) => {
  return (
    friend.hasStrava &&
    friend.age > 21 &&
    (friend.favSport === "running" || friend.favSport === "football")
  );
});
```

### `find`

Megkeresi az első elemet, amire ráillik a feltétel, ha nem talál ilyet, `undefined`-ot ad vissza. Hasonlóan egy `boolean` értékkel kell visszatérnia a callbacknek.

```js
const users = [
  { name: "Barna", age: 23 },
  { name: "Eszter", age: 20 },
];
const result = users.find((u) => u.age > 21);
```

### `some` és `every`

Előbbi megnézi, hogy van-e a tömbnek olyan eleme, amire teljesül egy adott feltétel, utóbbi megnézi, hogy minden elemére teljesül-e. Hasonlóan egy olyan callbacket vár, ami `boolean` értékkel tér vissza.

```js
const nums = [1, 2, 3];
console.log(nums.some((n) => n > 2)); // true
console.log(nums.every((n) => n > 0)); // true
```

### `reduce`

Talán ez a legbonyolultabb megérték szemponjából. Olyasmi a gondolatmenet, mint a `fold` volt funkcionális programozáson. Fokozatosan `reduce`-oljuk a tömbünket, amíg csupán csak egy érték marad. Segítségével például ki tudjuk számolni az elemek összegét, szorzatát, egyéb érdekes dolgokat. Egy olyan callbacket használ, ami minden elemen végigmegy, és azon felül, hogy iterál az elemeken, megával visz egy `accumulatort`. Definiálnunk kell egy `initial value`-t is az accumulator számára.

```js
// callback szintaxisa:
const cb = (accumulator, currentValue) => {
  // return a módosított accumulatort, pl:
  return accumulator + curr;
};

// tömbön alkalmazva:
myArray.reduce(cb, 0); // ahol a 0 az initial value

// példa - számok összege:
const nums = [1, 2, 3, 4];
const sum = nums.reduce((acc, curr) => acc + curr, 0);

// példa - maximum elem megkeresése:
const nums = [10, 45, 2, 99, 30];
const max = nums.reduce((acc, curr) => (curr > acc ? curr : acc), nums[0]);
```

## Osztályok

JavaScriptben is tudunk objektumorientáltan programozni. Ezzel még fogunk találkozni és foglalkozni a következő gyakorlatokon, ha eljutunk odáig, hogy összetettebb alkalmazásokat fejlesztünk. Addig is, ha valakit érdekel a téma, akkor [ITT](https://github.com/baguadam/edu-kliens-2024-2025-2/tree/main/GY2) kaphat egy kis betekintést a README elején!
