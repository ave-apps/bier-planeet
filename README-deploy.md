# Bier-Planeet v1 — Deploy-instructies

Drie bestanden vormen de complete app:

| Bestand | Rol |
|---|---|
| `bier-planeet-v1.html` | De single-file front-end (Three.js + Leaflet, vanilla JS) |
| `Code.gs` | De Apps Script back-end (web-app, één deployment-URL) |
| `README-deploy.md` | Dit document |

Volg onderstaande stappen één keer; daarna hoef je voor inhoudelijke updates alleen de Sheet bij te werken.

---

## 1. Google Sheet aanmaken

1. Ga naar <https://sheets.new> en maak een nieuwe spreadsheet.
2. Geef het bestand een herkenbare naam, bijvoorbeeld **Bier-Planeet — datasource**.
3. Open `Extensies → Apps Script`. Er opent een nieuw tabblad met een leeg `Code.gs`-bestand.

## 2. Apps Script vullen

1. Verwijder de placeholder `function myFunction() {}`.
2. Plak de volledige inhoud van het meegeleverde `Code.gs` (uit deze map) in het editor-venster.
3. Druk op `Ctrl/Cmd + S` om op te slaan; geef het project desgewenst een naam, bijvoorbeeld *Bier-Planeet API*.

## 3. Eenmalig `setupSheets()` draaien

1. Selecteer in de functie-dropdown bovenin de editor `setupSheets`.
2. Klik op `Uitvoeren ▶`.
3. Eerste keer vraagt Apps Script om autorisatie:
   - Klik op **Machtigingen herzien**
   - Kies je Google-account
   - Klik op **Geavanceerd → Ga naar Bier-Planeet API (onveilig)**
   - Sta de scopes voor Spreadsheets toe.
4. Het script maakt 10 tabbladen aan, vult headers, en plaatst seed-data (12 brouwerijen, 13 bieren, 6 smaakprofielen, 6 evenementen, 2 winkels, 1 redactie, 1 vergelijkingssite, 2 historische locaties, 2 ingrediëntleveranciers en circa 14 relaties).
5. Bij succes verschijnt een dialoog met de tekst *"Bier-Planeet sheets aangemaakt …"*.

> **Belangrijk over lat/lng:** alle coördinaten worden geschreven als tekst met een leidende apostrof (`'52.342…`). Dit is de officiële workaround tegen de NL-locale die punten in decimalen vervangt door komma's. `doGet()` strip de apostrof bij uitlezen — je hoeft hier niets aan te doen, ook niet als je later handmatig pins toevoegt: zet altijd een `'` voor lat en lng.

## 4. Deployen als web-app

1. Klik rechtsboven op `Implementeren → Nieuwe implementatie`.
2. Kies bij **Type** het tandwieltje en selecteer `Web-app`.
3. Vul in:
   - **Beschrijving**: `Bier-Planeet API v1`
   - **Uitvoeren als**: `Ikzelf (jouw account)`
   - **Wie heeft toegang**: `Iedereen`
4. Klik op **Implementeren**.
5. Apps Script vraagt opnieuw bevestiging — accepteer.
6. Kopieer de **Web-app-URL** (eindigt op `/exec`). Dit is je `GAS_URL`.

## 5. URL koppelen in `bier-planeet-v1.html`

1. Open `bier-planeet-v1.html` in een editor.
2. Zoek bovenin de constante:
   ```js
   const GAS_URL = '';
   ```
3. Plak de URL tussen de quotes:
   ```js
   const GAS_URL = 'https://script.google.com/macros/s/AKfy.../exec';
   ```
4. Sla het bestand op, dubbelklik om te openen — de globe laadt en haalt JSONP-data binnen via `?actie=alles&callback=...`.

## 6. Updates in de toekomst

- **Inhoudelijk** (een brouwerij toevoegen, een evenementdatum aanpassen): bewerk de Sheet rechtstreeks. De volgende keer dat de pagina laadt, ziet de gebruiker de wijziging. Wil je sneller schakelen, klik dan op de refresh-knop voor evenementen — die haalt alleen de Evenementen-sheet op via `?actie=evenementen`.
- **Code wijzigen** (`Code.gs`): pas de code aan in de Apps-Script-editor. Klik daarna op `Implementeren → Implementatie beheren → bewerken (potlood) → Versie: nieuwe versie → Implementeren`. **De web-app-URL blijft gelijk** — geen wijziging in `bier-planeet-v1.html` nodig.
- **Een handmatige pin toevoegen**: open de juiste sheet, voeg een rij toe, vul lat/lng als `'52.1234` (apostrof-prefix), kies een unieke `id` en bewaar. Geen extra script nodig.

## 7. Veelvoorkomende valkuilen

| Symptoom | Oorzaak | Oplossing |
|---|---|---|
| Pins op een rare plek midden op de oceaan | NL-komma in lat/lng | Zet apostrof voor de waarde: `'52.342` i.p.v. `52,342` |
| `Fout: Script function not found: doGet` | Implementatie niet als web-app gepubliceerd | Stap 4 opnieuw doorlopen |
| Pagina blijft hangen op "Geodata laden…" | `GAS_URL` leeg of foute URL | Stap 5 controleren |
| `403 Forbidden` van Apps Script | Toegang niet op *Iedereen* gezet | Implementatie beheren → bewerken → toegang aanpassen |
| Datums tonen als `1970-…` | Lege datum-cel in Evenementen | Vul `start`, `eind`, `boek_deadline` in alle gepubliceerde evenement-rijen |

## 8. Privacy-noot voor v2

In v1 worden alle pins uit de seed-data getoond. In v2 komen daar:
- thuisbrouwer-opt-in via Jotform (alleen tonen na opt-in, jitter ±500 m op coördinaten);
- ingrediëntleveranciers en smaakprofiel-cluster-relaties als zichtbare overlays;
- schrijfacties via `doPost` voor reserveringsformulieren.

Tot dat punt is de back-end **read-only** en zijn alle gegevens publieke contactinformatie van bekende brouwerijen, festivals en winkels.

---

## Update naar v2

### 1. Apps Script
1. Open Apps Script (Extensies → Apps Script in je Bier-Planeet Sheet).
2. Vervang volledige inhoud van `Code.gs` door het nieuwe bestand uit deze map.
3. Druk `Ctrl/Cmd + S` om op te slaan.
4. Selecteer in de functie-dropdown `setupCustomSheets` en klik **Uitvoeren ▶**. Dit voegt drie tabbladen toe (`EigenBrouwers`, `EigenToplocaties`, `EigenBijdragen`) zonder bestaande data aan te raken. Idempotent: opnieuw draaien is veilig.
5. **Implementeren → Implementaties beheren → potlood-icoon → Versie: Nieuwe versie → Implementeren.** De web-app-URL blijft hetzelfde, dus `GAS_URL` in `bier-planeet-v1.html` hoeft niet te veranderen.

### 2. Frontend
Hard refresh van de pagina (Ctrl+Shift+R). Schemering-thema (warm amberbruin) laadt als default; in de header verschijnen vier nieuwe knoppen:
- ☀ / ☾ — wissel thema (Schemering ↔ Middernacht)
- ＋ Eigen pin — handmatige bijdrage in drie tabs
- ⤓ Untappd — CSV-import
- ✎ Mijn bijdragen — overzicht en verwijderen van eigen records

### 3. Eerste gebruik
- Klik op **+ Eigen pin** voor handmatige bijdragen — kies eerst een pseudoniem; de app maakt lokaal een gebruikers-ID aan dat aan elke bijdrage wordt gekoppeld.
- Klik op **⤓ Untappd** om een CSV-export uit Untappd Insiders in te lezen. Het CSV wordt lokaal in de browser geparseerd; alleen wat je in het filter-dialoog aanvinkt gaat naar de Sheet.
- Klik op **✎ Mijn bijdragen** om eigen records te bekijken of te verwijderen. Een Untappd-import kun je in één klap wegklappen.

### 4. Test-mode (alleen voor ontwikkelaar)

Activeer in DevTools-console:

```js
localStorage.setItem('bier_dev_mode','true')
```

Hard refresh: linksonder verschijnt een `[DEV-MODE]`-indicator. In het Untappd-modaal zijn dan twee extra knoppen zichtbaar:
- **Laad test-CSV** — leest `test/untappd-test-export.csv` lokaal in en draait door dezelfde parse-pipeline als een echte upload.
- **Wis test-data** — verwijdert alle records met `gid: GID-test0001` uit de drie eigen-sheets in één call.

Test-records gebruiken een vaste, reproduceerbare `untappd_user_hash` afgeleid van de string `'testbierdrinker'`, zodat ze ook bij her-import dezelfde slot bezetten.

---

## Update naar v3 (kroonkurk-pins, etiket-labels, Wijzig-knop)

### 1. Apps Script
1. Open Apps Script (Extensies → Apps Script in je Bier-Planeet Sheet).
2. Vervang volledige inhoud van `Code.gs` door het nieuwe bestand.
3. Druk `Ctrl/Cmd + S` om op te slaan.
4. Selecteer in de functie-dropdown `setupCustomSheetsV3` en klik **Uitvoeren ▶**. Dit voegt de kolom `dominant_biertype` toe aan `Brouwerijen` en `EigenBrouwers`. Bestaande data blijft intact. Idempotent: opnieuw draaien is veilig.
5. **Implementeren → Implementaties beheren → potlood-icoon → Versie: Nieuwe versie → Implementeren.** URL blijft hetzelfde — geen wijziging in `bier-planeet-v1.html` nodig.

### 2. Frontend
Hard refresh (Ctrl+Shift+R). Pins zien er anders uit (kroonkurk-textuur met 21 randpuntjes), labels zijn capsule-etiketten met serif-naam en mono-sublabel, en in *Mijn bijdragen* staat naast elke Verwijder-knop een **Wijzig**-knop.

### 3. Optioneel: vul biertype-velden
Open de Sheet, ga naar tabblad `Brouwerijen` of `EigenBrouwers`. Vul de nieuwe kolom `dominant_biertype` in voor brouwerijen waar dit zinvol is. Geldige waarden: `stout`, `porter`, `imperial-stout`, `lambic`, `geuze`, `kriek`, `witbier`, `witte`, `ipa`, `dipa`, `pale-ale`, `tripel`, `quadrupel`, `trappist`, `pilsner`, `lager`, `helles`, `saison`, `farmhouse`. Lege waarde = geen rand-hint.

De seed-data is al voorgevuld voor de bekende trappisten/abdijen (Westmalle, Westvleteren, Rochefort, Chimay, Orval, Achel, La Trappe → `trappist`), De Molen → `imperial-stout`, Jopen en 't IJ → `tripel`, Uiltje → `ipa`. Bij verse `setupSheets()`-installs zie je deze waarden direct; bij upgrade naar v3 op een bestaande Sheet blijven oude rijen leeg en kun je ze handmatig aanvullen.

### 4. Rand-hints — wat zie je?
- **Evenementen** (festivals, proeverijen): de pin krijgt een buitenring in de seizoens-kleur op basis van de `start`-datum (lente=hopgroen, zomer=gerstgeel, herfst=bok-aarde, winter=donker paars).
- **Brouwerijen** (regulier of eigen) met ingevuld `dominant_biertype`: de pin krijgt een ring in de bijbehorende biertype-kleur (trappist=warm amber, lambic=wijnrood, ipa=hoppig groengeel, …).
- Pins zonder bekende hint krijgen géén ring — afwezigheid is ook een keuze.
- Het bijbehorende etiket-label krijgt dezelfde rand-hint als buitenring, zodat pin en etiket visueel bij elkaar horen.

### 5. Wijzig-knop
In *Mijn bijdragen* staat naast elke Verwijder-knop een **Wijzig**-knop voor eigen handmatige items (Untappd-import is read-only). Klik opent het formulier-modaal met alle velden voor-ingevuld. Bij klik op **Wijzigingen opslaan** worden de niet-beschermde kolommen overschreven (id, gid, handle, untappd_user_hash en aangemaakt blijven onaangeroerd). De server weigert wijziging als de `gid` van de huidige gebruiker niet matcht met de eigenaar van de rij.
