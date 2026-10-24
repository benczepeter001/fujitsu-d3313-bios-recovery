# Fujitsu FUTRO S930 / D3313-E1  
## BIOS-helyreállítás, firmware-elemzés és a gépspecifikus OEM-azonosság visszaállítása

### Forenzikus szemléletű műszaki esettanulmány egy Fujitsu D3313-E1 alaplap firmware-helyreállításáról és platformazonosságának rekonstrukciójáról

> **Állapot:** BIOS-helyreállítás sikeres · gépspecifikus OEM-azonosság visszaállítva · eredeti LAN-azonosság visszaállítva · a különálló mSATA-probléma vizsgálata folyamatban

[English version](README.md)

---

## Összefoglaló

Ez a repository egy Fujitsu D3313-E1 alaplapra épülő Fujitsu FUTRO S930 vékonykliens forenzikus szemléletű vizsgálatát és helyreállítását dokumentálja.

A rendszer működésképtelen állapotban került a vizsgálatba. A rendelkezésre álló előzmények szerint közvetlenül a meghibásodás előtt processzor-újrapasztázási kísérlet történt, amelynek során a hűtőegységet eltávolították, az alaplap hátoldalán található fém rögzítőlemez pedig elmozdult. Mivel a processzor környezetét a visszaszerelési próbálkozások során jelentős mechanikai nyomás érhette, a kezdeti hibát nem lehetett biztonsággal firmware-problémának tekinteni. Mechanikai sérülés, tápellátási hiba, alaplapi meghibásodás, SPI-firmware sérülés vagy ezek kombinációja egyaránt lehetséges magyarázat maradt.

A vizsgálat ezért nem a hagyományos „írjuk újra a BIOS-t és nézzük meg” megközelítést követte, hanem bizonyíték-alapú hibakeresési módszert.

A folyamat része volt több alkalommal végzett alaplapi elektromos mérés, CH341A programozóval végzett külső SPI-kiolvasás, két egymástól független 8 MiB-os firmware-dump kriptográfiai ellenőrzése, hivatalos Fujitsu firmware-rel való összehasonlítás, UEFI Firmware Volume- és FFS-elemzés, Fujitsu recovery-kísérletek, DOS alatti Realtek-diagnosztika, közvetlen Realtek eFuse-vizsgálat, Linux alatti belső SPI-kiolvasás, blokk- és byte-szintű firmware-összehasonlítás, végül pedig egyetlen 64 KiB-os SPI-régió szelektív, működő rendszerből történő visszaállítása.

A vizsgálat során több jelentős eredmény született.

Az eredeti gép SPI-image-e és a megfelelő hivatalos Fujitsu R1.16 firmware **128 darab 64 KiB-os blokkból 127-ben teljesen azonosnak bizonyult**. A teljes gépspecifikus eltérés egyetlen 64 KiB-os OEM/manufacturing régióban koncentrálódott:

```text
0x440000–0x44FFFF
```

Ez a régió több gépspecifikus mezőt tartalmazott, köztük két egymással nagyrészt redundáns struktúrát, amelyekben szerepelt a `MAC Address #0` mező is. A gyári Fujitsu firmware ezen a helyen `FF:FF:FF:FF:FF:FF` template-értéket tartalmazott, míg az eredeti gép dumpjában a tényleges Fujitsu által hozzárendelt MAC-cím szerepelt.

A sikeres Fujitsu BIOS recovery után a rendszer újra POST-olt és bootolt, a hálózati vezérlő azonban elvesztette állandó MAC-azonosságát. A DOS alatti Realtek-diagnosztika nem talált érvényes külső EEPROM-ot, míg a Linux alatti közvetlen PGTool-vizsgálat látszólag programozatlan eFuse-t mutatott:

```text
Efuse Write Count = 0
```

A recovery utáni fizikai SPI-tartalom belső kiolvasása ezután megmutatta, hogy a gépspecifikus OEM MAC-mező visszaállt a gyári `FF` template-értékre.

A Realtek eFuse végleges programozása helyett megtartottuk a működő recovery utáni firmware-állapotot, és kizárólag az eredeti 64 KiB-os OEM-régiót állítottuk vissza. A szelektív írás után a fizikai flash tartalmát külön verify művelettel is ellenőriztük.

Hidegindítás után a BIOS ismét az eredeti Fujitsu LAN-címet mutatta, és a Linux is visszakapta a hozzá tartozó állandó hálózati azonosságot.

A bizonyítékok ezért azt a modellt támasztják alá, hogy a vizsgált D3313-E1 rendszeren a Realtek vezérlő gépspecifikus MAC-azonosságát a Fujitsu platformfirmware inicializálja, nem pedig a vezérlő korábban programozott eFuse-tartalmából származik.

A projekt egy általánosabb firmware-helyreállítási elvet is szemléltet:

> **Csak azt a legkisebb firmware-régiót állítsuk vissza, amelyről bizonyítottuk, hogy valóban módosításra szorul, ahelyett hogy alapértelmezésként a teljes image-et újraírnánk.**

---

## Kulcsszavak

Fujitsu D3313-E1 · FUTRO S930 · BIOS recovery · UEFI · SPI flash · firmware forensics · OEM-adatok · manufacturing adatok · flashrom · CH341A · MX25L6473F · Realtek RTL8168G · MAC-cím · eFuse · firmware-helyreállítás · forenzikus firmware-elemzés

---

# 1. A vizsgálat célja és hatóköre

A projekt eredetileg egy működésképtelennek tűnő Fujitsu FUTRO S930 javítási kísérleteként indult.

A munka akkor alakult át strukturált firmware-vizsgálattá, amikor világossá vált, hogy a rendszer alapvető bootolhatóságának helyreállítása önmagában nem állította vissza a gép eredeti azonosságát és konfigurációját.

A vizsgálat fő céljai ezért a következők lettek:

1. megállapítani, bizonyítható-e firmware-eredetű hiba úgy, hogy közben nem zárjuk ki idő előtt a mechanikai és elektromos okokat;
2. megbízható másolatot készíteni az eredeti SPI-firmware-ről;
3. összehasonlítani a konkrét gép firmware-ét a megfelelő hivatalos Fujitsu firmware-rel;
4. azonosítani a gépspecifikus régiókat, és ahol lehetséges, meghatározni azok funkcióját;
5. a dokumentált recovery mechanizmus segítségével helyreállítani a POST- és bootképességet;
6. megvizsgálni, miért veszítette el a hálózati vezérlő az állandó MAC-azonosságát a recovery után;
7. meghatározni, hogy ez az azonosító a Realtek EEPROM/eFuse tárolójából vagy a Fujitsu platformfirmware-ből származik-e;
8. helyreállítani az eredeti gépspecifikus konfigurációt a lehető legkisebb SPI-régió módosításával;
9. olyan reprodukálható bizonyítéki láncot létrehozni, amely később függetlenül ellenőrizhető és továbbfejleszthető.

Ez a repository ezért nem általános BIOS-flashelési útmutató.

Ez egy **bizonyíték-alapú firmware-helyreállítási és platformazonosság-rekonstrukciós esettanulmány**.

---

# 2. Bizonyítéki kategóriák

Annak érdekében, hogy a közvetlenül mért tények ne keveredjenek az értelmezéssel, a projektben szereplő állításokat elvben négy kategóriába soroljuk.

### `[OFFICIAL]`

Gyártói dokumentációból, hivatalos firmware-csomagból, hivatalos segédprogramból vagy más elsődleges technikai forrásból származó információ.

### `[OBSERVED]`

A vizsgált hardveren közvetlenül mért, kiolvasott, naplózott, kinyert vagy reprodukált információ.

### `[INFERRED]`

Több megfigyelés által erősen alátámasztott következtetés, amely azonban önmagában nem közvetlen mérési eredmény.

### `[OPEN QUESTION]`

További bizonyítást vagy reprodukciót igénylő hipotézis, illetve jelenleg megoldatlan működés.

Ez a különbségtétel lényeges.

Firmware-kutatás során könnyen előfordulhat, hogy egy valószínű magyarázat sokkal hamarabb kialakul, mint hogy ténylegesen bizonyíthatóvá válna. A repository egyik célja éppen ezért az, hogy világosan elválassza egymástól:

**mi történt**,  
**mit mértünk**,  
és **mit gondolunk jelenleg arról, hogy a mérések mit jelentenek**.

---

# 3. A vizsgált platform

A vizsgált rendszer:

**Fujitsu FUTRO S930**

alaplapja:

**Fujitsu D3313-E1**

A vizsgálat szempontjából releváns hardverelemek:

- AMD GX-424CC négymagos beágyazott processzor;
- Fujitsu D3313-E1 alaplap;
- AMI Aptio-alapú firmware;
- 8 MiB-os Macronix SPI flash;
- fizikailag MX25L6473F családba azonosított Macronix flash;
- Realtek RTL8111/8168G családba tartozó Gigabit Ethernet vezérlő;
- AMD FCH SATA vezérlő;
- ASMedia ASM1061/ASM1062 SATA vezérlő;
- 240 GB-os Kingston SATA SSD;
- 16 GB-os mSATA SSD;
- Debian Linux.

A vizsgálat során azonosított firmware-verzió:

```text
V4.6.5.4 R1.16.0 for D3313-E1x
```

A nyilvános repositoryban minden gépspecifikus azonosító — például sorozatszám, PC ID, UUID, teljes MAC-cím, HDCP-hoz kapcsolódó adat és más manufacturing mezők — szándékosan anonimizálva szerepel.

---

# 4. Kiinduló hibaállapot

A strukturált vizsgálat kezdetekor a rendszer már nem működött.

A gép előéletéről kapott információ szerint közvetlenül a meghibásodás előtt processzor-újrapasztázási kísérlet történt.

A visszaszerelés során az alaplap hátoldalán található fém rögzítőlemez állítólag elmozdult a helyéről. Emiatt a hűtőegységet többször újra kellett pozicionálni, és eközben jelentős mechanikai nyomás érhette a processzor környezetét és az alaplapot.

Ezt az információt:

**`[REPORTED HISTORY]`**

kategóriába soroljuk, vagyis nem saját kísérleti megfigyelésnek tekintjük.

A mechanikai előzmény azért volt fontos, mert kizárta azt a lehetőséget, hogy már a vizsgálat elején egyszerűen BIOS-hibát feltételezzünk.

A lehetséges hibaforrások között szerepelt:

- PCB-repedés;
- sérült forrasztási kötés;
- processzor környéki alkatrészhiba;
- VRM- vagy tápági meghibásodás;
- zárlat;
- mechanikai deformáció;
- sérült SPI-firmware;
- érvénytelen firmware-állapot;
- illetve több hiba egyidejű jelenléte.

Az elérhető tápágakat és alaplapi mérési pontokat három külön alkalommal multiméterrel ellenőriztük.

Nem találtunk olyan egyértelmű elektromos hibát, amely önmagában megmagyarázta volna a teljes bootképtelenséget.

Ez **nem bizonyította**, hogy az alaplap mechanikailag sértetlen.

Azt azonban indokolta, hogy a firmware-vizsgálatot is folytassuk.

A hardveres és firmware-es hipotéziseket ezért párhuzamosan tartottuk nyitva.

---

# 5. Külső SPI-kiolvasás

Az első firmware-kiolvasást egy alacsony költségű WCH CH341A USB SPI-programozóval és SOIC8 tesztcsipesszel végeztük.

A programozó azonosítói:

```text
VID: 1A86
PID: 5512
Device: WCH CH341A
```

Windows alatt több driverrel és programozóeszközzel történt kísérletezés.

A `flashrom` USB-n ebben a környezetben nem működött megbízhatóan, és többek között az alábbi hibát adta:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

Az IMSProg ezzel szemben sikeresen felismerte a programozót:

```text
WCH CH341A
Device rev 3.0.4
```

A legnagyobb gyakorlati problémát végül nem maga a programozó, hanem a SOIC8 csipesz mechanikai bizonytalansága jelentette.

A csipesz elmozdulhatott vagy elveszíthette a kontaktust a flash tokozásán.

Ezért egyetlen látszólag sikeres kiolvasást nem tekintettünk elegendő bizonyítéknak.

---

# 6. Kettős kiolvasás és integritás-ellenőrzés

Két egymástól független teljes SPI-kiolvasás készült.

Mindkét image mérete pontosan:

```text
8 388 608 byte
```

volt, ami egy 8 MiB-os SPI flash teljes tartalmának felel meg.

Mindkét kiolvasáshoz külön SHA256 ellenőrzőösszeget számítottunk.

A két image ugyanazt az SHA256 digestet adta:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

Az egyező digest bizonyítja, hogy a két egymástól függetlenül kiolvasott 8 MiB-os image bitre pontosan azonos volt.

Az SHA256 értéket reprodukálhatósági és provenienciaazonosítóként tesszük közzé. Ennek segítségével később ellenőrizhető, hogy egy másik példány teljesen azonos-e a vizsgálat során használt eredeti image-dzsel anélkül, hogy magát a személyre szabott firmware-t közzé kellene tenni.

A nyers SPI-image szándékosan nem kerül publikálásra, mert gépspecifikus manufacturing és azonosítási adatokat tartalmaz.

A kettős kiolvasás jelentősége módszertani.

Egy bizonytalan SOIC8 kapcsolat hibás adatokat eredményezhet, de két egymástól függetlenül megszerzett, teljes 8 MiB-os és bitre pontosan azonos image lényegesen erősebb bizonyíték arra, hogy a flash tényleges tartalma helyesen került kiolvasásra.

Ebből alakult ki a projekt egyik alapvető munkafolyamata:

```text
OLVASD KI
  ↓
OLVASD KI ÚJRA
  ↓
HASONLÍTSD ÖSSZE
  ↓
HASH-ELD
  ↓
ELEMEZD
  ↓
CSAK EZUTÁN ÍRJ
```

Ugyanezt az elvet később a helyreállított rendszerből végzett belső SPI-kiolvasásnál is alkalmaztuk.

---

# 7. Összehasonlítás a hivatalos Fujitsu firmware-rel

A megfelelő hivatalos Fujitsu R1.16 firmware-image-et külön beszereztük.

A vizsgálat egyik központi kérdése a következő volt:

> **Miben különbözik egy konkrétan legyártott D3313-E1 firmware-tartalma a Fujitsu által terjesztett generikus firmware-től?**

Ahelyett, hogy a teljes 8 MiB-os image-et egyetlen átláthatatlan bináris objektumként kezeltük volna, 64 KiB-os blokkokra bontottuk.

Egy 8 MiB-os image:

```text
128 × 64 KiB
```

blokkból áll.

Az eredmény meglepően egyértelmű lett.

---

# 8. Jelentős strukturális felismerés: 128 blokkból 127 azonos

Az eredeti gép SPI-dumpja és a hivatalos Fujitsu R1.16 firmware:

**128 blokkból 127-ben teljesen azonos volt.**

Mindössze egyetlen 64 KiB-os régió tért el:

```text
0x440000–0x44FFFF
```

Ez a megfigyelés lényegesen megváltoztatta a vizsgálat irányát.

A konkrét gép firmware-e nem széles körben tért el a gyári image-től.

Éppen ellenkezőleg: a teljes 8 MiB-os firmware szinte egészében azonos volt a hivatalos Fujitsu kiadással.

A gépspecifikus testreszabás egyetlen, nagyon kis régióba koncentrálódott.

Ezután második ellenőrzést végeztünk.

A hivatalos Fujitsu R1.16 image:

```text
0x440000–0x44FFFF
```

blokkját kicseréltük az eredeti gép SPI-dumpjának ugyanazon blokkjára.

Az így kapott teljes 8 MiB-os image bitre pontosan azonos lett az eredeti gép SPI-dumpjával.

A kapcsolat tehát így írható fel:

```text
Hivatalos Fujitsu R1.16 firmware
            +
az eredeti gép 64 KiB-os OEM-blokkja
            =
a konkrét gép teljes eredeti SPI-image-e
```

A vizsgált rendszer esetében ez a projekt egyik legerősebb strukturális eredménye.

---

# 9. UEFI Firmware Volume- és FFS-elemzés

A firmware-t UEFI Firmware Volume és Firmware File System szinten is megvizsgáltuk.

Az elemzett image-ben:

```text
8 érvényes Firmware Volume
84 FFS fájl
0 hibás FV fejléc
0 hibás FFS checksum
```

volt azonosítható.

Ez további bizonyítékot adott arra, hogy a kiolvasott image belsőleg koherens UEFI firmware.

A gépspecifikus eltérések két kis firmware-struktúrában koncentrálódtak, amelyek hozzávetőlegesen innen kezdődnek:

```text
0x440000
```

és:

```text
0x443000
```

Mindkettő egy FREEFORM FFS objektumot tartalmazott.

Az első megfigyelt FFS GUID:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
```

A második:

```text
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

A két struktúra rendkívül hasonló.

Úgy tűnik, ugyanazon gépspecifikus manufacturing/OEM adat párhuzamos vagy redundáns reprezentációit tartalmazzák.

A redundancia pontos modellje jelenleg még nem teljesen ismert.

Lehetséges magyarázatok:

- elsődleges és tartalék példány;
- aktív és készenléti rekord;
- generációszámláló;
- verziózott manufacturing rekord;
- más integritási vagy fallback mechanizmus.

Ez további kutatási irány marad.

---

# 10. OEM/manufacturing adatstruktúrák

A gépspecifikus régióban több felismerhető ASCII mezőnév található.

A megfigyelt címkék között szerepelt:

```text
BdSn
PcId
BUID
UUID
MAC Address #0
HDCP
TPMD
OEMB
JPND
MMCD
CRID
SDEL
SLPD
HDLS
IBSD
AESD
```

Egyes mezők jelentése viszonylag egyértelmű.

Mások csak részben vagy egyáltalán nem dokumentáltak.

A lényeges megfigyelés az volt, hogy a hivatalos Fujitsu firmware több ilyen mezőben üres vagy template-értékeket tartalmaz, miközben az eredeti gép dumpjában tényleges gépspecifikus manufacturing adatok találhatók.

Ez egy általánosabb architekturális modellt támaszt alá:

```text
generikus gyártói firmware
          +
manufacturing személyre szabás
          =
egy konkrét gép firmware-állapota
```

Ezért a nyilvánosan letölthető gyártói firmware-image nem feltétlenül tekinthető egy adott gép teljes eredeti állapotának.

---

# 11. A gépspecifikus MAC-mező felismerése

Mindkét OEM-struktúrában megtalálható volt:

```text
MAC Address #0
```

amelyet hat byte követett.

Az eredeti gép dumpjában mindkét redundáns példány ugyanazt a gépspecifikus Ethernet-címet tartalmazta.

A tényleges cím a nyilvános repositoryban:

```text
<REDACTED_FUJITSU_MAC>
```

formában szerepel.

A hivatalos Fujitsu R1.16 image azonos mezőiben viszont:

```text
FF FF FF FF FF FF
```

vagy:

```text
FF:FF:FF:FF:FF:FF
```

szerepelt.

Ez jelentős eredmény.

A hivatalos firmware nem egy generikus Fujitsu MAC-címet tartalmazott.

Hanem egy **még nem személyre szabott manufacturing template értéket**.

A konkrét gép Ethernet-azonossága tehát később, a gyártási vagy személyre szabási folyamat során kerülhetett ebbe a struktúrába.

---

# 12. A nyers SPI-image adatvédelmi és forenzikai jelentősége

Ennek a felismerésnek a javításon túlmutató következményei is vannak.

Egy nyers SPI-dumpot nem szabad egyszerűen „BIOS-fájlként” kezelni.

Tartalmazhat:

- sorozatszámot;
- PC ID-t;
- UUID-t;
- MAC-címet;
- HDCP-hoz kapcsolódó adatokat;
- platformazonosítókat;
- konfigurációs adatokat;
- manufacturing metaadatokat;
- és esetleg további, jelenleg még nem teljesen visszafejtett érzékeny mezőket.

Ezért ez a repository **nem teszi közzé**:

- az eredeti személyre szabott SPI-dumpot;
- a személyre szabott recovery ROM-ot;
- valódi sorozatszámokat;
- valódi PC ID-t;
- teljes UUID-ket;
- a teljes gépspecifikus MAC-címet;
- HDCP-hoz kapcsolódó adatokat;
- más egyedi manufacturing értékeket.

Csak anonimizált példák, offsetek, struktúraleírások és adatvédelmileg ellenőrzött képek kerülhetnek nyilvánosságra.

---

# 13. Fujitsu BootStick elemzése

A recovery során használt Fujitsu BootStick mechanizmust külön is megvizsgáltuk.

A segédprogram hagyományos FreeDOS-alapú rendszerindító adathordozót készített, amely tartalmazott:

- MBR-t;
- egy aktív FAT32 partíciót;
- FreeDOS bootkódot;
- `KERNEL.SYS` fájlt;
- `COMMAND.COM` fájlt.

Az újonnan létrehozott BootStick alapértelmezett `AUTOEXEC.BAT` fájlja nem tartalmazott automatikus BIOS-flash parancsot.

Ez azért volt lényeges, mert két egymással könnyen összekeverhető elemet különített el:

1. a Fujitsu BootStick által létrehozott generikus rendszerindító környezetet;
2. az adott alaplaphoz és BIOS-csomaghoz tartozó tényleges recovery mechanizmust.

A sikeres recovery végül egy 2 GB-os USB-adathordozóról történt.

A hibalehetőségek csökkentése érdekében egyszerű hátlapi USB 2.0 portot választottunk.

Nem állítjuk, hogy ez a port elektromosan rövidebb, kizárólagosan szükséges vagy hivatalosan kijelölt recovery port.

Ez gyakorlati kompatibilitási döntés volt.

---

# 14. Recovery jumper konfiguráció

A vizsgált D3313-E1 alaplapon a jumper elrendezése:

```text
5     3     1
●     ●     ●

●           ●
6    [ ]    2
      ↑
 hiányzó pin
 4-es pozíció
```

A megfigyelt működési állások:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

A hibakeresés korábbi szakaszában az alaplap tévedésből több alkalommal is elindult RESERVED állásban.

A vizsgált példányon nem figyeltünk meg ehhez egyértelműen köthető visszafordíthatatlan károsodást.

Ez azonban **nem jelenti azt, hogy a RESERVED állás biztonságos**.

A pontos funkcióját nem fejtettük vissza, ezért használata nem javasolt.

---

# 15. A sikeres recovery konfigurációja

A sikeres firmware-recoveryhez a hardverkonfigurációt tudatosan minimalizáltuk.

Csatlakoztatva maradt:

- alaplap;
- processzor;
- egy RAM-modul a DIMM1 foglalatban;
- CMOS elem;
- buzzer;
- kijelző;
- billentyűzet;
- recovery USB.

A nem szükséges háttértárakat és perifériákat eltávolítottuk.

A jumpert RECOVERY állásba tettük.

A folyamat során rövid, periodikus sípolások jelentkeztek.

A látszólag sikeres befejezéskor ismétlődő:

```text
hosszú – hosszú – rövid – rövid
```

hangmintát figyeltünk meg.

Ezután a gépet:

1. kikapcsoltuk;
2. leválasztottuk a tápellátásról;
3. eltávolítottuk a recovery USB-t;
4. a jumpert visszatettük DEFAULT állásba.

A következő normál indításkor visszatért a videokimenet, és a BIOS Setup ismét elérhetővé vált.

A korábban működésképtelen alaplap újra POST-olt.

A BIOS-recovery tehát sikeres volt.

---

# 16. Váratlan hálózati azonossághiba a recovery után

A POST helyreállítása után új rendellenesség jelent meg.

A BIOS a Realtek hálózati vezérlő MAC-címeként:

```text
00:00:00:00:00:00
```

értéket mutatott.

A Debian ennek ellenére elindult, és a hálózat működött.

Az interfész fizikai linket épített fel, hálózati konfigurációt kapott, és SSH-n keresztül elérhető volt.

Első látásra ez akár egyszerű BIOS-kijelzési hibának is tűnhetett volna.

A Linux naplói azonban mást mutattak.

Az `r8169` driver ezt jelezte:

```text
can't read MAC address, setting random one
```

Az `ethtool` pedig:

```text
Permanent address: not set
```

állapotot mutatott.

Az operációs rendszer tehát nem kapott érvényes állandó hardveres MAC-címet.

A hálózat azért maradt működőképes, mert a Linux generált egy helyettesítő címet, amelyet később a systemd MAC-policy stabilizált.

A gép tehát visszakapta a hálózati kapcsolatot, de elvesztette eredeti hardveres hálózati azonosságát.

---

# 17. Az operációs rendszer és a hardverállapot szétválasztása

A következő szükséges kérdés az volt:

> **Lehetséges, hogy ez csak Linux-driverprobléma?**

A hipotézis ellenőrzéséhez a vizsgálatot kivittük a normál Linux hálózati stackből.

DOS-alapú Realtek-diagnosztikát választottunk, hogy a vezérlőt a Debian és az `r8169` drivertől függetlenül lehessen megvizsgálni.

---

# 18. Realtek RSET8168 diagnosztika

A Realtek DOS diagnosztikai segédprogramját:

```text
RSET8168 v1.24
```

külön FreeDOS USB-ről indítottuk.

A program közvetlenül a hálózati vezérlőt vizsgálta.

A státuszképernyő:

```text
Ethernet Address: 00 00 00 00 00 00
```

értéket mutatott.

A MAC-regiszter nézet szintén nullás adatokat jelzett.

Ez fontos bizonyítéki lépés volt.

A hiányzó MAC nem a Debian hálózati stackje miatt alakult ki.

A Realtek vezérlő Linux nélkül sem rendelkezett érvényes gépazonossággal.

---

# 19. EEPROM-diagnosztika eredménye

Az RSET8168 EEPROM Diagnostic Test ezt jelezte:

```text
No Valid External EEPROM!!
```

majd:

```text
Data from EFuse
```

végül:

```text
EFuse Fail
```

Ez új hipotézist vetett fel.

Ha nincs érvényes külső EEPROM, akkor elképzelhető volt, hogy a MAC az RTL8168G belső eFuse területén található.

Ha pedig az eFuse a recovery során sérült vagy elvesztette a tartalmát, akkor közvetlen Realtek-programozás is lehetséges javítási iránynak tűnt.

Ebben a szakaszban ez még ésszerű, de nem bizonyított magyarázat volt.

---

# 20. Közvetlen Realtek eFuse-vizsgálat

Az eFuse-hipotézis közvetlen teszteléséhez Linux alatt Realtek PGTool-kompatibilis segédprogramot használtunk.

A toolkészlet tartalmazta:

```text
8168GEF.cfg
```

konfigurációt az RTL8168G család eFuse-kezeléséhez.

A hozzá tartozó saját kernelmodul azonban elavult Linux API-t használt, és kezdetben nem fordult le a Debian 6.12 kernellel.

Az eredeti:

```c
vma->vm_flags |= VM_IO;
```

sort:

```c
vm_flags_set(vma, VM_IO);
```

hívásra módosítottuk.

A kompatibilitási módosítás után a modul sikeresen lefordult és betöltődött.

A normál `r8169` drivert leállítottuk, és a Realtek vezérlőt a PGTool saját driverével olvastuk.

---

# 21. Döntő eFuse-eredmény

A csak olvasási módban végzett eFuse-kiolvasás során az olvasható területen kizárólag:

```text
FF
```

értékek jelentek meg.

Ennél is fontosabb, hogy a program ezt közölte:

```text
Efuse Write Count = 0
```

miközben lényegében a teljes eFuse kapacitás még szabad volt.

Ez lényeges különbség.

Egy kizárólag `FF` értékeket tartalmazó eFuse elméletileg lehetne törölt vagy hiányos adat is.

A nulla írásszámláló azonban erősen arra utal, hogy ezt a területet **korábban soha nem programozták**.

Az EEPROM olvasási útvonala emellett:

```text
Not Support This Adapter
```

eredményt adott.

A megfigyelések együtt ezért ellene szóltak annak a modellnek, hogy az eredeti Fujitsu MAC tartósan a Realtek EEPROM/eFuse tárolójában lett volna.

Egy másik modell vált sokkal valószínűbbé:

> **A Fujitsu firmware a Realtek vezérlőt az alaplapi SPI flashben tárolt gépspecifikus platformadatokból inicializálhatja.**

Ekkor ez még következtetés volt.

A bizonyításhoz a recovery utáni fizikai SPI-tartalmat is közvetlenül össze kellett hasonlítani.

---

# 22. Miért nem programoztuk egyszerűen be a Realtek eFuse-t?

A PGTool technikailag lehetővé tette volna node ID/MAC programozását a Realtek nem felejtő tárolójába.

Az eredeti MAC-cím ismert volt.

Az eFuse üresnek látszott.

Ez tehát gyors kerülőmegoldás lehetett volna.

Tudatosan nem ezt választottuk.

Az okok módszertani és technikai jellegűek voltak.

Az eFuse-programozás:

- tartós;
- nem azonos egy normál, újraírható flash művelettel;
- véges programozható kapacitással rendelkezik;
- potenciálisan visszafordíthatatlan;
- és semmi nem bizonyította, hogy a Fujitsu eredeti platformterve ezt a tárolási módot használta.

A projekt célja nem egyszerűen az volt, hogy:

> „legyen újra valamilyen állandó MAC-cím”.

A cél az volt, hogy:

> **az eredeti gép által használt mechanizmust állítsuk helyre.**

A Realtek eFuse programozása ezért csak tartalék lehetőség maradt.

---

# 23. Belső SPI-hozzáférés Linuxból

Ekkorra a külső SOIC8 csipesz kábele használhatatlanná vált.

A külső programozó javítása helyett először megvizsgáltuk, hogy a már bootoló Debian rendszerből hozzáférhető-e közvetlenül a SPI flash.

A:

```text
flashrom -p internal
```

parancs sikeresen felismerte az:

```text
AMD FCH
```

chipsetet.

A `flashrom` több kompatibilis 8 MiB-os Macronix flashdefiníciót ajánlott fel.

A fizikai chip azonosítása alapján kifejezetten az:

```text
MX25L6473F
```

definíciót választottuk.

Két teljes, egymástól független 8 MiB-os belső kiolvasást végeztünk.

A két image SHA256 értéke azonos lett.

Ez ugyanazt a bizonyítási elvet reprodukálta, amelyet a külső kiolvasásnál már alkalmaztunk:

```text
két független kiolvasás
        +
azonos kriptográfiai hash
        =
megbízható recovery utáni SPI-referencia
```

Így megbízható képet kaptunk arról, hogy **mi található fizikailag a flash chipben a Fujitsu recovery után**.

---

# 24. Döntő felismerés: az OEM MAC állapota recovery után

A recovery utáni SPI-dumpban ismét megkerestük a két:

```text
MAC Address #0
```

struktúrát.

Mindkettő továbbra is jelen volt.

A hozzájuk tartozó hat byte azonban most:

```text
FF FF FF FF FF FF
```

értéket tartalmazott.

Ez pontosan ugyanaz volt, mint a hivatalos Fujitsu R1.16 gyári image-ben.

A három állapot így közvetlenül összehasonlíthatóvá vált.

### Az eredeti gép SPI-je

```text
MAC Address #0 → <REDACTED_FUJITSU_MAC>
```

### Hivatalos Fujitsu R1.16 image

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

### Fizikai SPI a recovery után

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

Ez a megfigyelés több korábban különálló jelenséget összekapcsolt.

Megmagyarázta:

- a BIOS nullás MAC-kijelzését;
- az RSET8168 nullás Ethernet Address értékét;
- a Linux `Permanent address: not set` állapotát;
- a kernel által generált helyettesítő MAC-et;
- és a látszólag üres Realtek eFuse-t.

A vezérlő tehát nem feltétlenül egy saját, belső MAC-címet „veszített el”.

A Fujitsu platformfirmware-ből hiányzott az a gépspecifikus érték, amelyből a vezérlőt inicializálni kellett.

---

# 25. Mit mond ez a Fujitsu recovery működéséről?

Itt különösen fontos az óvatos értelmezés.

A bizonyítékok **nem** elegendőek ahhoz az egyszerűsítő állításhoz, hogy:

> „a Fujitsu recovery egyszerűen felülírja az OEM-régiót a gyári image-dzsel”.

A közvetlenül bizonyított tény ennél szűkebb és egyben érdekesebb.

A recoveryhez használt forrásimage tartalmazta az eredeti személyre szabott adatot.

A recovery után azonban a fizikai SPI releváns OEM-struktúráiban gyári template MAC-érték szerepelt.

Ez azt jelenti, hogy a recovery **nem egyszerű, teljes chipre kiterjedő bitről bitre másolást végez a megadott 8 MiB-os image-ből**.

Valamilyen további recovery-logika működik.

Lehetséges modellek:

- szelektív Firmware Volume-írás;
- manufacturing régió újragenerálása;
- bizonyos személyre szabott struktúrák figyelmen kívül hagyása;
- külön OEM-kezelési szabályok;
- valamely belső recovery-komponensből történő rekonstrukció;
- vagy más alaplapspecifikus recovery-policy.

A pontos algoritmus:

**`[OPEN QUESTION]`**

és a projekt egyik legerősebb további kutatási iránya.

---

# 26. Miért nem írtuk vissza a teljes régi 8 MiB-os image-et?

Rendelkezésre állt az eredeti teljes 8 MiB-os firmware-dump.

A legegyszerűbb javítás ezért az lehetett volna, hogy ezt teljes egészében visszaírjuk.

Ezt tudatosan elutasítottuk.

Az eredeti és a recovery utáni SPI-image több 64 KiB-os blokkban is eltért, többek között:

```text
2
54
64
65
66
67
68
```

blokkokban.

Jelentős különbségek voltak olyan területeken is, amelyek futás közben változó firmware/NVRAM állapothoz kapcsolódhatnak.

Ez önmagában nem rendellenes.

Egy működő UEFI rendszer jogosan módosíthat:

- bootváltozókat;
- NVRAM-rekordokat;
- számlálókat;
- futásidejű konfigurációt;
- eszközállapotot;
- recovery-állapotot.

A teljes korábbi image vak visszaírása ezért az OEM-azonossággal együtt olyan, a hibához nem kapcsolódó futásidejű állapotokat is visszaállított volna, amelyekhez nem akartunk hozzányúlni.

A vezérelv ezért:

> **Tartsuk meg a jelenleg működő firmware-állapotot, és csak azt állítsuk vissza, amelyről bizonyítottuk, hogy hiányzik.**

---

# 27. A javítási jelölt elkészítése

A megbízható recovery utáni SPI-image szolgált alapként.

Kizárólag:

```text
0x440000–0x44FFFF
```

régióját cseréltük ki az eredeti, recovery előtti dump megfelelő 64 KiB-os blokkjára.

Az így létrejött fájl:

```text
D3313_REPAIRED_CANDIDATE.bin
```

nevet kapta.

A javítási jelölt és a recovery utáni aktuális firmware byte-szintű összehasonlítása:

```text
3526 eltérő byte
```

eredményt adott.

Minden eltérés:

```text
egyetlen 64 KiB-os blokkban
```

található, mégpedig az előzőleg azonosított OEM/manufacturing régióban.

Ez volt a kívánt állapot.

A működő recovery utáni firmware minden más része — köztük a futás közben változott állapotok — érintetlen maradt.

Csak a gépspecifikus OEM-régió tért el.

---

# 28. Fizikai ellenőrzés írás előtt

A `flashrom` számára külön layoutot definiáltunk:

```text
0x440000:0x44ffff OEM
```

Mielőtt tényleges írás történt volna, a javítási jelöltet összevetettük a fizikai flash jelenlegi tartalmával.

A verify eredménye:

```text
3526 eltérő byte
```

volt.

Ez pontosan megegyezett a korábbi, független fájlszintű összehasonlítás eredményével.

Két külön módszer tehát ugyanazt mutatta:

```text
fájlszintű bináris összehasonlítás → 3526 byte
fizikai flash verify               → 3526 byte
```

Ez az egyezés jelentősen csökkentette annak kockázatát, hogy rossz régiót vagy rossz javítási image-et írjunk.

A tényleges írás csak ezután történt meg.

---

# 29. Szelektív 64 KiB-os SPI-visszaállítás

A teljes 8 MiB-os flash-t **nem** írtuk újra.

Kizárólag:

```text
0x440000–0x44FFFF
```

régió került kijelölésre.

A `flashrom` ezt jelezte:

```text
Erase/write done from 440000 to 44ffff
```

majd:

```text
Verifying flash... VERIFIED.
```

Az írás után külön második verify műveletet is végrehajtottunk.

Ennek eredménye ismét:

```text
VERIFIED
```

volt.

Ez a második, független ellenőrzés tudatos döntés volt.

Nem tekintettük elegendőnek az írás utáni automatikus sikerüzenetet.

A fizikai flash tartalmát külön újra ellenőriztük a kívánt javítási állapottal szemben.

---

# 30. A helyreállítás eredménye

A gépet teljesen leállítottuk, majd hidegindítást végeztünk.

A BIOS Setup ezután ismét az eredeti Fujitsu LAN MAC-címet mutatta.

A repositoryban a valódi érték helyett:

```text
<REDACTED_FUJITSU_MAC>
```

szerepel.

A Debian indulása után a Linux hálózati interfésze szintén ezt az eredeti hardveres azonosságot használta.

A korábbi:

```text
Permanent address: not set
```

állapot megszűnt.

Ezzel teljessé vált a bizonyítéki lánc.

---

# 31. A kialakult bizonyítéki lánc

A teljes folyamat a következőképpen foglalható össze:

```text
Eredeti gép SPI-je
        ↓
gépspecifikus MAC jelen van a Fujitsu OEM-régióban
        ↓
hivatalos Fujitsu firmware
        ↓
ugyanaz a mező FF:FF:FF:FF:FF:FF template
        ↓
Fujitsu recovery helyreállítja a POST-ot
        ↓
BIOS MAC = 00:00:00:00:00:00
        ↓
Linux nem talál permanent MAC-et
        ↓
RSET8168 Linuxon kívül is nullás címet mutat
        ↓
nincs érvényes külső EEPROM
        ↓
Realtek eFuse közvetlen kiolvasása
        ↓
üres FF állapot + Efuse Write Count = 0
        ↓
recovery utáni SPI belső kiolvasása
        ↓
a Fujitsu OEM MAC-mező ismét FF template
        ↓
csak az eredeti 64 KiB-os OEM-blokk visszaállítása
        ↓
fizikai írás független ellenőrzése
        ↓
hidegindítás
        ↓
eredeti Fujitsu MAC visszatér a BIOS-ban
        ↓
Linux permanent hálózati azonossága helyreáll
```

A megfigyelések együtt erős támogatást adnak a következő értelmezéshez:

> **A vizsgált D3313-E1 rendszeren a Realtek RTL8168G családba tartozó vezérlő gépspecifikus Ethernet-azonosságát a Fujitsu platformfirmware inicializálja, nem pedig egy korábban programozott Realtek eFuse-ból olvassa ki.**

Ez a következtetés a vizsgált konkrét hardverre vonatkozik.

Ahhoz, hogy az egész D3313 platformcsaládra általánosítható legyen, további alaplapokon végzett reprodukció szükséges.

---

# 32. Mit bizonyított a vizsgálat?

A rendelkezésre álló bizonyítékok az alábbi megállapításokat támogatják:

1. a működésképtelen D3313-E1 alaplap Fujitsu recovery segítségével ismét POST-képessé tehető;
2. két független SPI-kiolvasás és kriptográfiai összehasonlítás megbízható firmware-referenciát biztosít;
3. az eredeti gép image-e és a hivatalos Fujitsu R1.16 firmware 128 darab 64 KiB-os blokkból 127-ben azonos;
4. a fő gépspecifikus személyre szabás egyetlen 64 KiB-os OEM/manufacturing régióban koncentrálódik;
5. ebben a régióban több egyedi gép- és platformazonosító található;
6. a hivatalos gyári image `FF` template-értéket tartalmaz a LAN MAC-mezőben;
7. az eredeti gép SPI-je ugyanebben a mezőben a tényleges Fujitsu MAC-címet tartalmazza;
8. a recovery után a fizikai SPI ezen a helyen visszatért a gyári template-állapotba;
9. a Realtek vezérlő nem rendelkezett használható külső EEPROM-mal;
10. az eFuse a vizsgált példányban programozatlannak látszott;
11. a vizsgált gép LAN-azonossága ezért nem programozott Realtek eFuse-tárolóból származott;
12. a teljes SPI belsőleg is megbízhatóan kiolvasható Linux `flashrom` segítségével;
13. a hiányzó OEM-régió pontosan izolálható;
14. kizárólag ez a 64 KiB-os régió írható vissza;
15. a fizikai írás külön verify művelettel ellenőrizhető;
16. az OEM-régió visszaállítása helyreállította az eredeti LAN-azonosságot a BIOS-ban és Linux alatt is.

---

# 33. Amit a vizsgálat nem bizonyít

A projekt **nem** bizonyítja, hogy:

- minden D3313-E1 pontosan ugyanígy működik;
- minden FUTRO S930 ugyanilyen mechanizmussal tárolja a hálózati azonosságot;
- minden Fujitsu recovery elveszíti a személyre szabott MAC-adatot;
- a recovery algoritmust teljesen visszafejtettük;
- a RESERVED jumperállás biztonságos;
- a használt USB-port hivatalos recovery-követelmény;
- ezen a platformon minden RTL8168G eFuse-ja üres;
- a megoldatlan mSATA-probléma biztosan hardveres eredetű;
- vagy hogy a projektben alkalmazott firmware-elemzési és recovery technikákat ez a munka találta fel.

A repository legerősebb állításai tudatosan a vizsgált konkrét rendszeren közvetlenül megfigyelhető eredményekre korlátozódnak.

---

# 34. Különálló, megoldatlan probléma: 16 GB mSATA

Az OEM-azonosság helyreállítása után külön megvizsgáltuk a gépbe szerelt 16 GB-os mSATA SSD-t.

Linux alatt két SATA vezérlő egyértelműen felismerhető:

```text
AMD FCH AHCI
ASMedia ASM1061/ASM1062 AHCI
```

A 240 GB-os Kingston SATA SSD stabilan felismerhető, és:

```text
6.0 Gbps
```

sebességű SATA-linket épít fel.

A 16 GB-os mSATA eszköz ezzel szemben:

- nem jelenik meg az `lsblk` kimenetében;
- nem jut el ATA eszközazonosításig;
- nem jelenik meg blokkos eszközként;
- az érintett SATA útvonalakon `SATA link down` állapot marad.

Az eredeti Fujitsu OEM-blokk helyreállítása ezen a viselkedésen nem változtatott.

Jelenleg ezért nincs bizonyíték arra, hogy az mSATA-probléma ugyanahhoz a firmware-hibához tartozik.

Lehetséges okok:

- hibás mSATA SSD;
- érintkezési probléma;
- hiányzó foglalati tápellátás;
- sérült SATA jelút;
- alaplapi hardverhiba;
- vagy más, jelenleg még nem azonosított konfigurációs mechanizmus.

Ez továbbra is:

**`[OPEN QUESTION]`**

és szándékosan elkülönítve szerepel a már lezárt firmware-helyreállítástól.

---

# 35. Külön hardveres incidens: a SATA-adapter elmozdulása

A vizsgálat során a Kingston SATA SSD-t tartó adapter a gép működése közben mechanikailag elmozdult.

A Linux azonnal többek között:

- ATA bus error;
- BadCRC;
- handshake/link hiba;
- I/O error

üzeneteket naplózott.

A fájlrendszer ennek következtében read-only állapotba is kerülhetett.

Az adapter stabil visszahelyezése és újraindítás után ezek a SATA-hibák eltűntek.

Ezt az incidenst nem szabad összekeverni a különálló mSATA-problémával.

Ugyanakkor egyértelműen megmutatta, hogy az ideiglenes mechanikai rögzítés nem alkalmas tartós használatra.

A tervezett javítás:

```text
M2 nylon/műanyag távtartók
körülbelül 6 mm
```

és megfelelő csavarok alkalmazása.

---

# 36. Módszertani elv: a minimálisan szükséges beavatkozás

A helyreállításból egy általános elv is kirajzolódott:

> **Ne írjunk újra több firmware-t annál, mint amennyiről bizonyítottuk, hogy ténylegesen meg kell változtatni.**

Ebben az esetben:

- rendelkezésre állt a teljes eredeti SPI-image;
- rendelkezésre állt egy teljes, működő recovery utáni image;
- azonosítottuk a gépspecifikus régiót;
- mindössze 3526 byte tért el egyetlen 64 KiB-os blokkon belül;
- ezért a firmware aktuális, működő állapotának többi része érintetlenül maradhatott.

A végső művelet:

```text
64 KiB-ot módosított egy 8 MiB-os flashből
```

Ez csökkenti annak kockázatát, hogy szükségtelenül változzon:

- NVRAM;
- bootváltozó;
- futásidejű állapot;
- recovery által létrehozott adat;
- ismeretlen funkciójú firmware-struktúra.

Ez az elv más firmware-helyreállítási munkafolyamatokban is hasznos lehet.

---

# 37. Reprodukálhatóság

A legerősebb eredményeket további hardvereken is reprodukálni kellene.

Különösen értékes kísérlet lenne egy második D3313-E1 rendszer vizsgálata, amelynél össze lehetne hasonlítani:

- az OEM/manufacturing blokk helyét;
- az FFS GUID-okat;
- a redundáns rekordok struktúráját;
- a gyári template-értékeket;
- az eredeti gép MAC-tárolását;
- a Realtek eFuse állapotát;
- a Fujitsu recovery viselkedését;
- a recovery utáni OEM-állapotot;
- a szelektív OEM-helyreállítás eredményét.

A reprodukció lehetővé tenné annak pontosabb meghatározását, hogy az itt dokumentált működés:

- egyetlen konkrét példányra jellemző;
- a D3313-E1 modell tulajdonsága;
- vagy a szélesebb D3313 platformcsaládra is általánosítható.

---

# 38. Korlátok

A vizsgálatnak több fontos korlátja van.

### Egyetlen elsődleges példány

A teljes vizsgálat egyetlen D3313-E1 rendszeren történt.

Ez nem teszi lehetővé populációszintű általánosítások megfogalmazását.

### A recovery algoritmus nem teljesen ismert

A Fujitsu recovery megfigyelhető eredménye dokumentált, de a belső recovery implementációt még nem fejtettük vissza.

### Részben dokumentálatlan OEM-mezők

Több gépspecifikus mezőnevet azonosítottunk, de pontos funkciójuk jelentős része továbbra sem ismert.

### Nem történt destruktív redundanciateszt

A két látszólag redundáns OEM-struktúrát szándékosan nem sértettük meg azért, hogy megfigyeljük a fallback viselkedést.

### Az mSATA-probléma megoldatlan

A nem detektált 16 GB-os mSATA meghajtót még nem próbáltuk ki függetlenül más rendszerben vagy külső adapterrel.

### Adatvédelmi korlátozások

Az eredeti személyre szabott firmware teljes egészében nem publikálható, mert gépspecifikus adatokat tartalmaz.

Ezeket a korlátokat szándékosan dokumentáljuk, hogy az adat hiánya ne legyen összetéveszthető negatív eredménnyel.

---

# 39. További kutatási irányok

A helyreállított rendszer több további vizsgálathoz is platformot biztosít.

## 39.1. A Fujitsu recovery tényleges írási térképének rekonstruálása

Az egyik legfontosabb nyitott kérdés, hogy a recoveryhez adott forrásimage-ben jelen lévő személyre szabott adat miért nem maradt meg a fizikai SPI-ben.

Kontrollált teszt-image-ek készíthetők, amelyekben nem kritikus régiókban előre meghatározott markerértékeket módosítunk.

A recovery után a fizikai SPI ismét kiolvasható és összehasonlítható.

Ez megmutathatná:

- mely régiókat másolja;
- melyeket hagyja figyelmen kívül;
- melyeket rekonstruálja;
- melyeket őrzi meg;
- és léteznek-e speciális manufacturing szabályok.

Az eredmény egy formális **D3313 recovery write-map** lehetne.

---

## 39.2. Az OEM/manufacturing struktúra teljes visszafejtése

A 64 KiB-os régió láthatóan sokkal több információt tartalmaz, mint pusztán a LAN MAC-cím.

Több rendszer anonimizált dumpjainak összehasonlításával vizsgálható lenne:

- mely mezők állandók;
- melyek modellfüggők;
- melyek alaplapspecifikusak;
- melyek gépenként egyediek;
- vannak-e számlálók;
- milyen integritási mezők vannak;
- melyek feature flag-ek;
- melyek kapcsolódhatnak biztonsági funkciókhoz.

Ez hosszabb távon a D3313 manufacturing adatstruktúra dokumentált specifikációjához vezethet.

---

## 39.3. A redundáns OEM-struktúrák működése

A két kapcsolódó FFS-struktúra további kutatást igényel.

Nyitott kérdések:

- egyikük elsődleges példány?
- a másik tartalék?
- van generációszámláló?
- érvényességi flag alapján választ a firmware?
- képes egyikből rekonstruálni a másikat?
- mi történik, ha csak az egyik példány tartalmaz érvényes MAC-et?

Ezeket célszerű lenne külön, tartalék tesztplatformon vizsgálni, hogy ne veszélyeztessük a már helyreállított rendszert.

---

## 39.4. A NIC inicializálásáért felelős UEFI modul azonosítása

Az OEM-blokk visszaállítása után a helyes Realtek MAC-cím visszatért.

A következő logikus kérdés:

> **Melyik firmware-komponens olvassa ki a Fujitsu OEM-adatokat és írja azokat a Realtek vezérlőbe?**

Lehetséges kutatási módszerek:

- DXE modulok elemzése;
- PCI inicializáció követése;
- string- és GUID-korreláció;
- MMIO-regiszterek vizsgálata;
- emulátorral támogatott elemzés;
- bootidő alatti instrumentáció.

A felelős modul azonosítása a jelenlegi viselkedésalapú bizonyítékot részletes implementációs modellé alakíthatná.

---

## 39.5. Más, OEM-adatokból inicializált hardverfunkciók

A MAC eredménye szélesebb kérdést vet fel.

Ha a hálózati vezérlő gépspecifikus konfigurációt kap az OEM-régióból, akkor lehetséges, hogy más hardverfunkciók is hasonlóan működnek.

Lehetséges vizsgálati célok:

- SATA/mSATA viselkedés;
- TPM konfiguráció;
- HDCP-adatok;
- alaplapi funkciók engedélyezése;
- licencinformációk;
- PCIe eszközkonfiguráció;
- platformazonosság.

Az mSATA problémáját ezért továbbra is külön kell kezelni, miközben az OEM-függő konfiguráció tesztelhető hipotézis maradhat.

---

## 39.6. D3313 E, B és S variánsok összehasonlítása

A D3313 család több változatban létezik.

Keresztplatformos összehasonlítással meghatározható lenne:

- azonosak-e az OEM FFS GUID-ok;
- ugyanazon offseteken található-e a manufacturing régió;
- mennyire kompatibilis az adatstruktúra;
- hasonlóan működik-e a recovery;
- mely dokumentációk használhatók biztonságosan több variáns architekturális referenciájaként.

Ez segíthet elválasztani az általános platformarchitektúrát a konkrét modell implementációjától.

---

## 39.7. Automatizált firmware-differenciálás

A jelenlegi elemzés több lépése automatizálható.

Egy jövőbeli segédprogram például elvégezhetné:

```text
image-méret ellenőrzés
SHA256 számítás
blokk-szintű diff
FV validáció
FFS validáció
OEM string felismerés
gépspecifikus mezők osztályozása
adatvédelmi kockázatok felismerése
flashrom layout generálás
```

Ezzel hasonló firmware-helyreállítások gyorsabbá és kevésbé hibakockázatossá válhatnának.

---

## 39.8. Firmware privacy sanitizer

A projekt bizonyítja, hogy az SPI-image-ek olyan gépspecifikus információkat tartalmazhatnak, amelyeket nem célszerű nyilvánosan terjeszteni.

Külön sanitizer eszköz készülhetne többek között az alábbi adatok felismerésére vagy anonimizálására:

- MAC-cím;
- sorozatszám;
- UUID;
- PC ID;
- manufacturing azonosítók;
- ismert credential-szerű struktúrák.

Egy adatvédelmi szempontból biztonságos firmware-megosztó eszköz a D3313 platformon túl is hasznos lehetne.

---

## 39.9. Recovery hangjelzések elemzése

A recovery folyamat jól elkülöníthető hangjelzéseket adott.

A teljes hangfolyamat pontos időzítéssel történő rögzítése lehetővé tehetné annak vizsgálatát, hogy egyes hangminták megfelelnek-e:

- eszközfelismerésnek;
- törlésnek;
- programozásnak;
- verify fázisnak;
- sikeres befejezésnek;
- hibának.

Ez különösen olyan esetekben lehet hasznos, amikor recovery alatt nincs videokimenet.

---

## 39.10. Hardver- és firmware-hibák szétválasztásának módszertana

A kiinduló helyzet azért volt különösen érdekes esettanulmány, mert egyszerre volt reális a mechanikai sérülés és a firmware-hiba lehetősége.

A kialakult diagnosztikai sor:

```text
fizikai ellenőrzés
        ↓
multiméteres mérések
        ↓
külső SPI-kiolvasás
        ↓
kettős kiolvasás ellenőrzése
        ↓
gyári firmware-rel való összehasonlítás
        ↓
UEFI struktúraelemzés
        ↓
recovery
        ↓
operációs rendszer alatti diagnosztika
        ↓
DOS hardverdiagnosztika
        ↓
vezérlő NVM közvetlen vizsgálata
        ↓
belső SPI-kiolvasás
        ↓
minimális szelektív helyreállítás
```

Ez a folyamat általános keretként is használható lehet olyan beágyazott rendszerek vizsgálatakor, ahol a meghibásodás eredete nem ismert.

---

# 40. Szélesebb firmware-forenzikai jelentőség

A projekt egyik általános tanulsága:

> **A „gyári BIOS-image” és „egy konkrét gép teljes firmware-állapota” nem feltétlenül ugyanaz.**

Egy tényleges rendszer firmware-azonossága több rétegből állhat:

```text
generikus gyártói firmware
gépspecifikus manufacturing adatok
futásidejű NVRAM
hardverazonosság
licencinformáció
biztonsági konfiguráció
platformspecifikus kalibráció
```

Egy generikus gyártói image vak visszaírása ezért helyreállíthatja a bootképességet úgy, hogy közben elvesznek egyedi rendszertulajdonságok.

Ebben az esetben a jól látható elveszett tulajdonság a LAN-azonosság volt.

Más platformokon ilyen egyedi adatok lehetnek például:

- rendszer-sorozatszámok;
- UUID-k;
- Windows/OEM licencadatok;
- kalibrációs értékek;
- TPM konfiguráció;
- alaplapi feature flag-ek;
- hálózati azonosítók.

Firmware-javítás során ezért különbséget kell tenni **a bootolhatóság helyreállítása** és **a teljes gépspecifikus állapot helyreállítása** között.

---

# 41. Biztonsági megfontolások

Az SPI flash módosítása működésképtelenné teheti az alaplapot.

Az itt alkalmazott módszertan ezért erősen preferálja:

- több független kiolvasást;
- kriptográfiai hash-ellenőrzést;
- az eredeti dumpok változatlan archiválását;
- a flash chip pontos azonosítását;
- írás előtti verify műveletet;
- a lehető legkisebb írási területet;
- írás utáni független ellenőrzést.

Az OTP/eFuse programozást külön kell kezelni a normál flash-programozástól.

Visszafordíthatatlan vagy korlátozottan írható tároló módosítása előtt ismerni kell:

- milyen adat kerül beírásra;
- visszaállítható-e a művelet;
- valóban ezt a tárolási mechanizmust használta-e eredetileg az adott platform;
- mennyi írható kapacitás maradt;
- létezik-e kevésbé invazív helyreállítási lehetőség.

---

# 42. A repository adatvédelmi szabályai

A nyilvános repository nem tartalmazhat:

- eredeti személyre szabott SPI-image-et;
- teljes gépsorozatszámot;
- PC ID-t;
- UUID-t;
- teljes gépspecifikus MAC-címet;
- HDCP- vagy más gépspecifikus manufacturing titkot;
- személyes operációsrendszer-felhasználónevet;
- privát helyi fájlútvonalat;
- EXIF/GPS metaadatot;
- olvasható azonosító matricát;
- gépazonosságot tartalmazó QR-kódot vagy vonalkódot.

Minden nyilvánosan közzétett képet, videót, hangfelvételt és firmware-részletet külön adatvédelmi ellenőrzésnek kell alávetni.

Nyers médiafájl közvetlenül nem kerülhet a repositoryba.

---

# 43. Tervezett repository-struktúra

A repository várható felépítése:

```text
/
├── README.md
├── README.hu.md
├── LICENSE
├── CITATION.cff
├── SOURCES.md
├── DISCLAIMER.md
├── docs/
│   ├── en/
│   └── hu/
├── scripts/
├── examples/
└── media/
    └── public/
```

Személyre szabott firmware-image soha nem kerülhet a nyilvános könyvtárstruktúrába.

---

# 44. Korábbi munkák és forrásmegjelölés

A projekt **nem állítja**, hogy a firmware-recovery, UEFI-elemzés, SPI-programozás vagy Realtek-diagnosztika alaptechnikáit ez a munka találta fel.

A vizsgálat már létező gyártói dokumentációkra, nyílt forrású eszközökre és korábbi közösségi kutatásokra épít.

A fontosabb előzményterületek:

- Fujitsu D3313 dokumentáció;
- Fujitsu BIOS- és recovery-anyagok;
- kapcsolódó D3313 platformok későbbi Kontron dokumentációja;
- `flashrom` projekt;
- Realtek RSET8168;
- Realtek PGTool dokumentáció és eszközök;
- Fujitsu FUTRO rendszerekkel kapcsolatos közösségi munka;
- korábbi D3313 firmware-módosítási és elemzési projektek;
- általános UEFI reverse engineering és firmware-repair kutatások.

A repository külön:

```text
SOURCES.md
```

fájlban fogja tartalmazni a konkrét szerzői hivatkozásokat és közvetlen forráslinkeket.

A projekt célja nem annak állítása, hogy az alkalmazott módszerek itt születtek.

A hozzájárulás a már ismert módszerek **dokumentált kombinálása, független ellenőrzése és alkalmazása egy konkrét D3313-E1 helyreállítási esetben, valamint az ebből származó platformspecifikus megfigyelések rögzítése**.

---

# 45. Az esettanulmány kutatási hozzájárulása

A munka legfontosabb eredménye nem pusztán az, hogy egy számítógép újra működik.

A projekt dokumentált kapcsolatot teremt:

```text
hivatalos firmware
        ↕
személyre szabott gép-firmware
        ↕
manufacturing adatok
        ↕
firmware recovery
        ↕
platformazonosság elvesztése
        ↕
vezérlőszintű diagnosztika
        ↕
közvetlen NVM-vizsgálat
        ↕
recovery utáni fizikai SPI-állapot
        ↕
minimális szelektív helyreállítás
        ↕
eredeti gépazonosság visszaállítása
```

Ez a bizonyítéki lánc több újrahasznosítható megfigyelést ad:

- konkrét helyet a D3313-E1 gépspecifikus adatainak;
- bizonyítékot arra, hogy a gyári firmware személyre szabási template-et tartalmaz;
- bizonyítékot arra, hogy a vizsgált NIC eFuse-ja nem volt programozva;
- bizonyítékot arra, hogy a platformfirmware képes a NIC azonosságát inicializálni;
- bizonyítékot arra, hogy a Fujitsu recovery nem egyszerű nyers full-chip másolásként működik;
- demonstrált minimális régiós helyreállítási módszert.

Ezek az eredmények hasznosak lehetnek olyan kutatásokhoz, amelyek témája:

- Fujitsu D3313 firmware-architektúra;
- beágyazott rendszerek személyre szabása;
- firmware-ben tárolt hardverazonosság;
- firmware recovery mechanizmusok;
- EEPROM nélküli PCIe-eszközök inicializációja;
- BIOS-forenzika;
- régi ipari rendszerek digitális megőrzése.

---

# 46. Végkövetkeztetés

A vizsgálat kezdetén nem lehetett meghatározni, hogy a FUTRO S930 hibájának oka:

- mechanikai alaplapi sérülés;
- processzor környéki sérülés;
- tápellátási hiba;
- SPI-firmware sérülés;
- vagy ezek kombinációja.

A végállapot lényegesen jobban ismert.

A rendszer jelenleg:

- sikeresen POST-ol;
- működő BIOS Setupot biztosít;
- Debian Linuxot indít;
- Ethernet-hálózaton működik;
- tartalmazza a helyreállított, gépspecifikus Fujitsu OEM-régiót;
- és ismét az eredeti Fujitsu LAN-azonosságot inicializálja.

A recovery utáni MAC-hibát nem Realtek eFuse-programozással oldottuk meg.

A közvetlen vizsgálat alapján az eFuse látszólag korábban soha nem volt programozva.

A döntő információ ehelyett a Fujitsu platform gépspecifikus SPI/OEM-régiójában található.

A végső javítási folyamat ezért:

```text
megbízható recovery utáni SPI-kiolvasás
        ↓
összehasonlítás az eredeti firmware-rel
        ↓
egyetlen 64 KiB-os gépspecifikus régió izolálása
        ↓
fizikai verify írás előtt
        ↓
szelektív in-system flash írás
        ↓
független ellenőrzés írás után
        ↓
hidegindítás
        ↓
eredeti platformazonosság helyreállítása
```

sorozatból állt.

A 16 GB-os mSATA felismerési problémája továbbra is külön dokumentált, nyitott kérdés, és jelenleg nem tekintjük a sikeresen lezárt firmware-helyreállítás részének.

A BIOS-helyreállítás és a Fujitsu gépspecifikus OEM-azonosság visszaállítása ezért:

# **SIKERESEN LEZÁRULT**

A következő fázis már elsősorban nem javítási feladat.

A helyreállított platform, a felhalmozott mérési eredmények, a firmware-összehasonlítások és a reprodukálható bizonyítéki lánc alapot biztosítanak **a D3313 firmware-architektúrájának, manufacturing adatmodelljének és recovery mechanizmusának további kutatásához**.

---

## Jogi és biztonsági nyilatkozat

Ez a repository saját tulajdonú és ellenőrzött hardveren végzett kísérleti firmware- és hardvermunkát dokumentál.

A firmware módosítása, SPI-programozás, recovery jumper használata és alaplapi beavatkozás véglegesen károsíthatja a hardvert vagy megsemmisítheti a gépspecifikus adatokat.

Az anyag kutatási, dokumentációs, megőrzési és oktatási céllal kerül közzétételre.

Hasonló eljárások végrehajtásakor mindenki saját felelősségére jár el, és saját feladata megbízható mentések készítése, valamint a konkrét hardverrel kapcsolatos kockázatok megértése.

---

## Jelenlegi kutatási állapot

```text
BIOS recovery                          LEZÁRVA
POST helyreállítása                    LEZÁRVA
Debian boot                            LEZÁRVA
Belső SPI-kiolvasás                    LEZÁRVA
OEM-régió azonosítása                  LEZÁRVA
Eredeti LAN-azonosság visszaállítása   LEZÁRVA
Szelektív SPI-írás ellenőrzése         LEZÁRVA
Realtek eFuse vizsgálata               LEZÁRVA
Fujitsu recovery algoritmus            NYITOTT
OEM-struktúra teljes visszafejtése     NYITOTT
Második alaplapon reprodukció          NYITOTT
16 GB mSATA vizsgálata                 NYITOTT
```
