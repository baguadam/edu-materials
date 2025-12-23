# PHP alapok, HTTP, kimenetgenerálás
A félév második felében a `szerveroldali webprogramozással` foglalkozunk, ehhez pedig a PHP nyelvet használjuk, úgyhogy bár általában az emberek sztereotípiák alapján a PHP-t nem kifejezetten kedvelik, tegyünk rá egy kísérletet, hátha mi megkedveljük! 

## PHP nyelvi alapok
A PHP egy szerveroldali szkriptnyelv, amit alapvetően webfejlesztésre használunk. A kódot a szerver futtatja, ebből is adódik, hogy innentől kezdve áttérünk a szerveroldali programozásra. Megjelenik a kommunikáció a `kliens` és a `szerver` között, amiben fontos szerepe lesz a `HTTP` protokollnak. Az alap koncepció, hogy mindig a kliens kezdeményez, a szerver pedig válaszol. A kliens általában egy böngésző, de akár lehetne más is, például egy mobilalkalmazás. A kliens küld egy `HTTP kérést` a szervernek, a szerver pedig válaszol egy `HTTP válasszal`, visszaadva a kért erőforrást. 

A PHP hasonlóan a JavaScript-hez, dinamikusan típusos nyelv, tehát nem szükséges a változók típusát előre deklarálni, a típusok kiderülnek futási időben. Viszont van lehetőségünk arra, hogy megköveteljük a típusokat: 
> ### 💡 FONTOS
>
> Tudjuk deklarálni a változók típusát, illetve a függvények paramétereinek és visszatérési értékének típusát is. Ehhez használhatjuk a `declare(strict_types=1);` direktívát a fájl elején, ami bekapcsolja a szigorú típusellenőrzést. Ez azt jelenti, hogy ha egy változónak vagy függvényparaméternek egy adott típust deklarálunk, akkor csak azzal a típussal rendelkező értékeket lehet hozzárendelni vagy átadni neki. Ha eltérő típusú értéket próbálunk hozzárendelni vagy átadni, akkor egy `TypeError` kivétel keletkezik futási időben. Ez segít elkerülni a típushibákat és növeli a kód megbízhatóságát.
>
> ```php
>declare(strict_types=1);
>
>function add(int $a, int $b): int {
>   return $a + $b;
>}
>
>var_dump(add(1, 2));
>var_dump(add(1.5, 2.5)); // Hibát dob
> ```

PHP-ban is találkozhatunk egyszerű típusokkal, ezek: `integer`, `float`, `string`, `boolean`. Ezen kívül vannak összetett típusok is, mint például a `tömbök` (arrays) és az `objektumok` (objects), illetve speciális típusok, ilyen a `NULL`. 

Ha változót szeretnénk létrehozni, azt a `$` jellel kezdjük, majd pedig a változó neve következik. 
```php
$magic_number = 42;
$pi = 3.14;
$name = "John Doe";
$is_active = true;
```

Fontos különbség JS-hez képest, hogy itt a `+` operátort kizárolólag összeadásra haszáljuk, míg a `string konkatenációhoz` a `.` operátor szolgál. Erre nagyon figyeljetek! 

```php
$first_name = "John";
$last_name = "Doe";
$full_name = $first_name . " " . $last_name;
```

Ezen felül találkozhatunk még néhány hasznos operátorral, ilyen például a `??` (null coalescing), ami akkor nagyon jó, ha egy változó értékét szeretnénk lekérdezni, de nem vagyunk benne biztosak, hogy létezik-e. Ha nem létezik, akkor egy alapértelmezett értéket ad vissza.
```php
$username = $_GET['username'] ?? 'guest'; // tehát ha nincs 'username' a GET paraméterek között, akkor 'guest' lesz az értéke
```

Hasznos beépített függvények a `var_dump()` és a `print_r()`, amik segítségével ki tudjuk íratni a változók tartalmát, illetve típusát, ezeket leginkább debugolás során használjuk, ha szeretnénk részletes információt kapni arról, hogy mi is van egy változóban. 

### Tömbök, asszociatív tömbök
Eddig nagyjából megismerkedtünk a legalapabb dolgokkal, úgyhogy nézzük meg azt, hogyan tudunk adatokat tárolni összetettebb formában. Nyilván az egyik legelementálisabb adatszerkezet erre a `tömb`. Tömböt az `array()` függvénnyel vagy a `[]` szintaxissal tudunk létrehozni. 
```php
$fruits = array("apple", "banana", "cherry");
$vegetables = ["carrot", "broccoli", "spinach"];
```

A tömbök indexelése itt is alapértelmezett 0-tól kezdődik, hasonlóan tudok beleindexelni, mint bármelyik normális nyelvben:
```php
echo $fruits[0]; // kiírja: apple
```

Iterálni is tudunk a tömbökön, erre a legcélszerűbb a `foreach` ciklus használata:
```php
foreach ($fruits as $fruit) {
    echo $fruit . "\n";
}

// előfordulhat, hogy szeretném tudni az indexet is az elem értéke mellett, ekkor:
foreach ($fruits as $index => $fruit) {
    echo "Index: " . $index . ", Fruit: " . $fruit . "\n";
}

// FONTOS, hogy ezeket a változókat bárhogy elnevezhetjük, nem kötelező a $index elnevezés, a szintaxis a fontos.
```

Javascriptban találkoztunk az `objektumokkal`, amikor összetettebb struktúrákat hoztunk létre. PHP-ban erre az `asszociatív tömbök` szolgálnak, amik hasonlóan működne, `kulcs-érték` párokat tárolunk bennük, egyedül a szintaxis lesz kicsit más, mint korábban. 
```php
// egyszerű asszociatív tömb létrehozása
$person = [
    "name" => "John Doe",
    "age" => 30,
    "city" => "New York"
]

// nyilván az asszociatív tömbök is lehetnek ennél összetettebbek, például tömböt is tárolhatunk értékként
$student = [
    "name" => "Alice",
    "age" => 22,
    "courses" => ["Math", "Physics", "Chemistry"]
];

// az iterálás viszont hasonlóan működik, mint a sima tömböknél, ugyanúgy foreach a jó megközelítés, és ahogy a fenti példában
// el tudtuk kérni a tömb indexét, itt ennek megfelelően a kulcsot tudjuk lekérdezni.
foreach ($person as $key => $value) {
    echo "Key: " . $key . ", Value: " . $value . "\n";
}
```

### Néhány hasznos beépített függvény
PHP-ban rengeteg beépített függvény található, amik megkönnyítik a fejlesztést. Ezek közül néhány nagyon hasznos lehet a mindennapokban:
- `count($array)`: Visszaadja a tömb elemeinek számát.
- `array_push($array, $value)`: Hozzáad egy elemet a tömb végéhez.
- `array_pop($array)`: Eltávolítja és visszaadja a tömb utolsó elemét.
- `in_array($value, $array)`: Ellenőrzi, hogy egy érték szerepel-e a tömbben.
- `array_keys($array)`: Visszaadja a tömb összes kulcsát egy új tömbben.
- `array_values($array)`: Visszaadja a tömb összes értékét egy új tömbben.
- `trim($string)`: Eltávolítja a string elejéről és végéről a whitespace karaktereket.
- `strtolower($string)`: Átalakítja a stringet kisbetűssé.
- `strtoupper($string)`: Átalakítja a stringet nagybetűssé.

### Osztályok és objektumok
Mint azt egy modern programozási nyelvtől elvárhatjuk, PHP-ban is van lehetőségünk `objektumorientált programozásra` (OOP). Osztályokat, interface-eket hozhatunk létre, műökdik az interface-ek megvalósítása, az öröklődés, és minden egyéb, amit elváránk. Nézzünk erre néhány nyelvi példát, amikket találkozhattatok már a `Storage` osztály esetén: 

Teljesen egyszerűen tudunk osztályt létrehozni, megadni, hogy egy-egy adattagnak mi legyen a típusa, illetve a láthatósága.
```php
class Person {
    public string $name;
    public int $age;

    public function sayHello() {
        echo "Hello, my name is {$this->name}";
    }
}

// Példányosítás
$p = new Person();
$p->name = "Anna";
$p->age = 22;

$p->sayHello();
```

Tudunk az osztályokhoz - értelemszerűen - konstruktort is definiálni, nyilván ez az objektum létrehozásakor fut le. Fontos, hogy amikor osztályon belül hivatkozunk az adattagokra vagy metódusokra, akkor a `$this` kulcsszót kell használnunk.
```php
class User {
    public string $username;
    public string $email;

    public function __construct(string $username, string $email) {
        $this->username = $username;
        $this->email = $email;
    }
}

$u = new User("Bela", "bela@example.com");
```

Tudunk öröklődni is, akár olyan formában is, hogy a leszármazott osztály meghívja a szülő osztály konstruktorát:
```php
class Person {
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

class Student extends Person {
    public function __construct($name, $age, public string $major) {
        parent::__construct($name, $age);
    }
}

$s = new Student("Anna", 21, "Computer Science");
```

Definiálhatunk interface-eket is, amiket aztán az osztályok megvalósítanak: 
```php
interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    public function log(string $message): void {
        file_put_contents("log.txt", $message . PHP_EOL, FILE_APPEND);
    }
}
```

Ezen felül nyilván működik itt is az `abstract` osztályok használata, illetve a metódusok, adattagok lehetnek `static`-ok, itt a példát meghagyom rátok, a szintaxis ugyanaz, mint a legtöbb nyelvben: oda kell írni ezeket a megfelelő helyre. 

## Kimenet generálás 
A PHP egyik legerősebb tulajdonsága az, hogy dinamikus HTML kimenetet képes generálni. A kimenete lesz a HTTP válasz törzse, vagyis amit echozunk / kiírunk → azt látja a böngésző. Alapvetően egy `.php` fájl kétféle tartalmat tartalmazhat: PHP kódot és HTML kódot. A PHP kódot a `<?php ... ?>` tagek közé helyezzük, míg a HTML kódot simán írhatjuk a fájlba. Amikor a szerver feldolgozza a PHP fájlt, akkor a PHP kódot végrehajtja, és az eredményt beilleszti a HTML kódba, majd ezt küldi vissza a kliensnek. Tehát a PHP és a HTML keverése egyáltalán nem baj, ami gondot okozhat, ha ezt rosszul tesszük. 
Erre egy egészen látványos példa, ha a teljes HTML-t csak az `echo` segítségével írjuk ki:
```php
echo "<ul>";
foreach ($errors as $e) {
    echo "<li>$e</li>";
}
echo "</ul>";
```
Egészen olvashatatlan lesz a kódunk így, ráadásul nehezen is karbantartható, ha szeretném kibővíteni, rögtön problémákra ütközök. Továbbá fontos szempont az is, hogy NEM `sablonszerű`, vagyis nem úgy néz ki, mint a valódi HTML (ami a célunk lenne).
Ehelyett sokkal jobb megoldás, ha az `alternatív szintaxist` használjuk, ahol a PHP kódot és a HTML kódot is jól elkülönítjük egymástól. Például:
```php
<?php if (condition): ?>
    <p>HTML blokk</p>
<?php endif ?>
```

> ### 💡 FONTOS
>
> Teljesen hasonlóan műödik a `foreach ... endforeach`, `for ... endfor`, `while ... endwhile` szerkezet is, ezeket bátran használhatjátok, ha szeretnétek a HTML kimenetet tisztábban tartani.

```php
<ul>
<?php foreach ($errors as $error): ?>
    <li><?= $error ?></li>
<?php endforeach; ?>
</ul>
```

> ### 💡 FONTOS
>
> Felmerülhet, hogy mi is a különbség a `<?= ... ?>` és a `<?php echo ... ?>` között. A kettő teljesen ugyanazt csinálja, a `<?= ... ?>` egy rövidebb szintaxis az érték kiíratására. Ez a rövidített forma különösen hasznos, amikor csak egy változó vagy kifejezés értékét szeretnénk kiírni a HTML-be, anélkül, hogy teljes `echo` utasítást írnánk.

Teljesen nyugodt szívvel használhatjuk ezt a fajta szintaxist összetettebb HTML kimenetek generálására is, például: 
```php
<?php
$student = [
    "name" => "Alice",
    "age" => 22,
    "courses" => ["Math", "Physics", "Chemistry"]
];
?>

<h1>Student Information</h1>
<?php foreach ($students as $s): ?>
    <div class="card">
        <h3><?= $s["name"] ?></h3>
        <ul>
        <?php foreach ($s["subjects"] as $sub): ?>
            <li><?= $sub ?></li>
        <?php endforeach ?>
        </ul>
    </div>
<?php endforeach ?>
```

# GET, POST, kommunikáció kliens és szerver között
A webalkalmazások egyik legfontosabb része a kliens és a szerver közötti kommunikáció. A kliens általában egy böngésző, ami HTTP kéréseket küld a szervernek, a szerver pedig válaszol ezekre a kérésekre HTTP válaszokkal. Két leggyakrabban használt HTTP metódus a `GET` és a `POST`, mi is ezekkel ismerkedtünk meg a gyakorlatokon. 

> ### GET és POST
>
> A `GET` és `POST` egyeránt HTTP kérések típusai, metódusok, aminek a segítségével a kliens (általában egy böngésző) adatokat küld a szervernek. A `GET` metódus az adatokat az URL-ben továbbítja, míg a `POST` metódus az adatokat a kérés törzsében (body) küldi el. A kettő közötti fő különbség az, hogy a `GET` kérések általában kisebb mennyiségű adat továbbítására alkalmasak, és az adatok láthatóak az URL-ben, míg a `POST` kérések nagyobb mennyiségű adat továbbítására is alkalmasak, és az adatok nem láthatóak az URL-ben. A `GET` kéréseket általában akkor használjuk, amikor adatokat szeretnénk lekérdezni a szervertől, míg a `POST` kéréseket akkor, amikor adatokat szeretnénk küldeni a szervernek, például egy űrlap beküldésekor.
>
> A PHP-ban a `$_GET` és `$_POST` szuperglobális tömbök segítségével érhetjük el a `GET` és `POST` kérésekben küldött adatokat. Például, ha egy űrlapot `POST` metódussal küldünk el, akkor az űrlap mezőinek értékei a `$_POST` tömbben lesznek elérhetőek. (Szuperglobálisok: speciális változók, amelyek minden helyről elérhetőek a kódban, például függvényekből is.)