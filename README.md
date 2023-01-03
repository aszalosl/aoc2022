
# Advent of Code 2022 in Kona (in Hungarian)

Az elmúlt években már  többször próbálkoztam az [Advent of Code](https://adventofcode.com) teljesítésével. A decemberi időszak valóban a lecsendesülés időszaka lehetne, ha nem ekkor lennének a ZHk, pótZHk, beadandók, első vizsgák. S ez nem csak a diákok, hanem a tanárjaik számára is kihívást jelent, hisz illene mindent időre átolvasni, kijavítani. Ennek ellenére 2018-ban sikerült Python-ban, majd 2021-ben Scala-ban teljesíteni a kihívást, igaz másodjára csak pár hetes késéssel. 18-ban a Python mellett próbáltam Racket-ben is megoldani a feladatokat, hogy alaposabban megismerjek egy funkcionális nyelvet, de valójában ez csak 21-ben jött össze a Scala-val. Több évig ott mocorgott bennem, hogy illene egy vektor alapú nyelvet is elsajátítani. Elég sokat töprengtem azon, hogy az [APL](https://aplwiki.com), a [J](https://code.jsoftware.com/wiki/Main_Page), a [K](https://k.miraheze.org/wiki/Main_Page) és a [Q](https://code.kx.com/q/learn/) közül melyikkel lenne érdemes próbálkozni; végül az egyszerűsége miatt nálam a K győzött. Persze ez azt is jelenti, hogy pár esetben majd körülményesebben lehet ugyanazt megfogalmazni, mint más nyelveken lehetne, de ha már eddig is zsákbafutásról volt szó, akkor még egy megszorítás már nem sokat ad hozzá. 

Még a nyolcvanas években hozzászoktam a programozási nyelvek nyelvjárásaihoz, a legjellemzőbb a ZX Spectrum és Commodore C64 BASIC-je közti eltérés volt, mert szinte minden egyes programsort át kellett írni, hogy működjön. Aztán a szabványosabbnak tekinthető eszközökkel és programnyelvekkel ez szinte eltűnt, a Turbo Pascal, a C, a Java szabványos, egységes volt, az egyik helyen megírt program minden módosítás nélkül futott a másik gépen is. A visszafele kompatibilitás elvárt tulajdonság szinte mindenütt, van is belőle nagy galiba, ha nem teljesül - lásd a Python 2-3 váltást -, de az sem túlságosan jó, ha ez fékezi az adott nyelvet, mint például a Java esetén. 

[Arthur Whitney](https://en.wikipedia.org/wiki/Arthur_Whitney_(computer_scientist)) - a K programozási nyelv atyja - számára ez sose volt szempont, a K különféle verziói tekinthetőek egymás nyelvjárásainak. Ahogy az igények változnak, úgy változott a nyelv is, került bele újabb és újabb eszköz. A rossznyelvek szerint az újabb verziók megírásához egy sort sem használ fel a régi kódokból, mindig mindent a nulláról kezd. Már a kilencedik verziónál tartunk, de mivel ezer dollárokban mérhető összegért lehet hozzáférni az egyes variánsokhoz, eléggé elterjedt a ["generikusok"](https://k.miraheze.org/wiki/Running_K) használata, amikor független személyek az egyes verziókra nagyon hasonlító variánsokat készítenek, jellemzően már nyílt forrással. Ezek közül többet is kipróbáltam, számomra a legegyszerűbb volt a [Kona](http://kona.github.io/#/) variánst működésre bírni (Linux, OSX és Windows alatt is), és ennél tudom leginkább kihasználni a már meglévő [K2 leírásokat](https://k.miraheze.org/wiki/Learning_Resources) illetve [K oktatóanyagokat](https://nsl.com/k/training/).

Miután akadályoztatás miatt nem tudtam időben elkezdeni a kihívást, nem az a célom, hogy karácsonyra önállóan végezzek az összes feladattal, hanem az, hogy ráérezzek a vektor-alapú programozás ízére, megismerjem a Kona lehetőségeit és korlátait, s ha lehet, véges időn belül egyedül oldjam meg a feladatot; s nem titkolt célom az sem, hogy más figyelmét is felhívjam erre a nyelvre, s magyar nyelvű oktatóanyagot biztosítsak számára. Miután elsődleges cél a tanulás/tanítás, emiatt ha minden kötél szakad, akkor ráfanyalodok mások - mint például [ngn/k](https://codeberg.org/ngn/k/src/branch/master/aoc/22) vagy [qbists](https://github.com/qbists/studyq/blob/main/aoc/2022/01.md) - megoldásaira, amelyek persze véletlenül sem Kona-ban íródtak, de utat mutathatnak a saját megoldáshoz, s megmutathatnak pár trükköt, melyekre nem biztos, hogy egyedül rájönnék.

Szeretném, ha mindaz megragadna bennem, amit egyszer sikerült összehoznom, ezért magamnak is leírom, hogy milyen eszközöket használok, azokat milyen néven találom meg a manual-ban, s hogyan jutok előre lépésről lépésre. Ha ez másnak is segít, az meg kész nyereség. 

## 1. Calorie Counting

Az input formátuma a következő:

```		
1000
2000
3000

4000

5000
6000

7000
8000
9000

10000
```		

A feladat nem más, mint az üres sorokkal elválasztott részeket összegezni, majd kiválasztani a maximálist.

Az input beolvasása igen egyszerűen megy (load text file)  _0:`test01.txt_, majd mivel minden érték egy szám, sorokat konvertáljuk egyből egészekké is (format)  _0$_. Ezzel csak az a baj, hogy ez az üres sorokkal nem tud mit kezdeni, s ezeknél hibát jelez (0N). Ez az ami pont jó lesz nekünk, ugyanis ezet tekintjük majd elválasztásnak. Annak érdekében, hogy az első sorunk is megmaradjon, az egész input elé berakunk még egy ilyen jelzést _0N,_ s a könnyebb hivatkozás érdekében ezt tároljuk egy változóban.

```
b:0N,0$0:`test01.txt
```

Azt, hogy hol is találhatjuk meg az elválasztókat, az egyenlőséggel (equal) határozzuk meg: _0N=b_, s mivel itt egy értéket hasonlítunk össze egy listával, ennek eredménye egy bitlista lesz, ott kapunk egyest, ahol az egyenlőség teljesül, s mindenütt másutt nullát. Ez kiváló inputja lesz a where (&) operátornak, amely ilyen esetben összegyűjti az egyesek indexeit. A Drop/Cut operátornak listát adva bal argumentumként a *Cut* műveletet hajtja végre, azaz az inputból származó listánkat allisták listájává alakítja. Ez már majdnem jó is lenne, de az elválasztóink ott találhatóak minden egyes sor elején, amitől azért jó lenne megszabadulni. Ha a Drop/Cut bal argumentuma egy egész (esetünkben 1), akkor a *Drop* műveletről lesz szó, és így minden egyes allistának első elemétől megszabadulhatunk (Drop Each), de ha már itt vagyunk, az egyes részlistákat összegezzük is (Add Over Each):

```
s:+/'1_'(&0N=b) _ b
```
   
Az első kérdést megválaszolásához nincs más dolgunk, mint kiválasztani a kapott számok közül a maximálist (Max Over) :

```
|/s
```
    
A feladat második részében az allisták összegei közül a három legnagyobbat kell összegezni. Ehhez csökkenő sorrenbe rendezzük az lista elemeit (Grade-down azután pedig Index), majd vesszük az első három elemet (Take 3) és szummázzuk azokat (Plus Over)

```    
+/3#s[>s]
```
    
## 2. Rock Paper Scissors
Ahogy a feladat elnevezése is mutatja, a kő-papír-olló játékon alapul a feladat. Az inputban szereplő betűpár alapján kell pontokat számolni, majd ezeket összegezni. A három-három lehetőség miatt kilenc különféle sor jelenhet meg az inputban, akár ezeket is lehetne használni, ám próbáljunk egy kicsit takarékoskodni a memóriával, és a _string_ adattípus helyett a _symbol_-t használni, amely valójában egy dinamikus felsorolásos típus. Az input formátuma a következő: 

```
A Y
B X
C Z
```

Nincs más dolgunk, mint a sor első és harmadik karakteréből egy sztringet alkotni, majd ezt konvertálni szimbólummá. Erre én egy egyargumentumú függvényt vezetek be, melynek alapértelmezett változója az _x_, és az Index-Item műveletet használja az egyes betűk kinyerésére, majd a Format segítségével konvertálja a kívánt alakra:

```
f:{`$x@0 2}
```

Ezek után készítünk egy asszociatív tömböt (dict), mely a korai K verzióknál még a _Make/Unmake_ segítségével történik, s az egymásba ágyazott feltételes utasítások helyett ebben a tömbben tároljuk a feladatspecifikus információkat, az inputhoz tartozó pontszámot. Ezután nincs más dolgunk, mint az input minden egyes sora helyett a neki megfelelő értéket venni, és ezeket összegezni.

```
d1: .((`AX;4);(`AY;8);(`AZ;3);(`BX;1);(`BY;5);(`BZ;9);(`CX;7);(`CY;2);(`CZ;6))
+/d1@f'0:`input02.txt
```

A feladat második részében megváltozik az input jelentése. Ezt az asszociatív tömb lecserélésével kezelhetjük le, minden más marad ugyanaz.

```
d2: .((`AX;3);(`AY;4);(`AZ;8);(`BX;1);(`BY;5);(`BZ;9);(`CX;2);(`CY;6);(`CZ;7))
+/d2@f'0:`input02.txt
```

## 3. Rucksack Reorganization

Ebben a feladatban az input a következőképpen néz ki:
```
vJrwpWtwJgWrhcsFMMfFFhFp
jqHRNqRjqzjGDLGLrsFMfFZSrLrFZsSL
PmmdzqPrVvPwwTWBwg
wMqvLMZHhHMvwLHjbvcjnnSBnvTQFn
ttgJtRGJQctTZtZT
CrZsJsPPZsGzwwsLwLmpwMDw
```

Az első esetben minden sort meg kell felezni, és megkeresni azt a karaktert, amely mindkét félben szerepel, majd ezekhez a karakterekhez tartozó kódokat kell összegezni. 

A megoldáshoz készítünk egy függvényt, mely a sorhoz megadja a közös karakter kódját. K-ban megszokott, hogy a kifejezéseket jobbról-balra olvassuk, ám ha a függvény törzsében több művelet is szerepel, akkor az egyes műveletek balról-jobbra hajtódnak végre. Most ezek a  műveletek lokális változók értékadásai lesznek. _x_ jelöli a függvény egyetlen paraméterét, ami majd az input egyik sora lesz. Az _l_ jelöli a sor hosszának a felét (Count, Divide), az _a_ a sor első felét (Take), a _b_ a sor második felét (Drop), míg a _o_ a feladat kiírása szerinti egyetlen közös karakter ASCII kódját.

A két sztring _metszetét_ a `_lin` beépített függvénnyel lehet kiszámolni. Adott a két sztring mint két karakterlista, és teszteljük, hogy az első karakterei közül melyik fordul elő a másodikban. Ennek megfelelően az eredmény egy bitvektor lesz, melyből ki kell nyerni a közös karaktereket, azaz az első sztring azon betűit, amelyekhez a bitvektorban igaz (1) érték tartozik. Itt a megfelelő indexet a Where szolgáltatja. Mivel a kapott eredmény egy egyelemű lista lesz, ennek az első elemét kell venni (First), majd pedig az ASCII kódját (_ic). A nagybetűk kódjai 65 és 90, míg a kisbetűké 97 és 122 közé esnek, a feladat szerint pedig 27-52 illetve 1-26 közé kell konvertálni, így egy if-then-else szerkezetet használunk, ahol a szögletes zárójelben elől a feltétel szerepel, majd az igaz ághoz tartozó kifejezés, amit a hamis ághoz tartozó követ.  

Nem maradt más dolgunk, mint alkalmazni a függvényt az input minden sorára (Apply-monadic Each), majd összegezni a kódokat (Plus Over) 

```
f:{l:(#x)%2;a:l # x;b:l _ x;o:_ic *a[&a _lin b];:[96<o;o-96;o-38] }
+/f @' 0:`input03.txt
```

A második esetben nem ketté kell szelni a sort, hanem hármassával csoportosítani. Mielőtt ezt megcsinálnánk, kezdjünk azzal a függvénnyel, amely a három sorból álló listához hozzárendeli a neki rendelt kódot. Az _a_, _b_ és a _c_ rendre az egyes sorokat jelölik (First, Index/At), a _d_ az első két sor "metszetét", míg az _o_ a három sor metszete első karakterének ASCII kódját.

```
 f:{a:*x;b:x@1;d:a[&a _lin b];c:x@2;o: _ic *d[&d _lin c];:[96<o;o-96;o-38]}
 ```

Ahhoz, hogy hármassával szét tudjuk szabdalni az inputot, nem ártana tudni, hogy hány sorból áll, ezért az egész inputot beolvassuk az _i_ változóba, majd ebből kiszámoljuk, hogy hány csoportunk is lesz (Count és Divide), majd a szétszabdaláshoz egy aritmetikai sorozatot hozunk létre (Enumerate és Times), és végül a Cut segítségével az input listáját allisták listájává alakítjuk. Erre már alkalmazható a függvényünk (Apply Each) és szummázhatunk (Plus Over) 

```
 +/f@'(3*!(#i)%3) _ i:0:`input03.txt
```

## 4. Camp Cleanup
## 5. Supply Stacks
## 6. Tuning Trouble
## 7. No Space Left On Device
## 8. Treetop Tree House
## 9. Rope Bridge
## 10. Cathode-Ray Tube
## 11. Monkey in the Middle
## 12. Hill Climbing Algorithm
## 13. Distress Signal
## 14. Regolith Reservoir
## 15. Beacon Exclusion Zone
## 16. Proboscidea Volcanium
## 17. Pyroclastic Flow
## 18. Boiling Boulders
## 19. Not Enough Minerals
## 20. Grove Positioning System
## 21. Monkey Math
## 22. Monkey Map
## 23. Unstable Diffusion
## 24. Blizzard Basin
## 25. Full of Hot Air
