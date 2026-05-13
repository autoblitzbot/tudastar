# Telegram biztonság – Business Bots kockázat és bevált gyakorlatok

Források:
- https://x.com/officer_secret/status/2054039379908145165
- https://x.com/officer_secret/status/1964011360888541637

Magyar fordítás Vladimir S. / Officer’s Notes bejegyzéseiből.

---

## Telegram Business Bots: új kockázat

Új frissítés érkezett a Telegramon…

A **Business Bots** funkció — amellyel egy szkriptet kapcsolhatsz a személyes fiókodhoz automatikus válaszokhoz — mostantól teljesen ingyenes.

Korábban Telegram Premium kellett hozzá. Most már mindenki számára elérhető — és ez nagyon érdekes, egyben akár elég árnyékos lehetőségeket is megnyit.

Képzeld el: ülsz egy bárban, kimész pisilni, és csak 10 másodpercre ott hagyod a feloldott telefonodat az asztalon. Ez bőven elég ahhoz, hogy valaki feltörjön — sebezhetőség kihasználása nélkül.

A támadó — az új „haverod”, egy féltékeny feleség, egy munkatárs vagy egy csapdának szánt lány — felkapja a telefonodat, bemegy a Telegram Business beállításokba, és hozzáköt egy előre elkészített botot a fiókodhoz.

Visszajössz, felveszed a telefonod, és semmit sem veszel észre. Minden beszélgetésed teljesen normálisnak tűnik, sehol nincs botról szóló értesítés.

**DE!** Ha egy előre felkészített új névjegy ír neked — vagy ha maga a bot kezdeményezi a beszélgetést —, a bot azonnal elfogja az üzenetet. A bot a Telegram szerverein fut, ezért akkor is tovább olvassa az üzeneteidet és dolgozza fel a kiváltó eseményeket, ha kikapcsolod a telefonod, kiveszed a SIM-kártyát, és bezárod az összes aktív munkamenetet — mert közvetlen API-tokenje van a fiókodhoz.

Van egy másik remek hiba is az időzített üzenetekkel. A támadó a telefonodról küld magának egy üzenetet azzal a beállítással, hogy „küldés, amikor a névjegy online lesz”, majd azonnal törli a csevegést a te oldaladon. A felületed teljesen tisztának látszik. De amikor a feltétel teljesül, a Telegram szerverei mégis elküldik a kézbesítési eseményt.

A tanulság egyszerű: egy feloldott okostelefon valaki más kezében 10 másodpercnél tovább már kompromittált eszköznek számít.

Nem árt most rögtön ellenőrizni:

**Beállítások → Telegram Business → Chat Bots**

Győződj meg róla, hogy nincs-e ott csendben egy gyanús bot, amely továbbítja az összes adatodat valakinek.

---

## Telegram biztonsági bevált gyakorlatok

Üdv! Ebben a jegyzetben adok néhány gyors tippet, amelyek segítenek nyugodtabban aludni, ha Telegramot használsz.

A Telegram az egyik leggyakoribb kommunikációs eszköz a kriptós világban, és ennek jó oka van. Az appot gyakran használják munkára és kapcsolattartásra is, ezért logikus, hogy a csalók és hackerek is itt keresnek áldozatokat.

Nézzük meg, hogyan ne válj áldozattá.

### Alap tippek

Figyelj az imposztorokra. Gondosan ellenőrizd a Telegram biót, mert a csaló bármilyen nevet beírhat a biójába, miközben a saját felhasználónevét üresen hagyja.

Figyelj a hamis Telegram-belépési értesítésekre is: ezek gyakran adathalász linket tartalmaznak. Alaposan ellenőrizd őket; valódi értesítésnek a hivatalos Telegram hírek és tippek csatornában kell megjelennie.

Léteznek hamis botok is — igen, botok, nem csak felhasználói fiókok, tudnak elsőként rád írni — és így tovább.

Egyik Telegram chat sem végponttól végpontig titkosított: sem az 1:1 beszélgetések, sem a csoportok. Alapból csak TLS védi őket. Ha jól emlékszem, egyedül a **Secret Chat** végponttól végpontig titkosított.

### Ajánlott beállítások

- **Telefonszám → Ki láthatja a telefonszámomat:** Senki
- **Adatok és tárhely → Automatikus média-letöltés:** kikapcsolva
- **Telefonszám → Ki találhat meg a számom alapján:** Névjegyeim
- **Utoljára látott és online → Ki láthatja az időbélyegemet:** Senki
- **Profilkép → Ki láthatja a profilképemet:** Névjegyeim
- **Hívások → Ki hívhat engem:** Névjegyeim — vagy Senki, ha úgy jobb
- **Hívások → Peer-to-peer:** Névjegyeim — vagy Senki, ha nem akarod megosztani az IP-címedet a beszélgetőpartnerrel

Amikor hívást indítasz, a jobb felső sarokban négy emojit fogsz látni. Kérd meg a másik felet, hogy mondja be ezeket, és hasonlítsd össze a sajátjaiddal. Egyezniük kell. Ez védelem a közbeékelődéses támadások ellen.

- **Továbbított üzenetek → Ki adhat linket a fiókomhoz, amikor továbbítja az üzeneteimet:** Névjegyeim
- Soha ne adj névjegyeket a Telegramhoz — ha vannak, töröld őket —, és mindig használj VPN-t
- **Csoportok és csatornák → Ki adhat hozzá engem:** Névjegyeim
- Állíts be 2FA-t, vagyis felhőjelszót
- Állíts be felhős e-mailes 2FA-t is
- Kapcsold ki a matricák ismétlődő animációját. Animált matricák = veszély
- Kapcsold ki az automatikus letöltést Wi-Fi-n és mobilneten is: **Privacy & Security → Data Settings**
- Kapcsold ki mindenkinek a P2P hívásokat, mert kiszivárogtathatják az IP-címedet
- Ugyanez igaz a Secret Chatre is: a végponttól végpontig titkosítás azt jelenti, hogy az IP-címed ismertté válhat annak, akivel beszélgetsz — és fordítva
- Kapcsold ki a link- és kép-előnézeteket a Secret Chatben — a Privacy and Security részben lejjebb görgetve találod
- Kapcsold ki a GIF-ek automatikus lejátszását

Telegram Premium előfizetést — illetve számokat és felhasználóneveket — már TON-nal is lehet venni: https://fragment.com/premium

A holland rendőrség hozzáférhet rejtett Telegram-számokhoz — használj eldobható számot.

### Botok, PDF-ek és munkamenetek

Soha ne aktiválj semmilyen Telegram botot a `/start` paranccsal. Ne is nyúlj Telegram botokhoz.

Csak a nyilvános csoportban használt botok tekinthetők viszonylag biztonságosnak, amelyeket nyilvános chatben parancsokkal kezelsz. Soha ne írj privátban Telegram botnak. Bármelyik gomb tartalmazhat SQL injection sérülékenységet vagy akár rosszabbat.

Ha PDF-et kell megnyitnod — például önéletrajzot —, használd a https://dangerzone.rocks eszközt, vagy kérd, hogy Google Drive-ra töltsék fel, és előnézetben nézd meg.

Figyeld az aktív munkameneteket. Zárd le az inaktív sessionöket. Figyelj a session-lopó kártevőkre.

Ha üzenetet kapsz arról, hogy beléptek a fiókodba, ellenőrizd, hogy valóban a hivatalos Telegram értesítési vagy hírcsatornából jött-e. A csalók megszemélyesíthetik ezt az értesítési csatornát, hogy kicsalják tőled az SMS-ben kapott egyszer használatos kódot.

Nézd át a Telegram FAQ-t is: https://telegram.org/faq

### Telefonszám és VPN

Telegram-belépéshez használj másik telefonszámot — vagy akár virtuális számot —, ne a valódi mobiltelefonszámodat. Viszont ha egyszer használatos számot használsz, később más is megszerezheti ugyanazt a számot, és hozzáférhet a fiókodhoz.

Az IP-címed elrejtéséhez használj VPN-t — amelyet például a Telegram is átadhat hatósági kérésre.

Nézd át ezt a listát is: https://telegra.ph/How-to-configure-Telegram-security-and-privacy-07-21

Kövesd a Telegram Tips csatornát: https://t.me/TelegramTips

### Eszközök és rendszeres ellenőrzés

Szükséges egy külön, biztonságos eszköz, amelyen be van jelentkezve a fiókod.

Rendszeresen ellenőrizni kell a fiókba bejelentkezett alkalmazás működését és a szolgáltatási értesítések chatjét — legalább ötnaponta egyszer.

Minél több eszköz van bejelentkezve a fiókba, annál nagyobb a fiók kompromittálásának kockázata. Ugyanakkor egy bejelentkezett eszköz a Telegram-fiók biztonságának fenntartásában is eszköz lehet.

Ez a projekt a Telegram korlátait írja le:

- https://github.com/tginfo/Telegram-Limits
- https://crowdin.com/project/telegram-limits

Maradj biztonságban!

---

## Ha bannoltak vagy korlátoztak

Úgy tűnik, bannoltak… Mit tegyek?

Bárki áldozata lehet tömeges jelentgetésnek. Küldj levelet az **abuse@telegram.org** címre, vagy vedd fel a kapcsolatot a támogatási ügynökkel: **+42470** — add hozzá ezt a számot a névjegyekhez.

Emellett érdemes a Telegram beállításaiban található ügyfélszolgálatot is megkeresni: **Ask a Question**.

Ezután töltsd ki ezt az űrlapot:

- https://telegram.org/support

Ezeket is meg lehet próbálni:

- https://t.me/notoscam
- https://t.me/AmeliaTearheart
- https://t.me/BotSupport

E-mail címek:

- dmca@telegram.org
- security@telegram.org
- recover@telegram.org
- abuse@telegram.org

A hivatalos Telegram-oldalakon, kliensekben stb. történt változások automatikus észlelése:

- https://t.me/tgcrawl

Ha korlátoztak, keresd fel a @SpamBotot, és menj végig a lépéseken:

- https://t.me/SpamBot
