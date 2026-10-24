# Biztonsági irányelvek

## Hatókör

Ez a repozitórium a Fujitsu FUTRO S930 / D3313-E1 platformmal kapcsolatos független firmware-helyreállítási, hardverelemzési és biztonsági jellegű műszaki kutatásokat dokumentálja.

Ez a biztonsági szabályzat az alábbiakra vonatkozik:

- a repozitóriumban közvetlenül karbantartott fájlokra;
- a projekthez kifejezetten fejlesztett szkriptekre, segédprogramokra és szoftverekre;
- a repozitórium konfigurációjára;
- GitHub Actions munkafolyamatokra vagy más, később hozzáadott automatizmusokra;
- titkok, hitelesítő adatok, tokenek vagy érzékeny gépspecifikus adatok véletlen közzétételére;
- a projekt saját kódja által bevezetett sérülékenységekre;
- olyan nem várt működésre, amely a dokumentált célnál szélesebb körben módosíthat adatokat vagy hardvert.

Ez a szabályzat **nem teszi ezt a repozitóriumot** a Fujitsu, Realtek, AMD, AMI, flashrom, Linux, Debian vagy bármely más, a dokumentációban említett harmadik fél projektjének, gyártójának, termékének, firmware-ének vagy szolgáltatásának biztonsági kapcsolattartójává.

A harmadik fél termékét érintő sérülékenységet általában az adott gyártónak vagy upstream projektnek kell jelenteni.

---

## Támogatott verziók

Ez a projekt elsősorban kutatási repozitórium, nem pedig hagyományos, verziózott szoftvertermék.

A biztonsági javítások a jelenlegi alapértelmezett ágra vonatkoznak:

| Verzió / ág | Biztonsági támogatás |
| --- | --- |
| `main` | Támogatott |
| Korábbi commitok | Nincs aktív karbantartás |
| Forkok és harmadik féltől származó másolatok | A projekt nem tartja karban |

Ha később külön verziószámokkal vagy release ágakkal rendelkező szoftverkomponensek jelennek meg, ez a szabályzat ennek megfelelően frissíthető.

---

## Sérülékenység jelentése

Kérjük, **ne nyisson nyilvános GitHub issue-t**, ha a jelentés az alábbiak valamelyikét tartalmazza:

- kihasználható biztonsági hibát;
- hitelesítő adatot vagy titkot;
- API-kulcsot vagy hozzáférési tokent;
- privát gépspecifikus azonosítót;
- kitakarás nélküli firmware-adatot;
- személyes információt;
- olyan műszaki részletet, amely a javítás elkészülte előtt érdemben növelheti a visszaélés kockázatát.

Ehelyett használja a repozitórium **GitHub Private Vulnerability Reporting** funkcióját.

A repozitórium **Security / Biztonság** részében a privát sebezhetőségi jelentési lehetőséggel közvetlenül a repozitórium karbantartójának küldhető a jelentés.

Ha a privát jelentési lehetőség valamilyen okból nem elérhető, a repozitórium metaadataiban megadott kapcsolattartási módon lehet elérni a projekt tulajdonosát.

---

## Mit tartalmazzon egy jelentés?

Egy hasznos sérülékenységi jelentés lehetőség szerint tartalmazza:

- a probléma rövid leírását;
- az érintett fájlt, komponenst, szkriptet vagy workflow-t;
- az érintett commitot vagy verziót;
- a reprodukálás lépéseit;
- az elvárt működést;
- a ténylegesen megfigyelt működést;
- a lehetséges biztonsági hatást;
- szükség esetén proof-of-concept információt;
- javasolt javítást vagy mitigációt, ha rendelkezésre áll.

Kérjük, távolítsa el vagy takarja ki a szükségtelen:

- jelszavakat;
- hitelesítési tokeneket;
- privát kulcsokat;
- sorozatszámokat;
- UUID-ket;
- teljes MAC-címeket;
- személyre szabott firmware-image-eket;
- HDCP- vagy más hasonló gépspecifikus titkokat;
- személyes adatokat.

Érzékeny információt csak akkor adjon meg, ha az valóban szükséges a sérülékenység megértéséhez vagy reprodukálásához.

---

## Felelős közzététel

Kérjük, biztosítson ésszerű időt a jelentett sérülékenység kivizsgálására és – ahol indokolt – javítására, mielőtt részletes kihasználási információt hozna nyilvánosságra.

A jelentések értékelésének szempontjai:

- reprodukálhatóság;
- műszaki hatás;
- érintett projektkomponensek;
- kihasználhatóság valószínűsége;
- felhasználókra vagy hardverre gyakorolt lehetséges kár;
- valamint az, hogy a probléma a repozitórium hatókörébe tartozik-e.

Ez egy függetlenül karbantartott kutatási projekt, ezért nincs garantált válaszidő vagy javítási SLA.

A jelentés lezárható vagy más kapcsolattartóhoz irányítható, ha harmadik fél termékét érinti, és nem a repozitórium által karbantartott kódot vagy anyagot.

---

## Kutatás és jogosultság

A repozitórium többek között az alábbi témákat érinti:

- firmware-elemzés;
- BIOS-helyreállítás;
- SPI flash hozzáférés;
- hardverdiagnosztika;
- reverse engineering;
- hálózati vezérlők konfigurációja;
- alacsony szintű rendszerelemzés.

Biztonsági kutatást kizárólag:

- saját tulajdonú hardveren;
- vizsgálatra kifejezetten engedélyezett rendszeren;
- jogszerűen vizsgálható adaton;
- vagy olyan környezetben szabad végezni, amelyhez egyértelmű engedély áll rendelkezésre.

A projekt nem ad engedélyt harmadik fél rendszereinek jogosulatlan tesztelésére, elérésére, módosítására, megfigyelésére, működésük akadályozására vagy kihasználására.

---

## Hardverbiztonság

A repozitóriumban dokumentált egyes műveletek helytelen használat esetén visszafordíthatatlan hardver- vagy firmware-károsodást okozhatnak.

Példák:

- SPI flash írás;
- BIOS-módosítás;
- EEPROM-programozás;
- OTP/eFuse programozás;
- forrasztás és kiforrasztás;
- hőlégfúvós javítás;
- alaplapi elektromos mérés.

Egy veszélyes eljárással kapcsolatos biztonsági jelentésnél egyértelműen külön kell választani:

- a természetéből adódó, dokumentált hardverkockázatot;
- a helytelen használatot;
- a hiányos dokumentációt;
- és a projekt nem várt hibás működését.

Az olyan visszafordíthatatlan tárolók, mint az OTP vagy eFuse memória, különösen nagy körültekintést igényelnek.

---

## Érzékeny firmware- és gépspecifikus adatok

Firmware-image-ek olyan információkat is tartalmazhatnak, amelyeket nem szabad nyilvánosan közzétenni, például:

- sorozatszámokat;
- UUID-ket;
- PC ID-ket;
- MAC-címeket;
- manufacturing adatokat;
- licencelési információkat;
- HDCP-adatokat;
- biztonsági konfigurációt;
- eszközspecifikus azonosítókat.

Kérjük, ne csatoljon kitakarás nélküli személyre szabott firmware-image-et nyilvános issue-hoz.

Ha ilyen anyag szükséges egy biztonsági probléma bizonyításához, használja a repozitórium privát sebezhetőségi jelentési csatornáját, és csak a feltétlenül szükséges adatot adja meg.

---

## Titkok és hitelesítő adatok

Soha ne commitoljon szándékosan:

- jelszavakat;
- privát kulcsokat;
- hozzáférési tokeneket;
- API-kulcsokat;
- munkamenet-hitelesítő adatokat;
- helyreállítási kódokat;
- felhőszolgáltatás-hitelesítő adatokat;
- aláírási titkokat.

A GitHub secret scanning és push protection bizonyos típusú kiszivárgott titkokat képes felismerni, de az automatikus érzékelés nem helyettesíti a körültekintő ellenőrzést.

Ha egy hitelesítő adat véletlenül commitba kerül, úgy kell tekinteni, hogy kiszivároghatott, ezért azt azonnal vissza kell vonni vagy cserélni kell.

Egy titok törlése a legfrissebb commitból nem jelenti automatikusan azt, hogy az eltűnt a teljes Git-előzményből.

---

## Mikor megfelelő a nyilvános issue?

Nyilvános GitHub issue megfelelő például:

- dokumentációs hibákhoz;
- hibás linkekhez;
- érzékeny adatot nem tartalmazó reprodukálhatósági problémákhoz;
- hardverkompatibilitási megfigyelésekhez;
- feature requesthez;
- pontosítási kéréshez;
- olyan nem biztonsági hibához, amely nem okoz lényeges biztonsági kockázatot.

Ha bizonytalan abban, hogy egy probléma nyilvánosan közölhető-e, először használja a privát sebezhetőségi jelentést.

---

## Harmadik fél sérülékenységei

Ha a repozitóriumban említett harmadik fél komponensében található sérülékenység, azt általában közvetlenül az illetékes gyártónak vagy upstream karbantartónak kell jelenteni.

Például:

- Fujitsu firmware vagy segédprogram;
- Realtek szoftver vagy hardver;
- AMD komponens;
- AMI firmware;
- Debian vagy Linux;
- flashrom;
- CH341A szoftver;
- más harmadik féltől származó eszköz.

A repozitórium releváns esetben dokumentálhat már nyilvánosan ismert sérülékenységeket, de nem működik ezen projektek hivatalos biztonsági kapcsolattartójaként.

---

## Közzététel

A repozitórium saját kódját érintő sérülékenység biztonsági információi később nyilvánosságra hozhatók, miután:

1. a probléma megfelelően kivizsgálásra került;
2. az érzékeny információk eltávolításra kerültek;
3. ahol lehetséges, megfelelő javítás vagy mitigáció készült;
4. a közzététel már nem okoz szükségtelen további kockázatot.

A megoldott problémák dokumentálhatók:

- GitHub Security Advisoryban;
- release note-ban;
- a repozitórium dokumentációjában;
- vagy a commitelőzményekben.

---

## Jogi és licencelési információk

Ez a biztonsági szabályzat nem helyettesíti a repozitórium jogi, kutatási, biztonsági vagy licencelési nyilatkozatait.

Lásd:

- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [LICENSE](LICENSE)
- [README.md](README.md)
- [README.hu.md](README.hu.md)

A harmadik felek jogai és licencei továbbra is az adott jogosultakhoz tartoznak.

---

## Kapcsolat

Elsődleges biztonsági jelentési csatorna:

**GitHub Private Vulnerability Reporting ehhez a repozitóriumhoz**

Repozitórium:

`https://github.com/benczepeter001/fujitsu-d3313-bios-recovery`

Nem érzékeny kérdésekhez a normál GitHub issue-rendszer használható.

---

Copyright © 2026 Bencze Péter és az egyes közreműködők saját eredeti hozzájárulásaik tekintetében.
