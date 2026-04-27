# Redux Toolkit

A félév második felében a `Redux`-szal ismerkedtünk meg, ami egy állapotkezelő könyvtár főként React alkalmazásokhoz. Az alapgondolat az, hogy az alkalmazás állapotát ne szétszórva, különböző komponensekben tartsuk, hanem egy **központi "tárhelyen" (store)**.

## Miért használjuk a Reduxot?

A Redux célja, hogy:

- **Központosítsa** az állapotkezelés logikáját (`slice`, `reducer`, `action`)
- **Átláthatóvá** tegye az adatfolyamot az alkalmazásban
- **Könnyen integrálhatóvá** tegye az aszinkron műveleteket (`thunk`, `middleware`)
- **Hatékonyabbá** tegye nagy, komplex alkalmazások fejlesztését

Ez különösen akkor hasznos, ha:

- Több komponens osztozik ugyanazon állapoton
- Már nem elegendő a `prop drilling`
- A `Context API` nem ad elég rugalmasságot a bonyolult adatkezeléshez

## Mit ad hozzá a Redux Toolkit?

A `Redux Toolkit` a Redux hivatalos ajánlott eszköztára, amely leegyszerűsíti az állapotkezelés implementálását:

- Csökkenti a boilerplate kódot
- Automatikusan generál `action creator`-okat és `reducer`-eket a `createSlice` segítségével
- Egyszerűsíti a store konfigurálását (`configureStore`)

## Folyamatábra, fontosabb elemek

[Forrás](https://www.esri.com/arcgis-blog/products/3d-gis/3d-gis/react-redux-building-modern-web-apps-with-the-arcgis-js-api)
![image](https://github.com/user-attachments/assets/70a73bf1-eeb0-4965-8424-82f2c9902182)

Az ábra vizuálisan bemutatja, hogyan működik a Redux alapú állapotkezelés. Nézzük, hogy melyik elem mit jelent, milyen szerepet tölt be. Ebben a részben leginkább a mögöttes működés és a "mi történik a háttérben" dolgok vannak összegyűjtve, nem pedig az "így érdemes/ajánlott megírni". Ma már nem írunk nulláról reducer függvényeket, és nem használunk kézzel írt action objektumokat – ezek helyett használjuk a `Redux Toolkit` eszközeit. Tehát célja, hogy segítsen megérteni, mi történik a "motorháztető" alatt, amikor az RTK-t használjuk.

### Store

A `Store` a Redux "szívében" található. Ez tárolja az alkalmazás globális állapotát (state), valamint a reducereket, amik felelősek az állapot frissítéséért.

### Action, Action Creator

Az ` action` valójában egy egyszerű JavaScript objektum, ami azt írja le, hogy mi történt az alkalmazásban. Ezek igazából az alkalmazás reakciói valamilyen eseményre, mint például egy gomnyomásra. Ilyenkor az action beérkezik a `store`-ba, ott megkapja a `Reducer`, ami eldönti, hogy az érkezett objektum alapján hogyan cselekedjen, milyen változásra van szükség az állapottérben.

```js
// action például:
{
  type: 'counter/increment',
  payload: 1
}

```

Az `action creator` egy függvény, ami visszaad egy action objektumot:

```js
// egyszerű, natív példa egy action creatorra
const increment = () => ({
  type: "counter/increment",
  payload: 1,
});
```

### Middleware

A `middleware` egy köztes réteg az `action` és a `reducer` között, amely lehetőséget ad pl. aszinkron műveletek (API hívások) kezelésére. A Redux Toolkit beépítve tartalmazza a `redux-thunk` middleware-t, ami lehetővé teszi, hogy `action` helyett függvényt dispatch-eljünk.

### Reducer

A `reducer` egy tiszta függvény, ami megkapja a jelenlegi állapotot (`state`) és egy `action`t, majd a beérkezett action objectnek megfelelően "szétswitchel", és visszaad egy új, frissített `state`et. Például:

```js
// egyszerű, natív példa a recurre: ha "increment" típisú action érkezett be, akkor a value értéket megnöveljük,
// by default pedig visszaadjuk az eredeti state-et
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case "counter/increment":
      return { ...state, value: state.value + 1 };
    default:
      return state;
  }
}
```

### Selector

A `selector` egy egyszerű függvény, amely a globális állapotból egy adott részt kér le, miközben:

- elkülíti a megjelenítési logikát az állapotkezeléstől
- hatékonyabbá teszi a komponens újrarendelését

```js
export const selectCounterValue = (state: CounterState) => state.value;
```

### View (Komponensek)

A `View` a React komponenseket jelenti, amelyek megjelenítik az adatokat és reagálnak a felhasználói interakciókra. A Redux állapotát a `useSelector()` segítségével olvassuk, míg a `useDispatch()`-al küldünk `action`-öket.

## Naprakész megoldások RTK-val

No, és akkor nézzük meg, hogyan tudjuk mindezt implementálni az `RTK` segítségével. Megnézzük, mik a követendő lépések, találtok rá példát, hogyan:

1. készítsünk egy `slice`-ot (benne `reducer`-rel és `action`-ökkel)
2. hozzunk létre egy `store`-t
3. kössük be a `store`-t a React alkalmazásba
4. használunk `selector`-okat
5. küldünk be `action`-öket a komponensekből

### Slice létrehozása

```jsx
// features/counter/counterSlice.ts
import { createSlice } from "@reduxjs/toolkit";

// típus
interface CounterState {
  value: number;
  prevValues: number[];
  hasPrimeValue: boolean;
}

// initial state
const initialState: CounterState = {
  value: 0,
  prevValues: [],
  hasPrimeValue: false,
};

// slice létrehozása
const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment(state) {
      state.value += 1;
    },
    decrement(state) {
      state.value -= 1;
    },
    incrementByAmount(state, action) {
      state.value += action.payload;
    },
  },
});

// action creator exportok
// azért ezzel a szintaxissal exportáljuk, hogy külön-külön tudjuk importálni
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// reducer export
export default counterSlice.reducer;
```

Gondoljuk végig, honnan indultunk, és ezzel az implementációval milyen absztrakciós szintre jutottunk el. Míg korábban - akár a teljesen nativ megoldásra gondoltok, akár a később látott `createAction` és `createReducer` megoldásra - külön kezeltük és definiáltuk az `action`-öket, illetve a `reducer`-t, addig most egyetlen helyen, nagyon kényelmes szintaxissal meg tudjuk írni mindkettőt.

A `createSlice` automatikusan legenerálja az `action creator`-okat és a `reducer`-t is a fenti szintaxist követve. Így ötvözve a kettőt nem kell nekünk manuálisan megírni az actionöket, illetve arra sincs szükség, hogy létrehozzunk egy reducert akár egyszerű függvény módjára, akár használva a RTK által kezünkbe adott eszközöket. A fenti kód által kapunk három `action`-t a kezünkbe, olyan, mintha azt mondanának pl, hogy:

```js
{
  type: 'counter/incrementByAmount',
  payload: 5
}
```

Tehát az így kapott `incremenentByAmount` teljesen úgy viselkedik, mintha azt a `creatAction`-nel hoztuk volna létre:

```js
const incrementByAmmount = createAction("counter/incrementByAmount");
```

Illetve a `reducer`-t is megkapjuk a `createSlice`-tól, ami ekvivalens azzal, mintha megírtuk volna az alábbi egyszerű függvényt:

```js
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case "counter/increment":
      return { ...state, value: state.value + 1 };
    case "counter/decrement":
      return { ...state, value: state.value - 1 };
    case "counter/incrementByAmount":
      return { ...state, value: state.value + action.payload };
    default:
      return state;
  }
}
```

Ahhoz, hogy használni tudjuk ezeket az actionöket, fontos, hogy exportáljuk őket, illetve szintén fontos, hogy a reducert-t is exportáljuk, hiszen másképp nem tudjuk majd bekötni a store-ba.

### Store létrehozása

```js
// app/store.ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export const store = configureStore({
  reducer: counterReducer,
});
```

A `configureStore` automatikus beállítja az alap `thunk middleware`-t, a `reducer` alatt pedig egyelőre megadunk egy `counterReducer`-t (tovább olvasva látsz majd példát arra, hogyan tudunk több reduce-t is kezelni itt). Mivel tehát jelenleg csupán egy `slice`-unk van, így erre tekinthettek úgy, mintha kitöltené a teljes `store`-t. Ezzel gyakorlatilag egy globális állapotot hoztunk létre, és egyelőre működés szintjén nem különbözünk attól, mint amikor `slice`-nélkül írtunk egy sima `reducer`-t, és azt adtuk meg a `configureStore`-nak.

![image](https://github.com/user-attachments/assets/64fcc5f6-452e-44be-ab08-8822d0bcb227)

### Store "provide-olása" a React számára

```tsx
// main.tsx
import ReactDOM from "react-dom/client";
import App from "./App";
import { Provider } from "react-redux";
import { store } from "./app/store";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
);
```

A `Provider` komponens teszi elérhetővé a Redux állapotot a komponensfán belül, hasonlóképpen gondolhattok erre, mint amit a `Context` esetén láttunk

### Selectorok írása

Ahhoz, hogy le tudjuk kérdezni az állapottér bizonyos részeit, `selector`okat írunk. Ennek egészen egyszerű a szintaxisa:

```js
// features/counter/counterSlice.ts
export const selectCounterValue = (state) => state.value;
```

Az alap gondolat ezen a ponton az, hogy nem szeretnénk mindig a teljes `state`-et visszaadni, hanem csak egy-egy részét, amire éppen szükségünk van. Amiket ilyen formában definiálunk, `selector function`-nek hívjuk. Egy `selector` function - ahogy azt látjuk a fenti példában - megkapja paraméterül a teljes `state`-et, és visszaadja annak egy részét, itt például a `value`-t. Írhatnánk ennél összetetteb selectorokat is minden probléma nélkül, nem szükséges csupán az adott értéket visszaadni. Teljesen valid megközelítés lenne azt mondani mondjuk, hogy:

```js
export const selectNonNegativeCounterValue = (state) => {
  if (state.value < 0) {
    return 0;
  }
  return state.value;
};
```

Gyakran előfordul, hogy nem csak az állapot egy részére van szükségünk, hanem valamilyen `származtatott értékre` – például kétszerezzük meg egy számot, számoljuk össze egy tömb hosszát, stb.

Ha ezt egy sima selectorral csináljuk, akkor felesleges újraszámolások és ezáltal újrarenderelések történhetnek, még akkor is, ha a számítás bemenete nem változott, csak más részletei az állapotnak. Ez rontja a hatékonyságot. Ha viszonylag komplex számítás van/sokat iterálok, akkor értelemszerűen nem szeretnék mindig újra- és újraszámolni az értéket, akkor is, ha az nem változik, amiből ez az érték származik. Márpedig `createSelector` használata nélkül pontosan ez történt. Segítségével viszont `memorizált selectorokat` hozhatunk létre, amik csak akkor számolják újra az értéket, ha a bemeneti értékek változnak, értsd: ha változik az érték, amiből származtatunk.

`createSelector – Memorizált selectorok`

A `reselect` könyvtár `createSelector` függvényével `memorizált selectorokat` hozhatunk létre. Ez azt jelenti, hogy ha a bemeneti értékek nem változnak, a selector nem számolja újra az eredményt, hanem visszaadja a korábban eltárolt értéket. Ennek a szintaxisa a következő

```js
// features/counter/counterSlice.ts
import { createSelector } from "reselect";

export const selectCounterValue = (state) => state.value;

// memorizált selector
export const selectDoubleCounter = createSelector(
  [selectCounterValue],
  (value) => value * 2,
);
```

Itt a createSelector első paramétere egy tömb, amiben felsoroljuk a `base selectorokat` (egyszerű állapotlekérő függvények, mint például a `selectCounterValue`), a második paraméter pedig egy függvény, ami megkapja ezek kimeneteit, és visszaadja a számított értéket. Gondold végig, mi történik a fenti példában! A megadott `base selector` a `selectCounterValue`, aminek a visszatérési értéke a `value`, mi pedig ebből adjuk vissza a származtott kétszeres értéket.

Kicsit összetettebb szintaxis: a következő példában három értékből szeretnénk származtatni valamit. Ehhez van három `base selector`-unk, az egyik visszaadja `a`-t, a másik `b`-t, a harmadik pedig `c`-t. Ezeket szépen felsoroljuk első paraméterként egy tömbben: `[selectA, selectB, selectC]`. Ezt követően elkérjük ezeknek a visszatérési értékét: `(a, b, c)`, majd pedig visszatérünk a kívánt származtatott értékkel.

```js
const selectA = (state) => state.a;
const selectB = (state) => state.b;
const selectC = (state) => state.c;

export const selectABC = createSelector(
  [selectA, selectB, selectC],
  (a, b, c) => {
    return a + b + c;
  },
);
```

Ez a selector csak akkor fut újra, ha `state.a`, `state.b` vagy `state.c` változik. Fontos, hogy `VAGY`-ot használok és nem `ÉS`-t, hiszen a három közül bármelyik megváltozása új számított értéket eredményez.

Helyezzük el azért itt egyben is, hogyan nézne ki a megírt `counterSlice.ts` a `selector function`-ökkel kiegészítve:

```js
// features/counter/counterSlice.ts
import { createSlice, createSelector } from "@reduxjs/toolkit";

// típus
interface CounterState {
  value: number;
  prevValues: number[];
  hasPrimeValue: boolean;
}

// initial state
const initialState: CounterState = {
  value: 0,
  prevValues: [],
  hasPrimeValue: false,
};

// slice létrehozása
const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment(state) {
      state.value += 1;
    },
    decrement(state) {
      state.value -= 1;
    },
    incrementByAmmount(state, action) {
      state.value += action.payload;
    },
  },
});

// action creator exportok
// azért ezzel a szintaxissal exportáljuk, hogy külön-külön tudjuk importálni
export const { increment, decrement, incrementByAmmount } = counterSlice.actions;

// selectorok
export const selectCounterValue = (state: CounterState) => state.value;
export const selectDoubleCounter = createSelector(
  [selectCounterValue],
  (value) => value * 2,
);

// reducer export
export default counterSlice.reducer;
```

### Használat a komponensekből

A megírt `selector`-okat egyszerűen a `useSelector` hook segítségével tudjuk használni, átadva neki paraméterül magát a selectort. Erre egy egyszerű példa:

```jsx
import { useSelector } from "react-redux";
import { selectCounterValue } from "./counterSelectors";

const CounterValue = () => {
  const count = useSelector(selectCounterValue);

  return <p>Aktuális érték: {count}</p>;
};

export default CounterValue;
```

Itt a következő történik:

- a `useSelector` figyeli a `store`-t, ha változik a `value` értéke, akkor a komponens újrarenderelődik

Ha pedig `action`-t szeretnénk beküldeni a store-ba, ehhez először is szükségünk lesz a `useDispatch` hook használatára, ezzel kezünkbe kapunk egy `dispatch()`-et, ami meghívja az action creatorokat, így megkapva paraméterül a létrehozott `action object`-ez. Az action általa bekerül a store-ba, a reducer pedig megfelelően módosítja az állapotot:

```jsx
import { useDispatch } from "react-redux";
import { increment, decrement } from "./counterSlice";

const CounterButtons = () => {
  const dispatch = useDispatch();

  return (
    <div>
      <button onClick={() => dispatch(decrement())}>–</button>
      <button onClick={() => dispatch(increment())}>+</button>
    </div>
  );
};

export default CounterButtons;
```

## Mi van akkor, ha több slice-unk van?

Értelemszerűen egy nagyobb alkalmazásban több különböző slice-unk is lehet, hiszen ez az egésznek a lényege: Leválasztunk különálló `ALTEREKET` az állapotunkból. Általában `feature`-ök szerint szoktuk ezeket a slice-okat létrehozni, például lehet egy `counterSlice`, egy `todoSlice`, egy `userSlice`, stb. Alkalmazásstruktúrában ezek általában külön mappákban helyezkednek el, például a `features` könyvtárban, ahol minden egyes `feature`-nek van egy saját mappája, amiben benne van a slice, a reducer, a selectorok, stb. Egy valid struktúra lehet például a következő:

```src/
  features/
    counter/
      counterSlice.ts
      // ide tartozó komponensek, stb
    todo/
      todoSlice.ts
      // ide tartozó komponensek, stb
    user/
      userSlice.ts
      // ide tartozó komponensek, stb
  app/
    store.ts
```

Ezen a ponton az érdekes rész nem az, hogyan hozzuk létre ezeket a `slice`-okat, hiszen teljesen ugyanúgy történik minden, mint a fent leírt mintában. Sokkan fontosabb az, hogyan tudjuk az egészet működésre bírni, tehát miként érem el azt, hogy a `store` felismerje, hogy ő milyen `alterekkel` rendelkezik (altér alatt azt értem, hogy leválasztunk különböző részeket az állapotunkból), tudja, hogy amikor egy `action` érkezik, akkor azt melyik `reducer`-nek kell kezelnie. Illetve amikor `selector`-okat használunk, melyik altéren keresztül keresse a lekérendő értékeket.

Első körben a `store` implementációja változik. Míg korábban egyértelműen azt adtuk meg, hogy `reducer: counterReducer`, addig most egy `object`-et adunk meg, amiben felsoroljuk a különböző `slice`-okat, illetve hogy ezek milyen `reducer`-eket tartalmaznak. Ezáltal a `store`-unk tudni fogja, hogy milyen altérrel rendelkezik, és hogy melyik `reducer`-t kell használni egy adott `action` esetén.

```js
// app/store.ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
});
```

Ha emlékeztek, eddig úgy írtuk meg a selector functionöket, hogy a `state`-nek mindig megadtuk a típusát:

```js
const selectCounterValue = (state: CounterState) => state.value;
```

Mi a helyzet ezzel most? Egyrészt ha csak annyit mondanék, hogy `state: CounterState`, most már hibás lenne, ugyanis a `state` jelen implemenációban nem egy `CounterState` típus. Ahhoz, hogy ezt kicsit jobban megértsük, nézzük meg a következő két ábrát.

Az első ábra azt az implementációt mutatja, ahol csak egy `slice`-unk van, és a `store`-unk teljes állapotát ez a `slice`-unk adja. Ezért is tudtuk azt mondani, hogy a `state`-nek a típusa `CounterState`, hiszen a teljes állapotunk egy `CounterState` típusú objektum. És ebből kifolyólag a `state`-en keresztül közvetlenül el tudtuk érni a `value`-t.

A második ábra azt mutatja, amikor több `slice`-unk van, és a fentebb már emlegetett `alterek` hogyan jelennek meg ilyenkor vizuálisan. Láthatjuk, hogy a `store`-nak van egy `counter` altere, amiben a `counter`-hez tartozó állapot van, illetve egy `user` altere, amiben a `user`-hez tartozó állapot van. Tehát a `state` ennek a kettőnek az ötvözete. A legegyszerűbb - és ezt most részletesebb magyarázat nélkül csak bedobom ide a dokumentációból: [Redux](https://react-redux.js.org/using-react-redux/usage-with-typescript) -, ha a store létrehozásakor dinamikusan kinyerjük a `RootState` típusát a `store.getState`-ből:

```js
// app/store.ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
});

// ez működni fog, ha kiegészítjük a store-t egyéb reducer-ekkel, hiszen a RootState mindig a store aktuális állapotát
// fogja visszaadni, így nagyobb további dolgunk itt nincs, majd ezzel az exportált RootState-tel type-oljuk a state-et
export type RootState = ReturnType<typeof store.getState>
```

### Selectorok

Ami még változik ezen kívül, az a selectorok működése. Megint vegyük elő az előbbi ábránkat, hogyan is épül fel a `store`-unk az alterekkel együtt. Míg korábban tehát azt tudtam mondani, hogy:

```js
const selectCounterValue = (state: CounterState) => state.value;
```

Most már nem tudom elérni közvetlenül a `state`-en keresztül a `value`-t, hiszen szétszedtük alterekre. Semmi gond, egyszerűen csak követjük a képen jól látható vonalatat, és nem közvetlenül a `state`-ből kérjük le a `value`-t, hanem először bemegyünk a `counter` altérbe, majd onnan jön a `value`. Ami ezen felül még változik, hogy a `state`-nek a típusa már nem `CounterState`, hanem a `RootState` lesz, amit fentebb exportálunk. Így tehát a selectorunk így fog kinézni:

```js
export const selectCounterValue = (state: RootState) => state.counter.value;
```

> ### 💡 ÖSSZEGZÉS
>
> Összegezve tehát, amikor több `slice`-unk van, lényegében két helyen kell belenyúlni az implementációba. Egyrészt a `store` kicsit máshogy fog kinézni, hiszen egy `object`-et adunk meg a `reducer`-nek, ahol összekötjük a `slice`-okat a hozzájuk kapcsolódó `reducer`-ekkel (FONTOS: figyeljetek a name-ingre, ha a `slice` neve `counter`, akkor a `store`-ban is `counter`-nek kell megadni, amihez hozzákötöd a `reducer`-ét). Másrészt a `selector`-okban is meg kell változtatni a lekérdezés módját, hiszen most már nem közvetlenül a `state`-ből kérjük le az értékeket, hanem először bemegyünk az adott alterbe, majd onnan kérjük le a szükséges értékeket. Nyilván ehhez szükség van arra, hogy a `state`-nek a típusát is megváltoztassuk, hiszen most már nem egy `CounterState`-ről beszélünk, hanem egy `RootState`-ről, ami a teljes állapotunkat reprezentálja. Ez a `RootState` pedig dinamikusan kinyerhető a `store.getState`-ből, így nem kell manuálisan frissíteni, ha újabb slice-okat adunk hozzá a store-hoz.
