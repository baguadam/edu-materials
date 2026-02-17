# Osztályok - JavaScript

> ### 💡 FONTOS
>
> A JavaScript öröklődési modellje prototípus-alapú, nem klasszikus értelemben vett osztály-alapú. Ez azt jelenti, hogy az objektumok nem más osztályokból másolnak viselkedést, hanem egy úgynevezett prototípusláncon (`prototype chain`) keresztül öröklik a tulajdonságokat és metódusokat.
>
> Minden JS objektumnak van egy belső hivatkozása egy másik objektumra, ezt nevezzük prototípusnak. Ha egy property-t vagy metódust nem találunk az adott objektumon, akkor automatikusan továbbkeressük a prototípusán, majd annak prototípusán, és így tovább. Ezt a folyamatot hívjuk `property lookup`-nak a prototype chain mentén.
>

A JS is lehetőséget biztosít objektumoritentált programozásra, így tudunk **osztályokat** definiálni. Viszont fontos, hogy JS-ben az, hogy léterhotunk egy osztályt, csupán egy "szintaktikus cukorka", egy kényelmesebb szintaxis arra, hogy prototípus alapú öröklődést valósítsunk meg. Így JS-ben az osztályok valójából speciális függvények, így a működésük is ennek megfelelően alakul. Ezért is van az, hogy a JS-ben nem csak class deklarációval, hanem egy sima class expressionnel is létrehozhatunk osztályokat:

```js
class FileReader {
  constructor(file) {
    this.file = file;
  }
}

// vagy
const FileReader = class {
  constructor(file) {
    this.file = file;
  }
};
```

Tudunk definiálni itt is gettereket, metódusokat, statikus metódusokat, statikus adattagokat, adattagokat default értékkel. Ezekre néhány példa:

```js
class Person {
  // A fieldek lényegében object property-k, így nem használunk pl const meg hasonló keywordoket velük
  static welcomeMessage = "";

  age; // Default value nélkül by default undefined
  name;

  // Fieldek esetén nem használjuk a private vagy a public keywordoket, viszont tudunk private fieldeket létrehozni,
  // ezeket a # jelöli:
  #sex; // private fieldeket mindenképp előre kell deklarálni

  constructor(age, name) {
    this.age = age;
    this.name = name;
  }

  static printMessage(message) {
    console.log(message);
  }
}
```

A JS támogatja az osztályok közötti öröklődést az `extends` kulcsszó segítségével, illetve a szülő osztály konstruktorának meghívását a `super()` függvényhívással. A szintaxis hasonlít más objektumorientált nyelvekhez, azonban a háttérben itt is a prototípuslánc valósítja meg az öröklést.

Fontos szabály, hogy származtatott osztály esetén, ha saját konstruktort definiálunk, akkor abban MUSZÁJ meghívni a `super()`-t, mielőtt `this`-re hivatkoznánk/példánytagokat használnánk.

```js
class Printer {
  constructor(fileSource) {
    this.fileSource = fileSource;
  }

  print() {
    console.log(`Printing a Printerből: ${this.fileSource}`);
  }
}

class DatabasePrinter extends Printer {
  constructor(fileSource) {
    super(fileSource);
  }

  print() {
    console.log(`Printing a DatabasePrinterből: ${this.fileSource}`);
  }
}

const printer = new DatabasePrinter("HM");
printer.print(); // "Printing a DatabasePrinterből: HM"
```
