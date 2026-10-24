# Fujitsu FUTRO S930 / D3313-E1 BIOS-helyreállítás és firmware-elemzés

Független műszaki vizsgálat egy Fujitsu FUTRO S930, D3313-E1 alaplap BIOS-helyreállításáról, SPI firmware-struktúrájáról, Fujitsu gépspecifikus OEM-adatairól, a Realtek hálózati identitásról és a firmware célzott helyreállításáról.

> **Fontos:** ez a repository egy konkrét kutatási esetet dokumentál. Nem hivatalos Fujitsu javítási eljárás, és nem feltételezhető, hogy változtatás nélkül minden D3313-E1 vagy FUTRO S930 rendszerre alkalmazható.

English version: [README.md](README.md)

---

## 1. Hatókör és célok

A projekt eredetileg egy olyan Fujitsu FUTRO S930 / D3313-E1 rendszer helyreállításaként indult, amely már nem bootolt megfelelően.

A vizsgálat fokozatosan firmware-forenzikai esettanulmánnyá bővült, az alábbi célokkal:

- az eredeti SPI-tartalom megőrzése bármilyen módosítás előtt;
- annak megállapítása, hogy az eredeti firmware strukturálisan érvényes-e;
- a gépspecifikus firmware összehasonlítása a hivatalos Fujitsu firmware-rel;
- a gépspecifikus OEM/manufacturing adatok azonosítása;
- annak megértése, hogy a Fujitsu BIOS recovery miért állította helyre a bootolhatóságot, de az eredeti LAN-identitást nem;
- annak meghatározása, hogy a Realtek hálózati vezérlő EEPROM/eFuse memóriában tárolta-e a permanens MAC-címet, vagy a platform firmware-ből kapta;
- kizárólag a bizonyítékok által indokolt legkisebb firmware-régió helyreállítása;
- a teljes bizonyítási lánc és a fennmaradó bizonytalanságok dokumentálása.

A BIOS- és OEM-identitás helyreállításának fő vizsgálata sikeresen befejezettnek tekinthető.

A különálló 16 GB-os mSATA-felismerési probléma továbbra is nyitott kérdés, és szándékosan külön problémaként kezeljük.

---

## 2. Bizonyítékkategóriák

Annak érdekében, hogy az értelmezést ne tüntessük fel mérési eredményként, a repository öt bizonyítékkategóriát használ.

### `[OFFICIAL]`

Gyártói dokumentációból, hivatalos firmware-csomagból, szabványdokumentációból, upstream szoftverdokumentációból vagy más hiteles elsődleges forrásból származó információ.

### `[OBSERVED]`

A vizsgált hardveren vagy szoftveren közvetlenül mért, rögzített, kiolvasott, naplózott, kinyert, reprodukált vagy függetlenül ellenőrzött információ.

### `[INFERRED]`

A rendelkezésre álló megfigyelések által alátámasztott következtetés, amelyet közvetlenül nem mértünk.

### `[REPORTED HISTORY]`

Olyan információ, amely a kontrollált vizsgálat előtti eseményt vagy állapotot ír le, és a megőrzött bizonyítékok alapján utólag már nem ellenőrizhető függetlenül.

Ezt a kategóriát történeti háttér dokumentálására használjuk anélkül, hogy visszaemlékezést kontrollált mérésként mutatnánk be.

### `[OPEN QUESTION]`

További bizonyítékot vagy független reprodukciót igénylő hipotézis, tisztázatlan mechanizmus vagy viselkedés.

Ezeket a jelöléseket minden olyan ponton használjuk, ahol a különbség érdemben befolyásolja az értelmezést.

---

## 3. A vizsgált platform

Az elsődlegesen vizsgált rendszer:

- **Rendszer:** Fujitsu FUTRO S930
- **Alaplap:** Fujitsu D3313-E1
- **Alaplap-azonosító:** S26361-D3313-E1
- **Processzorplatform:** AMD embedded platform; a vizsgált gép AMD GX-424CC-osztályú konfigurációt használ
- **Firmware-család:** Fujitsu / American Megatrends UEFI
- **SPI flash:** Macronix MX25L6473F-osztályú, 8 MiB kapacitású SPI flash
- **LAN-vezérlő:** Realtek RTL8111/RTL8168G család
- **Elsődleges kutatási operációs rendszer:** Debian Linux
- **Külső programozó:** CH341A
- **A vizsgálat során használt SPI-szoftverek:** IMSProg, AsProgrammer és flashrom
- **Firmware-elemző eszközök:** UEFITool és kapcsolódó bináriselemző eszközök

A tágabb hardvervizsgálat során használt tárolók között volt:

- hagyományos SATA SSD;
- egy külön 16 GB-os mSATA eszköz, amelynek felismerési problémája továbbra is megoldatlan.

A repository firmware-következtetései elsősorban erre a konkrét vizsgált példányra vonatkoznak.

---

## 4. Kiinduló hibajelenség

`[REPORTED HISTORY]`

A rendszer az alaplap és a processzor környezetében végzett fizikai hardvermunka után már nem bootolt normálisan.

A kontrollált firmware-vizsgálat megkezdése előtt jelentős fizikai beavatkozás történt.

Mivel a meghibásodás pillanatában fennálló elektromos állapot nem került rögzítésre, ez a repository **nem állít bizonyított ok-okozati kapcsolatot** egyetlen konkrét mechanikai művelet és a BIOS-probléma között.

A vizsgálat során több lehetséges magyarázatot is figyelembe vettünk:

- sérült firmware;
- megváltozott firmware-állapot;
- elektromos vagy mechanikai alaplaphiba;
- CPU/socket/kontaktprobléma;
- tárolóprobléma;
- recovery-állapot megváltozása;
- korábban meglévő, ettől független hiba.

A későbbi sikeres firmware-helyreállítás azt mutatja, hogy a firmware-állapotnak lényeges szerepe volt, de ez utólag nem bizonyítja a kezdeti fizikai okot.

---

## 5. Külső SPI-kiolvasás

Mielőtt bármit írtunk volna az alaplapra, az eredeti 8 MiB-os SPI flash teljes tartalmát külsőleg kiolvastuk.

A művelethez CH341A programozót és SOIC8 csatlakozási módszert használtunk.

A programozó WCH CH341A-osztályú eszközként jelent meg.

A Windows alatti kísérletek során különböző driver- és szoftverkombinációk eltérő működést mutattak, többek között:

```text
VID_1A86
PID_5512
```

és például:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

hibát.

Az IMSProg sikeresen felismerte a programozót és eszközrevíziót is jelzett.

A flash egy MXIC/Macronix 64 Mbit-osztályú, 8 MiB-os SPI eszközként került azonosításra.

Ennek a szakasznak a célja a bizonyíték megőrzése volt, nem a javítás.

---

## 6. Kettős kiolvasás és integritás-ellenőrzés

`[OBSERVED]`

Módosítás előtt két egymástól független teljes SPI-kiolvasás készült.

Mindkét image mérete:

```text
8 388 608 bájt
```

volt.

A két független kiolvasást összehasonlítottuk.

Bitre azonosak voltak.

A megőrzött eredeti image SHA-256 hash-e:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

A munkafolyamat alapelve:

```text
KIOLVASÁS
   ↓
ÚJRA KIOLVASÁS
   ↓
ÖSSZEHASONLÍTÁS
   ↓
HASH
   ↓
ARCHIVÁLÁS
   ↓
ELEMZÉS
   ↓
CSAK EZUTÁN ÍRÁS
```

Az érintetlen eredeti image-et minden későbbi kísérleti vagy javított image-től elkülönítve őriztük meg.

---

## 7. Összehasonlítás a hivatalos Fujitsu firmware-rel

Elsődleges tiszta referencia-image-ként a D3313-E1x hivatalos Fujitsu firmware-csomagját használtuk.

A vizsgált hivatalos verzió:

```text
V4.6.5.4 R1.16.0 for D3313-E1x
```

A releváns hivatalos firmware-image-ek között 8 MiB méretű fájlok szerepeltek, például:

```text
D3313-E1.rom
D3313-E1.103
D3313-E1.109
D3313-E1.110
D3313-E1.114
```

A fő hivatalos ROM mérete:

```text
8 388 608 bájt
```

A vizsgált hivatalos `D3313-E1.rom` SHA-256 hash-e:

```text
1E51B1392A291CEC24575170F5527FE541048463366D2FA541E90A93EA371F59
```

A hivatalos csomag Fujitsu flash- és recovery-komponenseket is tartalmazott, köztük:

```text
DOS\D3313-E1.UPC
DOS\D3313-E1.UPD
DOS\DosFlash.BAT
DOS\EfiFlash.exe
```

A vizsgált `DosFlash.BAT` az alábbi parancsot hívja:

```text
EfiFlash.exe /AUTO /NOMCU
```

---

## 8. Fő strukturális eredmény: 128 blokkból 127 azonos

A 8 MiB-os image-eket 64 KiB-os blokkokra osztottuk.

Egy 8 MiB-os image:

```text
128 × 64 KiB
```

blokkból áll.

Az eredeti gépspecifikus image és a hivatalos Fujitsu image összehasonlításának eredménye:

```text
128 blokkból 127 azonos
```

Mindössze egyetlen 64 KiB-os blokk különbözött:

```text
0x440000–0x44FFFF
```

Ez az eredmény a gépspecifikus eltérést az egész firmware nagyon kis részére szűkítette.

`[INFERRED]`

Az eredmény erősen arra utalt, hogy a fő végrehajtható firmware lényegében megegyezik a hivatalos Fujitsu kóddal, míg a különböző 64 KiB-os terület gépspecifikus vagy gyártási adatokat tartalmaz.

---

## 9. UEFI Firmware Volume és FFS elemzés

Az image-ek UEFI firmware-struktúráját is elemeztük.

A vizsgálat eredménye:

```text
8 érvényes Firmware Volume
84 FFS fájl
0 hibás FV fejléc
0 hibás FFS checksum
```

A releváns Firmware Volume-ok közelítő helyei között szerepelt:

```text
0x400000
0x440000
0x443000
0x44A000
0x44C000
0x474000
0x48C000
0x740000
```

A `0x440000` környéki gépspecifikus terület tehát strukturált UEFI firmware-régióban található, nem egyszerűen használaton kívüli területen.

Az elemzés során azonosított FREEFORM FFS-struktúrák között például az alábbi GUID-ok szerepeltek:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

Az egyes struktúrák pontos szemantikai szerepét pusztán a helyük alapján nem tekintettük bizonyítottnak.

---

## 10. OEM/manufacturing adatstruktúrák

A különböző régió vizsgálata felismerhető Fujitsu/OEM-jellegű mezőket mutatott.

Például:

```text
BdSn
PcId
BUID
UUID
MAC Address #0
HDCP
TPMD
OEMB
```

A régióban lévő információ egyértelműen eltér a hagyományos végrehajtható BIOS-kódtól.

`[OBSERVED]`

A gépspecifikus azonosítók az SPI image-be vannak beágyazva.

`[INFERRED]`

A blokk OEM/manufacturing identitási régióként működik, amely az adott gép gyártásakor vagy konfigurálásakor provisionált adatokat tartalmaz.

Mivel ezek a struktúrák érzékeny azonosítókat is tartalmazhatnak, a teljes személyre szabott eredeti firmware-image **nem kerül nyilvánosságra** ebben a repositoryban.

---

## 11. A gépspecifikus MAC-mező felfedezése

Különösen fontos mező volt:

```text
MAC Address #0
```

Az eredeti megőrzött firmware-ben ez a mező a gép Fujitsu által kiosztott Ethernet-identitását tartalmazta.

A tényleges értéket szándékosan nem publikáljuk, ezért itt:

```text
<REDACTED_FUJITSU_MAC>
```

alakban szerepel.

A hivatalos Fujitsu firmware megfelelő helyén ezzel szemben:

```text
FF:FF:FF:FF:FF:FF
```

volt.

Ez az egyik legerősebb bizonyítéka annak, hogy a hivatalos image generikus sablon, míg a gépből kiolvasott firmware gyártási személyre szabást tartalmaz.

---

## 12. Az SPI-dumpok adatvédelmi és forenzikai jelentősége

Egy nyers SPI-image nem pusztán „BIOS-kód”.

Tartalmazhat:

- sorozatszámokat;
- alaplapsorozatszámokat;
- UUID-ket;
- PC ID-ket;
- MAC-címeket;
- gyártási azonosítókat;
- HDCP-adatokat;
- licencinformációkat;
- biztonsági beállításokat;
- kalibrációs vagy platformspecifikus adatokat;
- más, dokumentálatlan gépspecifikus struktúrákat.

Ezért:

> **A személyre szabott firmware-dumpot potenciálisan érzékeny műszaki bizonyítékként kell kezelni.**

A repository ezért a teljes firmware helyett:

- hasheket;
- offseteket;
- strukturális eredményeket;
- anonimizált példákat;
- reprodukálható módszertant

publikál.

---

## 13. A Fujitsu BootStick vizsgálata

A Fujitsu recovery-megoldását a hivatalos csomag alapján vizsgáltuk.

A BootStick eljárás kis kapacitású USB-meghajtón hozott létre recovery-környezetet.

A sikeres konfiguráció hagyományos USB 2.0-osztályú pendrive-ot használt.

A létrejövő környezet DOS/FAT-jellegű bootmédiát használt.

A releváns komponensek között szerepeltek:

```text
D3313-E1.rom
D3313-E1.UPD
EfiFlash.exe
DosFlash.BAT
```

A Fujitsu recovery-környezetet és a normál `EfiFlash` frissítési eljárást nem szabad automatikusan ugyanannak a mechanizmusnak tekinteni.

A recovery külön bootloader/recovery logikát is használhat.

---

## 14. Recovery jumper

A vizsgált alaplap az alábbi konfigurációs pozíciókat jelzi:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

A helyreállítás a hardware recovery jumper megfelelő pozíciójával történt.

Ez összhangban van a Fujitsu dokumentációjával, amely sérült BIOS esetére külön recovery módot ír le.

Másik rendszeren történő helyreállítás előtt mindig az adott alaplap megfelelő dokumentációját kell ellenőrizni.

---

## 15. Sikeres recovery-konfiguráció

`[OBSERVED]`

A megfelelő Fujitsu recovery-média és helyesen beállított recovery jumper mellett az alaplap belépett a helyreállítási folyamatba.

A folyamat alatt hangjelzések voltak hallhatók.

A vizsgálat során megfigyelt jellegzetes minta:

```text
hosszú – hosszú – rövid – rövid
```

A folyamat befejezése és a jumper normál pozícióba történő visszaállítása után:

- visszatért a POST;
- a gép bootolt;
- a Debian elindult;
- a gép már nem volt az eredeti nem bootoló állapotban.

Az elsődleges BIOS-recovery tehát sikeres volt.

---

## 16. A recovery utáni hálózati identitási hiba

A sikeres Fujitsu recovery után azonban egy új anomália maradt.

Az onboard Realtek Ethernet-vezérlő már nem mutatta az eredeti permanens MAC-identitást.

BIOS/firmware szinten a releváns identitás lényegében üresnek látszott.

Megfigyelt érték:

```text
00:00:00:00:00:00
```

A Linux `r8169` driver olyan működést mutatott, amely egy használható hardveres MAC hiányával egyezett, és ideiglenes/véletlenszerű címet állított be.

Megfigyelt diagnosztikai szövegek:

```text
can't read MAC address, setting random one
```

és:

```text
Permanent address: not set
```

A hálózat tehát működhetett, miközben a gép eredeti platformidentitása hiányzott.

---

## 17. Az operációs rendszer viselkedésének elkülönítése a hardverállapottól

A Linux által generált MAC-címet nem tekintettük permanens vezérlőidentitás bizonyítékának.

A vizsgálat külön kezelte:

1. az operációs rendszer által pillanatnyilag használt runtime címet;
2. a vezérlőben tartósan programozott címet;
3. a platform firmware-ben tárolt Fujitsu gépspecifikus értéket.

A Linux diagnosztikában látható permanens cím hiánya azt mutatta, hogy a probléma mélyebb egy egyszerű operációs rendszeres hálózati konfigurációs hibánál.

A Linux interfészbeállításának módosítása önmagában nem állítaná vissza az eredeti Fujitsu provisioning-mechanizmust.

---

## 18. Realtek RSET8168 diagnosztika

A vezérlőt Realtek DOS diagnosztikai eszközökkel is vizsgáltuk.

Az RSET8168 v1.24 többek között az alábbiakat jelezte:

```text
No Valid External EEPROM!!
```

Az eszköz eFuse-adatra utaló szöveget is mutatott:

```text
Data from EFuse
```

valamint:

```text
EFuse Fail
```

üzenetet.

Ezek alapján meg kellett vizsgálni, hogy az eredeti permanens identitás:

- külső EEPROM-ban;
- belső eFuse-ban;
- platform firmware-ben;
- vagy több mechanizmus kombinációjával

volt-e tárolva.

Egyetlen diagnosztikai üzenetből nem vontunk le végleges következtetést.

---

## 19. EEPROM-diagnosztika eredménye

További Realtek-eszközökkel a külső EEPROM útvonalát is vizsgáltuk.

Az adapter a releváns EEPROM-műveletnél például:

```text
Not Support This Adapter
```

jellegű eredményt adott.

Nem találtunk bizonyítékot arra, hogy egy hagyományos, feltöltött külső EEPROM tartalmazná az eredeti Fujitsu MAC-identitást.

`[OBSERVED]`

A vizsgált rendszer a tesztelt diagnosztikai módszerekkel nem mutatott olyan használható külső EEPROM-ot, amely az eredeti permanens címet tartalmazta volna.

---

## 20. A Realtek eFuse közvetlen vizsgálata

Mivel az eFuse programozása visszafordíthatatlan lehet, bármilyen írás előtt kizárólag olvasással ellenőriztük az állapotát.

A vizsgálathoz Realtek PGTool-kompatibilis diagnosztikát használtunk.

Az alapelv:

> **Előbb olvasni, és visszafordíthatatlan tárolót bizonyíték nélkül soha nem programozni.**

A releváns eFuse-terület üres/programozatlan állapotot mutatott.

---

## 21. Kritikus eFuse-eredmény

A kulcsfontosságú eredmény:

```text
Efuse Write Count = 0
```

volt.

A releváns eFuse-adat:

```text
FF
```

értéket mutatott, nem pedig korábban beprogramozott gépspecifikus MAC-et.

`[OBSERVED]`

Nem találtunk bizonyítékot arra, hogy a vizsgált vezérlő eFuse memóriájába valaha beprogramozták volna az eredeti Fujitsu Ethernet-identitást.

Ez döntő eredménynek bizonyult.

---

## 22. Miért nem programoztuk a Realtek eFuse-t?

Az eredeti MAC eFuse-ba történő beírása új, permanens hardverállapotot hozott volna létre.

A bizonyítékok azonban azt mutatták, hogy:

- az eredeti gép SPI-image-e tartalmazta a Fujitsu MAC-et;
- a hivatalos Fujitsu firmware megfelelő mezőjében `FF:FF:FF:FF:FF:FF` volt;
- a Realtek eFuse write count `0` volt;
- tehát a vezérlő eredeti identitása láthatóan nem egy korábban programozott eFuse-ból származott.

`[INFERRED]`

Az eFuse programozása nem az eredeti gyártási mechanizmust rekonstruálta volna.

Ezért az eFuse-t szándékosan **nem programoztuk**.

---

## 23. Belső SPI-hozzáférés Linux alatt

Miután az alaplap ismét bootolható volt, a futó rendszerből is hozzáfértünk az SPI flashhez a flashrom segítségével.

Használt mód:

```text
flashrom -p internal
```

A flashrom felismerte az AMD platform/FCH hozzáférési útvonalát és a Macronix SPI flash eszközt.

Ez kontrollált összehasonlítást és később célzott írást tett lehetővé anélkül, hogy újra és újra fizikailag csipeszt kellett volna az SPI chipre rögzíteni.

Az eredeti, külső programozóval megőrzött dump továbbra is a pre-repair állapot elsődleges bizonyítéka maradt.

---

## 24. A döntő recovery utáni OEM-megállapítás

Teljes post-recovery SPI-image készült, amelyet összehasonlítottunk:

- az eredeti megőrzött gépspecifikus dumppal;
- a hivatalos Fujitsu firmware-rel.

A post-recovery image OEM MAC mezőjében:

```text
FF FF FF FF FF FF
```

volt azon a helyen, ahol az eredeti firmware:

```text
<REDACTED_FUJITSU_MAC>
```

értéket tartalmazott.

Ez közvetlenül bizonyította, hogy a Fujitsu recovery helyreállította a bootolható firmware-t, de a gépspecifikus MAC mezőt generikus/sablon állapotban hagyta.

A recovery tehát **nem rekonstruálta teljes egészében a gép manufacturing-personalizációját**.

---

## 25. Mit jelent ez a Fujitsu recovery-mechanizmus szempontjából?

Az összesített bizonyíték az alábbi értelmezést támasztja alá:

1. A Fujitsu recovery sikeresen állít helyre működő firmware-állapotot.
2. A hivatalos image generikus OEM-sablont tartalmaz.
3. A gépspecifikus értékek nem feltétlenül kerülnek vissza a recovery során.
4. A vizsgált Realtek vezérlő eFuse memóriájában nem volt megtalálható az eredeti MAC.
5. A platform firmware OEM/manufacturing régiója biztosítja a gépspecifikus Ethernet-identitást.

`[INFERRED]`

A vizsgált D3313-E1 példányon az eredeti permanens LAN-identitás a platform firmware-ből származik, nem pedig egy korábban beprogramozott Realtek eFuse-ból.

Ez a következtetés kizárólag a vizsgált példányra bizonyított, amíg további rendszereken nem reprodukálják.

---

## 26. Miért nem állítottuk vissza a teljes történeti SPI-image-et?

A legegyszerűbb javítás az eredeti 8 MiB-os dump teljes visszaírása lett volna.

Ezt szándékosan elutasítottuk.

A rendszer közben már átesett:

- recovery-n;
- bootállapot-változásokon;
- firmware runtime változásokon;
- NVRAM-változásokon;
- további diagnosztikán.

A teljes történeti image visszaírása ezzel együtt más, a javításhoz nem szükséges régi állapotokat is visszaállított volna.

Ezért a vizsgálat alapelve:

> **Csak azt a legkisebb régiót állítsuk vissza, amelyről a bizonyítékok igazolják, hogy a szükséges gépspecifikus adatot tartalmazza.**

Ez egyszerre csökkenti a beavatkozás mértékét és erősebb ok-okozati tesztet ad.

---

## 27. A javítási candidate elkészítése

A javított candidate image-et az aktuális post-recovery firmware-ből építettük fel.

Csak az eredeti gépspecifikus OEM-régió került át a megőrzött pre-recovery image-ből:

```text
0x440000–0x44FFFF
```

A candidate külön fájlban került megőrzésre, például:

```text
D3313_REPAIRED_CANDIDATE.bin
```

Az összehasonlítás körülbelül:

```text
3526 eltérő bájtot
```

mutatott a post-recovery image és a candidate között.

Az eltérések az intended OEM-restoration területhez kapcsolódtak, nem egy teljes 8 MiB-os rollbackhez.

---

## 28. Írás előtti fizikai és logikai ellenőrzés

A célzott írás előtt ellenőriztük, hogy:

- a gép bootolható;
- az eredeti SPI-mentés továbbra is megvan;
- az aktuális SPI-image kiolvasásra került;
- a candidate az aktuális image-ből és az eredeti OEM blokkból épült;
- a célcímtartomány helyes;
- az írás mindössze 64 KiB-ra korlátozódik;
- az írási mechanizmus támogat utólagos verificationt.

A cél a firmware-beavatkozás és a fizikai beavatkozás minimalizálása volt.

---

## 29. Célzott 64 KiB-os helyreállítás

A flashrom layouthoz az OEM-régiót az alábbi módon definiáltuk:

```text
0x440000:0x44ffff OEM
```

Csak ezt a régiót írtuk.

A flashrom a megfelelő címtartományra:

```text
Erase/write done from 440000 to 44ffff
```

eredményt adott.

Az írás utáni verification sikeres volt:

```text
Verifying flash... VERIFIED.
```

A firmware többi része a recovery utáni aktuális állapotban maradt.

---

## 30. A helyreállítás eredménye

A célzott írás után a gépet teljesen leállítottuk és hideg bootot végeztünk.

`[OBSERVED]`

Az eredeti Fujitsu gépspecifikus Ethernet-identitás visszatért.

A hálózati stack ismét a gép permanens LAN-identitását látta, nem pedig Linux által generált véletlenszerű címet.

A gép továbbra is bootolható maradt.

A célzott 64 KiB-os helyreállítás tehát úgy állította vissza az elveszett identitást, hogy nem írta vissza a teljes 8 MiB-os történeti firmware-image-et.

---

## 31. A teljes bizonyítási lánc

A bizonyítási lánc:

```text
Eredeti gép SPI-je
        ↓
OEM-régióban az eredeti Fujitsu MAC
        ↓
Hivatalos Fujitsu firmware-ben FF:FF:FF:FF:FF:FF
        ↓
Fujitsu recovery helyreállítja a bootolhatóságot
        ↓
Post-recovery OEM mező továbbra is FF:FF:FF:FF:FF:FF
        ↓
Linux/Realtek nem talál permanens MAC-et
        ↓
RSET8168 nem talál érvényes külső EEPROM-ot
        ↓
Realtek eFuse vizsgálat:
Efuse Write Count = 0
        ↓
eFuse nincs programozva
        ↓
Csak az eredeti 0x440000–0x44FFFF OEM-régió visszaállítása
        ↓
flashrom verification sikeres
        ↓
hideg boot
        ↓
eredeti Fujitsu LAN-identitás visszatér
```

Ez a lánc lényegesen erősebb bizonyítékot ad, mint bármelyik diagnosztikai eszköz eredménye önmagában.

---

## 32. Mit bizonyított a vizsgálat?

A vizsgált D3313-E1 példányon a bizonyítékok alapján:

- az eredeti külső SPI-kiolvasás reprodukálható volt;
- az eredeti 8 MiB-os firmware túlnyomó része megegyezett a hivatalos Fujitsu firmware-rel;
- 128 darab 64 KiB-os blokkból 127 megegyezett;
- a gépspecifikus személyre szabás a `0x440000–0x44FFFF` régióra koncentrálódott;
- ez a régió tartalmazta a Fujitsu által kiosztott eredeti MAC-identitást;
- a hivatalos firmware-ben ugyanott `FF:FF:FF:FF:FF:FF` generikus érték szerepelt;
- a Fujitsu recovery helyreállította a bootolhatóságot, de ezt a személyre szabott MAC-et nem;
- a vizsgált Realtek eFuse `Efuse Write Count = 0` állapotot mutatott;
- ezért az eredeti MAC helyreállítása nem eFuse-programozással történt;
- az eredeti 64 KiB-os OEM-régió célzott visszaállítása visszahozta az eredeti permanens LAN-identitást.

---

## 33. Mit nem bizonyít a vizsgálat?

A repository **nem bizonyítja**, hogy:

- minden D3313-E1 ugyanilyen personalization layoutot használ;
- minden FUTRO S930 ugyanezen mechanizmuson keresztül tárolja a MAC-et;
- minden Realtek RTL8111/8168G vezérlő eFuse-a üres ezen a platformon;
- minden Fujitsu recovery-csomag azonos módon kezeli a personalizációt;
- minden firmware-verzióban azonosak az offsetek;
- a teljes 64 KiB-os régió minden OEM-mezőjének jelentése ismert;
- a kezdeti BIOS-hiba fizikai oka bizonyított;
- ez a célzott írás más gépeken univerzálisan biztonságos.

Ehhez további független reprodukció szükséges.

---

## 34. Külön nyitott probléma: 16 GB mSATA

A vizsgált gépen külön 16 GB-os mSATA-probléma is fennáll.

A tünetek között szerepel, hogy az eszköz nem jelenik meg megbízhatóan vagy nem létesít normál SATA-linket az adott konfigurációban.

Ez a probléma **nem része a befejezett BIOS/OEM-identitás helyreállításnak**.

Lehetséges okként továbbra is vizsgálható:

- maga az eszköz hibája;
- csatlakozási/kontaktprobléma;
- tápellátás;
- SATA routing;
- vezérlőkonfiguráció;
- alaplapi hardverhiba.

`[OPEN QUESTION]`

Az mSATA-probléma továbbra is megoldatlan.

---

## 35. Külön hardveres incidens: mozgó SATA-adapter

A tágabb hardvermunka során egy SATA-adapter/csatlakozó fizikailag mozgatható volt és helyzetet tudott változtatni.

Ezt azért dokumentáljuk, mert a mechanikai instabilitás megnehezítheti a tárolóhibák értelmezését.

Ugyanakkor:

> **Nem bizonyított közvetlen ok-okozati kapcsolat az adapter mozgása és a firmware/OEM helyreállítás eredménye között.**

Ezért ezt külön hardveres megfigyelésként kezeljük.

---

## 36. A minimálisan szükséges módosítás elve

A módszertani eredmény nem csupán az, hogy a gép megjavult.

A javítás mindössze:

```text
64 KiB
```

módosításával sikerült egy:

```text
8 MiB
```

méretű SPI firmware-ben.

Ez egy általános firmware-recovery elvet támaszt alá:

> **Ha a bizonyítékok azonosítják a szükséges gépspecifikus állapotot, őrizd meg a jelenlegi ismert működő firmware-t, és csak a legkisebb szükséges régiót állítsd vissza.**

Ez csökkenti a szükségtelen változtatásokat például az alábbi területeken:

- NVRAM;
- bootváltozók;
- firmware runtime state;
- egyéb konfiguráció;
- ismeretlen struktúrák;
- recovery során létrejött módosítások.

---

## 37. Reprodukálhatóság

Egy hasonló vizsgálatnak reprodukálható bizonyítási láncot kell megőriznie.

Ajánlott minimum:

```text
1. Pontos alaplaprevízió azonosítása
2. Pontos SPI flash azonosítása
3. Teljes flash kiolvasása
4. Teljes flash újbóli kiolvasása
5. A két kiolvasás összehasonlítása
6. Kriptográfiai hash készítése
7. Eredeti image-ek változatlan archiválása
8. Hivatalos firmware beszerzése hiteles forrásból
9. Blokkszintű összehasonlítás
10. Strukturális eltérések elemzése
11. Megfigyelés és következtetés elkülönítése
12. Visszafordíthatatlan írás kerülése bizonyíték nélkül
13. Javítás előtt az aktuális firmware ismételt kiolvasása
14. Csak a minimálisan indokolt régió módosítása
15. Írás utáni verification
16. Hideg boot és független eredményellenőrzés
```

További D3313-E1 alaplapokon végzett független reprodukció jelentősen erősítené az általánosabb következtetéseket.

---

## 38. Korlátok

A vizsgálatnak fontos korlátai vannak.

Elsősorban egyetlen fizikai gépre vonatkozik.

Bizonyos megfigyelések akkor készültek, amikor a rendszer már átesett:

- korábbi hibakeresésen;
- recovery-próbálkozásokon;
- fizikai hardverkezelésen;
- firmware-műveleteken;
- operációs rendszeres változtatásokon.

Nem minden történeti állapot került megőrzésre.

Egyes mechanizmusokat több, egymást erősítő bizonyíték alapján következtetünk ki, nem pedig közvetlen Fujitsu-dokumentációból.

A vendor diagnosztikai eszközöknek is lehet dokumentálatlan viselkedésük.

Ezért a repository következetesen elkülöníti a közvetlen megfigyelést az értelmezéstől.

---

## 39. További kutatási irányok

Hasznos további munka lehet:

1. az OEM-régió összehasonlításának reprodukálása további D3313-E1 alaplapokon;
2. D3313-A/B/E/G/S változatok összehasonlítása, ahol ez jogszerűen és gyakorlatilag lehetséges;
3. a `0x440000–0x44FFFF` régió teljes struktúrájának feltérképezése;
4. annak meghatározása, hogyan provisionálják a Fujitsu manufacturing eszközök az OEM blokkot;
5. a `BdSn`, `PcId`, `BUID`, `UUID`, MAC, HDCP és TPM-adatok kapcsolatának vizsgálata;
6. több hivatalos firmware-verzió összehasonlítása;
7. annak vizsgálata, változott-e a recovery működése a Fujitsu BIOS-verziók között;
8. a Realtek teljes inicializációs útvonalának vizsgálata;
9. biztonságos blokkszintű összehasonlító és anonimizáló szkriptek készítése;
10. a különálló 16 GB mSATA-probléma megoldása.

---

## 40. A firmware-forenzikai eredmény tágabb jelentősége

Az eset fontos különbségre világít rá:

> **Egy generikus gyártói firmware-image és egy konkrét gép teljes firmware-állapota nem feltétlenül ugyanaz.**

Egy működő platform több rétegből állhat:

```text
generikus végrehajtható firmware
        +
manufacturing personalizáció
        +
gépspecifikus identitás
        +
runtime NVRAM / konfiguráció
        +
licencelési / biztonsági adatok
        +
lehetséges kalibrációs vagy platformspecifikus állapot
```

Egy recovery-eljárás ezek közül csak bizonyos rétegeket állíthat helyre.

Ennek jelentősége van:

- firmware-forenzikában;
- hardvermegőrzésben;
- alaplapcserénél;
- jogszerű javításnál;
- gépidentitás megőrzésénél;
- reverse engineering során;
- biztonságos szelektív helyreállításnál.

---

## 41. Biztonsági szempontok

Firmware- és alaplapszintű beavatkozás tartós hardverkárt okozhat.

Ajánlott:

- több független SPI-kiolvasás;
- összehasonlítás írás előtt;
- kriptográfiai hash;
- eredeti bizonyíték változatlan megőrzése;
- pontos flash-chip azonosítás;
- megfelelő feszültség és programozóbeállítás ellenőrzése;
- a lehető legkisebb írási tartomány;
- minden írás utáni verification;
- OTP/eFuse műveletek visszafordíthatatlanként kezelése.

A Realtek eFuse-t ebben a vizsgálatban szándékosan **nem** programoztuk, mert a bizonyítékok nem támasztották alá, hogy ez lett volna az eredeti provisioning-mechanizmus.

Lásd:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

## 42. A repository adatvédelmi szabályai

A nyilvános repository szükségtelenül nem tartalmazhat kitakarás nélküli gépspecifikus bizonyítékot.

Ne kerüljön nyilvánosságra:

- teljes személyre szabott SPI-image;
- gépsorozatszám;
- szükségtelen alaplapsorozatszám;
- PC ID;
- UUID;
- teljes gépspecifikus MAC-cím;
- HDCP-adat;
- privát manufacturing adat;
- hitelesítő adat;
- személyes helyi fájlrendszer-útvonal;
- más, nem szükséges személyes információ.

A nyilvános bizonyíték lehetőség szerint:

- anonimizált részlet;
- hash;
- offset;
- kitakart érték;
- strukturális leírás.

A képernyőképeknél és fényképeknél is ellenőrizni kell:

- felhasználónevet;
- tálcát;
- IP-címet;
- fájlnevet és útvonalat;
- háttérben nyitott ablakokat;
- matricákat;
- QR-/vonalkódokat;
- EXIF/GPS metaadatot;
- más azonosító információt.

---

## 43. Repository-struktúra

A projekt jelenlegi alapfájljai:

```text
README.md
README.hu.md

LICENSE

CITATION.cff

SOURCES.md

DISCLAIMER.md
DISCLAIMER.hu.md

SECURITY.md
SECURITY.hu.md

CONTRIBUTING.md
CONTRIBUTING.hu.md

.github/
└── ISSUE_TEMPLATE/
```

Személyre szabott nyers firmware szándékosan nincs a nyilvános repositoryban.

Médiaanyag sem kerül bele addig, amíg nem publikálható szükségtelen azonosító információ felfedése nélkül.

---

## 44. Korábbi munkák és forrásmegjelölés

A vizsgálat meglévő kutatásokra és eszközökre épít, és nem állítja, hogy az alapul szolgáló technikákat vagy eszközöket maga találta volna fel.

Releváns korábbi munkák többek között:

- Fujitsu műszaki és firmware-dokumentáció;
- Fujitsu BIOS Flash Tools dokumentáció;
- hivatalos D3313 firmware-csomagok;
- flashrom;
- UEFITool;
- IMSProg;
- Realtek diagnosztikai eszközök;
- közösségi D3313/FUTRO kutatások;
- más kutatók firmware-elemzési és recovery-technikái.

A részletes forrásjegyzék:

[SOURCES.md](SOURCES.md)

A harmadik felek saját szerzői jogaik és licenceik jogosultjai maradnak.

---

## 45. Az esettanulmány kutatási hozzájárulása

A repository hozzájárulása a bizonyítékok dokumentált kombinációja és ellenőrzése egy konkrét D3313-E1 esetben.

A projekt különösen dokumentálja:

- az eredeti SPI függetlenül ellenőrzött kiolvasását;
- a hivatalos Fujitsu firmware-rel végzett blokkszintű összehasonlítást;
- a gépspecifikus adatok lokalizálását egyetlen 64 KiB-os régióra;
- az OEM MAC-mező azonosítását;
- a Fujitsu recovery működésének megfigyelését;
- a Realtek EEPROM/eFuse állapotának vizsgálatát;
- az alá nem támasztott, visszafordíthatatlan eFuse-programozás szándékos elkerülését;
- kontrollált javítási candidate létrehozását;
- kizárólag az OEM blokk belső, célzott visszaállítását;
- az eredeti permanens LAN-identitás sikeres visszatérését.

A kutatási érték a teljes bizonyítási láncban rejlik, nem egyetlen eszközben vagy parancsban.

---

## 46. Következtetés

A BIOS/OEM-helyreállítási vizsgálat elérte elsődleges célját.

A Fujitsu FUTRO S930 / D3313-E1 ismét bootolható állapotba került.

A generikus recovery után elveszett gépspecifikus Ethernet-identitást az SPI firmware Fujitsu OEM/manufacturing régiójához sikerült kötni.

A vizsgált Realtek vezérlőn nem találtunk bizonyítékot arra, hogy az eredeti identitás korábban eFuse-ba lett volna programozva.

Mindössze:

```text
0x440000–0x44FFFF
```

visszaállítása az eredeti megőrzött firmware-ből visszaadta az eredeti LAN-identitást úgy, hogy a firmware többi része a helyreállított állapotban maradt.

A vizsgálat tehát azt támasztja alá, hogy ezen a példányon a permanens hálózati identitás a Fujitsu platform firmware-en keresztül került provisionálásra, nem pedig egy korábban programozott Realtek eFuse-ból származott.

---

## 47. Jelenlegi kutatási állapot

### BIOS / firmware / OEM recovery

# **SIKERESEN BEFEJEZETTNEK TEKINTHETŐ**

Elkészült:

- eredeti SPI megőrzése;
- kettős kiolvasás ellenőrzése;
- hivatalos firmware összehasonlítása;
- UEFI-struktúra elemzése;
- OEM-régió lokalizálása;
- Fujitsu recovery;
- recovery utáni identitáshiba reprodukálása;
- Realtek EEPROM/eFuse vizsgálata;
- eFuse érintetlenül hagyása;
- OEM repair candidate elkészítése;
- célzott 64 KiB-os írás;
- flash verification;
- eredeti permanens LAN-identitás helyreállítása.

### 16 GB mSATA vizsgálat

# **NYITOTT**

Az mSATA-probléma külön hardver-/tárolóvizsgálat, és nem módosítja a befejezett BIOS/OEM-recovery eredményét.

---

## 48. Licenc

A repository megfelelő saját kutatási szövegei, dokumentációja, ábrái, elemzései és egyéb eredeti projektanyagai az alábbi licenc alatt kerülnek terjesztésre:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

Lásd:

[LICENSE](LICENSE)

A repository licence **nem licenceli újra automatikusan**:

- a Fujitsu firmware-t;
- a Realtek szoftvert;
- gyártói utilityket;
- harmadik fél dokumentációját;
- harmadik fél kódját;
- védjegyeket;
- más harmadik féltől származó anyagot.

Ha később forráskód kerül a repositoryba, az külön szoftverlicencet használhat.

---

## 49. Hivatkozás

A hivatkozási metaadatokat a:

[CITATION.cff](CITATION.cff)

fájl tartalmazza.

A munka hivatkozásakor lehetőség szerint a repository mellett az adott verziót, release-t vagy commitot is meg kell adni.

Stabil kutatási baseline:

```text
v1.0.0
```

Repository:

```text
https://github.com/benczepeter001/fujitsu-d3313-bios-recovery
```

---

## 50. Jogi és kutatási nyilatkozat

Ez egy független kutatási projekt.

Nem áll kapcsolatban, nem támogatja, nem szponzorálja és nem készítette a Fujitsu, Realtek, AMD, AMI, Macronix, WCH vagy más említett gyártó vagy projekt.

A harmadik felek terméknevei és védjegyei kizárólag azonosítás és műszaki dokumentáció céljából szerepelnek.

A firmware módosítása:

- bootolhatatlanná teheti a hardvert;
- megsemmisíthet gépspecifikus adatokat;
- károsíthat flash memóriát vagy más alkatrészt;
- visszafordíthatatlan konfigurációváltozást okozhat.

Csak olyan hardveren és adaton végezz vizsgálatot, amely a tulajdonodban van, vagy amelynek vizsgálatára kifejezett engedéllyel rendelkezel.

A részletes jogi, biztonsági, adatvédelmi és újrafelhasználási feltételek:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [SECURITY.md](SECURITY.md)
- [SECURITY.hu.md](SECURITY.hu.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [LICENSE](LICENSE)

Műszaki javításokkal, reprodukciós eredményekkel vagy kutatási kapcsolatfelvétellel kapcsolatban:

**Bencze Péter**  
**E-mail:** bencze.peter86@proton.me

---

Copyright © 2026 Bencze Péter és az egyes közreműködők saját eredeti hozzájárulásaik tekintetében.
