# Közreműködés a Fujitsu D3313-E1 BIOS-helyreállítási kutatási projektben

Köszönjük, ha hozzájárulnál a projekthez.

Ez a repozitórium a Fujitsu FUTRO S930 / D3313-E1 platform BIOS-helyreállításával, firmware-elemzésével, hardverviselkedésével, gépspecifikus OEM-adataival és platformazonosságának helyreállításával kapcsolatos független műszaki kutatást dokumentálja.

Szívesen fogadunk olyan hozzájárulásokat, amelyek javítják:

- a műszaki pontosságot;
- a reprodukálhatóságot;
- a dokumentációt;
- a forrásmegjelölést;
- a hardverkompatibilitási információkat;
- a firmware-struktúrák megértését;
- az eszközöket és segédprogramokat;
- az adatvédelmet és biztonságot;
- vagy a kutatási dokumentáció minőségét.

Mivel a projekt firmware-rel, hardverazonossággal, potenciálisan destruktív műveletekkel és harmadik felek szellemi tulajdonával is foglalkozik, a hozzájárulások az átlagos dokumentációs projektnél nagyobb körültekintést igényelnek.

---

# 1. Közreműködés előtt

Kérjük, először olvasd el:

- [README.md](README.md)
- [README.hu.md](README.hu.md)
- [SOURCES.md](SOURCES.md)
- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)
- [SECURITY.md](SECURITY.md)
- [SECURITY.hu.md](SECURITY.hu.md)
- [LICENSE](LICENSE)

Ezek határozzák meg a projekt:

- kutatási hatókörét;
- bizonyítási követelményeit;
- biztonsági határait;
- adatvédelmi szabályait;
- licencelési rendszerét;
- harmadik féltől származó anyagokra vonatkozó korlátait;
- biztonsági problémák jelentési módját.

---

# 2. Milyen hozzájárulások hasznosak?

Hasznos hozzájárulás lehet például:

- műszaki pontatlanság javítása;
- reprodukálható mérési eredmény;
- további D3313 sorozatú hardvermegfigyelés;
- firmware-struktúra elemzése;
- UEFI Firmware Volume vagy FFS-megállapítás;
- recovery-eljárással kapcsolatos tapasztalat;
- anonimizált képernyőkép vagy fénykép;
- dokumentáció javítása;
- fordítás javítása;
- forrás- és hivatkozásjavítás;
- reprodukciós teszt;
- adatvédelmi javítás;
- a kutatást támogató szkript vagy segédprogram;
- D3313 revíziók összehasonlítása;
- meglévő hipotézis megerősítése vagy cáfolata.

A negatív eredmények is értékesek.

Egy jól dokumentált sikertelen kísérlet megakadályozhatja, hogy mások veszélyes vagy eredménytelen módszereket ismételjenek meg.

---

# 3. Bizonyítási szintek

A projekt különbséget tesz a különböző bizonyítéktípusok között.

Műszaki információ beküldésekor kérjük jelezd, hogy az adat:

### `[OFFICIAL]`

Gyártói dokumentációból, hivatalos firmware-ből, vendor utilityből, szabványból vagy más elsődleges, hiteles műszaki forrásból származik.

### `[OBSERVED]`

Tényleges hardveren vagy szoftveren közvetlenül mért, rögzített, kiolvasott, naplózott, kinyert vagy reprodukált információ.

### `[INFERRED]`

A rendelkezésre álló bizonyítékok által alátámasztott, de közvetlenül nem mért következtetés.

### `[REPORTED HISTORY]`

Olyan korábbi esemény vagy állapot, amely a kontrollált vizsgálat előtt történt, és utólag nem ellenőrizhető függetlenül.

### `[OPEN QUESTION]`

További kutatást igénylő hipotézis vagy tisztázatlan viselkedés.

Következtetést vagy feltételezést ne tüntess fel közvetlen megfigyelésként.

---

# 4. Reprodukálhatóság

A műszaki állítások lehetőség szerint tartalmazzanak elegendő információt ahhoz, hogy más kutató reprodukálni vagy függetlenül értékelni tudja az eredményt.

Hasznos kiegészítő adatok lehetnek:

- pontos alaplaprevízió;
- hardvermodell;
- firmware-verzió;
- flash chip típusa;
- használt szoftver vagy eszköz verziója;
- használt parancs;
- releváns kimenet;
- kriptográfiai hash;
- offset vagy címtartomány;
- tesztkörülmények;
- tápellátási állapot;
- jumperpozíció;
- ismételt mérések száma;
- fénykép vagy képernyőkép;
- összehasonlítási módszertan.

Firmware-kiolvasásnál erősen ajánlott:

- két egymástól független kiolvasás;
- kriptográfiai ellenőrzés.

Egyetlen látszólag sikeres SPI-kiolvasást nem szabad automatikusan hiteles bizonyítékként kezelni.

---

# 5. Firmware-hozzájárulások

## Ne tölts fel nyilvánosan személyre szabott firmware-image-et

Egy nyers firmware tartalmazhat:

- sorozatszámot;
- UUID-t;
- PC ID-t;
- MAC-címet;
- HDCP-hez kapcsolódó adatot;
- OEM manufacturing adatot;
- licencelési információt;
- eszközspecifikus biztonsági adatot;
- más gépspecifikus azonosítót.

**Ne küldj be kitakarás nélküli személyre szabott SPI-dumpot:**

- commitban;
- pull requestben;
- nyilvános issue-ban;
- GitHub Discussionben;
- release-ben;
- vagy más nyilvános repozitóriumi helyen.

Firmware-bizonyíték esetén inkább használj:

- kriptográfiai hash-t;
- anonimizált részletet;
- strukturális leírást;
- kitakart hexdumpot;
- offsetet;
- fájlméretet;
- blokkszintű összehasonlítást;
- reprodukálható kiolvasási eljárást.

Ha teljes személyre szabott firmware valóban szükséges egy biztonsági probléma kivizsgálásához, használd a [SECURITY.md](SECURITY.md) szerinti privát sebezhetőségi jelentést.

---

# 6. Harmadik féltől származó firmware és szerzői joggal védett anyag

Ne küldj be harmadik féltől származó anyagot, ha nem rendelkezel megfelelő joggal annak terjesztésére.

Ilyen lehet például:

- Fujitsu firmware;
- BIOS-frissítő csomag;
- gyártói utility;
- Realtek szoftver;
- AMD szoftver;
- AMI komponens;
- zárt driver;
- szerzői joggal védett kézikönyv;
- harmadik fél forráskódja;
- kereskedelmi diagnosztikai eszköz.

Az, hogy egy harmadik féltől származó anyag kutatási szempontból hasznos, nem jelenti automatikusan azt, hogy jogszerűen újraterjeszthető.

Lehetőség szerint inkább add meg:

- a hivatalos forrást;
- hivatkozást;
- kriptográfiai hash-t;
- fájlnevet;
- verziószámot;
- a jogtulajdonostól való beszerzés módját.

Harmadik féltől származó anyag nem kerül automatikusan CC BY-NC-SA 4.0 licenc alá attól, hogy ebben a repozitóriumban szó esik róla.

---

# 7. Adatvédelmi követelmények

Képernyőkép, fotó, log, firmware-részlet vagy diagnosztikai kimenet beküldése előtt ellenőrizd, hogy nem tartalmaz-e szükségtelenül érzékeny adatot.

Távolítsd el vagy takard ki például:

- sorozatszámokat;
- UUID-ket;
- teljes MAC-címeket;
- PC ID-ket;
- felhasználóneveket;
- helyi fájlrendszer-útvonalakat;
- e-mail-címeket;
- szükségtelen IP-címeket;
- hitelesítési tokeneket;
- API-kulcsokat;
- QR-kódokat;
- vonalkódokat;
- EXIF helyadatokat;
- HDCP- vagy más gépspecifikus titkokat.

Ha egy azonosító műszakilag releváns, de nyilvánosan nem közölhető, használj helyőrzőt, például:

```text
<REDACTED_FUJITSU_MAC>
```

vagy:

```text
<REDACTED_SERIAL_NUMBER>
```

---

# 8. Biztonsági szempontból érzékeny megállapítások

Kihasználható sérülékenység vagy véletlen hitelesítőadat-kiszivárgás esetén ne nyiss nyilvános issue-t.

Használd a GitHub Private Vulnerability Reporting funkciót a:

[SECURITY.md](SECURITY.md)

szerint.

Jellemzően először privát csatornán jelentendő például:

- kihasználható projektkód-sérülékenység;
- kiszivárgott hitelesítő adat;
- hozzáférési token;
- privát kulcs;
- érzékeny személyre szabott firmware;
- olyan sérülékenység, amelynek azonnali publikálása érdemben növelné a visszaélés kockázatát.

---

# 9. Hardverbiztonság

A projektben dokumentált egyes eljárások tartósan károsíthatják a hardvert.

Példák:

- SPI flash programozás;
- BIOS-módosítás;
- EEPROM-írás;
- OTP/eFuse programozás;
- forrasztás;
- kiforrasztás;
- hőlégfúvós rework;
- közvetlen feszültségbetáplálás;
- alaplapi elektromos vizsgálat.

Hardvert vagy firmware-t módosító eljárásnál dokumentáld:

- az előfeltételeket;
- tápellátási állapotot;
- szükséges feszültséget;
- visszafordíthatatlan lépéseket;
- mentési követelményeket;
- várt eredményt;
- lehetséges hibamódokat;
- helyreállítási lehetőséget, ha létezik.

Visszafordíthatatlan műveletet egyértelműen jelölni kell.

Ne mutass be destruktív vagy visszafordíthatatlan eljárást rutinfeladatként, ha létezik kevésbé invazív alternatíva.

---

# 10. Csak engedélyezett kutatás

Biztonsági elemzéshez, reverse engineeringhez, hálózati vizsgálathoz, rendszerhozzáféréshez vagy digitális forenzikához kapcsolódó hozzájárulás kizárólag olyan rendszert vagy adatot érinthet, amelynek vizsgálatára a közreműködő jogosult.

Ne küldj be jogosulatlan:

- rendszerhozzáférésből;
- fiókhozzáférésből;
- hálózati lehallgatásból;
- hitelesítőadat-használatból;
- adatkivonásból;
- megfigyelésből;
- kihasználásból

származó anyagot.

A projekt jogszerű javítást, interoperabilitást, megőrzést, védekező elemzést és engedélyezett kutatást dokumentál.

---

# 11. Dokumentációs stílus

A dokumentációban a pontosság fontosabb, mint a túlzott bizonyosság.

Ha egy mechanizmus nincs közvetlenül bizonyítva, inkább:

> A megfigyelt eredmény arra utal...

mint:

> Biztosan ez történik.

Ha csak egy példányon vizsgáltuk:

> A vizsgált D3313-E1 példányon...

helyesebb, mint:

> Minden D3313-E1-en...

kivéve, ha több példány vagy hiteles elsődleges dokumentáció igazolja az általánosítást.

Lehetőség szerint:

- különítsd el a megfigyelést az értelmezéstől;
- őrizd meg a releváns parancskimenetet;
- jelöld a bizonytalanságot;
- dokumentáld a sikertelen kísérleteket;
- add meg a hardverrevíziót;
- hivatkozz elsődleges forrásokra;
- kerüld az alá nem támasztott általánosítást.

---

# 12. Angol és magyar dokumentáció

A projekt fontosabb dokumentumait lehetőség szerint angolul és magyarul is fenntartjuk.

Jelenlegi főbb dokumentumpárok:

- `README.md` / `README.hu.md`
- `DISCLAIMER.md` / `DISCLAIMER.hu.md`
- `SECURITY.md` / `SECURITY.hu.md`
- `CONTRIBUTING.md` / `CONTRIBUTING.hu.md`

Nem kötelező mindkét nyelvi változatot beküldeni, ha a közreműködő nem tud megbízható fordítást készíteni.

Jelentős tartalmi módosítás esetén azonban jelezni kell, ha a másik nyelvi változat frissítése is szükséges.

Gépi fordítás használható kiindulási alapként, de a műszaki terminológiát és jelentést publikálás előtt kézzel ellenőrizni kell.

---

# 13. Issue-k

Nyilvános GitHub issue megfelelő például:

- dokumentációs hibára;
- hibás linkre;
- reprodukálhatósági kérdésre;
- nem érzékeny hardvermegfigyelésre;
- kompatibilitási jelentésre;
- kutatási irány javaslatára;
- feature requestre;
- nem biztonsági hibára.

Új issue létrehozása előtt érdemes ellenőrizni, hogy a témát nem dokumentáltuk-e már.

A jó issue-cím egyértelműen megnevezi a problémát.

Például:

```text
D3313-E1 R1.18: OEM block differs from documented 0x440000–0x44FFFF region
```

jobb, mint:

```text
BIOS problem
```

---

# 14. Pull requestek

A pull request legyen lehetőség szerint fókuszált és könnyen áttekinthető.

Egy PR lehetőleg egy összefüggő változtatást tartalmazzon.

A pull request leírásában térj ki arra:

- mi változott;
- miért szükséges;
- hogyan lett ellenőrizve;
- milyen hardvert vagy firmware-verziót érint;
- eltávolítottad-e az érzékeny adatokat;
- tartalmaz-e harmadik féltől származó anyagot;
- szükséges-e másik nyelvi dokumentum frissítése.

Nagyobb firmware-elemzési hozzájárulásnál adj rövid bizonyítási láncot is.

---

# 15. Commitüzenetek

Használj rövid, de informatív commitüzenetet.

Jó példák:

```text
Document D3313-E1 R1.18 OEM region comparison
```

```text
Correct RTL8168G eFuse interpretation
```

```text
Add second-board SPI block comparison
```

```text
Update Hungarian recovery documentation
```

Kerüld az olyan túl általános üzeneteket, mint:

```text
fix
```

```text
update
```

```text
changes
```

ha pontosabb megfogalmazás is lehetséges.

---

# 16. Dokumentáció licencelése

Eltérő kifejezett jelölés hiányában a repozitórium megfelelő saját kutatási és dokumentációs anyagai az alábbi licenc alatt kerülnek terjesztésre:

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International  
(CC BY-NC-SA 4.0)**

Ha saját dokumentációt, kutatási szöveget, saját ábrát, saját fényképet, saját mérési eredményt vagy más szerzői jogi védelem alatt álló eredeti anyagot küldesz be a CC BY-NC-SA 4.0 alatt álló projektdokumentációba, elfogadod, hogy az elfogadott hozzájárulás a projekt részeként **CC BY-NC-SA 4.0** alatt terjeszthető, kivéve ha a befogadás előtt ettől eltérő megállapodás születik.

A beküldött anyaghoz megfelelő joggal kell rendelkezned.

Saját hozzájárulásod szerzői joga nálad marad, kivéve ha kifejezetten másként állapodunk meg.

A beküldés önmagában nem ruházza át a szerzői jogot a repozitórium tulajdonosára.

Lásd:

- [LICENSE](LICENSE)
- [DISCLAIMER.md](DISCLAIMER.md)

---

# 17. Szoftver és forráskód licencelése

A Creative Commons licencek nem elsődlegesen szoftverkód licencelésére szolgálnak.

Ezért a repozitórium `LICENSE` fájlja **nem jelenti automatikusan azt, hogy minden forráskód CC BY-NC-SA 4.0 licenc alatt áll**.

Ha egy szoftverkomponens, szkript vagy könyvtár külön szoftverlicencet tartalmaz, az irányadó az adott kódra.

Amíg egy új kódkomponens külön licencelése nincs meghatározva:

> **Jelentősebb új forráskód beküldése előtt nyiss issue-t.**

Így a megfelelő licencet még a kód projektbe történő befogadása előtt meg lehet határozni.

Ne feltételezd, hogy a forráskód beküldésével automatikusan a dokumentáció CC-licence alá kerül.

---

# 18. A közreműködő jogai és nyilatkozatai

Hozzájárulás beküldésével kijelented, hogy:

- a hozzájárulást te készítetted, **vagy**
- rendelkezel megfelelő joggal és engedéllyel annak beküldésére;
- az anyag tudomásod szerint nem tartalmaz olyan tartalmat, amelynek újraterjesztésére nem vagy jogosult;
- az érzékeny információk szükség szerint eltávolításra kerültek;
- a harmadik féltől származó anyagok egyértelműen jelölve vannak;
- a vonatkozó licenceket betartod.

Ne másolj más forrásból kódot, dokumentációt, fényképet, firmware-t, ábrát vagy más védett tartalmat megfelelő jogosultság nélkül.

---

# 19. Forrásmegjelölés

Jelentős külső kutatást, korábbi munkát, dokumentációt és műszaki forrást megfelelően hivatkozni kell.

A releváns forrásokat a:

[SOURCES.md](SOURCES.md)

fájlhoz is hozzá lehet adni.

Lehetőség szerint az alábbi sorrendet részesítsd előnyben:

1. elsődleges gyártói dokumentáció;
2. szabvány;
3. upstream szoftverdokumentáció;
4. eredeti kutatási forrás;
5. közösségi beszélgetés támogató bizonyítékként.

A közösségi tapasztalat értékes lehet, de nem automatikusan egyenértékű az elsődleges műszaki dokumentációval.

---

# 20. Műszaki nézeteltérések

A bizonyítékokkal alátámasztott szakmai vita üdvözlendő.

Ha egy hozzájárulás megkérdőjelez egy meglévő következtetést, a cáfoló megfigyelést add meg, ne pusztán azt írd, hogy a jelenlegi következtetés hibás.

Például:

> Egy második, R1.18 firmware-t futtató D3313-E1-en két független SPI-kiolvasás más offseten találja a feltöltött MAC-struktúrát.

hasznosabb, mint:

> A dokumentált offset rossz.

A meglévő következtetéseket módosítani kell, ha erősebb bizonyíték válik elérhetővé.

---

# 21. Kísérleti munka

Firmware-állapotot módosító kísérleteket lehetőség szerint helyreállítható állapotból kell indítani.

Ajánlott előkészületek:

- ellenőrzött eredeti SPI-mentés;
- több független kiolvasás;
- kriptográfiai hash;
- dokumentált flash-chip azonosítás;
- ismert recovery-eljárás;
- minimális írási tartomány;
- írás utáni verification.

Egy hipotézis tesztelésekor lehetőség szerint a lehető legkisebb szükséges régiót módosítsd.

---

# 22. Repozitóriumi baseline

Az első stabil, dokumentált kutatási baseline tagje:

```text
v1.0.0
```

Kísérleti munkánál ajánlott megjelölni azt a baseline-t vagy commitot, amelyből a vizsgálat kiindult.

Ez segít elkülöníteni a későbbi kísérleti állapotokat a dokumentált recovery-baseline-tól.

---

# 23. Mit utasíthatunk el?

Egy hozzájárulás elutasítható, ha:

- kitakarás nélküli érzékeny adatot tartalmaz;
- harmadik fél anyagát megfelelő jog nélkül újraterjeszti;
- bizonyíték nélküli műszaki állítást tényként közöl;
- ésszerűen nem ellenőrizhető;
- veszélyes eljárást megfelelő figyelmeztetés nélkül mutat be;
- jogosulatlan hozzáférést vagy tesztelést ösztönöz;
- ellentétes a repozitórium jogi vagy biztonsági szabályaival;
- hozzáadott érték nélkül duplikál meglévő tartalmat;
- harmadik fél anyagát próbálja újralicencelni;
- rosszindulatú kódot tartalmaz;
- szándékosan gyengíti az adatvédelmi vagy biztonsági védelmeket.

Az elutasítás nem feltétlenül jelenti azt, hogy maga a kutatási kérdés érdektelen.

Előfordulhat, hogy biztonságosabb, jobban dokumentált vagy jogilag megfelelő formában újra benyújtható.

---

# 24. Javasolt hozzájárulási folyamat

Egyszerű dokumentációs javítás esetén:

```text
Fork / branch
   ↓
módosítás
   ↓
érzékeny adatok ellenőrzése
   ↓
commit
   ↓
pull request
```

Nagyobb firmware- vagy hardverkutatás esetén:

```text
eredeti bizonyíték megőrzése
        ↓
hardver- és firmware-verzió dokumentálása
        ↓
reprodukálható mérés
        ↓
eredmény ellenőrzése
        ↓
érzékeny adatok eltávolítása
        ↓
megfigyelés és következtetés szétválasztása
        ↓
korábbi munkák hivatkozása
        ↓
fókuszált pull request
```

Biztonsági szempontból érzékeny megállapításnál:

```text
ne publikáld nyilvánosan
        ↓
GitHub Private Vulnerability Reporting
```

---

# 25. Kérdések

Általános közreműködési kérdéshez használható nyilvános GitHub issue.

Biztonsági szempontból érzékeny ügyben a privát sebezhetőségi jelentési eljárást használd.

Jogi, licencelési vagy újrafelhasználási kérdésekhez lásd:

- [LICENSE](LICENSE)
- [DISCLAIMER.md](DISCLAIMER.md)
- [DISCLAIMER.hu.md](DISCLAIMER.hu.md)

---

Köszönjük, hogy hozzájárulsz a projekt műszaki pontosságához, reprodukálhatóságához, biztonságához és hosszú távú értékéhez.

---

Copyright © 2026 Bencze Péter és az egyes közreműködők saját eredeti hozzájárulásaik tekintetében.
