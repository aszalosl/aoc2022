
# Advent of Code 2022 in Kona (in Hungarian)

## Bevezetés

Az elmúlt években már  többször próbálkoztam az [Advent of Code](https://adventofcode.com) teljesítésével. A decemberi időszak valóban a lecsendesülés időszaka lehetne, ha nem ekkor lennének a ZHk, pótZHk, beadandók, első vizsgák. S ez nem csak a diákok, hanem a tanárjaik számára is kihívást jelent, hisz illene mindent időre átolvasni, kijavítani. Ennek ellenére 2018-ban sikerült Python-ban, majd 2021-ben Scala-ban teljesíteni a kihívást, igaz másodjára csak pár hetes késéssel. 18-ban a Python mellett próbáltam Racket-ben is megoldani a feladatokat, hogy alaposabban megismerjek egy funkcionális nyelvet, de valójában ez csak 21-ben jött össze a Scala-val. Több évig ott mocorgott bennem, hogy illene egy vektor alapú nyelvet is elsajátítani. Elég sokat töprengtem azon, hogy az [APL](https://aplwiki.com), a [J](https://code.jsoftware.com/wiki/Main_Page), a [K](https://k.miraheze.org/wiki/Main_Page) és a [Q](https://code.kx.com/q/learn/) közül melyikkel lenne érdemes próbálkozni; végül az egyszerűsége miatt nálam a K győzött. Persze ez azt is jelenti, hogy pár esetben majd körülményesebben lehet ugyanazt megfogalmazni, mint más nyelveken lehetne, de ha már eddig is zsákbafutásról volt szó, akkor még egy megszorítás már nem sokat ad hozzá.

Még a nyolcvanas években hozzászoktam a programozási nyelvek nyelvjárásaihoz, a legjellemzőbb a ZX Spectrum és Commodore C64 BASIC-je közti eltérés volt, mert szinte minden egyes programsort át kellett írni, hogy működjön. Aztán a szabványosabbnak tekinthető eszközökkel és programnyelvekkel ez szinte eltűnt, a Turbo Pascal, a C, a Java szabványos, egységes volt, az egyik helyen megírt program minden módosítás nélkül futott a másik gépen is. A visszafele kompatibilitás elvárt tulajdonság szinte mindenütt, van is belőle nagy galiba, ha nem teljesül - lásd a Python 2-3 váltást -, de az sem túlságosan jó, ha ez fékezi az adott nyelvet, mint például a Java esetén.

[Arthur Whitney](https://en.wikipedia.org/wiki/Arthur_Whitney_(computer_scientist)) - a K programozási nyelv atyja - számára ez sose volt szempont, a K különféle verziói tekinthetőek egymás nyelvjárásainak. Ahogy az igények változnak, úgy változott a nyelv is, került bele újabb és újabb eszköz. A rossznyelvek szerint az újabb verziók megírásához egy sort sem használ fel a régi kódokból, mindig mindent a nulláról kezd. Már a kilencedik verziónál tartunk, de mivel ezer dollárokban mérhető összegért lehet hozzáférni az egyes variánsokhoz, eléggé elterjedt a ["generikusok"](https://k.miraheze.org/wiki/Running_K) használata, amikor független személyek az egyes verziókra nagyon hasonlító variánsokat készítenek, jellemzően már nyílt forrással. Ezek közül többet is kipróbáltam, számomra a legegyszerűbb volt a [Kona](http://kona.github.io/#/) variánst működésre bírni (Linux, OSX és Windows alatt is), és ennél tudom leginkább kihasználni a már meglévő [K2 leírásokat](https://k.miraheze.org/wiki/Learning_Resources) illetve [K oktatóanyagokat](https://nsl.com/k/training/).

Miután akadályoztatás miatt nem tudtam időben elkezdeni a kihívást, nem az a célom, hogy karácsonyra önállóan végezzek az összes feladattal, hanem az, hogy ráérezzek a vektor-alapú programozás ízére, megismerjem a Kona lehetőségeit és korlátait, s ha lehet, véges időn belül egyedül oldjam meg a feladatot; s nem titkolt célom az sem, hogy más figyelmét is felhívjam erre a nyelvre, s magyar nyelvű oktatóanyagot biztosítsak számára. Miután elsődleges cél a tanulás/tanítás, emiatt ha minden kötél szakad, akkor ráfanyalodok mások - mint például [ngn/k](https://codeberg.org/ngn/k/src/branch/master/aoc/22) vagy [qbists](https://github.com/qbists/studyq/blob/main/aoc/2022/01.md) - megoldásaira, amelyek persze véletlenül sem Kona-ban íródtak, de utat mutathatnak a saját megoldáshoz, s megmutathatnak pár trükköt, melyekre nem biztos, hogy egyedül rájönnék.

Szeretném, ha mindaz megragadna bennem, amit egyszer sikerült összehoznom, ezért magamnak is leírom, hogy milyen eszközöket használok, azokat milyen néven találom meg a manual-ban, s hogyan jutok előre lépésről lépésre. Ha ez másnak is segít, az meg kész nyereség.

## 1. Calorie Counting

Az input formátuma a következő:

```txt
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

Az input beolvasása igen egyszerűen megy (load text file)  `0:`test01.txt`, majd mivel minden érték egy szám, sorokat konvertáljuk egyből egészekké is (Format)  _0$_. Ezzel csak az a baj, hogy ez az üres sorokkal nem tud mit kezdeni, s ezeknél hibát jelez (0N). Ez az ami pont jó lesz nekünk, ugyanis ezeket tekintjük majd elválasztásnak. Annak érdekében, hogy az első sorunk is megmaradjon, az egész input elé berakunk még egy ilyen jelzést _0N,_ s a könnyebb hivatkozás érdekében ezt tároljuk egy változóban.

```kona
b:0N,0$0:`test01.txt
```

Azt, hogy hol is találhatjuk meg az elválasztókat, az egyenlőséggel (equal) határozzuk meg: _0N=b_, s mivel itt egy értéket hasonlítunk össze egy listával, ennek eredménye egy bitlista lesz, ott kapunk egyest, ahol az egyenlőség teljesül, s mindenütt másutt nullát. Ez kiváló inputja lesz a where (&) operátornak, amely ilyen esetben összegyűjti az egyesek indexeit. A Drop/Cut operátornak listát adva bal argumentumként a _Cut_ műveletet hajtja végre, azaz az inputból származó listánkat allisták listájává alakítja. Ez már majdnem jó is lenne, de az elválasztóink ott találhatóak minden egyes sor elején, amitől azért jó lenne megszabadulni. Ha a Drop/Cut bal argumentuma egy egész (esetünkben 1), akkor a _Drop_ műveletről lesz szó, és így minden egyes allistának első elemétől megszabadulhatunk (Drop Each), de ha már itt vagyunk, az egyes részlistákat összegezzük is (Add Over Each):

```kona
s:+/'1_'(&0N=b) _ b
```

Az első kérdést megválaszolásához nincs más dolgunk, mint kiválasztani a kapott számok közül a maximálist (Max Over) :

```kona
|/s
```

A feladat második részében az allisták összegei közül a három legnagyobbat kell összegezni. Ehhez csökkenő sorrenbe rendezzük az lista elemeit (Grade-down azután pedig Index), majd vesszük az első három elemet (Take 3) és szummázzuk azokat (Plus Over)

```kona
+/3#s[>s]
```

## 2. Rock Paper Scissors

Ahogy a feladat elnevezése is mutatja, a kő-papír-olló játékon alapul a feladat. Az inputban szereplő betűpár alapján kell pontokat számolni, majd ezeket összegezni. A három-három lehetőség miatt kilenc különféle sor jelenhet meg az inputban, akár ezeket is lehetne használni, ám próbáljunk egy kicsit takarékoskodni a memóriával, és a _string_ adattípus helyett a _symbol_-t használni, amely valójában egy dinamikus felsorolásos típus. Az input formátuma a következő:

```txt
A Y
B X
C Z
```

Nincs más dolgunk, mint a sor első és harmadik karakteréből egy sztringet alkotni, majd ezt konvertálni szimbólummá. Erre én egy egyargumentumú függvényt vezetek be, melynek alapértelmezett változója az _x_, és az Index-Item műveletet használja az egyes betűk kinyerésére, majd a Format segítségével konvertálja a kívánt alakra:

```kona
f:{`$x@0 2}
```

Ezek után készítünk egy asszociatív tömböt (dict), mely a korai K verzióknál még a _Make/Unmake_ segítségével történik, s az egymásba ágyazott feltételes utasítások helyett ebben a tömbben tároljuk a feladatspecifikus információkat, az inputhoz tartozó pontszámot. Ezután nincs más dolgunk, mint az input minden egyes sora helyett a neki megfelelő értéket venni, és ezeket összegezni.

```kona
d1: .((`AX;4);(`AY;8);(`AZ;3);(`BX;1);(`BY;5);(`BZ;9);(`CX;7);(`CY;2);(`CZ;6))
+/d1@f'0:`input02.txt
```

A feladat második részében megváltozik az input jelentése. Ezt az asszociatív tömb lecserélésével kezelhetjük le, minden más marad ugyanaz.

```kona
d2: .((`AX;3);(`AY;4);(`AZ;8);(`BX;1);(`BY;5);(`BZ;9);(`CX;2);(`CY;6);(`CZ;7))
+/d2@f'0:`input02.txt
```

## 3. Rucksack Reorganization

Ebben a feladatban az input a következőképpen néz ki:

```txt
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

```kona
f:{l:(#x)%2;a:l # x;b:l _ x;o:_ic *a[&a _lin b];:[96<o;o-96;o-38] }
+/f @' 0:`input03.txt
```

A második esetben nem ketté kell szelni a sort, hanem hármassával csoportosítani. Mielőtt ezt megcsinálnánk, kezdjünk azzal a függvénnyel, amely a három sorból álló listához hozzárendeli a neki rendelt kódot. Az _a_, _b_ és a _c_ rendre az egyes sorokat jelölik (First, Index/At), a _d_ az első két sor "metszetét", míg az _o_ a három sor metszete első karakterének ASCII kódját.

```kona
 f:{a:*x;b:x@1;d:a[&a _lin b];c:x@2;o: _ic *d[&d _lin c];:[96<o;o-96;o-38]}
 ```

Ahhoz, hogy hármassával szét tudjuk szabdalni az inputot, nem ártana tudni, hogy hány sorból áll, ezért az egész inputot beolvassuk az _i_ változóba, majd ebből kiszámoljuk, hogy hány csoportunk is lesz (Count és Divide), majd a szétszabdaláshoz egy aritmetikai sorozatot hozunk létre (Enumerate és Times), és végül a Cut segítségével az input listáját allisták listájává alakítjuk. Erre már alkalmazható a függvényünk (Apply Each) és szummázhatunk (Plus Over) 

```kona
 +/f@'(3*!(#i)%3) _ i:0:`input03.txt
```

## 4. Camp Cleanup

A soron következő feladatban az egyes sorok 2-2 intervallumot tartalmaznak. A feladatunk az egymást átfedő intervallumpárok számát meghatározni. Látjuk, hogy az intervallumot a mínusz jel jelöli, és vesszővel választjuk el a két intervallumot. Először is a négy számot kellene kinyerni. Más esetben a reguláris kifejezés egy kézenfekvő megoldás, de most ezzel nem élhetünk. Viszont a Scan nem dokomentált tulajdonsága, hogy előtte egy karaktert, mögötte egy sztringet szerepeltetve az adott karakter előfordulásainál feldarabolja a sztringet. Most csak azt tart vissza bennünket, hogy második és harmadik számot vessző választja el, nem pedig mínusz jel.

```txt
2-4,6-8
2-3,4-5
5-7,7-9
2-8,3-7
6-6,4-6
2-6,4-8
```

Ezen könnyű segíteni, csak az `_ssr` beépített függvényt kell használni, ahol az első argumentum az eredeti szöveg, a második hogy mit szeretnénk lecserélni, míg a harmadik, hogy mire. A csere után már jöhet a Scan, majd a Format a sztring darabjainak számmá alakítására. Mindezt az `f` függvény tartalmazza. Ha _x0_, _x1_, _x2_, _x3_ jelöli a négy számot, akkor az első intervallum akkor tartalmazza a másodikat, ha _x0<x2_ és _x3<x1_, míg fordítva  _x2<x0_ és _x1<x3_. Ezt egy az egyben le is lehetne fordítani képletre, de gyorsabban végzünk, ha _(x0-x2)(x1-x3)_ szorzatot tekintjük. Ha az negatív (pontosabban nem pozitív), akkor az előbbi két feltétel valamelyike teljesül. Használható lenne a  `x[i]` jelölés is, de ennél rövidebb a `x@i`, sőt a First még rövidebb. Mivel kisebb-egyenlő nem létezik a rendszerben, ezért a nagyobbat kell tagadni (Negate). A `g` függvény tartalmazza ezt a vizsgálatot, s nincs más hátra, mint a beolvasott inputot átengedni előbb az `f`, majd a `g` függvényen, s mivel darabszámra vagyunk kíváncsiak, már csak összegezni kell a megkapott logikai értékeket.

```kona
f:{0$"-"\ _ssr[x;",";"-"]}
g: {~0<((*x)-(x@2))*((x@1)-(x@3))}
+/g'f@'0:`test04.txt
```

A feladat második felében az átfedések számát kell megadni. Ez hasonló az előbbihez, csak az _(x0-x2)(x1-x3)_ szorzat nem pozitívságát kell vizsgálni, amit itt a `h` függvény tartalmaz.

```kona
h: {~0<((*x)-(x@3))*((x@1)-(x@2))}
+/h'f@'0:`test04.txt
```

## 5. Supply Stacks

Ebben a feladatban egy darut kell szimululálnunk. Az input megadja az egyes oszlopok tartalmát, illetve a végrehajtandó feladatsort.

```txt
    [D]    
[N] [C]    
[Z] [M] [P]
 1   2   3 

move 1 from 2 to 1
move 3 from 1 to 3
move 2 from 2 to 1
move 1 from 1 to 2
```

Első dolgunk az állapot beolvasása. Ehhez a következő változókat használjuk: _a_ a teljes input, _b_ az első üres sorig terjedő sorok száma, _c_ az első _b_ sor oszlopaiból kiválogatva a nekünk szükségesek, _d_ pedig ugyanez szóközök nélkül, illetve hogy a nullával kezdődő indexelés ne kavarjon be, amikor a feladat egyessel kezdődően hivatkozik az oszlopokra, még beszúrunk egy virtuális oszlopot a többi elé. A _c_ meghatározásakor minden negyedik karakter-oszlopra szükség lesz, ám az első jó oszlop az egyes indexű. Nincs más hátra, mint megszabadulni a sorkezdő szóközöktől, erre egy külön függvényt csinálunk, mely minden karakterre teszteli, hogy az szóköz-e, s ezeket a Where-t használva kihagyja. Szerencsére a sorban hátrább (oszlopban lentebb) már nincs szóköz, különben kellene a a Scan és az Or is, hogy azokat semlegesítsük.

```kona
a:0:"test05.txt"
b:a?""
c:(+b#a)@1+4*!1+(#*a)%4
d:"0",{a:~x=" ";b:|\a;x@&b}'c
```

A feladat megoldásához szükségünk van a parancssorozatra is. Sajnos ez tele van szemetelve mindenféle szavakkal, pedig nekünk csak a számok kellenének. A példában csak egyjegyű számok szerepelnek, a valós feladatban hosszabbak is, ezért óvatosabban járunk el. Készítünk egy függvényt, mely elsőként meghatározza a szóközök helyét, hasonlóan a _d_ kezdő lépéséhez. Majd a szóközöknél - pontosabban előttük - szétvagdaljuk a sztringet (Cut). Emiatt a _move_ szó ki is esik, így a páratlan sorszámú (páros indexű) elemeit kell megtartani a listának. Miután ezek egy-egy számot, meg egy azt megelőző szóközt tartalmaznak, alkalmazható mindegyikre a Format (parse), mikor is egész számot nyerünk ki belőle. Ezt függvényt a fájl első b+1 sora utáni részére kell alkalmazni, tehát a Drop lesz a barátunk.

Ezek után lényegében szó szerint követni kell az előírt utasításokat. A Take segítségével lemásolunk annyi elemet a megadott indexű oszlopról, amennyit át kell mozgatnunk (ez kerül a _q_ változóba), majd következőleg valóban töröljük is ezeket az elemeket (Drop), s végül - a feladatnak megfelelően fordított sorrendben (Reverse) a céloszlopot kiegészítjük. Annak érdekében, hogy a műveleteink végrehajtódjanak az egyes oszlopokat tartalmazó sztringeket felülírjuk az új értékkel.
A függvény amely ezt megvalósítja két argumentumot használ. Az _x_ lesz az oszlopok listája, míg az _y_ a műveletet jelző számhármas. Érdemes megfigyelni, hogy a függvény utolsó kifejezésnél visszaadja _x_ aktuális értékét. Erre pedig már lehet is alkalmazni a soron következő művelet hármasát. Azt, hogy minden műveletetet sorra végrehajtsunk, az Over dyad és Scan dyad révén érhetjük el.  Ha tesztelni kívánjuk a programunkat és látni akarjuk a köztes eredményeket is, akkor az utóbbit használhatjuk fel: `d f\s`, míg ha csak le akarjuk futtatni, akkor az előbbit:

```kona
s:{0$'((&x = " ") _ x) @ 0 2 4}'(b+1) _ a
f:{q:(*y)#x[y@1];x[y@1]:(*y) _ x[y@1];x[y@2]:(|q),x[y@2];x}
d f/s
```

A második feladatban az a trükk, hogy nem kell a leválasztott részt megfordítani, amikor áthelyezzük, emiatt a kapott függvény kicsit rövidebb lesz.

```kona
g:{q:(*y)#x[y@1];x[y@1]:(*y) _ x[y@1];x[y@2]:q,x[y@2];x}
d g/s
```

## 6. Tuning Trouble

A soron következő feladatnál azt a legelső pozíciót kell meghatározni, amelyet követő 4 karakter mind különböző. Az input csupán egy sor, a mintában így néz ki.

```txt
mjqjpqmgbljsphdztnvjfqwrcgsmlb
```

Az inputnak csak az első sorával kell dolgozni, ezért is használjuk a First-öt. Majd el kellene dönteni, hogy marker-rel van-e dolgunk. A függvényünknek egy paramétere van, hogy melyik karaktertől számoljuk a dolgokat. Első lépésben ennyit eldobunk (Drop), majd vesszük a négy soron következőt (Take), és a Range - más néven _uniq_ - segítségével kiválogatjuk az egyedi karaktereket. Ha pont négyen vannak, akkor nincs ismétlődés. Vegyük az input hosszát, egy eddig növekvő számsorozatot, és ezek mindegyikére alkalmazzuk az előbbi függvényt. (Ha túl sokat vágunk le, akkor nem lesz elég hosszú a maradék, emiatt a függvény hamis értékkel tér vissza, ami nem baj). Megkeressük, hogy melyik indexnél van az első igaz érték, s hozzáadunk még négyet, mert az azt követő karakter a kérdéses.

```kona
a:*0:"input06.txt"
f:{4=#?4#x _ a}
4+(f'!#a)?1
```

A feladat második felében csak a korábbi konstanst kellett átírni másra.

```kona
g:{14=#?14#x _ a}
14+(g'!#a)?1
```

## 7. No Space Left On Device

```txt
$ cd /
$ ls
dir a
14848514 b.txt
8504156 c.dat
dir d
$ cd a
$ ls
dir e
29116 f
2557 g
62596 h.lst
$ cd e
$ ls
584 i
$ cd ..
$ cd ..
$ cd d
$ ls
4060174 j
8033020 d.log
5626152 d.ext
7214296 k
```

A feladat inputja kellően szemét. Elég sok olyan sor van benne, mely csak zavart okoz, ezektől jó lenne megszabadulni. Másrészt szellemesen a könyvtárrendszer közepén fejezi be a sétát, ami a megoldást nem zavarja, de erre figyelemmel kell lenni a program megírásakor.

Kezdjük azzal, beolvassuk az inputot az _a_ változóban, de egyből el is hagyjuk az input első sorát (Drop), mert arra nem lesz szükségünk. Az _fw_ függvény neve a _first word_ rövidítése, és az korábbiak szerint igen egyértelmű a definíciója. Mely sorokat hagyjuk el? Amely sor a _dir_ szóval kezdődik, az most nem hordoz a feladat megoldásához semmi információt, mert utána úgyis lesz egy rá vonatkozó _cd_. Hogyan akadunk rá ezekre a sorokra? Alkalmazzuk az _fw_ függvényt az input összes sorára (Each) - amivel megkapjuk a sorok első szavait, és ezt már csak össze kell hasonlítani a _dir_ szócskával (Match Each-left). Ezzel minden sorra kapunk egy igaz-hamis értéket - megválaszolva azt kérdést, hogy ezzel a szóval kezdődik-e a sor.

Vannak további felesleges soraink, melyekkel körbenéznénk: `$ ls`. Mivel az input sztringek listája, ez pedig itt egy string a String match (`_sm`) szintén logikai értékek sorozatával tér vissza. A két bitsorozatnak venni kell a diszjunkcióját (Or), majd tagadni (Not), hogy pont ott kapjunk egyest, amely sort meg akarjuk őrizni. Ezek után egy korábban már látott trükkel (Where + At) már csak a valóban szükséges soraink maradtak meg.

```kona
a:1_0:"test07.txt"
fw:{(x?" ")#x}
b:a@&~(a _sm "$ ls")|(fw'a)~\:"dir"
```

Annak érdekében, hogy a gyökérkönyvtárban kössünk ki, össze kellene számolni, hogy hányszor lépünk befele, és hányszor kifele. Az utóbbi egyszerű: `cd ..`, viszont az előbbi kicsit komplikáltabb, ezért az összes `cd` parancs számát határozzuk meg, ami nem nehéz, mert csak ilyen parancsaink maradtak, azaz ha az első szó egy karakterből álló sztring - emiatt kell a vesszőt használni (Enlist) - ami pont a dollár prompt, akkor kész is vagyunk. A szerkezet hasonló a _dir_-nél látotthoz, csak miután itt darabszámra vagyunk kíváncsiak, összegezni kell a logikai értékeket (Add Over). A visszalépés is ugyanaz mint az előbb, s itt is összegezni kell. Ezzel a két darabszám a _c_ és _d_ változóba kerül. Már csak a sorok listáját kell megfelelő számú visszalépést tartalmazó sorral:

```kona
c:+/(fw'b)~\:,"$"
d:+/b _sm "$ cd .."
b:b,(c-2*d)#,"$ cd .."
```

A megoldásunkban újra a `xf/y` szerkezetet kívánjuk használni, ahol _x_ tartalmazza az aktuális állapotot, az _y_ pedig az input még fel nem dolgozott részét. Mire is van szükségünk? Mivel kijjebb és beljebb mozgunk a fájlszerkezetben, észben kell tartani, hogy a külső könyvtáraban már mekkora méretet számoltunk már össze, másrészt számolni kell, hogy a feladatban szereplő méret hol tart.

A szóban forgó függvény a Scheme _cond_ feltételes szerkezetét követi. Az egyes feltételeket azok igaz ága követi, esetlegesen egy else ággal - arra az esetre ha egyik feltétel sem teljesül. Lássuk milyen esetekkel kell foglalkoznunk!

+ A sor egy visszalépést tartalmaz. Ha a feladatban megadott méretet nem éri el a listánk első eleme, akkor hozzá kell adnunk a pár második tagjához, egyébként nem. Viszont ennek az alkönyvtárnak a mérete hozzáadódik a tartalmazó könyvtár méretéhez, azaz az első számmal meg kell növelni a másodikat. A listánk - ami a páros első tagja - első elemére hivatkozhatunk a First-First-tel, a második elemére a `x@0@1` helyett az At-Over segítségével hivatkozunk. Újabb feltételes szerkezet használata helyett aritmetikai kifejezést használunk.

+ A második feltétel arról szól, hogy beljebb kerülünk a fájlrendszerben. Ekkor egy nullát szúrunk a listánk elejére.

+ Ha nem _cd_ parancs szerepel a sorban, akkor egy fájlról van szó, melynek a méretével meg kell növelni a lista első elemét

A listánk egy nulla számból áll, míg a számláló megint nulla értékkel indul.  

```kona
f: {:[y~"$ cd ..";(((**x)+(x@/0 1)),2_*x;(x[1] + (**x)*(100001>**x)));(fw@y)~,"$";(0,*x;x[1]);(((0$y)+**x),1 _ *x;x[1])]}
(,0;0) f/ b
```

A második feladatnál azt a könyvtárat kell megtalálni, mellyel épp megfelelő méretű helyet tudunk felszabadítani. Most nem kell növelni a számlálót, hanem feljegyezni minden egyes könyvtár méretét. Ezzel csak az első eset kódja módosult, illetve az, hogy az induló paraméterünk második tagja egy üres lista. Eredményül egy teljes elfoglalt terület és a könyvtárak méreteinek listáját kapjuk meg. Ezt a páros az _e_ változóban mentjük el. Az _l_ a felszabadítantó terület méretét tartalmazza. Ezért tekintjük mindazokat a méreteket, melyek az _l_-nél nagyobbat, majd tekintjük ezek minimumát, s ezzel kész is vagyunk.

```kona
g: {:[y~"$ cd ..";(((**x)+(x@/0 1)),2_*x;(**x),x[1]);(fw@y)~,"$";(0,*x;x[1]);(((0$y)+**x),1 _ *x;x[1])]}
e: (,0;()) g/ b
l: 30000000-(70000000-**e)
&/(e[1][&e[1] >' l])
```

## 8. Treetop Tree House

Ez a feladat kifejezetten kedvez a vektor-nyelveknek, szinte egyértelmű a megoldása. A feladat arról szól - hasonlóan a [skyscraper](https://www.brainbashers.com/skyscrapers.asp) rejtvényhez - hogy hány fa látható a négy égtáj valamelyikéből. Ehhez az kell, hogy magasabb legyen, mint az előtte álló házak. Az input az egyes fák magasságát adja meg egy-egy karakterrel.

```txt
30373
25512
65332
33549
35390
```

Az karakter-egész átalakítást a `_ic` függvény oldja meg, s jól nevelt vektor-nyelvre jellemzően egyből átadhatjuk a teljes inputot, az összes sor minden karaktere konvertálódik egy egésszé. Lusta módon az így kapott ASCII kódokkal is dolgozhatnánk tovább, de jobb kisebb számokra alakítani ezeket. Udvarias lenne 48-at levonni, hogy egyezen a feladatben látott számjegy, meg a belőle származó szám, de van egy szerencsés mellékhatása, ha ezt egy kisebb számmal helyettesítjük, mert így minden kerethez tartozó érték láthatónak minősül. Azon lehetne vitatkozni, hogy a _0_ magasságú fa látható-e vagy sem, de a feladat kitűzője szerint igen, akkor fogadjuk el. A mátrixunk az _m_ nevet kapta, és _s_ jelöli a méretét. Szerencsére négyzetes a feladat, így egy méret elég mindkét irányba.

A megoldás egy trükkön múlik, az _f_ függvényen. Ez két paramétert használ, az `x` egészet, és az `y` mátrixot. Leválasztjuk a mátrix első x sorát (Take), majd oszloponként maximumot számolunk (Max Over), az eredményt összehasonlítjuk a soron következő x+1-dik sorral, s ahol az nagyobb, ott van egy látható fa. A mátrixot rögitjük (parciális függvényt definiálunk), és a leválasztható sorok összes lehetőségére végrehajtjuk ezt a parciális függvényt. Ahogy két vektor összehasonlítása egy bitvektort ad, ezek egymásutánja egy bitmátrixot, ahol ott áll egyes, ahol látható fa van. Az, hogy a legkisebb fa nálunk legalább 1, az első sorra konstans igazat ad.

Ezek után nincs más dolgunk, mint a mátrixunkat megfelelő módon tükrözni, s megkapjuk mind a négy irányt: ha nincs transzformáció, akkor az az északi irány (_a_); ha vizszintesen tükrözzük (Reverse), az a déli irány (_b_), ha transzponáljuk mátrixot (Flip) az a nyugati irány (_c_),  ha pedig előbb trükközzük, majd transzponáljuk, a keleti irányt adja (_d_). Itt vigyázni kell az eredmény visszaalakításával, hogy jó sorrendben történjen!

Ezek után nincs más dolgunk, mint venni a bitmátrixok diszjunkcióját (Or), majd összegezni az egészet (duplán Add Over).

```kona
 m: (_ic 0:"test08.txt")-47
 s:#m
 f:{(|/x#y)<y[x]}
 a:f[;m]'!s
 b:|f[;|m]'!s
 c:+f[;+m]'!s
 d:+|f[;|+m]'!s
 +/(+/a|b|c|d)
 ```

A feladat második részében az a lényegi kérdés, hogy egy-egy irányba meddig láthatunk el. Az adott fánál alacsonyabb fákat mindenképpen látjuk, viszont az azonos magasságú vagy még magasabb fák mögé már nem látunk.
Ha adott az aktuális fa magassága (_a_), valamint adott irányban álló fák magasságai (_b_), akkor az `b<a` egy bitsorozatot ad vissza, s ebben az első hamis érték (0) jelöli azt a fát amit még látunk, de mögé már nem.
A Find megadja ennek az indexét, de mivel az indexelés nullával kezdődik, ezt még eggyel növelnünk kell.
Ha ez a nullás nem létezik, akkor rossz értéket kapnánk, ezért bevetünk egy minimumot (Min), melynek a másik argumentuma a _b_ hossza (Count).
Az _a_ és _b_ meghatározásához ki kell indulnunk egy a mátrix egy sorából, ez lesz a függvény _y_ argumentuma, és a sor egy elemének indexéből (_x_). Az At és Drop használatával elérjük, hogy az index által jelölt elem, valamint a mögötte álló számokat veszik fel a lokális változóink.
A _g_ függvénynek egy indexet és egy sort kell átadni, de az indexnél felsoroljuk az összes lehetőséget (Enumerate Each), majd mindezt egy külső függvénynek tekintve átadjuk a mátrix összes sorát (Each). Ezzel a végeredmény  (_e1_) egy mátrix melynek elemeit a keleti irányba látható fák számai alkotják. Annak érdekében, hogy ugyanezt megkapjuk a nyugati irányra (_e2_), a mátrix sorait kell egyenként megfordítani (Reverse monadic Each). A Reverse-t direkt alkalmazva mátrixra vízszintesen tükrözzük a mátrixot, nem pedig függőlegesen. A transzponálás újra használható, ekkor a déli irányt kapjuk meg (_e3_). Az északi irányhoz a transzponálást és a tükrözést kell kombinálni (_e4_).
Végül a feladat nem kér mást, mint az egyes irányokhoz tartozó számok szorzatának a maximumát. Ehhez nincs másra szükségünk, mint a négy mátrix elemenkénti szorzatára (ami véletlenül sem mátrixszorzat), s két Max-Over segítségével mindkét irányban helyettesítjük a sort/oszlopot a maximális elemével.

```kona
 g:{a:y@x;b:(x+1) _ y; (#b)&1+(a>b)?0}
 e1:{g[;x]' (!s)}' m
 e2:|:'{g[;x]' (!s)}' |:'m
 e3:+{g[;x]' (!s)}' +m
 e4:+|:'{g[;x]' (!s)}' |:'+m
 |/|/e1*e2*e3*e4
```

## 9. Rope Bridge

A feladatban egy kötélke mozgását kell követni. Az input a kötél elejének a mozgását írja le. A kötél vége kis késlekedéssel követi az elejét, és az kérdés, hogy hány pozíciót látogatott meg a kötél vége.

```txt
R 4
U 4
L 3
D 1
R 4
D 1
L 5
R 2
```

A feladat megoldása során jellemzően a már használt műveleteket alkazzuk. Az _i_ és az _n_ a teszt és valódi inputot tartalmazza.
Ahogy fentebb is látható: elől szerepel egy irányt jelző betű, amit egy szám követ. Jó órás debuggolást igényelt, az a tény, hogy ez a szám nem csak egy számjegyből állhat, bár a teszt ilyet nem tartalmaz.
A beolvasáshoz készítünk egy segédfüggvényt, amit nem látunk el névvel, csak egyszerűen felhasználjuk. Ennek az input egy sora lesz az argumentuma, ebből kell a kezdő karaktert kinyerni (First) majd egy szimbólummá kovertálni (Dollar Backtick), az első két karaktert eldobni (Drop), hogy megkapjuk a számot tartalmazó sztringet, ezt számmá konvertálni (Dollar Int), és alkalmazni a szimbólumra Take műveletet, hogy elegendő számban megismételjük. Miután minden betűt kellő számban megismételtünk, jöhet a Join Over, hogy egy listába szervezzük a sok apró listát.

Az _s_ az előjel függvényt jelöli, az itt látható függvény más nyelveken is közkedvelt megvalósítás.

A kötél vége akkor mozog, ha nem szomszédos az elejével. Egy-egy pozíciót két számmal jelölünk, így a _c_ függvénynek négy argumentuma lesz. Ha az eltérés bármely irányban túllépi az egyet - a függvény igaz értéket ad vissza - , akkor már nem érnek össze, s ekkor kell mozdítani a kötél végét. A _d_ függvény megadja az kötél végének valamely irányú koordinátáját, az argumentumai pedig a kötél elejének és végének ugyanezen irányú koordinátái. Az _f_ és _g_ két függény, mely a kötél eleje és vége pozíciójához megadja a végének frissített pozícióit, akár mozog, akár nem. Az _a_ és _b_ két dictionary, s miattuk kell szimbólumokat használni, mert le vagyunk korlátozva ilyen kulcsok használatára.

A _h_ függvény a kötél egy állapotát várja (a négy számmal leírt két pozíciót), illetve egy irányt jelző szimbólumot. Majd ebből előállít egy újabb állapotot. Már korábban is alkalmaztuk a trükköt, mikor a függvény-Over segítségével folyamatosan frissítjük az állapotot, s közben feldolgozzuk a lista minden elemét. Itt nem az kérdés, hogy végül hova kerül végül a kötél, ezért a függvény-Scan variánst alkalmazzuk, mint valamilyen _trace_ parancs a nyolcvanas évekből. Viszont nincs szükségünk minden így megkapott adatra, csak a kötél végének pozíciójára. A _j_ függvényt ezt válogatja ki, sőt a Z80 processzornál látott binary-coded-decimal trükköt alkalmazzuk, a számpárból egy számot készítünk. Mi haszna van ennek? Sajnos a nyelvben nincs halmaz adattípus. Viszont van egy Range művelet, mely kiválogatja a lista egyedi elemeit, s nem marad nekünk más hátra, mint megszámolni, hányas is vannak (Count).

```kona
i:,/{(0$(2 _ x))#`$*x}'(0:"test09.txt")
n:,/{(0$(2 _ x))#`$*x}'(0:"input09.txt")
s:{(x>0)-(x<0)}
c:{[x;y;u;v]1<(_abs x-u)|_abs y-v}
d:{x+s[y-x]}
f:{[x;y;u;v] :[c[x;y;u;v];d[u;x];u]}
g:{[x;y;u;v] :[c[x;y;u;v];d[v;y];v]}
a: .((`R;1);(`L;-1);(`U;0);(`D,0))
b: .((`R;0);(`L;0);(`U;1);(`D,-1))
h: {p:(a@y)+*x; q:(b@y)+x@1; r:x@2; t:x@3; (p;q;f[p;q;r;t];g[p;q;r;t])}
j:{(x@3)+1000*x@2}
#?j' (0;0;0;0) h\ i
```

A feladat második részében hosszabb kötet kell szimulálni, amely 10 olyan darabból áll, melyek mindegyike úgy viselkedik, mint a korábbi kötelecske. Az előbb négy változót használtunk az állapot leírására, ugyanígy dolgozva most 20 változóra lenne szükségünk, tehát jobb, ha átírjuk a korábbi programot, és az egyes koordináták helyett pozíciókat jelölnek a változóink. Az _i_, _s_, _a_ és _b_ definíciója nem változott. A _c_, _d_ és _j_ szerepe ugyanaz, viszont a definíciója más.
A _c_-nek csak két argumentuma van - két pozíció, így ki kell nyerni az egyes koordinátákat, hogy kiszámolhassuk az eltérésüket. Ugyanez a helyzet a _d_ esetén is. Az _e_ függvény is két koordinátát vár - két egymást követő kötéldarabét -, s hátrább lévő pozícióját frissíti. A korábbi _h_ szerepét most az _m_ veszi át, ezzel tudjuk a kötél kezdetét az előírt iránynak megfelően elmozdítani.

A _p_ függvény egy teljes kötelet (pozíciósorozatot) vár, valamint egy irányt. A fejet ennek megfelelően elmozdítja (_q_),
majd a kötél hátralevő részére ettől a fejtől elindítva végrehajtja a frissítést. Az előbb látott `x f\ y` szerkezetet használjuk erre, ahol `x`-nek a kötél `y`-nak az irány felel meg. A Scan Dyad miatt a teljes kötelet kapjuk eredményül.
Emiatt ha eggyel kijjebb újra ezt a szerkezetet használjuk, immár a kiinduló kötéllel, és a mozgássorozattal, akkor minden közbenső állapotot visszakapunk. A _j_ korábban a második - egyben utolsó - pozíciót kódolta. Hosszabb kötél esetén ez nem esik egybe, így érdemes a last (Reverse - First) függvényt felhasználni, s egy számmá kódolni a pozíciót. Ezután a korábban látott módszerrel összeszámoljuk az egyedi pozíciók számát.

```kona
i:,/{(0$(2 _ x))#`$*x}'(0:"test09.txt")
s:{(x>0)-(x<0)}
a: .((`R;1);(`L;-1);(`U;0);(`D,0))
b: .((`R;0);(`L;0);(`U;1);(`D,-1))

c:{1<(_abs x[0]-*y)|_abs x[1]-y[1]}
d:{(y[0]+s[x[0]-*y];y[1]+s[x[1]-y@1])}
e:{ :[c[x;y];d[x;y];y]}
m:{(a[y]+*x; b[y]+x[1])}
p:{q:m[*x;y]; r:q e\ (1 _ x); r}
j:{k:*|x; k[1]+1000*k[0]}
#?j'( ((0;0);(0;0)) p\ i)
#?j'(((0;0);(0;0);(0;0);(0;0);(0;0);(0;0);(0;0);(0;0);(0;0);(0;0)) p\ n)
```

Az utolsó két sorból sejthető, hogy ez a kód megfelelő lesz a feladat mindkét részfeladatának megoldására, csak a kötelet másképp kell kódolni a két esetben.

## 10. Cathode-Ray Tube

A soron következő feladat egy egyszerű szimuláció. Két fajta sor szerepel az inputban, a NOP, illetve egy regiszter módosító utasítás. Az a nehezítés, hogy az előbbi egy ciklus, míg az utóbbi két ciklus alatt hajtódik végre, valamint a regiszter értéke csak az utasítás végrehajtása után módosul, míg a kérdés a ciklus közbeni regiszterértékre kíváncsi.

```txt
noop
addx 3
addx -5
```

Épp ezért az állapotunk egy páros lesz, első eleme a regiszter új értéke, míg a másik fele a végrehajtás alatti érték vagy értékek. Az _f_ függvény argumentumai az állapot és a soron következő sor tartalma lesz. Ebből elég az első betűt figyelni, mert ha az _n_, akkor egy ciklussal kell számolni, és a regiszter értéke nem módosul, azaz a kezdeti értéket kell továbbadni duplán. Ha viszont a regiszter értéke módosul, akkor ki kell vadászni a számot a mnemonik mögül. Akár az is megfelelő lett volna, hogy ekkor az input első öt karakterét eldobjuk, de elsőre kényelmesebb volt a szóközig terjedő rész kukázni, és a maradékot számmá alakítani. Természtesen már megint a `xf\y` szerkezetet használtam, mellyel az állapotváltozásokkal haladva fel tudjuk dolgozni a teljes inputot. Viszont nem lesz szükségünk mindarra, amit kiszámoltunk, az párosaink első tagját eldobhatjuk. Ha valaki figyelmes, akkor látja, hogy bekerült egy felesleges nulladik állapot, de ez pont segít a feladat megoldásában. A regiszter értékének történetét az _a_ listában mentettük el. A kérdésben feltett arithmetikai sorozatot nem nehéz előállítani az Enumerate és szorzás meg összeadás segítségével, és helyben definiált függvénnyel az index és érték szorzatokat könnyedén kiszámíthatjuk, míg az Add-Over pedig gondoskodik a szummázásról.

```kona
i:0:"test10a.txt"
f:{:["n"=*y;(*x;*x);((*x)+0$((1+y?" ") _ y); (*x;*x) ) ]}
a:,/{x[1]}'(1;1) f\ i
+/{x*a[x]} 20+40*!6
```

A feladat második részének a szövege igencsak el lett bonyolítva. Valójában az a kérdés, hogy mikor kerül közel a regiszter értéke egy folyamatosan változó értékhez (elektronsugár helyzete). Ez utóbbit könnyű generálni az Enumerate majd a Mod segítségével. Venni kell az eltérést az _a_ változóban tárolt értékektől, amit a Minus és abszolút érték számítással oldunk meg. A feladatban emlegetett három távolságnak, a -1, 0 és 1 felel meg, tehát csak azok melyeknél kettőnél kisebb értéket kapunk az eltérésre. A megoldás ebben az esetben egy kép, tehát a logikai értéknek megfelelő karakterét válasszuk a stringnek (At), majd az előbb látott módon generáljuk azokat a pontokat, ahol a karaktersorozatunkat szétvagdaljuk.

```kona
(40*!6) _ ".#"@ 2 > _abs (1 _ a) - (!240)!40
```

Nincs más dolgunk, mint hunyorogva kiolvasni a betűket.

```txt
("####.#..#...##.####.###....##.####.####."
 "...#.#.#.....#.#....#..#....#.#.......#."
 "..#..##......#.###..###.....#.###....#.."
 ".#...#.#.....#.#....#..#....#.#.....#..."
 "#....#.#..#..#.#....#..#.#..#.#....#...."
 "####.#..#..##..#....###...##..#....####.")
```

## 11. Monkey in the Middle

### Felütés

Ez a feladat számomra az eddigiek közül a legszemetebbnek tűnik, bár tudom, hogy még nem tartunk a felénél. Nem igazán K kompatibilis, ezért félrelibbentem a függönyt a feladat megoldásánák menetéről, és nem csak egy letisztázott megoldást mutatok be. A feladat inputjában egy pszeudokód szerepel, amit ha nagyon illedelmesek vagyunk, a programunkkal kellene értelmezni. Ezt a lépést hátrább rakom a teendők listájában, és a maradékra összpontosítok. Annak érdekében, hogy a pszeudókódokat értelmezni lehessen felhasználom az Eval műveletet, mely egy sztringben szereplő kódot futtat le - ezért is érdemes interpretert használni -, és minden majomhoz megadok egy-egy ilyen sztringet, melyek a _p_ változóban kapnak helyet. A lista minden eleméből egy függvény generálódik majd, ami megfelel annak, amit a feladat szövege leír, így lehet most a számolásra összpontosítani.

```kona
p: ("{w:(x*19)%3; :[w!23;(3;w);(2;w)]}"; "{w:(x+6)%3; :[w!19;(0;w);(2;w)]}"; "{w:(x*x)%3; :[w!13;(3;w);(1;w)]}"; "{w:(x+3)%3; :[w!17;(1;w);(0;w)]}")
```

### Számolgatunk

Mi a feladatunk egy állapota? Tudnunk kell, hogy mely tárgy mely majomnál van, illetve az adott tárgyhoz milyen szintű aggodalom járul. Ezzel a szinttel azonosítuk is ezt a tárgyat, mert más információra nincs szükségünk. Azaz egy-egy tárgyhoz egy páros tartozik, melynek első tagja legyen a majom sorszáma. A összes tárgyat így párok egy listája írja le. Mely majomhoz kerül legközelebb egy tárgy? Ehhez le kell futtatni a majomhoz tartozó programot - azaz az index a majom sorszáma, a pár első tagja lesz, míg egyetlen paramétere az aggodalom szintje, azaz a pár második tagja. Ezt az _f_ függvény írja le.

Sokkal egyszerűbb lett volna, ha a majmok nem a rangsor szerint dolgoznának - szigorúan egymás után -, hanem párhuzamosan. De mivel ez a feladat, előbb válogassuk ki a listából, hogy mely tárgyak vannak az _i._-dik majomnál. A sorszám lesz az `x` argumentum, maga a lista egy-egy tagja - azaz egy pár - pedig az `y` argumentum. Ha a pár első tagja egyezik a sorszámmal, akkor azzal most dolgoznunk kell. Az _a_ változó egy bitlistát tartalmaz, ahol az igaz jelöli az izgalmas párokat, amelyeket hamarosan feldolgozunk. Az _a_-hoz tartozó kódrészlet kapcsos zárójelén kívül újabb előfordulása van az `x`-nek és `y`-nak, amelyek már mást jelentenek, mint belül. Egy parciális függvényt alkottunk meg, melynek az _i._-dik sorszámát a mostani `y` szolgáltatja, míg az `x` a teljes listát jelöli. Az Each miatt páronként használja fel az előbbi parcinális függvényt. A _b_ változóba már az kerül, hogy mely majomhoz kerültek és aggodalmi szinten az _i_-dik majom által megvizsgált tárgyak, ehhez a Where + At segítségével kiválasztottuk a párokat, és mindre (Each) végrehajtottuk az _f_ függvényt.

Természetesen szükség van a többi majomnál lévő tárgyakra is. Ehhez az _a_ vektort negáljuk (Negate), majd ismét jön a Where-At páros, és a kiválogatott párok a _c_ változóba kerülnek. A függvény visszatérési értéke a _b_ és _c_ "uniója" lesz (Join). Ezzel kész is a _g_ függvény.
Ha az _i_ tárolja a párok listáját, és 4 majomról van szó összesen - ahogy a feladatismertetőben, akkor a   `i g/0 1 2 3` leír egy majom-fordulót.
Miután a feladat kérdése, hogy melyik majom hány tárgyat vizsgált meg, nem csak az egyes fordulók végeredményére vagyunk kíváncsiak, hanem a teljes folyamatra.

Ezért a tesztben szereplő feladat esetén 20 fordulóra lenne szükség, ami _g_ nyolcvan futását igényli. Ám miután nem a végeredmény - mely majomnál mi található a folyamat végén - érdekel minket, elég lesz ennél eggyel kevesebb is, viszont az Over helyett Scan-t kell használni, hogy a közbenső értékekhez is hozzáférjünk. Azt, hogy körbe-körbe megy a szerep a majmok között, az előző feladat megoldásánál látható Enumerate-Mod párossal érjük el, s a párok listájának listáját a _t_ változóba mentjük. A feladat megválaszolásához arra van szükség, hogy adott majom hány tárggyal foglalkozott a folyamat során. Tehát a párok első tagját kell kinyerni. Ezt az Each dupla alkalmazásával érjük el, illetve egy függvénnyel, mely a First-öt használja. Ennek erdményeképpen kapunk egy szép mátrixot. Arra vagyunk kíváncsiak, hogy az első sorában hány nullás, a második sorában hány egyes szerepel, és így szépen tovább. Ehhez készítünk egy vektort, melyben sorra ezek a számok szerepelnek - újfent az Enumerate-Mod párossal -, s jöhet az egyenlőség, persze soronként (Each). Ezzel az egy érték egy lista egyenlőségénél a lista minden elemét összehasonlítja a megadott elemmel, s kapunk egy bitsorozatot. Azt, hogy hány igaz/egyes szerepel a sorban, annak összegzésével tudjuk meg: Add Over Each. Nincs más dolgunk mint minden negyedik értéket összeadni, mert így követik egymást az ugyanahhoz a majomhoz tartozó értékek. Ez kicsit komplikált, ezért trükközünk egy kicsit, a vektorból egy mátrixot készítünk a Reshape segítségével. Ha valamelyik méretet _-1_-nek választjuk, akkor az automatikusan beállítódik. Így elég azt megadni, hogy a mátrixnak legyen 20 sora, utána összegezzük az egymáshoz tartozó - egy oszlopban szereplő számokat - megszokott módon. Ennek eredménye kerül az _u_ változóba.

Nics más dolgunk, mint összeszorozni a két legnagyobb értéket, ehhez sorbarendezzük a számokat (Grade Down+At), majd vesszük a két elsőt ebből a rendezett listából, és vesszük a szorzatukat.

```kona
m:4
f:{ (. p[*x]) x[1]}
i:(0 79;0 98; 1 54; 1 65; 1 75; 1 74; 2 79; 2 60; 2 97; 3 74)
g:{a:{x=*y}[y;]'x; b:f'x@&a; c:x@&~a; b,c}
t: i g\((!-1+m*20)!m)
u:+/(20;-1)#+/'({*x}''t) =' ((!m*20)!m)
*/2#u@>u
```

### Az input értelmezése

Most már foglalkozhatunk a pszeudokódok feldolgozásával is! Egy-egy majomra vonatkozó kód a következőképpen néz ki:

```txt
Monkey 0:
  Starting items: 79, 98
  Operation: new = old * 19
  Test: divisible by 23
    If true: throw to monkey 2
    If false: throw to monkey 3

```

Az input több hasonlót tartalmaz, melyek egy-egy üres sorral vannak elválasztva. Egyszerűbb dolgunk lesz, ha az inputot ezek alapján feldaraboljuk. Az üres sort is beleszámítva 7 sorból áll egy ilyen rész, tehát vesszük a hét többszöröseit a teljes input hosszáig, és ezeknél szétszabdaljuk az inputot (Cut), s ennek eredménye lesz a _d_. Szükségünk volt a tárgyak listájára. Ezt a _j_ függvény segítségével alkotjuk meg. A _k_ változóba kigyűjtjük a majom azonosítóját, amihez szüksége lesz az első sor hosszára, ebből kettőt levonunk - egyet az indexelés kezdete, egyet meg a sorzáró kettőspont miatt -, majd az ezen a pozíción álló számjegyet számmá alakítjuk. A második sorban kettőspontot követően vannak a számok, ezért megkeressük a kettőspont helyét, majd ennél kettővel több karaktert eldobunk a sztring elejéről. A számunkra fontos számok a már vesszővel elválasztva szerepelnek, ami most nem zavaró, s trükkösen újra az Eval-ra hivatkozunk, ez automatikus számlistát generál. Már csak a párokat kell kialakítani, melyhez a Join Each-right kombinációt alkalmazzuk, ha az _l_ valódi listát tartalmaz. Abban a speciális esetben, ha csak egy érték szerepel itt (Atom), akkor az Enlist-et is használni kell, hogy megfelelő szerkezetet kapjunk. Végül a Join-Over Each adja meg a várt párok listáját: `,/j'd`.

```kona
d:(7*!(1+(#c)%7)) _ c:0:"test11.txt"
j:{k: 0$x[0]@-2+#*x; l:.(2+x[1]?":") _ x[1]; :[@l;,(k;l); k,/:l]}
i:,/j'd
m:#d
```

Meg kellene alkotnunk az egyes majmok programját. Szerencsére a szerkezet igen egyszerű, a harmadik sor egy képlettel ér véget, a negyediktől a hatodik pedig egy-egy számmal. A három utolsó sor feldolgozásához bevezetünk egy új függvényt: _e_, amely a második argumentumként megadott sztringből csak az elsőként megadott szám hosszúságú szuffixét hagyja meg. Mehetne mindez a sztring hossza szerinti játszadozással és a Drop segítségével, de a Reverse-Take-Reverse rövidebben leírható, mert csak megfordítjuk a sztringet, leválasztjuk a kellő számú karaktert most már az elejéről, s visszafordítjuk, hogy nagyobb számok esetén is helyes legyen.

A képletben a régi értéket az `old` testesíti meg, ezért egy olyan sztringet rakunk össze, mely egy olyan függvényt generál, melynek a változója pont ez, így például egy ilyen sztring lehet a következő: `{[old]w:(old * 19)%3; :[w! 7;(3;w);(2;w)]}`. Így semmi dolgunk nincs a képlettel, egy az egyben bemásolhatjuk, csak le kell választani a harmadik sorban az egyenlőségjel mögötti részt. Kicsit pontosabban célzunk, nem hagyjuk meg a kezdő szóközöt sem. A következő három sor megfelelő záró karaktereit pedig a megfelelelő helyekre rakjuk be. Fontos figyelni arra, hogy ezeket sztingként kezeljük, ezért nem csináltunk belőlük számokat. Ezek után peding nincs más dolgunk, mint a _d_ összes elemére alkalmazzuk ezt a függvényt.

```kona
e:{|x#|y}
q:{"{[old]w:(", ((2+x[2]?"=") _ x[2]), ")%3; :[w!", e[2;x[3]], ";(", e[1;x[5]], ";w);(", e[1;x[4]], ";w)]}" }
p:q'd
```

### Második rész

A lényegi változás, hogy

+ a húsz kör helyett tízezerrel kell dolgoznunk; ezért jó, hogy nem négyzetes vagy exponenciális bonyolúltságú megoldásunk van,
+ a hárommal történő osztás elmarad; emiatt egyre nagyobb számokkal dolgozunk, melyek azért a több ezernyi lépésben csak túllépnek a programnyelv ábrázolási tartományán.

Matematikusként könnyű dolgom van, a majmok programjaiban lévő tesztek különböző prímszámokat használnak, így ha számokat helyettesítjük e prímek szorzata szerint vett maradékkal, akkor nem lépünk túl az egészek értelmezési tartományán. A tesztfeladat eset `*/23 19 13 17` azaz `96577` a szorzat; míg élesben ugyanez már `*/7 19 5 11 17 13 2 3` azaz `9699690` lesz. Ezt a szorzatot berakjuk az _a_ változóba. Az lenne a szép, ha most erre is írnék egy függvényt, de a korábbi kódrészletek alapján úgy gondolom, hogy már mindenki ki tudja nyerni a negyedik sor végén található számot szám formájában is, onnan meg egyértelmű a függvény elkészítése, elemenkénti alkalmazása _d_-re, s a szorzat alkalmazása. Emiatt nézzük, mi mindennek kell változni a korábbi programban!

+ A hárommal való osztást az _a_ szerinti maradékkal kell kicserélni a _q_ függvényben,
+ bevezetünk egy `r` konstanst, ami a körök számát jelöli, ezzel lehet a megadott értékekhez hasonlítani a saját megoldásunkat, és a korábbi `20` számot mindenütt - azaz a _t_, az _u_ és a _v_ definíciójában - lecseréljük erre, azaz `r`-re.

```kona
q:{"{[old]w:(", ((2+x[2]?"=") _ x[2]), ")!a; :[w!", e[2;x[3]], ";(", e[1;x[5]], ";w);(", e[1;x[4]], ";w)]}" }
p:q'd
r:10000
t: i g\((!-1+m*r)!m)
v:(+/'({*x}''t) =' ((!m*r)!m))
u:+/(r;m)#v
```

Hasonlóan kell eljárni, mint korábban, természetesen ha ötszázszor több sorunk van, akkor negyedmilliószor nagyobb számot várhatunk megoldásként mint az előtt. Szerencsére ezt a 11 jegyű számot a K automatikusan kezeli, nem kell BigInt-re átírni a programunkat.

## 12. Hill Climbing Algorithm

A soron következő feladatban a legrövidebb utat kell megtalálni. Valamilyen formában a [Dijkstra algoritmus](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) minden évben előfordul.
Miután a hivatalos megvalósítás [kupacot](https://en.wikipedia.org/wiki/Heap_(data_structure)) használ, ezt inkább hanyagolom,
ahogy a szélességi keresést is, amely pedig a [sor](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)) használatára épít.
Azért megoldásom nyomokban tartalmazza a [szélességi keresést](https://en.wikipedia.org/wiki/Breadth-first_search), ám nem épít keresőfát, hanem egy mátrixszal helyettesíti,
amivel ugyan elveszítjük annak lehetőségét, hogy megadjuk a pontos utat, de ezt nem is várja el a feladat.
Már az eredeti térkép is egy mátrix formájában kapható meg, így egy hasonló mátrixot készítünk, melyben az egyes pontokhoz a hozzájuk tartozó mélységi számokat tároljuk.
Az inputban egyes betűk az adott pont magasságát jelölik, így `a` jelöli a legalacsonyabb, a `z` a legmagasabb pontot.
Mivel `a` és `z` nem csak egyszer szerepelhet, egy `S` és `E` betű jelöli a start és a cél helyét.

```txt
Sabqponm
abcryxxl
accszExk
acctuvwj
abdefghi
```

Ez a két nagybetű bekavar a beolvasásba, így bevezetünk egy _f_ függvényt, ami a kisbetűket és és ezt a két nagybetűt is magassággá alakítja, felhasználva a betűkhöz tartozó ASCII kódokat.
Miután az input sztringek - azaz karakterlisták - listája, ezért dupla Each segítségével tudunk feldolgozni külön-külön minden karaktert.
Viszont szükségünk van a két nagybetű helyére, ehhez szintén egy függvényt készítünk. A _w_ függvény a Find utasítást használja, s ezt ráeresztjük az input minden sorára. Ha az adott sorban nem található a keresetett karakter - ami igen általános -, akkor az utolsó karakter utáni pozíciót kapjuk vissza. Mivel biztosak lehetünk abban, hogy a keresett karakter valamelyik sorban csak megjelenik, így a visszakapott számok minimuma adja meg a keresett `x` pozíciót. Az `y` pizícióhoz pedig az kell, hogy a keresett minimum hol fordul elő, azaz a minimumra, és a számsorra alkalmazzuk az Equals-t, ami egy bitvektort generál, így a Where visszaadja azt a pozíciót, ahol az egyetlen egyes található.

```kona
f:{:[x=83;0;x=69;26;x-97]}
i:0:"test12.txt"
i:0:"input12.txt"
a:f'' _ic i
w:{s:x?'y; u:&/s; v:&u=s;(u,v)}
ws:w[i;"S"]; we:w[i;"E"]
```

Bevezetünk egy konstanst _m_ névvel, ami egy nagy számot jelöl -  olyan nagyot melyet a lépések számozása nem érhet el - így ezzel tudjuk jelölni, hogy hol nem jártunk még.
Az _a_ méreteit (Shape) követve létrehozunk egy hasonló _d_ mátrixot, kezdetben ezzel a konstanssal feltöltve, majd megváltoztatjuk az `S` helyén, a kiindulópontban egy egyest szerepeltetünk.

```kona
m:1000
d:(^a)#m
d[ws[0];ws@1]:1
```

A feladatnak megfelelően akkor tudunk egy szomszédos mezőre lépni, ha az maximum egy értékkel van magasabban. Összpontosítjuk a figyelmünket egy irányra, modjuk délre.
Minden mezőt az "alatta"/tőle délre lévővel kell összehasonlítani. Ehhez toljuk el az egész mátrixot eggyel feljebb: azaz töröljük az első sorát (Drop), s hogy a méretekkel ne legyen probléma,
majd alul egészítsük ki egy konstansokból álló sorral! Ez utóbbihoz szükség van a mátrix szélességére, azaz egy-egy sorának hosszára. Ezt a méretét (Shape) jelző számpár
második tagja adja meg, azaz itt is eldobhatjuk az elsőt, s a Take segítségével alakíthatunk ki a konstansunkból egy vektort, ám ne feletkezzünk meg az Enlist-ről,
mellyel a vektor mátrixszá minősül át, és már illeszhető (Join) az előbbi csonkhoz. Ebből az eltolt mátrixból kivonva az eredetit megkapjuk az egymás alatt szereplő mezők távolságát,
s ott lehet továbblépni, ahol maximum 1 ez a távolság, magyarul kisebb, mint 2. Ez az összehasonlítás egy bitmátrixot eredményez.

Annak érdekében, hogy ezt a műveletsort ne kelljen megismételni a fennmaradó három irányra a programkódban, ezt a bitmátrix generálást az _n_ függvényben eltároltuk, és ezt alkalmazzuk az eredeti feladatunk _a_ mátrixának különféle tükrözéseire, melyek végül kiadják a négy irányt. Mivel ezeket a bitmátrixokat újra és újra felhasználjuk, elmentjük négy változóba:

```kona
n:{s:2>((1 _ x), ,(1 _ ^x)#m) - x}
ad:n[a]; au:n[|a]; ar:n[+a]; al:n[|+a]
```

Ahol egyes bitek vannak az _ad_ mátrixban - mely a dél felé haladási lehetőségeket tárolja -,  ott a _d_ mátrixban feljegyzett lépésszámnál az alatta lévő mezőben feljegyzésre kerülő eggyel nagyobb lesz - feltéve, ha oda máshonnan nem jutottunk még el. Ha adott mezőből továbbléphetnénk, de a _d_ megfelelő mezője nagy számot tartalmaz, akkor felesleges figyelembe venni. Ha pedig valahonnan nem lehet továbblépni - az _ad_ ott nullást tartalmaz, akkor az a nagy számmal tudjuk jelezni.

A _g_ függvény a következőképpen működik: megkapja az _ad_ és _d_ mátrixokat, veszi ezeknek a pontonkénti szorzatát - tehát ahol az _ad_-ban 1 van, ott _d_ értéke megmarad, ahol pedig _0_, ott nullázzuk, majd _ad_ negáltjának szorzatát _d_-vel amit hozzá is adunk az előbbi szorzathoz, azaz a korábban kapott nullák helyett már nagy számok szerepelnek. Az így kapott mátrix utolsó sorát eldobjuk - pontosabban vesszük az előtte lévő sorokat, s hogy letoljuk ezt a csonkot, még kiegészítjük egy első sorral. Annak érdekében, hogy a léptetés megtörténjen, a csonk elemeit eggyel növeljük az összeillesztés előtt.

```kona
g:{(,(1 _ ^y)#m),1+((*^y)-1)#((x*y)+(m*~x))}
```

Ezzel már tudjuk kezelni a délre történő lépéseket. De ha a _d_ mátrixot tükrözzük ide-oda, akkor a többi irányyal is elboldogulunk. Persze az eredményt illik visszaforgatni. Végül tekintjük a négy irányban megtehető lépéseket, valamint a kiinduló állapotot, és tekintjük ezek minimumát. Ennek mi az értelme? A _d_ az adott pontba eljutás minimálás lépésszámát adja meg. Oda-vissza lépéssel a lépésszám már nagyobb lenne, de a minimum miatt nem írjuk felül. Ha viszont még felfedezetlen területre jutunk, az ott szereplő nagy számot egy kisebbre cseréljük le. Ha valahova több irányból is el lehet jutni, a versenyhelyzet miatt mindig a legkisebb szám fog bekerülni.

```kona
h:{gd:g[ad;x]; gu:|g[au;|x]; gr:+g[ar;+x]; gl:+|g[al;|+x]; x&gd&gu&gl&gr}
```

Korábban már használtuk a `xf/y` és `xf\y` szerkezeteket, ennek van egyargumentumú párja is. Mivel az _a_ mátrixból generált bitmátixok nem változnak, a _h_ függvénynek csak egy argumentuma van, a _d_ mátrix; s egy hasonló, csak egy lépéssel kiegészített mátrixot ad vissza, így arra megint alkalmazható a _h_ függvény, s ez akkor áll meg, ha újra ugyanazt az eredményt kapjuk vissza.
Ha már leállt a folyamat, akkor csak ki kell olvasni az `E` betű helyén álló számot, s mivel egytől indult a számozás, ebből egyet ki kell vonni:

```kona
z:h/d
z[*we;we@1]-1
```

A feladat második felében azt a pozíciót kell megkeresni, amelyhez `a` tartozik, és melyből a legkevesebb lépéssel érhető el `E`. Persze le lehetne futtatni az előbbi megoldást minden egyes olyan pozícióból, melyhez `a` tartozik, de az elég munkás lenne.
Inkább fordítsuk meg a helyzetet, és nézzük meg, hogy az `E`-ből hány lépésből lehet elérni az egyes pozíciókat. Mihez kell ezen változtani. Természetesen más lesz a kezdőpont, nem az _ws_-t hanem a _we_-t kell használni a _d_ módosításához. Az _n_ függvény szabta meg, hogy mikor lehet továbblépni. Ott a feltétel az volt, hogy az eltérés kisebb mint kettő. Ha megfordítjuk a keresés irányát, akkor fordítva végezzük el a kivonást, így módosított függvényben nagyobb mint minusz kettő szerepel helyette. A _z_-be most is begyűjtjük az összes számot. Viszont ezek közül csak az érdekes, melyhez `a` tartozik.

A kódolásban ez a `0` értéket jelenti, tehát az a kérdés, hogy az _a_ vektorban hol szerepel nulla. Ez újra bitmátrixot ad, ezzel beszorozva a _z_ mátrixot, már majdnem kész is vagyunk. Sajnos vannak olyan pozíciók, melyek a szabályok szerint nem elérhetők, közöttük olyan is, melyhez `a` tartozik, itt az _m_ érték található, illetve ott vannak a nullák. A Range segítségével felsorolhatjuk az egyedi értékeket - melyeket elmentünk az _y_ változóba -, de ehhez a mátrixból vektort kell csinálni, amit a Join-Over segítségével érhetünk el. Rendezzük sorba a számokat a megszokott módon, s a nullát követő számot kellene visszaadni, pontosabban ennél eggyel kisebbet, mert továbbra is eggyel indult a számozás.

```kona
d[we[0];we@1]:1
n:{s:-2<((1 _ x), ,(1 _ ^x)#m) - x}
y:?,/(a=0)*z
-1+y[<y]@1
```

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
