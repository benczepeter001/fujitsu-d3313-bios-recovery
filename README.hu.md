# Fujitsu FUTRO S930 / D3313-E1

## BIOS-helyreállítás, firmware-elemzés és a gépspecifikus OEM-azonosság visszaállítása

### Forenzikus szemléletű műszaki esettanulmány egy Fujitsu D3313-E1 alaplap firmware-helyreállításáról és platformazonosságának rekonstrukciójáról

> **Állapot:** BIOS-helyreállítás sikeres · gépspecifikus OEM-azonosság visszaállítva · eredeti LAN-azonosság visszaállítva · a különálló mSATA-probléma vizsgálata folyamatban

[English version](README.md)

---

## Projekt-navigáció

- [Angol dokumentáció](README.md)
- [Források és korábbi munkák](SOURCES.md)
- [Jogi és kutatási nyilatkozat](DISCLAIMER.md)
- [Magyar jogi nyilatkozat](DISCLAIMER.hu.md)
- [Licenc](LICENSE)
- [Hivatkozási metaadatok](CITATION.cff)

---

# Összefoglaló

Ez a repozitórium egy Fujitsu D3313-E1 alaplapra épülő Fujitsu FUTRO S930 vékonykliens forenzikus szemléletű vizsgálatát és helyreállítását dokumentálja.

A rendszer működésképtelen állapotban került vizsgálatra. A rendelkezésre álló előzmény szerint közvetlenül a meghibásodás előtt processzor-újrapasztázási kísérlet történt, amelynek során a hűtőegységet és az alaplap hátoldalán található fém rögzítőlemezt eltávolították. Mivel a processzor környezetét a visszaszerelési próbálkozások során jelentős mechanikai nyomás érhette, a kezdeti hibát nem lehetett kizárólag firmware-problémának tekinteni.

Mechanikai sérülés, tápellátási hiba, alaplapi meghibásodás, sérült SPI-firmware vagy ezek kombinációja egyaránt lehetséges magyarázat maradt.

A vizsgálat ezért nem a hagyományos „írjuk újra a BIOS-t és nézzük meg” megközelítést követte, hanem bizonyíték-alapú módszertant.

A folyamat része volt:

- ismételt alaplapi elektromos mérés;
- külső SPI-kiolvasás CH341A programozóval;
- két egymástól független 8 MiB-os firmware-kiolvasás;
- kriptográfiai hash-ellenőrzés;
- összehasonlítás hivatalos Fujitsu firmware-rel;
- UEFI Firmware Volume és FFS-elemzés;
- Fujitsu recovery-mode kísérletek;
- DOS-alapú Realtek-diagnosztika;
- közvetlen Realtek eFuse-vizsgálat;
- Linux alatti belső SPI-kiolvasás;
- firmware blokkszintű összehasonlítás;
- valamint egyetlen 64 KiB-os régió szelektív, működő rendszerből történő visszaállítása.

A vizsgálat során több jelentős eredmény született.

Az eredeti gép SPI-image-e és a megfelelő hivatalos Fujitsu R1.16 firmware **128 darab 64 KiB-os blokkból 127-ben teljesen azonosnak bizonyult**.

A teljes gépspecifikus eltérés egyetlen 64 KiB-os OEM/manufacturing régióban koncentrálódott:

```text
0x440000–0x44FFFF
```

Ebben a régióban több gépspecifikus mező szerepelt, köztük két olyan struktúra, amely tartalmazta:

```text
MAC Address #0
```

mezőt.

A hivatalos firmware-ben ennek értéke:

```text
FF:FF:FF:FF:FF:FF
```

volt, míg az eredeti gép dumpjában a tényleges Fujitsu által kiosztott Ethernet-azonosító szerepelt.

A Fujitsu BIOS-helyreállítás után a rendszer ismét POST-olt és bootolt, de a hálózati vezérlő elvesztette állandó MAC-azonosságát.

A DOS-alapú Realtek-diagnosztika nem talált érvényes külső EEPROM-ot, míg a közvetlen Linux PGTool-vizsgálat látszólag soha nem programozott eFuse-t mutatott:

```text
Efuse Write Count = 0
```

A helyreállítás utáni belső SPI-dump ezután azt mutatta, hogy a gépspecifikus OEM MAC-mező visszaállt a gyári `FF` sablonértékre.

A Realtek eFuse végleges programozása helyett megőrzésre került a működő recovery-állapotú firmware, és kizárólag az eredeti 64 KiB-os OEM-régió került visszaírásra.

A szelektív írás fizikai flash-szinten külön ellenőrzésre került.

Hidegindítás után a BIOS ismét az eredeti Fujitsu LAN-azonosságot mutatta, Linux alatt pedig helyreállt a megfelelő állandó hálózati identitás.

A bizonyítékok ezért azt a modellt támasztják alá, hogy a vizsgált D3313-E1 rendszerben a Realtek vezérlő gépspecifikus Ethernet-azonosságát a Fujitsu platformfirmware biztosítja, nem pedig korábban programozott Realtek eFuse.

A projekt általánosabb firmware-javítási tanulsága:

> **Mindig a lehető legkisebb, bizonyítottan helyreállításra szoruló régiót módosítsuk, ne pedig alapértelmezésként az egész firmware-image-et írjuk újra.**

---

# Kulcsszavak

Fujitsu D3313-E1 · Fujitsu FUTRO S930 · BIOS-helyreállítás · UEFI · SPI flash · firmware forenzika · OEM-adatok · manufacturing adatok · flashrom · CH341A · MX25L6473F · Realtek RTL8168G · MAC-cím · eFuse · firmware-helyreállítás · forenzikus firmware-elemzés

---

# 1. Hatókör és célok

A projekt eredetileg egy látszólag meghibásodott Fujitsu FUTRO S930 javításaként indult.

A munka strukturált firmware-vizsgálattá vált, amikor világossá vált, hogy a bootképesség helyreállítása önmagában nem állítja vissza automatikusan az adott gép eredeti identitását és konfigurációját.

A fő célok:

1. megállapítani, hogy firmware-rel kapcsolatos hiba bizonyítható-e anélkül, hogy mechanikai vagy elektromos okokat idő előtt kizárnánk;
2. megbízható másolatot készíteni az eredeti SPI-firmware-ről;
3. összehasonlítani a gép firmware-ét a megfelelő hivatalos Fujitsu firmware-rel;
4. azonosítani a gépspecifikus régiókat;
5. lehetőség szerint meghatározni ezek funkcióját;
6. visszaállítani a POST-ot és a normál bootképességet;
7. kivizsgálni az állandó hálózati identitás recovery utáni elvesztését;
8. megállapítani, hogy ez az identitás Realtek EEPROM/eFuse-ból vagy Fujitsu platformfirmware-ből származik-e;
9. visszaállítani az eredeti gépspecifikus konfigurációt a lehető legkisebb SPI-régió módosításával;
10. megőrizni egy reprodukálható bizonyítási láncot.

Ez a repozitórium tehát **nem általános BIOS-flashelési útmutató**.

Inkább:

> **bizonyíték-alapú firmware-helyreállítási és platformidentitás-rekonstrukciós esettanulmány.**

---

# 2. Bizonyítéktípusok

Azért, hogy a mérések ne keveredjenek az értelmezéssel, a megállapítások több kategóriába sorolhatók.

### `[OFFICIAL]`

Gyártói dokumentációból, hivatalos firmware-csomagból, gyártói segédprogramból vagy más elsődleges műszaki forrásból származó információ.

### `[OBSERVED]`

A vizsgált hardveren közvetlenül mért, kiolvasott, naplózott, kinyert vagy reprodukált adat.

### `[INFERRED]`

Több megfigyeléssel erősen alátámasztott, de közvetlenül nem mért következtetés.

### `[REPORTED HISTORY]`

A gép állapotával vagy korábbi kezelésével kapcsolatos, a strukturált vizsgálat előtt keletkezett információ.

### `[OPEN QUESTION]`

További bizonyítékot vagy reprodukciót igénylő hipotézis vagy nem tisztázott viselkedés.

Firmware-kutatás során gyakran alakul ki hihető magyarázat még azelőtt, hogy azt ténylegesen bizonyítani lehetne.

A repozitórium egyik célja ezért az elkülönítés:

- mi történt;
- mit mértünk;
- mit közöltek korábban;
- és mit valószínűsít jelenleg a bizonyíték.

---

# 3. Vizsgált platform

A vizsgált rendszer:

**Fujitsu FUTRO S930**

alaplapja:

**Fujitsu D3313-E1**

A vizsgálat szempontjából releváns hardverek:

- AMD GX-424CC négymagos beágyazott processzor;
- Fujitsu D3313-E1 alaplap;
- AMI Aptio-alapú firmware;
- 8 MiB Macronix SPI flash;
- fizikailag azonosított MX25L6473F-családba tartozó flash;
- Realtek RTL8111/8168G-családba tartozó Gigabit Ethernet vezérlő;
- AMD FCH SATA vezérlő;
- ASMedia ASM1061/ASM1062 SATA vezérlő;
- 240 GB Kingston SATA SSD;
- 16 GB mSATA SSD;
- Debian Linux.

A vizsgálat során azonosított firmware-verzió:

```text
V4.6.5.4 R1.16.0 for D3313-E1x
```

A gépspecifikus értékeket – többek között sorozatszámot, PC ID-t, UUID-t, MAC-címet, HDCP-adatokat és más gyártási azonosítókat – a nyilvános repozitórium szándékosan nem tartalmazza.

---

# 4. Kezdeti hibállapot

A rendszer már működésképtelen volt, amikor a strukturált vizsgálat megkezdődött.

A rendelkezésre álló előzmény szerint közvetlenül a meghibásodás előtt processzor-újrapasztázás történt.

A visszaszerelés során az alaplap mögötti fém rögzítőlemez elmozdult a helyéről. A hűtőegységet ezt követően több alkalommal nyomták és igazították vissza.

Ez az információ:

```text
[REPORTED HISTORY]
```

kategóriába tartozik.

Mivel mechanikai terhelés történt, nem lehetett biztonsággal feltételezni, hogy kizárólag firmware-hiba áll fenn.

Lehetséges kezdeti hibaforrások:

- PCB-sérülés;
- sérült forrasztási pont;
- processzor körüli alkatrészsérülés;
- VRM- vagy tápellátási hiba;
- rövidzárlat;
- mechanikai deformáció;
- sérült SPI-tartalom;
- érvénytelen firmware-állapot;
- vagy több hiba kombinációja.

Az elérhető alaplapi feszültségpontok és tápvonalak ismételten mérésre kerültek.

Nem sikerült olyan egyértelmű elektromos hibát azonosítani, amely önmagában megmagyarázta volna a teljes bootképtelenséget.

Ez **nem bizonyította**, hogy az alaplap mechanikailag sértetlen.

Arra viszont elegendő volt, hogy a firmware-vizsgálatot folytassuk, miközben a hardveres hipotéziseket továbbra is nyitva hagytuk.

---

# 5. Külső SPI-kiolvasás

A kezdeti firmware-kiolvasás egy olcsó WCH CH341A USB SPI-programozóval és SOIC8 csipesszel történt.

Programozóazonosítás:

```text
VID: 1A86
PID: 5512
Device: WCH CH341A
```

Windows alatt több driver és programozóeszköz is kipróbálásra került.

A `flashrom` USB-hozzáférése ebben a környezetben megbízhatatlan volt, többek között ilyen hibával:

```text
LIBUSB_ERROR_NOT_SUPPORTED
```

Az IMSProg sikeresen felismerte:

```text
WCH CH341A
Device rev 3.0.4
```

A legnagyobb gyakorlati probléma nem maga a programozó, hanem a SOIC8 csipesz mechanikai megbízhatatlansága volt.

A csipesz elmozdulhatott vagy elveszíthette az elektromos kontaktust.

Ezért egyetlen látszólag sikeres kiolvasás nem számított elegendő bizonyítéknak.

---

# 6. Kettős kiolvasás és integritás-ellenőrzés

Két egymástól független teljes SPI-kiolvasás készült.

Mindkét image mérete:

```text
8,388,608 byte
```

azaz 8 MiB volt.

Mindkét fájl SHA256 hash-e:

```text
7E138DA4B68FB7C86DBD841631D6D60937AF2EA109297A51476610C8F01E3603
```

A két azonos digest bizonyítja, hogy a két egymástól függetlenül elkészített firmware-kép bitre pontosan megegyezett.

A hash a proveniencia és reprodukálhatóság egyik nyilvános azonosítója.

A személyre szabott SPI-image maga szándékosan nem kerül nyilvános terjesztésre, mert gépspecifikus gyártási és identitásadatokat tartalmaz.

A módszertan:

```text
KIOLVASÁS
   ↓
ÚJRA KIOLVASÁS
   ↓
ÖSSZEHASONLÍTÁS
   ↓
HASH
   ↓
ELEMZÉS
   ↓
CSAK EZUTÁN ÍRÁS
```

Ugyanez az elv később a helyreállítás utáni belső SPI-kiolvasásnál is alkalmazásra került.

---

# 7. Összehasonlítás a hivatalos Fujitsu firmware-rel

A megfelelő hivatalos Fujitsu R1.16 firmware külön került beszerzésre.

A központi kérdés:

> **Miben különbözik egy legyártott D3313-E1 tényleges firmware-e a Fujitsu által terjesztett generikus firmware-től?**

A teljes 8 MiB-os image egyetlen bináris objektumként történő kezelése helyett mindkét firmware 64 KiB-os blokkokra lett bontva.

Egy 8 MiB-os image:

```text
128 × 64 KiB blokk
```

méretű.

Az eredmény meglepően egyértelmű volt.

---

# 8. Fő strukturális eredmény: 127/128 blokk egyezett

Az eredeti gép SPI-dumpja és a hivatalos Fujitsu R1.16 firmware:

**128 blokkból 127-ben teljesen azonos volt.**

Csak egyetlen 64 KiB-os régió tért el:

```text
0x440000–0x44FFFF
```

Ez alapvetően megváltoztatta a vizsgálat irányát.

A konkrét gép firmware-e nem általánosan különbözött a gyári kiadástól.

A teljes 8 MiB-os image szinte teljes egészében megegyezett a hivatalos firmware-rel.

A gépspecifikus eltérés egyetlen kis régióban koncentrálódott.

Egy további ellenőrzés is készült.

A hivatalos Fujitsu R1.16 image másolatában kizárólag:

```text
0x440000–0x44FFFF
```

került lecserélésre az eredeti gép megfelelő blokkjára.

Az így létrejött teljes 8 MiB-os image bitre pontosan megegyezett az eredeti gép SPI-image-ével.

A kapcsolat tehát:

```text
Hivatalos Fujitsu R1.16 firmware
              +
eredeti gépspecifikus 64 KiB OEM-blokk
              =
teljes eredeti gép SPI-image
```

Ez a vizsgált rendszeren a projekt egyik legerősebb strukturális eredménye.

---

# 9. UEFI Firmware Volume és FFS-elemzés

A firmware UEFI Firmware Volume és Firmware File System szinten is elemzésre került.

Az elemzett image:

```text
8 érvényes Firmware Volume
84 FFS fájl
0 hibás FV header
0 hibás FFS checksum
```

eredményt adott.

Ez további bizonyítékot jelentett arra, hogy a kiolvasott image belső szerkezete koherens.

A gépspecifikus eltérések elsősorban az alábbi címek körül koncentrálódtak:

```text
0x440000
```

és:

```text
0x443000
```

Megfigyelt FREEFORM FFS GUID-ok:

```text
0972EB29-E62F-4699-A19C-664AC3B7208B
```

és:

```text
8EEAF5CF-1C81-4FCC-AE1E-D8AD3CD15683
```

A két struktúra erősen hasonló.

Valószínűleg ugyanazon gépspecifikus gyártási információ redundáns vagy párhuzamos reprezentációját tartalmazzák.

A pontos redundanciamodell még nem ismert.

Lehetséges értelmezések:

- elsődleges és tartalék példány;
- aktív és standby rekord;
- generációalapú másolat;
- verziózott gyártási struktúra;
- integritási/fallback struktúra;
- vagy más gyártóspecifikus mechanizmus.

Ez továbbra is nyitott kutatási irány.

---

# 10. OEM/manufacturing adatstruktúrák

A gépspecifikus régióban felismerhető ASCII mezőnevek között szerepelt:

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

Egyes mezők funkciója viszonylag egyértelmű.

Mások részben vagy teljesen dokumentálatlanok.

A legfontosabb architekturális megfigyelés az, hogy a hivatalos Fujitsu firmware több mezőben sablon- vagy programozatlan értéket tartalmaz, míg a konkrét gép firmware-e kitöltött manufacturing adatokat hordoz.

Ez a modellt támogatja:

```text
generikus gyártói firmware
          +
gyártás közbeni személyre szabás
          =
egy konkrét gép firmware-állapota
```

Ezért egy nyilvános gyártói firmware-image nem tekinthető automatikusan egy konkrét legyártott gép teljes firmware-rekonstrukciójának.

---

# 11. A gépspecifikus MAC-mező azonosítása

Mindkét OEM-struktúra tartalmazta:

```text
MAC Address #0
```

mezőt, amelyet hat byte követett.

Az eredeti gép dumpjában mindkét másolat ugyanazt a gépspecifikus Ethernet-címet tartalmazta.

A tényleges értéket a nyilvános repozitórium így jelöli:

```text
<REDACTED_FUJITSU_MAC>
```

A hivatalos R1.16 firmware megfelelő mezője:

```text
FF FF FF FF FF FF
```

vagy:

```text
FF:FF:FF:FF:FF:FF
```

volt.

A hivatalos firmware tehát nem egy generikus Fujitsu MAC-ot tartalmazott.

Hanem egy **programozatlan manufacturing sablonértéket**.

Az egyedi Ethernet-azonosság a gyártás vagy személyre szabás során került a gépbe.

---

# 12. SPI-dumpok adatvédelmi és forenzikus jelentősége

Egy nyers SPI-image-et nem szabad automatikusan egyszerű „BIOS-fájlként” kezelni.

Tartalmazhat:

- sorozatszámokat;
- PC ID-ket;
- UUID-ket;
- MAC-címeket;
- HDCP-hez kapcsolódó adatokat;
- platformazonosítókat;
- konfigurációs adatokat;
- gyártási metaadatokat;
- más gépspecifikus értékeket.

Ezért a nyilvános repozitórium **nem tartalmazza**:

- az eredeti személyre szabott SPI-dumpot;
- személyre szabott recovery ROM-ot;
- valódi sorozatszámokat;
- valódi PC ID-ket;
- teljes UUID-ket;
- teljes gépspecifikus MAC-címet;
- HDCP-hez kapcsolódó adatokat;
- más gépspecifikus manufacturing titkokat.

Csak anonimizált példák, strukturális leírások, hash-ek, offsetek és megfelelően kitakart anyagok kerülhetnek nyilvánosságra.

---

# 13. Fujitsu BootStick vizsgálat

A Fujitsu BootStick mechanizmus külön vizsgálatra került.

A segédprogram egy hagyományos FreeDOS-alapú boot adathordozót hozott létre, amely tartalmazta:

- MBR-t;
- egy aktív FAT32 partíciót;
- FreeDOS bootkódot;
- `KERNEL.SYS` fájlt;
- `COMMAND.COM` fájlt.

Egy frissen generált BootStick alapértelmezett `AUTOEXEC.BAT` fájlja nem tartalmazott automatikus BIOS-flashelési parancsot.

Ez segített elkülöníteni:

1. a Fujitsu BootStick által létrehozott általános bootkörnyezetet;
2. a tényleges, alaplapspecifikus firmware-helyreállítási logikát.

A sikeres helyreállítás során végül egy 2 GB-os USB flash meghajtó került használatra.

Egy hátlapi USB 2.0 port lett kiválasztva a változók csökkentése érdekében.

Nem állítható, hogy ez a port elektromosan „rövidebb”, kizárólagos recovery-port, vagy hivatalosan ilyen célra kijelölt.

Ez gyakorlati kompatibilitási döntés volt.

---

# 14. Recovery jumper konfiguráció

A vizsgált D3313-E1-en megfigyelt jumperelrendezés:

```text
5     3     1
●     ●     ●

●           ●
6    [ ]    2
      ↑
 hiányzó pin
   4. pozíció
```

Megfigyelt pozíciók:

```text
DEFAULT   J5–3
RESERVED  J1–2
RECOVERY  J5–6
```

Egy korábbi hibakeresési szakaszban az alaplap véletlenül többször is a RESERVED állásban kapott tápot.

Ezen az egy példányon nem volt megfigyelhető ehhez egyértelműen köthető visszafordíthatatlan károsodás.

Ez **nem jelenti azt**, hogy a RESERVED pozíció biztonságos.

A funkciója nincs teljesen reverse engineeringgel feltárva, ezért használata nem ajánlott.

---

# 15. Sikeres recovery-konfiguráció

A firmware-helyreállítás szándékosan minimális hardverkonfigurációval történt.

Csatlakoztatott elemek:

- alaplap;
- processzor;
- egy RAM-modul DIMM1-ben;
- CMOS-elem;
- buzzer;
- kijelző;
- billentyűzet;
- recovery USB.

A nem szükséges adattárolók és perifériák eltávolításra kerültek.

A jumper RECOVERY állásba került.

Recovery közben periodikus rövid sípolások voltak megfigyelhetők.

A folyamat látszólagos befejezésekor az ismétlődő minta:

```text
hosszú – hosszú – rövid – rövid
```

volt.

Ezután a gép:

1. kikapcsolásra került;
2. áramtalanítás történt;
3. a recovery adathordozó eltávolításra került;
4. a jumper visszakerült DEFAULT állásba.

A következő normál bootkor visszatért a videokimenet, és elérhetővé vált a BIOS Setup.

A korábban nem működő alaplap ismét POST-olt.

A BIOS-helyreállítás sikerült.

---

# 16. Recovery utáni hálózati identitáshiba

A POST helyreállítása után egy új rendellenesség vált láthatóvá.

A BIOS a Realtek interfész MAC-címét így mutatta:

```text
00:00:00:00:00:00
```

A Debian ennek ellenére elindult, a hálózat pedig működött.

Az interfész fizikai linket hozott létre, hálózati konfigurációt kapott, és SSH-n elérhető volt.

A Linux naplók azonban megmutatták, hogy nem pusztán megjelenítési hibáról van szó.

Az `r8169` driver ezt jelezte:

```text
can't read MAC address, setting random one
```

Az `ethtool` pedig:

```text
Permanent address: not set
```

eredményt adott.

Az operációs rendszer tehát nem kapott érvényes állandó hardver-MAC-ot.

A hálózat azért működött, mert Linux ideiglenes címet generált.

A gép visszanyerte a hálózati működést, de elvesztette eredeti hardveres hálózati azonosságát.

---

# 17. Az operációs rendszer és a hardverállapot szétválasztása

A következő kérdés:

> **Ez csak Linux-driver probléma?**

Ennek tesztelésére a vizsgálat kikerült a Debian normál hálózati stackjéből.

DOS-alapú Realtek diagnosztikai eszköz került használatra a vezérlő független ellenőrzéséhez.

---

# 18. Realtek RSET8168 diagnosztika

A Realtek DOS-programja:

```text
RSET8168 v1.24
```

FreeDOS alól indult.

A státuszképernyő ezt mutatta:

```text
Ethernet Address: 00 00 00 00 00 00
```

A MAC-regiszter nézet szintén nullákat tartalmazott.

A hiányzó identitás tehát már Linuxon kívül is hiányzott.

---

# 19. EEPROM-diagnosztikai eredmény

Az RSET8168 ezt jelezte:

```text
No Valid External EEPROM!!
```

majd:

```text
Data from EFuse
```

és:

```text
EFuse Fail
```

Egy új hipotézis vált hihetővé.

Ha nincs érvényes külső EEPROM, lehetséges, hogy a rendszer normál esetben az RTL8168G belső eFuse-ában tárolja az Ethernet-azonosságot.

Ebben az esetben a közvetlen Realtek-programozás lehetett volna javítási út.

Ekkor azonban ez még nem volt bizonyított.

---

# 20. Közvetlen Realtek eFuse-vizsgálat

A vezérlő közvetlen vizsgálatához Linux alatt Realtek PGTool-kompatibilis eszköz került használatra.

A toolset többek között tartalmazta:

```text
8168GEF.cfg
```

konfigurációt az RTL8168G-családhoz.

A saját kernelmodul elavult Linux API-t használt.

Az eredeti:

```c
vma->vm_flags |= VM_IO;
```

kód Debian 6.12 kernelhez így került módosításra:

```c
vm_flags_set(vma, VM_IO);
```

Ezt követően a modul lefordult és betöltődött.

A normál `r8169` driver eltávolításra került, és a vezérlő a PGTool driveren keresztül vált közvetlenül elérhetővé.

---

# 21. Kritikus eFuse-eredmény

A kiolvasható eFuse-terület kizárólag:

```text
FF
```

értékeket tartalmazott.

Ennél is fontosabb:

```text
Efuse Write Count = 0
```

volt az eredmény, miközben a programozható eFuse-kapacitás lényegében teljes egészében szabad maradt.

Ez fontos különbség.

A kizárólag `FF` értékek önmagukban jelenthetnének törölt vagy hiányzó adatot.

A nulla írásszám azonban erősen arra utal, hogy a régiót **soha nem programozták**.

Az EEPROM-útvonal ezt is jelezte:

```text
Not Support This Adapter
```

Ezek együtt erősen ellene szóltak annak a feltételezésnek, hogy az eredeti Fujitsu MAC korábban tartósan Realtek EEPROM-ban vagy eFuse-ban lett volna eltárolva.

Egy új modell vált lényegesen valószínűbbé:

> **A Fujitsu firmware a hálózati vezérlőt az alaplapi SPI-ben tárolt gépspecifikus platformadatból inicializálja.**

Ehhez azonban még össze kellett hasonlítani a tényleges, recovery utáni fizikai SPI-tartalmat.

---

# 22. Miért nem programoztuk a Realtek eFuse-t

A PGTool környezet képes lett volna node ID/MAC programozására nem felejtő vezérlőtárolóba.

Az eredeti gép MAC-címe ismert volt.

Az eFuse üresnek látszott.

A programozás gyors workaround lehetett volna.

Ezt az opciót szándékosan elvetettük.

Az eFuse-programozás:

- tartós;
- eltér a normál újraírható flash használatától;
- véges programozható kapacitással rendelkezik;
- potenciálisan visszafordíthatatlan;
- és nem volt bizonyított, hogy az eredeti Fujitsu megoldás ezt használta.

A cél nem egyszerűen az volt:

> „legyen újra permanens MAC.”

Hanem:

> **állítsuk vissza azt a mechanizmust, amelyet az eredeti gép használt.**

Ezért a Realtek eFuse programozása csak tartalék lehetőség maradt.

---

# 23. Belső SPI-hozzáférés Linux alatt

Ekkorra a külső SOIC8 csipeszes kábel használhatatlanná vált.

Mivel a számítógép már bootolt, belső SPI-hozzáférés került tesztelésre Debian alatt.

A:

```text
flashrom -p internal
```

parancs azonosította:

```text
AMD FCH
```

chipsetet.

Több kompatibilis Macronix definíció jelent meg.

A fizikailag azonosított flashcsomag alapján explicit:

```text
MX25L6473F
```

került kiválasztásra.

Két teljes belső 8 MiB-os kiolvasás készült.

SHA256 hash-ük megegyezett.

A bizonyítási elv tehát ismét teljesült:

```text
két független kiolvasás
        +
azonos kriptográfiai hash
        =
megbízható recovery utáni SPI-baseline
```

A tényleges fizikai recovery utáni flashállapotról így megbízható image állt rendelkezésre.

---

# 24. Döntő recovery utáni OEM-megfigyelés

A két:

```text
MAC Address #0
```

struktúra ismét megtalálható volt a recovery utáni SPI-dumpban.

Strukturálisan mindkettő megmaradt.

A hozzájuk tartozó hat byte azonban:

```text
FF FF FF FF FF FF
```

értékre változott.

Ez pontosan megfelelt a hivatalos Fujitsu R1.16 sablonértékének.

A három releváns állapot így hasonlítható össze.

### Eredeti gép SPI

```text
MAC Address #0 → <REDACTED_FUJITSU_MAC>
```

### Hivatalos Fujitsu R1.16

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

### Fujitsu recovery utáni fizikai SPI

```text
MAC Address #0 → FF:FF:FF:FF:FF:FF
```

Ez összekötötte a korábban külön kezelt tüneteket:

- nullás BIOS MAC;
- nulla Ethernet-cím RSET8168 alatt;
- Linux `Permanent address: not set`;
- Linux által generált ideiglenes MAC;
- látszólag soha nem programozott Realtek eFuse.

A vezérlő nem feltétlenül egy saját belső MAC-ot veszített el.

A Fujitsu platformfirmware nem tartalmazta többé a vezérlő inicializálásához szükséges gépspecifikus értéket.

---

# 25. Mit jelent ez a Fujitsu recovery működésére nézve

Az eredményt óvatosan kell értelmezni.

A bizonyíték **nem** igazolja azt az egyszerűsített állítást, hogy:

> „A Fujitsu recovery egyszerűen felülírja az OEM-régiót a gyári image-dzsel.”

A valós megállapítás szűkebb.

A recovery forrásfirmware tartalmazott személyre szabott adatot.

A recovery után a fizikai SPI releváns OEM-struktúrái mégis gyári sablonértéket tartalmaztak.

Ezért a recovery folyamat **nem egyszerű, bitre pontos teljes-chip másolás** a megadott 8 MiB-os forrásból.

Valamilyen további recovery-logika működik.

Lehetséges modellek:

- szelektív Firmware Volume írás;
- manufacturing régió újragenerálása;
- egyes személyre szabott struktúrák kihagyása;
- OEM-régiók speciális kezelése;
- belső komponensből történő rekonstrukció;
- más, alaplapspecifikus szabály.

A pontos működés:

```text
[OPEN QUESTION]
```

marad.

---

# 26. Miért nem írtuk vissza a teljes régi SPI-image-et

Rendelkezésre állt a teljes eredeti 8 MiB-os dump.

A legegyszerűbbnek tűnő javítás tehát az egész történeti image visszaírása lett volna.

Ezt szándékosan elvetettük.

Az eredeti és a recovery utáni image összehasonlítása több 64 KiB-os blokkban is eltérést mutatott, többek között:

```text
2
54
64
65
66
67
68
```

A nagyobb eltérések részben olyan területeken voltak, amelyek futás közben változó firmware-állapothoz köthetők.

Egy működő UEFI-környezet jogosan módosíthat:

- bootváltozókat;
- NVRAM-rekordokat;
- számlálókat;
- runtime konfigurációt;
- eszközállapotot;
- recovery állapotot.

A teljes történeti image visszaírása ezért feleslegesen visszaállított volna más, közben legitim módon megváltozott firmware-állapotokat is.

A vezérelv:

> **A jelenleg működő firmware-t őrizzük meg, és csak azt az adatot állítsuk vissza, amelyről bizonyított, hogy hiányzik.**

---

# 27. A javítási candidate elkészítése

A megbízható recovery utáni SPI-image lett az alap.

Csak az alábbi régió:

```text
0x440000–0x44FFFF
```

került lecserélésre az eredeti gép recovery előtti dumpjának megfelelő 64 KiB-os blokkjára.

A létrejött candidate neve:

```text
D3313_REPAIRED_CANDIDATE.bin
```

A jelenlegi recovery-image és a candidate byte-szintű összehasonlítása:

```text
3526 eltérő byte
```

eredményt adott.

Minden eltérés:

```text
egyetlen 64 KiB-os blokk
```

területére esett, vagyis az azonosított OEM/manufacturing régióra.

A működő recovery firmware érintetlen maradt.

Csak a személyre szabott OEM-régió tért el.

---

# 28. Írás előtti fizikai ellenőrzés

A `flashrom` számára az OEM-régió:

```text
0x440000:0x44ffff OEM
```

layoutként került megadásra.

Írás előtt a candidate a tényleges fizikai flash-tartalommal is összevetésre került.

Az ellenőrzés:

```text
3526 differing bytes
```

eredményt adott.

Ez pontosan megegyezett a független fájlszintű összehasonlítás eredményével.

Két külön módszer tehát ugyanazt mutatta:

```text
fájlszintű bináris összehasonlítás → 3526 byte
fizikai flash verification         → 3526 byte
```

Csak ezután történt meg a fizikai írás.

---

# 29. Szelektív 64 KiB-os visszaállítás

A teljes 8 MiB-os SPI flash **nem** került újraírásra.

Csak:

```text
0x440000–0x44FFFF
```

került kiválasztásra.

A `flashrom` ezt jelezte:

```text
Erase/write done from 440000 to 44ffff
```

majd:

```text
Verifying flash... VERIFIED.
```

Ezután külön verification művelet is történt.

Az ismét:

```text
VERIFIED
```

eredményt adott.

Ez a második ellenőrzés szándékos volt.

A közvetlenül írás után megjelenő success message önmagában nem számított elegendő bizonyítéknak.

A fizikai eszköz külön is ellenőrzésre került az elvárt javított állapothoz képest.

---

# 30. Helyreállítási eredmény

A gép teljesen leállításra és áramtalanításra került, majd hidegindítás történt.

A BIOS Setup ismét az eredeti Fujitsu LAN-azonosságot mutatta.

A nyilvános repozitóriumban ez továbbra is:

```text
<REDACTED_FUJITSU_MAC>
```

formában szerepel.

A Debian indulása után Linux is a helyreállított eredeti hardveres identitást használta.

A korábbi:

```text
Permanent address: not set
```

állapot megszűnt.

A bizonyítási lánc teljessé vált.

---

# 31. A létrejött bizonyítási lánc

```text
Eredeti gép SPI
        ↓
gépspecifikus MAC jelen van a Fujitsu OEM-régióban
        ↓
hivatalos Fujitsu firmware
        ↓
ugyanez a mező FF:FF:FF:FF:FF:FF sablont tartalmaz
        ↓
Fujitsu recovery visszaállítja a POST-ot
        ↓
BIOS MAC 00:00:00:00:00:00 lesz
        ↓
Linux nem talál permanens MAC-ot
        ↓
RSET8168 Linuxon kívül is nullás címet mutat
        ↓
nincs érvényes külső EEPROM
        ↓
közvetlen Realtek eFuse-kiolvasás
        ↓
üres FF állapot + Efuse Write Count = 0
        ↓
recovery utáni SPI belső kiolvasása
        ↓
Fujitsu OEM MAC-mező FF sablont tartalmaz
        ↓
eredeti 64 KiB-os OEM-blokk visszaállítása
        ↓
fizikai írás független ellenőrzése
        ↓
hidegindítás
        ↓
eredeti Fujitsu MAC visszatér a BIOS-ban
        ↓
Linux permanens hálózati identitása helyreáll
```

A megfigyelések együtt erősen támogatják azt az értelmezést, hogy:

> **A vizsgált D3313-E1-en a Realtek RTL8168G-család vezérlője gépspecifikus Ethernet-azonosságát a Fujitsu platformfirmware-ből kapja inicializáláskor, nem pedig egy korábban programozott vezérlő-eFuse-ból.**

Ez a következtetés a vizsgált példányra vonatkozik.

További D3313 rendszereken végzett reprodukció szükséges ahhoz, hogy az egész platformcsaládra általánosítható legyen.

---

# 32. Mit bizonyított a vizsgálat

A rendelkezésre álló bizonyítékok az alábbiakat támasztják alá:

1. a működésképtelen D3313-E1 Fujitsu recovery segítségével ismét POST-képessé tehető;
2. két független SPI-kiolvasás és kriptográfiai ellenőrzés megbízható baseline-t adott;
3. az eredeti gép firmware-e és a hivatalos Fujitsu R1.16 128 darab 64 KiB-os blokkból 127-ben azonos volt;
4. a fő gépspecifikus személyre szabás egyetlen 64 KiB-os OEM/manufacturing régióban koncentrálódott;
5. ez a régió több gépspecifikus mezőt tartalmazott;
6. a hivatalos firmware `FF` sablonértéket használt a LAN MAC helyén;
7. az eredeti gép dumpja ugyanott a valódi Fujitsu MAC-ot tartalmazta;
8. recovery után a fizikai SPI visszaállt a gyári sablon-MAC állapotba;
9. a Realtek vezérlő nem mutatott érvényes külső EEPROM-ot;
10. az eFuse látszólag soha nem került programozásra;
11. ezért a vizsgált gép LAN-azonossága nem programozott Realtek eFuse-ból származott;
12. a teljes recovery utáni SPI Linux alól belső módszerrel kiolvasható volt;
13. a pontos hiányzó OEM-régió izolálható volt;
14. csak ez az egy 64 KiB-os régió volt visszaállítható;
15. a fizikai írás külön ellenőrizhető volt;
16. a Fujitsu OEM-régió visszaállítása helyreállította az eredeti LAN-azonosságot BIOS és Linux alatt is.

---

# 33. Mit nem bizonyított a vizsgálat

A projekt **nem bizonyítja**, hogy:

- minden D3313-E1 pontosan ugyanígy működik;
- minden FUTRO S930 ugyanezen a módon tárolja a hálózati identitást;
- minden Fujitsu recovery eltávolítja a személyre szabott MAC-adatot;
- a teljes recovery-algoritmus reverse engineeringje elkészült;
- a RESERVED jumperpozíció biztonságos;
- a használt USB-port hivatalos recovery-követelmény;
- a platform összes RTL8168G vezérlője programozatlan eFuse-t használ;
- a különálló mSATA-hiba firmware-rel kapcsolatos;
- vagy hogy a projekt találta fel a felhasznált firmware-elemzési módszereket.

Az állítások szándékosan a rendelkezésre álló bizonyítékokra korlátozódnak.

---

# 34. Külön nyitott probléma: 16 GB mSATA

Az OEM-azonosság helyreállítása után a beépített 16 GB-os mSATA SSD külön vizsgálat tárgya lett.

Linux sikeresen érzékeli:

```text
AMD FCH AHCI
ASMedia ASM1061/ASM1062 AHCI
```

vezérlőket.

A 240 GB-os Kingston SATA SSD megbízhatóan megjelenik, és:

```text
6.0 Gbps
```

sebességen linkel.

A 16 GB-os mSATA eszköz viszont:

- nem jelenik meg `lsblk` alatt;
- nem jut el ATA device identification állapotig;
- nem hoz létre felismerhető blokkeszközt;
- a releváns SATA-útvonal `SATA link down` állapotban marad.

Az eredeti Fujitsu OEM-blokk visszaállítása nem változtatott ezen.

Jelenleg nincs bizonyíték arra, hogy az mSATA-probléma része lenne a korábbi firmware-hibának.

Lehetséges okok:

- hibás mSATA SSD;
- csatlakozási/kontaktprobléma;
- hiányzó foglalattáp;
- sérült SATA jelút;
- alaplapi sérülés;
- egyelőre azonosítatlan más konfigurációs mechanizmus.

Ez továbbra is:

```text
[OPEN QUESTION]
```

és szándékosan elkülönül a befejezett firmware-helyreállítástól.

---

# 35. Külön hardveres incidens: mozgó SATA-adapter

Tesztelés közben a Kingston SATA SSD-t tartó adapter működés közben elmozdult.

Linux azonnal naplózott:

- ATA buszhibákat;
- BadCRC hibákat;
- handshake/link hibákat;
- I/O hibákat.

A fájlrendszer emiatt read-only állapotba kerülhetett.

Az adapter fizikai stabilizálása és reboot után a hibák megszűntek.

Ez külön incidens, nem azonos a nyitott mSATA-problémával.

Ugyanakkor bizonyítja, hogy az ideiglenes adapterrögzítés nem megfelelő tartós használatra.

A tervezett mechanikai megoldás:

```text
M2 nylon/műanyag távtartók
kb. 6 mm
```

megfelelő rögzítőelemekkel.

---

# 36. A minimálisan szükséges módosítás elve

A projekt egyik központi módszertani következtetése:

> **Ne írjunk újra több firmware-t annál, mint amennyiről bizonyított, hogy módosításra szorul.**

Ebben az esetben:

- rendelkezésre állt a teljes eredeti SPI-image;
- rendelkezésre állt a teljes működő recovery utáni image;
- a személyre szabott régió azonosításra került;
- csak 3526 byte tért el egyetlen 64 KiB-os blokkon belül;
- a működő firmware többi része érintetlenül megőrizhető volt.

A végső beavatkozás:

```text
64 KiB egy 8 MiB-os flashből
```

területre korlátozódott.

Ez csökkentette a szükségtelen módosításokat:

- NVRAM;
- bootváltozók;
- runtime állapot;
- recovery során keletkezett adatok;
- ismeretlen firmware-struktúrák

esetében.

Ez az elv más firmware-helyreállítási vizsgálatok során is hasznos lehet.

---

# 37. Reprodukálhatóság

A legerősebb eredményeket érdemes további hardvereken is reprodukálni.

Egy második D3313-E1-en különösen hasznos lenne összehasonlítani:

- az OEM/manufacturing blokk helyét;
- az FFS GUID-okat;
- a duplikált struktúrák elrendezését;
- a sablonértékeket;
- a gépspecifikus MAC tárolását;
- a Realtek eFuse állapotát;
- a Fujitsu recovery viselkedését;
- a recovery utáni OEM-állapotot;
- a szelektív OEM-visszaállítás eredményét.

Ez segítene eldönteni, hogy a megfigyelések:

- csak egy példányra jellemzők;
- D3313-E1 specifikusak;
- vagy szélesebb D3313 családra is általánosíthatók.

---

# 38. Korlátok

## 38.1. Egyetlen elsődleges vizsgálati példány

A teljes recovery-vizsgálat egy D3313-E1 rendszeren történt.

Ezért populációszintű általánosítás nem indokolt.

## 38.2. A recovery-algoritmus nincs teljesen reverse engineeringgel feltárva

A recovery megfigyelhető eredménye dokumentált, de a teljes belső implementáció nem ismert.

## 38.3. Részben dokumentálatlan OEM-mezők

Több gépspecifikus mezőnév azonosításra került, de sok pontos funkciója ismeretlen.

## 38.4. Nem történt destruktív redundanciateszt

A látszólag redundáns OEM-struktúrák fallback-viselkedése szándékos sérüléssel nem került tesztelésre.

## 38.5. Az mSATA-probléma nincs lezárva

A nem detektált 16 GB-os mSATA SSD még nem került teljes, független keresztpróbára.

## 38.6. Adatvédelmi korlátozások

A teljes személyre szabott firmware felelősen nem publikálható, mert gépspecifikus információkat tartalmaz.

---

# 39. Jövőbeli kutatási irányok

## 39.1. Fujitsu recovery write-map rekonstrukció

Az egyik legerősebb nyitott kérdés, hogy miért nem maradtak meg a recovery forrásban meglévő személyre szabott adatok a fizikai SPI-ben.

Kontrollált tesztimage-ekkel potenciálisan meghatározható lenne:

- mely régiókat másolja;
- melyeket hagyja ki;
- melyeket generálja újra;
- melyeket őrzi meg;
- van-e manufacturing-specifikus szabályrendszer.

Ebből formális **D3313 recovery write map** készülhetne.

---

## 39.2. OEM/manufacturing struktúra reverse engineeringje

A 64 KiB-os régió láthatóan sokkal többet tartalmaz, mint a LAN MAC.

Több anonimizált gép dumpjának összehasonlítása elkülöníthetné:

- invariáns mezőket;
- modellfüggő mezőket;
- boardspecifikus mezőket;
- gépazonosítókat;
- számlálókat;
- integritási mezőket;
- feature flageket;
- biztonsághoz kapcsolódó mezőket.

---

## 39.3. Redundáns OEM-struktúrák működése

A két hasonló FFS-struktúra kapcsán nyitott kérdések:

- melyik az elsődleges?
- melyik a tartalék?
- van-e generációszámláló?
- érvényességi flag választ közöttük?
- az egyikből rekonstruálható a másik?
- mi történik, ha csak az egyik érvényes?

Egy tartalék platform biztonságos kontrollkísérleteket tenne lehetővé.

---

## 39.4. A hálózati inicializációért felelős UEFI-modul

Az OEM-blokk visszaállítása helyreállította a Realtek-identitást.

A következő kérdés:

> **Mely firmware-komponens olvassa be a Fujitsu OEM-adatot és inicializálja a Realtek vezérlőt?**

Lehetséges módszerek:

- DXE-modul elemzés;
- PCI inicializáció követése;
- GUID/string korreláció;
- MMIO register tracing;
- emulátorral támogatott elemzés;
- boottime instrumentáció.

---

## 39.5. Más hardverek inicializálása OEM-adatból

A MAC-eredmény szélesebb architekturális kérdést vet fel.

Ha a hálózati identitás OEM-régióból származik, más hardverfunkciók is használhatják ugyanezt.

Lehetséges célok:

- SATA/mSATA viselkedés;
- TPM-konfiguráció;
- HDCP-információ;
- feature enablement;
- licensing adatok;
- PCIe-eszközkonfiguráció;
- platformazonosság.

---

## 39.6. D3313 E, B és S variánsok összehasonlítása

A D3313 család több variánsban létezik.

Keresztplatformos összehasonlítás megmutathatná:

- közösek-e az OEM FFS GUID-ok;
- ugyanazon offseteken van-e a manufacturing régió;
- kompatibilis-e a mezőstruktúra;
- hasonló-e a recovery-viselkedés;
- mely dokumentáció általánosítható biztonságosan.

---

## 39.7. Automatizált firmware-differenciálás

Az elemzés jelentős része automatizálható lehet.

Egy jövőbeli eszköz elvégezhetné:

```text
image-méret ellenőrzés
SHA256 hashing
blokkszintű összehasonlítás
FV-validáció
FFS-validáció
OEM string detektálás
gépspecifikus mezők osztályozása
adatvédelmi kockázatok felismerése
flashrom layout generálás
```

Ez gyorsabbá és kevésbé hibakockázatossá tehetné a hasonló vizsgálatokat.

---

## 39.8. Firmware adatvédelmi sanitizer

Mivel egy nyers SPI-image gépazonosító adatokat tartalmazhat, külön sanitizer képes lehetne felismerni vagy kitakarni:

- MAC-címeket;
- sorozatszámokat;
- UUID-ket;
- PC ID-ket;
- manufacturing azonosítókat;
- más ismert érzékeny struktúrákat.

Egy privacy-aware firmware-sharing tool a D3313 családon túl is hasznos lehetne.

---

## 39.9. Recovery hangjelzések elemzése

A recovery felismerhető sípolási mintákat produkált.

Teljes, pontos időzítéssel rögzített hangsorok esetleg összekapcsolhatók lennének:

- eszközfelismeréssel;
- törléssel;
- programozással;
- verificationnel;
- sikerrel;
- hibával.

Ez különösen akkor lehet hasznos, amikor nincs videokimenet.

---

## 39.10. Hardver- és firmware-hibák elkülönítése

Az eredeti meghibásodás azért volt diagnosztikailag érdekes, mert mechanikai sérülés és firmware-hiba egyaránt hihető volt.

A kialakult módszertan:

```text
fizikai vizsgálat
        ↓
multiméteres mérések
        ↓
külső SPI-kiolvasás
        ↓
dupla kiolvasás ellenőrzése
        ↓
gyári firmware-rel összehasonlítás
        ↓
UEFI strukturális elemzés
        ↓
recovery
        ↓
operációs rendszer szintű diagnosztika
        ↓
DOS hardverdiagnosztika
        ↓
közvetlen vezérlő-NVM vizsgálat
        ↓
belső SPI-kiolvasás
        ↓
minimális szelektív visszaállítás
```

Ez általános keretrendszerként is hasznos lehet olyan beágyazott rendszereknél, ahol a hiba eredete nem egyértelmű.

---

# 40. Szélesebb firmware-forenzikai jelentőség

Az egyik fontosabb általános tanulság:

> **A „gyári BIOS image” és egy konkrét gép teljes firmware-állapota nem feltétlenül ugyanaz.**

Egy legyártott rendszer több réteget tartalmazhat:

```text
generikus gyártói firmware
gépspecifikus manufacturing adatok
runtime NVRAM
hardverazonosság
licencelési információ
biztonsági konfiguráció
platformspecifikus kalibráció
```

Egy generikus gyártói image visszaírása helyreállíthatja a bootképességet, miközben eltávolíthat olyan tulajdonságokat, amelyek az adott konkrét géphez tartoznak.

Ebben az esetben a látható elveszett tulajdonság a LAN-azonosság volt.

Más rendszereken gépspecifikus adat lehet például:

- sorozatszám;
- UUID;
- OEM licencinformáció;
- kalibrációs adat;
- TPM-konfiguráció;
- boardspecifikus feature flag;
- hálózati identitás.

Firmware-javításkor ezért külön kell kezelni:

**bootképességet**

és:

**a teljes gépspecifikus állapot helyreállítását**.

---

# 41. Biztonsági szempontok

Az SPI flash módosítása bootképtelenné teheti a hardvert.

Az itt alkalmazott módszertan erősen preferálja:

- több egymástól független kiolvasást;
- kriptográfiai hash-ek használatát;
- az eredeti dumpok változatlan archiválását;
- a flash pontos azonosítását;
- írás előtti verificationt;
- a lehető legkisebb írási tartományt;
- külön, független írás utáni ellenőrzést.

Az OTP/eFuse programozást más kategóriaként kell kezelni, mint az újraírható flashmemóriát.

Visszafordíthatatlan vagy korlátozottan írható tároló módosítása előtt meg kell érteni:

- pontosan milyen adatot írunk;
- visszafordítható-e;
- eredetileg használta-e ezt a mechanizmust a rendszer;
- mennyi programozható kapacitás maradt;
- létezik-e kevésbé invazív helyreállítási út.

A teljes jogi és biztonsági nyilatkozat:

[DISCLAIMER.md](DISCLAIMER.md)

Magyar változat:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

# 42. A repozitórium adatvédelmi szabályai

A nyilvános repozitórium nem tartalmazhat kitakarás nélkül:

- személyre szabott nyers SPI-image-et;
- teljes gépsorozatszámot;
- PC ID-t;
- UUID-t;
- teljes gépspecifikus MAC-címet;
- HDCP- vagy hasonló gépspecifikus titkot;
- privát operációs rendszeres felhasználónevet;
- privát helyi fájlrendszer-útvonalat;
- szükségtelen EXIF/GPS-metaadatot;
- azonosítható címkét;
- eszközazonosságot tartalmazó QR-kódot vagy vonalkódot.

Képernyőképet, fotót, videót, hangfelvételt, firmware-részletet és logot publikálás előtt adatvédelmi szempontból ellenőrizni kell.

Személyre szabott nyers firmware-t közvetlenül commitolni nem szabad.

---

# 43. A repozitórium tervezett szerkezete

A projekt hosszabb távú struktúrája:

```text
/
├── README.md
├── README.hu.md
├── LICENSE
├── CITATION.cff
├── SOURCES.md
├── DISCLAIMER.md
├── DISCLAIMER.hu.md
├── docs/
│   ├── en/
│   └── hu/
├── scripts/
├── examples/
└── media/
    └── public/
```

Személyre szabott firmware-image-ek nem kerülhetnek a nyilvános fába.

---

# 44. Korábbi munkák és forrásmegjelölés

Ez a projekt **nem állítja**, hogy feltalálta volna az alkalmazott firmware-helyreállítási, UEFI-elemzési, SPI-programozási vagy Realtek diagnosztikai technikákat.

A munka gyártói dokumentációra, nyílt forrású eszközökre és korábbi közösségi kutatásokra épül.

Releváns területek:

- Fujitsu D3313 dokumentáció;
- Fujitsu BIOS- és recovery-dokumentáció;
- kapcsolódó D3313 platformokra vonatkozó Kontron-dokumentáció;
- `flashrom` projekt;
- Realtek RSET8168;
- Realtek PGTool dokumentáció és tooling;
- Fujitsu FUTRO rendszerekkel kapcsolatos közösségi kutatások;
- korábbi D3313 firmware-elemzési projektek;
- általános UEFI reverse engineering és javítási kutatások.

A konkrét hivatkozások és attribúciók:

[SOURCES.md](SOURCES.md)

A projekt saját hozzájárulása:

> **e módszerek dokumentált kombinálása, ellenőrzése és alkalmazása egy konkrét D3313-E1 recovery eseten, valamint az ebből származó platform-specifikus megfigyelések.**

---

# 45. Kutatási hozzájárulás

A projekt fő eredménye nem pusztán egy számítógép megjavítása.

A munka dokumentált kapcsolatot teremt:

```text
hivatalos firmware
        ↕
személyre szabott gép firmware
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
minimális szelektív visszaállítás
        ↕
helyreállított gépazonosság
```

Potenciálisan újrafelhasználható eredmények:

- konkrét D3313-E1 gépspecifikus adatrégió azonosítása;
- bizonyíték arra, hogy a hivatalos firmware személyre szabási sablonokat tartalmaz;
- bizonyíték arra, hogy a vizsgált Realtek eFuse soha nem került programozásra;
- bizonyíték arra, hogy platformfirmware biztosíthat NIC-identitást;
- bizonyíték arra, hogy a Fujitsu recovery nem egyszerű teljes-chip másolás;
- demonstrált minimális régiós helyreállítási módszer.

Ezek hasznosak lehetnek olyan kutatóknak, akik az alábbiakkal foglalkoznak:

- D3313 firmware-architektúra;
- embedded rendszer személyre szabás;
- firmware-identitástárolás;
- recovery mechanizmusok;
- EEPROM nélküli PCIe inicializáció;
- BIOS-forenzika;
- ipari és legacy rendszerek megőrzése.

---

# 46. Következtetés

A vizsgálat kezdetén nem lehetett eldönteni, hogy a FUTRO S930 hibája:

- mechanikai alaplapsérülés;
- processzorkörnyéki sérülés;
- tápellátási hiba;
- SPI-firmware-sérülés;
- vagy több hiba kombinációja.

A végső állapot lényegesen jobban ismert.

A rendszer jelenleg:

- sikeresen POST-ol;
- működő BIOS Setupot biztosít;
- Debian Linuxot bootol;
- Etherneten működik;
- tartalmazza a helyreállított gépspecifikus Fujitsu OEM-régiót;
- az eredeti Fujitsu LAN-azonosságot inicializálja.

A recovery utáni MAC-hiba **nem** Realtek eFuse programozással került javításra.

A közvetlen vizsgálat szerint az eFuse látszólag soha nem volt programozva.

A döntő gépazonosság ehelyett a Fujitsu SPI/OEM régióban volt megtalálható.

A végső helyreállítás ezért:

```text
megbízható recovery utáni SPI-kiolvasás
        ↓
összehasonlítás az eredeti firmware-rel
        ↓
egy 64 KiB-os gépspecifikus régió izolálása
        ↓
írás előtti fizikai verification
        ↓
szelektív in-system flash write
        ↓
független írás utáni verification
        ↓
hidegindítás
        ↓
eredeti platformazonosság helyreállítása
```

folyamatból állt.

A külön 16 GB-os mSATA detektálási probléma továbbra is nyitott, és jelenleg nem tekinthető a lezárt BIOS/OEM recovery részének.

A BIOS-helyreállítás és a Fujitsu gépspecifikus OEM-azonosság helyreállítása ezért:

# **SIKERESEN BEFEJEZETTNEK TEKINTHETŐ**

A következő szakasz már nem elsősorban javítási feladat.

A helyreállított platform, az összegyűjtött mérések, firmware-összehasonlítások és reprodukálható bizonyítási lánc további kutatások alapjául szolgálhat:

- D3313 firmware-architektúra;
- manufacturing struktúrák;
- gépszemélyre szabás;
- Fujitsu recovery működés;
- hardver inicializálása platformfirmware-ből.

---

# 47. Aktuális kutatási állapot

```text
BIOS recovery                         KÉSZ
POST helyreállítás                    KÉSZ
Debian boot                           KÉSZ
Belső SPI-kiolvasás                   KÉSZ
OEM-régió azonosítása                 KÉSZ
Eredeti LAN-azonosság helyreállítása  KÉSZ
Szelektív SPI írás ellenőrzése        KÉSZ
Realtek eFuse-vizsgálat               KÉSZ
Fujitsu recovery algoritmus           NYITOTT
OEM-struktúra teljes reverse engineering NYITOTT
Második alaplapos reprodukció         NYITOTT
16 GB mSATA vizsgálat                 NYITOTT
```

---

# 48. Licenc

Eltérő kifejezett jelölés hiányában a repozitóriumhoz kifejezetten létrehozott eredeti, szerzői jogi védelem alatt álló kutatási és dokumentációs anyagok licence:

**Creative Commons Nevezd meg! – Ne add el! – Így add tovább! 4.0 Nemzetközi  
(CC BY-NC-SA 4.0)**

A teljes hivatalos licencszöveg:

[LICENSE](LICENSE)

Röviden: az érintett anyagok nem kereskedelmi célra megoszthatók és átdolgozhatók a licenc forrásmegjelölési és azonos licenc alatti továbbadási követelményeinek betartásával.

Ez az összefoglaló **nem helyettesíti** a hivatalos licencszöveget.

### Fontos hatóköri különbség

A repozitórium tartalmazhat vagy tárgyalhat:

- harmadik féltől származó firmware-t;
- BIOS-komponenseket;
- gyártói segédprogramokat;
- harmadik fél dokumentációját;
- védjegyeket;
- harmadik fél szoftvereit;
- forráskódot vagy szkripteket.

Ezek az anyagok **nem kerülnek automatikusan CC BY-NC-SA 4.0 licenc alá** pusztán azért, mert szerepelnek vagy hivatkozás történik rájuk ebben a repozitóriumban.

A szoftverek és forráskódok külön licencfeltételeket használhatnak, ahol ez kifejezetten meghatározásra kerül.

A harmadik féltől származó anyagok a megfelelő jogtulajdonosok jogai és feltételei alatt maradnak.

A részletes licencelési határokról, jogi nyilatkozatról, kutatási célról, biztonsági rendelkezésekről és harmadik felek jogairól:

[DISCLAIMER.md](DISCLAIMER.md)

Magyar változat:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

A megfelelő saját eredeti anyagokra a szerzői jog jogosultja külön megállapodás alapján kereskedelmi licencet is felajánlhat.

---

# 49. Hivatkozás

Ha ez a projekt, annak mérései, strukturális megállapításai, módszertana vagy dokumentációja hozzájárul kutatáshoz, publikációhoz, javításhoz, műszaki elemzéshez vagy származékos kutatáshoz, kérjük, hivatkozzon rá.

A géppel olvasható hivatkozási metaadat:

[CITATION.cff](CITATION.cff)

Preferált projektnév:

> **Fujitsu FUTRO S930 / D3313-E1: BIOS Recovery, Firmware Analysis, and Restoration of Machine-Specific OEM Identity**

Szerző:

**Bencze Péter**

Repozitórium:

```text
https://github.com/benczepeter001/fujitsu-d3313-bios-recovery
```

Az alkalmazható eredeti kutatási és dokumentációs anyagok licence:

```text
CC-BY-NC-SA-4.0
```

---

# 50. Jogi és kutatási nyilatkozat

Ez független kutatási projekt.

Nem hivatalos Fujitsu-projekt, és nincs kapcsolatban a Fujitsuval vagy a repozitóriumban említett más gyártókkal; azok nem szponzorálják, nem tanúsítják, nem hagyják jóvá és nem támogatják hivatalosan.

Minden harmadik félhez tartozó terméknév, védjegy, firmware, szoftver, dokumentáció és más védett anyag a megfelelő jogosult tulajdonában marad.

Firmware-módosítás, SPI-programozás, hardveres rework és alacsony szintű diagnosztika tartós hardverkárosodást vagy gépspecifikus adatok elvesztését okozhatja.

A hasonló munkát reprodukáló személy felelőssége:

- saját biztonsági mentéseinek ellenőrzése;
- az adott hardver megfelelő ismerete;
- a kockázatok megértése;
- a vonatkozó jogszabályok és licencek betartása;
- kizárólag olyan rendszereken és adatokon végzett munka, amelyekhez rendelkezik szükséges jogosultsággal.

A részletes és irányadó nyilatkozat:

[DISCLAIMER.md](DISCLAIMER.md)

Magyar megfelelője:

[DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

Copyright © 2026 Bencze Péter és az egyes közreműködők saját eredeti hozzájárulásaik tekintetében.

A harmadik felek szerzői jogai, védjegyei, firmware-ei, szoftverei, dokumentációi és más védett anyagai továbbra is a megfelelő jogosultaknál maradnak.
