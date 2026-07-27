# CLAUDE.md — VO Turf List

Projektinstruktioner för fortsatt utveckling. Läs hela filen innan du börjar.

---

## 1. Om projektet

**VO Turf List** är en tillgänglighetsanpassad webbapp för GPS-spelet Turf, byggd
specifikt för skärmläsaranvändare (VoiceOver på iPhone i första hand, JAWS på Windows
sekundärt).

- **Användare:** Claudio, spelarnamn "Conejo". Skärmläsaranvändare, Stockholm.
- **Konversationsspråk:** svenska.
- **Leverans:** en enda fil, `index.html`, som publiceras via GitHub Pages.
  Filnamnet måste vara exakt `index.html`.
- **Nuvarande storlek:** ~3750 rader.
- **Inga byggverktyg, inga beroenden, ingen server.** Ren HTML + CSS + JS i en fil.

---

## 2. Arbetssätt — VIKTIGAST AV ALLT

### Diskutera alltid först, bygg sedan

**Skriv ALDRIG kod förrän användaren uttryckligen bekräftat.** Detta är den
regel som brutits oftast och orsakat mest frustration.

Flödet ska alltid vara:

1. Användaren beskriver vad han vill
2. Du **undersöker koden** och bekräftar hur det faktiskt fungerar idag
3. Du sammanfattar exakt vad som ska ändras, och ställer följdfrågor vid
   oklarheter
4. Användaren säger uttryckligen "ja", "nu kör vi", "klart att bygga" eller
   liknande
5. **Först då** bygger du

Användaren lägger ofta till fler punkter mitt i en diskussion ("jag hade ett par
saker till innan vi bygger"). Vänta in att han är klar. Fråga hellre "är det
något mer innan vi bygger?" än att börja tidigt.

### Undersök innan du påstår

Gissa aldrig om orsaken till en bugg. Använd `grep` och `view` för att läsa den
faktiska koden, och beskriv sedan vad du hittat. Flera av de viktigaste
buggfixarna i projektet kom från att faktiskt läsa koden istället för att anta.

### Var ärlig om begränsningar

Om ett verktyg inte fungerar, om du inte kan verifiera något, eller om du är
osäker — säg det rakt ut istället för att gissa. Användaren litar på att
information stämmer.

---

## 3. Obligatoriskt vid VARJE bygge

### Uppdatera tidsstämpeln — alltid

Längst ner i filen finns:

```html
<span class="update-line">Vibe coded with Claude by Conejo, last updated ÅÅÅÅ-MM-DD HH:MM</span>
```

Denna **ska alltid uppdateras till aktuellt datum och klockslag** vid varje nytt
bygge. Fråga inte, gör det automatiskt. Fråga användaren efter tid om du är
osäker på aktuell tid.

### Verifiera efter varje bygge

Kör alltid detta innan du presenterar resultatet:

```bash
python3 -c "
import re,subprocess
code=open('/mnt/user-data/outputs/index.html').read()
m=re.search(r'<script>([\s\S]*?)</script>',code)
with open('/tmp/test.mjs','w') as f: f.write(m.group(1))
r=subprocess.run(['node','--check','/tmp/test.mjs'],capture_output=True,text=True)
print('JS:','OK' if r.returncode==0 else r.stderr[:1000])

from html.parser import HTMLParser
class Checker(HTMLParser):
    def __init__(self):
        super().__init__(); self.stack=[]; self.errors=[]
    def handle_starttag(self,tag,attrs):
        void={'area','base','br','col','embed','hr','img','input','link','meta','param','source','track','wbr'}
        if tag not in void: self.stack.append((tag,self.getpos()))
    def handle_startendtag(self,tag,attrs): self.handle_starttag(tag,attrs)
    def handle_endtag(self,tag):
        if self.stack and self.stack[-1][0]==tag: self.stack.pop()
        else: self.errors.append(f'Unexpected </{tag}> at {self.getpos()}')
c=Checker(); c.feed(code)
print('HTML errors:',c.errors if c.errors else 'none')
print('Unclosed:',c.stack if c.stack else 'none')
"
```

Filen sparas som `.mjs` eftersom koden innehåller top-level await.

### Presentera med present_files

Avsluta alltid med `present_files` på `/mnt/user-data/outputs/index.html` plus en
kort sammanfattning på svenska av vad som ändrats.

---

## 4. Tillgänglighet — projektets kärna

Detta är en app för skärmläsaranvändare. Tillgänglighet är inte en detalj, det är
hela poängen.

### Undvik ARIA-brus

Den vanligaste tillgänglighetsbuggen i projektet har varit **för mycket ARIA**,
inte för lite. Konkreta lärdomar:

- `role="toolbar"` + `aria-label` → VoiceOver läser upp etiketten två gånger.
  Ta bort rollen, behåll etiketten.
- `role="list"` på en `<ul>` är redundant — `<ul>` har redan rätt roll.
- `<section aria-labelledby>` gör att VoiceOver annonserar sektionsnamnet
  automatiskt. Använd `<div>` + `<h2>` istället — rubriken är fortfarande
  navigerbar via rotorn, men utan extra uppläsning.
- `role="search"` skapar onödig uppläsning. Ta bort.
- `aria-live` på statusmeddelanden som uppdateras kontinuerligt (t.ex.
  uppdateringsklockslag) skapar konstant störande uppläsning. Använd bara
  `aria-live` på dedikerade återkopplingselement som uppdateras vid faktisk
  användarinteraktion.

**Tumregel:** semantisk HTML först. Lägg bara till ARIA när HTML inte räcker.

### Uppläsningsordning

Viktig information ska komma **direkt efter namnet**, inte i slutet:

- Zoner: `"Plattan. Unik zon. 150 m bort. Klockan 5. Ägd av X."`
- Vänner: `"TurferX. 80 m till Plattan. Äger 4 zoner. +15. Poäng 12450."`

### Meter kontra minuter

Bokstaven `m` betydde tidigare både meter och minuter, vilket gjorde att
talsyntesen läste fel. Lösningen:

- **Visuellt:** kompakt format behålls (`12m 35s`) via `formatDuration()`
- **Upplästt:** fullständiga ord via `formatDurationSpoken()`
  - Under 1 min: `"45 sekunder"`
  - 1 min–1 h: `"12 minuter och 35"`
  - 1 h+: `"1 timme, 12 minuter och 35"`

Bokstaven `m` får bara förekomma för avstånd i uppläst text.

### Element som byggs om medan man rör dem

Timers som bygger om DOM-element varje sekund kan förstöra ett element mitt i ett
VoiceOver-tryck, så att aktiveringen tyst misslyckas. Mönstret som löste det:
uppdatera bara textinnehållet om identiteten är oförändrad, och bygg bara om
elementet när det faktiskt behövs (se ägar-knappen i `renderDetailLive`).

### Övrigt

- Rubriker (`<h2>`) för alla sektioner så rotornavigering fungerar
- `tabindex="-1"` på rubriker som ska kunna ta emot fokus
- iOS Dynamic Type stöds via `rem`-baserad storlek
- Inga emoji i gränssnittet

---

## 5. Turf API — kritiska begränsningar

### Endpoints

```javascript
const API_BASE = 'https://api.turfgame.com/v5';           // /zones, /users
const PLAYER_LOCATION_API = 'https://api.turfgame.com/v4/users/location';
```

Officiell dokumentation finns på `https://api.turfgame.com/`.

### Hastighetsgräns: ETT anrop per sekund

Detta är den vanligaste orsaken till buggar i projektet. Symptom: "Only one
request per second allowed", "Kunde inte ladda zoner", data som ibland saknas.

**Regler:**

- Använd alltid `fetchJsonWithRetry(url, body)` — den gör automatiskt ett
  återförsök efter 1500 ms om hastighetsgränsen träffas
- Skicka **aldrig** flera anrop parallellt med `Promise.all`
- Sekvensera med `await sleep(1100)` mellan anrop
- Vid flera anrop i rad: hämta i samma ordning som informationen visas på sidan

### Poängfält — två olika

```javascript
u.points        // omgångspoäng (nollställs varje månad) → "Omgångspoäng"
u.totalPoints   // livstidspoäng → "Totala poäng"
```

Förväxla inte dessa. Profilknappen i verktygsfältet visar **omgångspoäng**.

### Unikazoner går inte att hämta via API

`/users` returnerar bara ett **antal** (`uniqueZonesTaken`), inte listan med
zonnamn. Därför krävs manuell filuppladdning från turf.lundkvist.com.

---

## 6. Arkitektur

### Vyer

`list-view`, `detail-view` (zon), `profile-view`, `player-detail-view`.
Navigering via History API. `playerDetailReturnView` spårar `'list'`/`'profile'`/`'detail'`.

### Centrala hjälpfunktioner

| Funktion | Syfte |
|---|---|
| `sleep(ms)` | Fördröjning |
| `fetchJsonWithRetry(url, body)` | POST med automatiskt återförsök vid hastighetsgräns |
| `getBlockStatus(zone)` | Returnerar `{blocked, label, css, remainingSec}` |
| `formatDuration(sec)` | Kompakt visuellt format |
| `formatDurationSpoken(sec)` | Fullständiga ord för uppläsning |
| `distStr(km)` | `"150 m"` eller `"1.23 km"` |
| `isUniqueZone(name)` | Sant om zonen **inte** finns i användarens lista |
| `nearestZoneToPlayer(player)` | Närmsta zon ur redan laddad lista |
| `fetchNearestZoneForFriend(player)` | Hämtar zoner runt vännens **egen** position |

### Verktygsfältet

Sticky, gult, tre rader:

- **Rad 0:** `#btn-profile` — visar namn, antal zoner, pph, omgångspoäng
- **Rad 1:** Uppdatera, Pausa, Avisera spelare
- **Rad 2:** Dölj tagna, Filter, Röstläge

Safari-fix för sticky-positionering:

```css
top: calc(env(safe-area-inset-top, 0px) - 1px);
```

`role="toolbar"` är medvetet borttaget (orsakade dubbel uppläsning).

### Filter

Alla zoner, Tillgängliga, Blockerade, Närmsta unikazoner, Sök zoner, Avancerad visning.

**Sök zoner** använder Nominatim för adressgeokodning, pausar automatiska
uppdateringar, rensar zonlistan, och visar upp till 5 adressförslag som knappar.
Avstånd och riktning beräknas relativt adressen, inte GPS-positionen.

**Avancerad visning** fungerar som Alla zoner (`getFilteredZones()` gör ingen
extra filtrering för den) men berikar varje zons text med två extra fält:

- **Höjdskillnad** relativt din position, direkt efter riktningen (`"15 m upp."`
  / `"20 m ner."`). Hämtas från **Open-Elevation** (`api.open-elevation.com`,
  öppen, nyckelfri, SRTM-baserad) — samma tjänst används för både din egen
  position och varje zon, så att jämförelsen inte blandar telefonens egen
  (ofta opålitliga) GPS-höjd med en separat datakälla. Ett enda batch-anrop
  per uppdatering (`ensureElevationData()`), cachas i `elevationCache` keyed på
  `lat.toFixed(4),lon.toFixed(4)`. Avrundas till närmaste 5 m, meddelas bara
  vid minst 10 m skillnad — annars visas inget höjdfält alls. Misslyckas
  anropet visas zonen ändå, bara utan höjd. En engångsdiagnos via `aria-live`
  (`#elevation-status`) meddelar en gång per app-session om hämtningen
  fungerade — session-scoped i minnet, sparas inte i localStorage.
- **Zontyp** (`zone.type.name` från Turfs egen `/zones`-respons, redan hämtad,
  inga extra anrop) direkt efter zonens namn, före "Unik zon." om båda gäller.
  Visas bara när namnet inte är det generiska `"Okänd"`/`"Unknown"`.

### Röstläge

`voiceMode` = `'vo'` (VO-aviseringar, standard) | `'voice'` (Inbyggd röst) | `'off'`.

VO-aviseringar och Röstläge bygger sina texter i **separata kodvägar**. Ändrar du
uppläst text måste du uppdatera båda. Detta har orsakat buggar flera gånger.

### localStorage

```
turf-lang, turf-interval, turf-speed, turf-volume, turf-voice-mode,
turf-compass, turf-username, turf-hide-taken, turf-player-notify,
turf-player-freq, turf-player-radius, turf-friends,
turf-friends-show-nearby, turf-players-hidden-nearby,
turf-unique-zones, turf-pause, turf-unique-auto-fetch,
turf-unique-count-baseline
```

---

## 7. Tvåspråkighet

Allt gränssnitt finns på svenska och engelska i objektet `T.sv` / `T.en`.

**När du lägger till text måste du:**

1. Lägga till nyckeln i **både** `T.sv` och `T.en`
2. Uppdatera den statiska HTML-texten (visas innan JS hunnit köra)
3. Lägga till uppdatering i `switchLang()` om elementet finns statiskt i HTML

Missas något av dessa syns fel språk vid språkbyte eller vid sidladdning.

Instruktionerna (tips) finns både som statisk HTML **och** som JS-arrayer.
`sectionSizes` måste matcha antalet tips per sektion.

---

## 8. Kända fallgropar

- **Deklarationsordning:** `let`-variabler som används i `loadPrefs()` måste
  deklareras före den körs, annars kraschar hela initialiseringen (temporal dead
  zone). Har hänt en gång.
- **CSS-specificitet:** inline `style="display:block"` vinner över
  `[hidden]`-attributets `display:none`. Använd en CSS-klass med
  `.klass[hidden]{display:none}` istället.
- **Kapplöpning vid uppstart:** zonlistan och spelarlistan hämtas parallellt. Om
  spelardata blir klar först saknas "Nära X"-info. Lösning: rendera om
  spelarlistan när zondata blir klar.
- **Normalisering av zonnamn:** unikazoner matchas mot en `Set` med
  gemener + trimmade namn (`uniqueZoneNamesNormalized`). Denna måste fyllas både
  vid filuppladdning **och** vid inläsning från localStorage.
- **`refreshAllBlockLabels()`** körs varje sekund och skriver över `aria-label`.
  Den måste använda samma etikettbyggare som den ursprungliga renderingen,
  annars försvinner t.ex. "Unik zon." inom en sekund.

---

## 9. Pågående arbete

### Automatisk hämtning av unikazoner (klar)

Webbmastern för turf.lundkvist.com har lagt till stöd specifikt för den här appen:
`GetZonefile.php?user=NAMN&country=voturf` returnerar samtliga unikazoner för
användaren, över alla regioner/länder, i ett enda KML-anrop.

Knappen "Aktivera automatisk hämtning" i unikazoner-sektionen på profilsidan
(ovanför filuppladdningsfältet) gör ett `fetch()` mot den URL:en och tolkar
svaret med samma `parseKml()`-funktion som filuppladdning redan använder. Vid
fel (t.ex. CORS) visas `fetchUniqueCorsError` via `aria-live`, och
filuppladdning finns kvar som fallback. Knappen är en engångsaktivering, inte
en toggle — efter lyckad aktivering visas den som en inaktiv statusbekräftelse
("Automatisk hämtning aktiverad") istället för att gå att stänga av igen.

**Automatisk uppdatering efter aktivering:** Turfs officiella API exponerar
bara ett antal (`u.uniqueZonesTaken`), inte namnen, men det antalet hämtas
redan på varje ordinarie uppdateringscykel i `refreshProfilePageSequential()`.
Så fort aktiveringen skett sparas det antalet som baslinje
(`turf-unique-count-baseline`). Vid varje efterföljande cykel jämförs nytt
antal mot baslinjen — har det ökat hämtas listan om igen tyst (`silent=true`,
ingen `aria-live`-avbrott) och baslinjen uppdateras. Misslyckas den tysta
hämtningen lämnas baslinjen orörd så nästa cykel försöker igen automatiskt.
Detta undviker att belasta turf.lundkvist.com med ett eget separat
pollningsintervall.

**Bakgrund:** filuppladdning stödjer `.txt` (tabb-separerad) och `.kml`.
Ett verkligt exportfilexempel (944 zoner, `Placemark`-struktur, alla regioner
i Sverige) har verifierats fungera med befintlig `parseKml()` utan ändringar.

---

## 10. Snabbreferens vid start av ny konversation

1. Läs denna fil
2. Läs `index.html` för att förstå nuläget
3. Fråga vad användaren vill åstadkomma
4. **Diskutera → bekräfta → bygg → verifiera → uppdatera tidsstämpel → presentera**
