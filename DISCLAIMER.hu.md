# Jogi, biztonsági és kutatási nyilatkozat

## Kutatási és oktatási cél

Ez a repository saját ellenőrzés alatt álló hardveren végzett független műszaki vizsgálatot dokumentál.

Az anyag célja:

- műszaki kutatás;
- firmware-elemzés;
- hardvermegőrzés;
- dokumentáció;
- reprodukálhatóság;
- oktatás;
- valamint a kutató ellenőrzése alatt álló hardver jogszerű javítása.

Az itt található anyag nem tekintendő hivatalos Fujitsu javítási eljárásnak, gyártói szervizkézikönyvnek, hitelesített mérnöki dokumentációnak vagy garantált helyreállítási módszernek.

---

## Független projekt

Ez a projekt független, és nem áll kapcsolatban, nem rendelkezik jóváhagyással, támogatással vagy szponzorációval többek között az alábbi szervezetektől és gyártóktól:

- Fujitsu;
- Kontron;
- Realtek;
- Macronix;
- WCH;
- AMI;
- valamint bármely más, ebben a repositoryban hivatkozott gyártó, szabványügyi szervezet, szoftverprojekt vagy közösség.

A terméknevek, védjegyek, modellnevek, szoftvernevek és vállalatnevek a megfelelő jogosultak tulajdonát képezik.

Ezek kizárólag műszaki azonosítás, dokumentáció, forráshivatkozás és kutatási célból szerepelnek.

---

## A kísérleti vizsgálat hatóköre

A repositoryban dokumentált eredmények elsősorban egyetlen, Fujitsu D3313-E1 alaplappal szerelt Fujitsu FUTRO S930 rendszer vizsgálatából származnak.

Eltérő, kifejezett megjelölés hiányában az ezen a gépen tett megfigyelések nem általánosíthatók automatikusan:

- minden D3313-E1 alaplapra;
- minden FUTRO S930 rendszerre;
- más D3313 variánsokra;
- más Fujitsu rendszerekre;
- más Realtek vezérlőrevíziókra;
- más firmware-verziókra.

A hardverrevíziók, firmware-verziók, gyártási konfigurációk, alkatrészhelyettesítések és futásidejű állapotok lényegesen módosíthatják a rendszer viselkedését.

---

## Bizonyítéki kategóriák

A projekt szükség szerint elkülöníti az alábbi kategóriákat:

```text
[OFFICIAL]
[OBSERVED]
[INFERRED]
[OPEN QUESTION]
```

### `[OFFICIAL]`

Gyártói dokumentációból, hivatalos firmware-csomagból, szabványból, upstream szoftverdokumentációból vagy más elsődleges forrásból származó információ.

### `[OBSERVED]`

A vizsgált hardveren közvetlenül mért, kiolvasott, naplózott, kinyert, reprodukált vagy ellenőrzött információ.

### `[INFERRED]`

Több megfigyelés által erősen alátámasztott következtetés, amely azonban önmagában nem közvetlen mérési eredmény.

### `[OPEN QUESTION]`

További bizonyítást, vizsgálatot vagy független reprodukciót igénylő hipotézis, mechanizmus vagy jelenség.

A kategóriák célja, hogy az értelmezés ne jelenjen meg közvetlen mérési tényként.

---

## Garancia kizárása

A repositoryban található minden információ, eljárás, parancs, offset, firmware-megfigyelés, szkript, ábra, konfigurációs példa, kutatási jegyzet és egyéb anyag:

> **adott állapotában, bármilyen kifejezett vagy hallgatólagos garancia nélkül**

kerül közzétételre.

Nem garantálható, hogy:

- egy dokumentált eljárás másik rendszeren is működni fog;
- egy másik D3313 alaplap ugyanezt a firmware-struktúrát használja;
- ugyanazok az offsetek más firmware-verziókban is érvényesek;
- a recovery eljárás sikeres lesz;
- egy másik flash chip azonos módon viselkedik;
- a hardver működőképes marad;
- a gépspecifikus adatok megmaradnak;
- a helyreállítási kísérlet visszafordítható lesz;
- vagy az itt dokumentált módszerekkel helyreállított rendszer alkalmas lesz éles, üzemi használatra.

---

## Maradandó hardverkárosodás kockázata

Firmware módosítása és alaplapi beavatkozás végleges hardverkárosodást okozhat.

Lehetséges következmények többek között:

- bootképtelen rendszer;
- sérült SPI-firmware;
- sérült flash-tároló;
- sérült alaplapi alkatrész;
- sérült tápellátási áramkör;
- sérült perifériavezérlő;
- firmware-konfiguráció elvesztése;
- alaplapspecifikus adatok elvesztése;
- recovery lehetőség elvesztése.

Helytelen feszültség, bekötés, programozókonfiguráció, csipeszorientáció, flash-chip kiválasztás vagy írási művelet véglegesen károsíthatja az eszközt.

A hasonló munkát végző személy felelőssége saját hardvere elektromos és firmware-jellemzőinek megértése.

---

## Adat- és gépazonosság elvesztésének kockázata

A firmware-tárolók olyan gépspecifikus információkat tartalmazhatnak, amelyek nem találhatók meg a generikus gyártói firmware-image-ekben.

Ilyenek lehetnek például:

- sorozatszámok;
- UUID-k;
- PC ID-k;
- MAC-címek;
- OEM/manufacturing adatok;
- licencinformációk;
- platformspecifikus konfiguráció;
- kalibrációs értékek;
- kriptográfiai vagy biztonsági jellegű adatok;
- más eszközspecifikus metaadatok.

Egy generikus firmware-image visszaállíthatja a bootképességet anélkül, hogy helyreállítaná egy konkrét gép teljes azonosságát vagy konfigurációját.

Ezért a repository nyomatékosan javasolja az eredeti firmware ellenőrzött példányainak megőrzését minden módosítás előtt.

---

## Mentési módszertan

Egyetlen látszólag sikeres firmware-kiolvasás nem tekinthető automatikusan megbízható biztonsági mentésnek.

A biztonságosabb munkafolyamat lehetőség szerint:

```text
OLVASD KI
  ↓
OLVASD KI ÚJRA
  ↓
HASONLÍTSD ÖSSZE
  ↓
HASH-ELD
  ↓
ARCHIVÁLD
  ↓
ELEMEZD
  ↓
CSAK EZUTÁN ÍRJ
```

A független kiolvasásokat össze kell hasonlítani és kriptográfiai hash-sel ellenőrizni.

Az eredeti dumpokat változatlanul kell megőrizni.

A módosított firmware-image-eket külön kell tárolni az eredeti bizonyítékoktól.

---

## Visszafordíthatatlan és korlátozottan írható tárolók

Különös óvatosság szükséges az alábbiak esetében:

- eFuse;
- OTP memória;
- egyszer programozható konfiguráció;
- security fuse;
- hardverazonosságot tároló fuse;
- egyéb korlátozottan írható vagy visszafordíthatatlan tároló.

Az ilyen tárolók programozása végleges lehet.

Az itt dokumentált vizsgálat során tudatosan elkerültük a Realtek eFuse programozását, miután a bizonyítékok arra utaltak, hogy ez nem az eredeti Fujitsu platformmechanizmus helyreállítását jelentette volna.

Visszafordíthatatlan tároló módosítása előtt tisztázni kell:

- pontosan milyen adat kerül beírásra;
- hogyan működik elektromosan az adott tároló;
- visszafordítható-e a művelet;
- az eredeti platform valóban ezt a tárolási mechanizmust használta-e;
- létezik-e kevésbé invazív helyreállítási lehetőség.

---

## A minimális beavatkozás elve

A projekt általános módszertani elve:

> **Csak azt a legkisebb firmware-régiót módosítsuk, amelyről a bizonyítékok alapján kimutatható, hogy módosításra szorul.**

Ez a megközelítés csökkenteni kívánja az olyan, a konkrét hibához nem feltétlenül kapcsolódó firmware-állapotok szükségtelen módosítását, mint:

- futásidejű NVRAM;
- bootváltozók;
- recovery-állapot;
- manufacturing adatok;
- számlálók;
- ismeretlen firmware-struktúrák.

Az elv csökkenti a szükségtelen beavatkozás mértékét, de nem szünteti meg a kockázatot.

---

## Adatvédelem és gépspecifikus információk

A nyers firmware-image-eket potenciálisan érzékeny műszaki bizonyítékként kell kezelni.

Tartalmazhatnak például:

- sorozatszámokat;
- MAC-címeket;
- UUID-ket;
- PC ID-ket;
- manufacturing azonosítókat;
- HDCP-hoz kapcsolódó információkat;
- licencadatokat;
- platformkonfigurációt;
- biztonsági jellegű metaadatokat;
- ismeretlen gépspecifikus struktúrákat.

Ezért a vizsgált rendszer személyre szabott nyers firmware-image-eit a repository szándékosan nem terjeszti nyilvánosan.

---

## Kriptográfiai hash-ek és proveniencia

Kriptográfiai hash-ek közzétehetők:

- proveniencia;
- reprodukálhatóság;
- artefaktumazonosítás;
- későbbi összehasonlítás

céljából.

A kriptográfiai digest lehetővé teszi annak megállapítását, hogy egy másik bináris fájl bitre pontosan azonos-e egy kutatási artefaktummal, anélkül hogy magát az artefaktumot közzé kellene tenni.

A hash közzététele nem jelenti az alapjául szolgáló firmware-image közzétételét.

---

## Képernyőképek, fényképek, logok és média

Képek, képernyőképek, felvételek, naplók és terminálkimenetek is tartalmazhatnak azonosításra alkalmas információkat.

Példák:

- felhasználónevek;
- helyi fájlútvonalak;
- IP-címek;
- MAC-címek;
- sorozatszámok;
- UUID-k;
- eszközfeliratok;
- matricák;
- QR-kódok;
- vonalkódok;
- EXIF metaadatok;
- GPS metaadatok;
- háttérben látható dokumentumok vagy tárgyak.

Ezért minden nyilvánosságra kerülő bizonyítékot publikálás előtt ellenőrizni és szükség esetén anonimizálni kell.

A nyers médiafájlokat nem szabad automatikusan nyilvános terjesztésre biztonságosnak tekinteni.

---

## Véletlen adatközlés

Gondos ellenőrzés mellett is előfordulhat gépspecifikus vagy személyes információ véletlen közzététele.

Ilyen eset észlelésekor felelős értesítés javasolt annak érdekében, hogy a repository javítható legyen.

Mivel a Git verziótörténetet őriz, a nyilvános repositoryk pedig klónozhatók, forkolhatók, archiválhatók és indexelhetők, egy adat eltávolítása az aktuális repositoryból nem garantálja, hogy a korábban közzétett információ minden példányból eltűnik.

---

## Harmadik féltől származó szoftverek és eszközök

A projekt többek között az alábbi külső szoftverekre és eszközökre hivatkozik:

- flashrom;
- UEFITool;
- IMSProg;
- Linux;
- systemd;
- Realtek diagnosztikai segédprogramok;
- Realtek PGTool-kompatibilis eszközök;
- közösségi fejlesztésű segédprogramok.

Ezek a projektek saját:

- szerzői jogaikat;
- licenceiket;
- dokumentációjukat;
- garanciális feltételeiket;
- biztonsági szabályaikat

megtartják.

Az itt történő hivatkozás vagy használat nem jelent tulajdonjogot, gyártói jóváhagyást vagy automatikus továbbterjesztési jogosultságot.

A hiteles licenc- és használati feltételekért mindig az eredeti projekt vagy gyártó dokumentációját kell figyelembe venni.

---

## Harmadik féltől származó firmware és zárt anyagok

A repository nem kíván zárt, gyártói firmware-t újraterjeszteni, kivéve ha annak továbbterjesztése kifejezetten engedélyezett.

A hivatalos firmware-csomagokra kizárólag:

- azonosítás;
- összehasonlítás;
- kutatási kontextus;
- reprodukálhatóság

céljából történik hivatkozás.

A vizsgált rendszer gépspecifikus firmware-dumpjai kutatási bizonyítéknak minősülnek, és alapértelmezés szerint nem kerülnek nyilvános terjesztésre.

A felhasználó felelőssége, hogy a firmware-eket és szoftvereket jogszerű forrásból szerezze be és betartsa a vonatkozó licencfeltételeket.

---

## Szerzői jog és forrásmegjelölés

A projekt nem állítja sajátjának mások által létrehozott:

- technikákat;
- segédprogramokat;
- firmware-formátumokat;
- szabványokat;
- korábbi kutatásokat.

A releváns korábbi munkák, gyártói dokumentációk, szoftverprojektek és közösségi kutatások a:

```text
SOURCES.md
```

fájlban szerepelnek megfelelő forrásmegjelöléssel.

A projekt saját hozzájárulása ezen módszerek dokumentált alkalmazása, kombinációja, ellenőrzése és értelmezése a vizsgált D3313-E1 esetben.

---

## Felelős felhasználás

Az itt dokumentált eljárások olyan hardverre vonatkoznak, amelyet:

- a kutató birtokol;
- a kutató ellenőriz;
- vagy amelyet jogszerűen bíztak rá vizsgálatra vagy javításra.

A repository semmilyen módon nem értelmezhető felhatalmazásként:

- engedély nélküli rendszerbeavatkozásra;
- harmadik fél hardverének megzavarására;
- hozzáférés-védelem jogosulatlan megkerülésére;
- bizalmas információ engedély nélküli megszerzésére;
- hardverazonosság megtévesztő vagy jogellenes módosítására.

A projekt jogszerű firmware-helyreállítással, hardvermegőrzéssel és műszaki elemzéssel foglalkozik.

---

## Biztonsági kutatási kontextus

A firmware-kutatási módszerek részben átfedhetnek számítógép-biztonsági technikákkal.

Ezek szerepeltetésének célja:

- védekező kutatás;
- javítás;
- hardvermegőrzés;
- interoperabilitás;
- diagnosztika;
- platformműködés megértése.

Minden felhasználó saját felelőssége, hogy tevékenysége megfeleljen a vonatkozó jogszabályoknak, szerződéses feltételeknek és engedélyezési határoknak.

---

## Nem minősül biztonsági tanúsításnak

Az itt dokumentált eljárások nem rendelkeznek biztonsági tanúsítással.

A repository nem igazolja:

- az elektromos biztonságot;
- az elektromágneses megfelelőséget;
- az üzemi alkalmazhatóságot;
- a jogszabályi megfelelést;
- a hardver hosszú távú megbízhatóságát.

Különös körültekintés szükséges:

- feszültség alatt álló alaplapok;
- külső programozók;
- szabadon hozzáférhető vezetők;
- módosított tápegységek;
- háttértárak;
- forrasztó- és hőlégfúvó berendezések

használatakor.

---

## Nem minősül mérnöki hitelesítésnek

Ez a repository műszaki kutatási dokumentáció.

Nem tekinthető:

- hivatalos mérnöki tanúsításnak;
- gyártói szervizengedélynek;
- hivatalos javítási dokumentációnak;
- biztonsági minősítésnek;
- garanciális útmutatásnak;
- jogi tanácsnak.

Szabályozott, biztonságkritikus, kereskedelmi vagy üzemi rendszereknél független szakértői felülvizsgálatra lehet szükség.

---

## Ellentmondó kísérleti eredmények

Ha egy későbbi vizsgálat ellentmond a repository valamely következtetésének, az önmagában nem jelenti azt, hogy a reprodukciót végző személy hibázott.

Lehetséges magyarázatok többek között:

- hardverrevíziók közötti eltérés;
- firmware-verziók közötti eltérés;
- gyártási variáció;
- nem dokumentált konfiguráció;
- alkatrészhelyettesítés;
- a jelenlegi értelmezés hibája.

Az ellentmondó eredmények értékes kutatási bizonyítéknak számítanak.

---

## Javítások és független reprodukció

A projekt szívesen fogad:

- műszaki javításokat;
- független reprodukciókat;
- negatív eredményeket;
- ellentmondó megfigyeléseket;
- új bizonyítékokat.

Ha új bizonyíték megváltoztat egy korábbi értelmezést, a repository frissíthető.

A Git verziótörténetének célja, hogy megőrizze a kutatási dokumentáció fejlődését.

---

## A következtetések hatóköre

A repository legerősebb következtetései a konkrétan vizsgált D3313-E1 rendszerre vonatkoznak mindaddig, amíg további reprodukció másképp nem bizonyítja.

A vizsgált gép esetében a bizonyítékok többek között az alábbiakra vonatkoznak:

- az eredeti SPI-image-re;
- az OEM-régióra;
- a Realtek eFuse állapotára;
- a recovery utáni firmware-re;
- a szelektív helyreállítás eredményére.

Ezeket a megfigyeléseket nem szabad automatikusan a teljes termékcsaládra általánosítani.

---

## Külső hivatkozások

A repository hivatkozásokat tartalmazhat:

- gyártói dokumentációkra;
- szabványokra;
- szoftverprojektekre;
- műszaki dokumentációkra;
- közösségi fórumokra;
- korábbi kutatásokra.

A külső tartalmak idővel módosulhatnak, áthelyezhetők vagy megszűnhetnek.

Egy link szerepeltetése nem jelenti a hivatkozott oldal minden állításának jóváhagyását.

A közösségi forrásokat a projekt tudatosan elkülöníti a gyártói dokumentációtól és a saját közvetlen kísérleti bizonyítékoktól.

---

## A repository jövőbeli bővítése

A repository később további tartalmakkal bővülhet, például:

- képernyőképekkel;
- diagramokkal;
- anonimizált bináris részletekkel;
- szkriptekkel;
- reprodukálhatósági eszközökkel;
- recovery dokumentációval;
- kutatási jegyzetekkel;
- független reprodukciókkal.

A jövőbeni tartalmakra ugyanazok az adatvédelmi, forrásmegjelölési, bizonyítéki és biztonsági alapelvek vonatkoznak.

---

## Kapcsolat

Műszaki javításokkal, reprodukciós eredményekkel vagy véletlenül közzétett érzékeny információ felelős jelzésével kapcsolatban:

**Bencze Péter**  
**E-mail:** bencze.peter86@proton.me

---

## Záró figyelmeztetés

Az, hogy a vizsgált rendszeren sikeres helyreállítás történt, nem jelenti azt, hogy az eljárás általánosan biztonságos.

A munka reprodukálása előtt meg kell érteni a konkrét hardvert, egymástól függetlenül ellenőrzött mentéseket kell készíteni, és minden firmware-írást potenciálisan destruktív műveletként kell kezelni.

Az itt közölt anyag kizárólag saját jogosultsági kereteken belül, megfelelő műszaki felkészültséggel és a kockázatok tudatos vállalásával használható.
