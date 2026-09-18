# Matek Gyakorló - Fejlesztői Dokumentáció

Ez a dokumentum a "Matek Gyakorló" webalkalmazás struktúráját, funkcionalitását és technikai megoldásait ismerteti. 

## 1. Architektúra
Az alkalmazás egyetlen HTML fájlból áll (`index.html`), ami megfelel az egyszerű szétosztás (Zero-Configuration Deploy) elvének. Nincsenek külső JS vagy CSS függőségek, így internetkapcsolat nélkül, lokális környezetben is tökéletesen fut (kivételt képez a Google Fonts külső meghívása, de hálózat hiányában a rendszer visszavált a böngésző alapértelmezett betűtípusára).

* **Technológiai Stack:** HTML5, CSS3 (Flexbox/Grid), Vanilla JavaScript (ES6).

## 2. Felhasználói Felület (UI) és Stílusok (CSS)

### 2.1. Dizájn alapelvek
Az alkalmazás gyerekbarát, világos színvilágot használ. Az elemek le vannak kerekítve, a gombokon pedig hover effektek és finom animációk gondoskodnak a reszponzív, interaktív érzetről.

### 2.2. A Nézetek
Az alkalmazás "Füles" (Tab) rendszert használ. Egyszerre csak egy aktív szekció látható a `tab-content active` és `display: block / none` CSS kapcsolókkal.
1. **Gyakorló feladatok (Generátor):** Interaktív vezérlők és a végeredményt megjelenítő A4 lapnézet.
2. **Szorzótábla:** 1-től 10-ig statikusan legenerált vizuális referenciatábla.
3. **Számházak:** A számok bontását szemléltető "házikós" infografika, 1-től 10-ig.

### 2.3. Nyomtatási optimalizálás (@media print)
Mivel a szoftver célja a fizikai papírra történő átvitel, a nyomtatási nézet felülbírálja a normál kijelzést.
* **Margó kontroll:** A `@page { margin: 0; }` paranccsal teljesen kikapcsoljuk a böngészők változó nyomtatási margóit.
* **Vertikális és Horizontális Középre Igazítás:** Az egész `body` és a kinyomtatandó `.tab-content` Flexbox containerként működik (`height: 100vh`, `justify-content: center`, `align-items: center`), ezáltal a tartalom (például a 20 darabos feladatlap) milliméterre pontosan a papír mértani közepére igazodik. Helyettesítő "margóként" a `padding: 1.5cm;` funkcionál.
* **Rácsok összehúzása:** A Szorzótábla és a Számházak esetén a grid oszlopszámot (4, illetve 5) és a betűméreteket specifikusan lekicsinyítettük annak érdekében, hogy minden fixen elférjen egyetlen A4-es (álló) oldalon.
* A navigációs elemek (gombok, fejlécek) a `.no-print` osztállyal elrejtésre kerülnek.

## 3. Háttérlogika (JavaScript)

### 3.1. getProblemsByType(range, type)
Ennek a függvénynek az a feladata, hogy generálja az **összes** lehetséges, szabályos variációt az adott számkörben és műveletben.
* **Összeadás:**
* **Kivonás:** A végeredmény nem lehet negatív (alsósoknak készül).
* **Szorzás & Osztás:** Vagy a hagyományos 1-10 szorzótábla logikája (100-as kör), vagy az eredmény bekorlátozása.

### 3.2. generateProblems()
A főszoftver, ami összeállítja a 20 darabos listát.
1. Kigyűjti a pipákat (kiválasztott műveletek).
2. Megakadályozza, hogy ne legyen pipa (alaphelyzetbe állítja az összeadást).
3. Elosztás (Kvótarendszer): Hogy elkerüljük, hogy az összeadás a puszta darabszáma miatt elnyomja a szorzást, a 20 kérdést matematikai úton elosztja a választott műveletek között
4. Miután kiszedte a véletlenszerű elemeket a halmazokból, a végleges, immár 20 elemű listát újra megkeveri a `shuffleArray` (Fisher-Yates) algoritmussal, így a különböző műveletek vegyesen jelennek meg a papíron.
5. Manipulálja a DOM-ot: A végeredményt HTML tagekbe burkolva betolja a `#math-grid`-be.

### 3.3 Statikus generátorok (Szorzótábla és Számházak)
A `buildMultiplicationTable()` és `buildNumberHouses()` dupla `for` ciklusokkal generálják le a repetitív HTML kódot oldalbetöltéskor (`window.onload`), kihasználva a böngésző erőforrásait ahelyett, hogy fix, hosszú HTML struktúrát hoztunk volna létre.

