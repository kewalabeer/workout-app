# Kernkracht — workout-app

Nederlandstalige workout-PWA voor thuistraining zonder materiaal. Zie
`briefing-workout-app.md` voor de volledige functionele briefing. Dit
document beschrijft hoe je de app deployt en legt het dataformaat vast,
zodat toekomstige Claude-sessies (bij het samenstellen van een nieuw
programmablok) er exact op kunnen aansluiten.

## Bestanden

- `index.html` — de volledige app: HTML, CSS en JavaScript inline. Geen
  build-stap, geen frameworks, geen externe requests na de eerste load.
- `manifest.webmanifest` — PWA-manifest met inline (data-URI) iconen.
- `sw.js` — service worker: cachet de app bij installatie en werkt
  daarna netwerk-eerst (altijd de nieuwste versie ophalen zolang er
  verbinding is), met een cache-fallback naar `index.html` voor de
  zeldzame keer dat er geen internet is.
- `README.md` — dit bestand.

## Deployen op GitHub Pages

1. Zet deze drie bestanden (`index.html`, `manifest.webmanifest`, `sw.js`)
   in de root van een GitHub-repository (of in `/docs` als je die map als
   Pages-bron instelt).
2. Ga naar **Settings → Pages** in de repository.
3. Kies bij **Source** de branch en map waarin de bestanden staan
   (bijv. `main` / `/root`).
4. Sla op. GitHub Pages publiceert de site op
   `https://<gebruikersnaam>.github.io/<repository>/`.
5. Open die URL op je telefoon in Chrome. Gebruik **Menu → App
   installeren** (of de installatie-banner die Chrome automatisch toont)
   om de PWA op je startscherm te zetten.
6. Een update van de app is simpelweg de bestanden opnieuw uploaden
   (`git push`). Omdat de service worker netwerk-eerst werkt, zie je een
   nieuwe versie normaal gesproken meteen bij de eerstvolgende keer dat
   je de app opent (met internetverbinding) — een handmatige refresh is
   zelden nodig. Wil je zeker weten dat een oude geïnstalleerde PWA de
   update pakt, verhoog dan ook `CACHE_NAME` in `sw.js`; dat forceert een
   opschoning van de oude cache bij `activate`.

Belangrijk: een PWA is alleen installeerbaar via HTTPS. GitHub Pages
levert dat standaard.

## Lokaal testen

Geen build-stap nodig. Serveer de map met een simpele statische server,
bijvoorbeeld:

```
python -m http.server 8080
```

en open `http://localhost:8080/index.html`. (Rechtstreeks openen als
`file://` werkt niet volledig omdat de service worker en manifest dan
niet correct registreren.)

## Dataformaat

Eén JSON-object, bewaard in `localStorage` onder de sleutel
`kernkracht_state_v1`, en identiek als exportbestand. Hoofdstructuur:

```json
{
  "schemaVersion": 1,
  "programma": {
    "blok": 1,
    "naam": "Blok 1",
    "duurWeken": 6,
    "startDatum": "2026-07-14",
    "verschovenDagen": 0,
    "blokreviewGedaan": false,
    "workouts": [
      {
        "id": "hoofd",
        "naam": "Hoofdworkout",
        "type": "hoofd",
        "duurMinTekst": "35 tot 40 min",
        "warmingUp": ["8x cat-camel", "10x schouderrollen"],
        "oefeningen": [
          { "oefeningId": "curlup", "sets": 3, "superset": null, "rustSec": 60 },
          { "oefeningId": "squat", "sets": 3, "superset": "4", "rustSec": 60 },
          { "oefeningId": "pushup", "sets": 3, "superset": "4", "rustSec": 60 }
        ]
      }
    ],
    "oefeningen": [
      {
        "id": "curlup",
        "naam": "McGill curl-up",
        "patroon": "anti-extensie",
        "animatie": "curlup",
        "startNiveau": 1,
        "niveaus": [
          { "n": 1, "naam": "Basis", "doelType": "holds", "min": 4, "max": 6, "holdSec": 8, "perKant": false }
        ],
        "uitleg": {
          "kort": "",
          "uitvoering": ["stap 1", "stap 2"],
          "fouten": ["fout 1"],
          "spieren": ["spiergroep 1"],
          "stoppen": "Bij pijn in rug of nek."
        }
      }
    ]
  },
  "voortgang": {
    "curlup": {
      "niveau": 1,
      "promotieTeller": 0,
      "zwaarTeller": 0,
      "nietLekkerTeller": 0,
      "vervangingsmarkering": false,
      "rangeAdjust": 0,
      "geintroduceerd": false
    }
  },
  "instellingen": {
    "trainingsdag": 2,
    "reminder": true,
    "reminderTijd": "20:00",
    "seizoen": true,
    "lengteCm": null,
    "laatsteExport": null,
    "disclaimerGezien": true,
    "autoRun": true,
    "spraak": true
  },
  "log": {
    "sessies": [
      {
        "datum": "2026-07-14",
        "workoutId": "hoofd",
        "sets": [
          { "oefeningId": "curlup", "set": 1, "niveau": 1, "kant": null, "reps": 5, "holdSec": 8, "seconden": null }
        ],
        "feedback": [ { "oefeningId": "curlup", "score": "goed" } ],
        "duurMin": 36
      }
    ],
    "gewicht": [ { "datum": "2026-07-14", "kg": 80.5 } ],
    "overgeslagenWeken": [ { "datum": "2026-07-14", "weekNummer": 2 } ],
    "blokreviews": [
      {
        "blok": 1,
        "datum": "2026-08-25",
        "perOefening": [ { "oefeningId": "curlup", "gevoel": "goed", "opmerking": "" } ],
        "algemeen": ""
      }
    ]
  }
}
```

### Velden en betekenis

**`programma`**
- `blok` (number), `naam` (string) — bloknummer en vrije naam.
- `duurWeken` (number) — lengte van het blok in weken (4–6).
- `startDatum` (ISO-datum) — datum van de eerste trainingsdag van het
  blok. Wordt bij onboarding automatisch bepaald als de eerstvolgende
  gekozen trainingsdag.
- `verschovenDagen` (number) — opgeteld bij `startDatum` om "week
  overslaan" te verwerken (elke overgeslagen week telt +7 op). Zo
  schuiven zowel de weekteller als de einddatum van het blok mee.
- `blokreviewGedaan` (boolean) — voorkomt dat de blokreview na
  bevestiging opnieuw wordt getoond.
- `workouts[].superset` — workouts met hetzelfde niet-null
  `superset`-label worden in de workout-flow afwisselend zonder extra
  rust uitgevoerd (bijv. squat/push-up als "4a/4b").
- `oefeningen[].niveaus[].perKant` — als `true` wordt de set gelogd voor
  links én rechts (geen rust tussen de kanten, wel na de set).
- `oefeningen[].niveaus[].doelType` — `"reps"`, `"holds"` (reps waarbij
  elke herhaling een `holdSec`-lange hold is) of `"seconden"` (één
  doorlopende hold, `min`/`max` zijn dan seconden).
- `oefeningen[].startNiveau` — alleen relevant bij import: het niveau
  waarmee de oefening in het nieuwe blok start (zie Import hieronder).
  Ontbreekt dit veld, dan start de oefening op niveau 1.
- `oefeningen[].animatie` — sleutel naar de ingebouwde SVG-animatie
  (zie `ANIMATIONS` in `index.html`). Nieuwe oefeningen die nog geen
  animatie hebben, krijgen `"animatie": null`; de app toont dan een
  neutrale placeholder in plaats van een geanimeerde stick figure.
- `oefeningen[].video` (optioneel) — directe YouTube-URL naar een
  uitlegvideo van die oefening, getoond als externe link (opent in een
  nieuw tabblad) bovenaan de volledige uitleg. Ontbreekt dit veld, dan
  valt de app terug op een YouTube-zoeklink met de oefeningnaam. Geen
  video embedden — dat zou alsnog een verbinding vereisen bij het
  openen van de app en een licentierisico met zich meebrengen (zie
  briefing, punt 2).

**`instellingen`** (aanvullend op de functionele instellingen)
- `autoRun` (boolean, default `true`) — of oefeningen met een vaste
  hold-tijd (`doelType` `"seconden"` of een niveau met `holdSec`)
  zichzelf automatisch starten, aftellen en loggen tijdens de
  workout-flow, zodat er geen tik nodig is. Oefeningen met vrije
  herhalingen (geen `holdSec`) kunnen niet automatisch geteld worden —
  daar blijft één tik per set nodig om de set te bevestigen. Elke stap
  in de workout-flow toont een badge ("Handsfree" of "1 tik per set")
  die dit per oefening/niveau aangeeft. De gebruiker kan dit tijdens de
  workout per sessie omschakelen met de knop "Automatisch" bovenaan elk
  scherm in de workout-flow; die schakelaar werkt los van de
  permanente instelling.
- `spraak` (boolean, default `true`) — of de app hardop (via de
  Web Speech API, `nl-NL`) aankondigt welke oefening en welk doel eraan
  komt, aftelt bij holds, en "klaar"/"rust"/"ga verder" uitspreekt.
  Puur lokale TTS van het besturingssysteem; geen externe request.
  Werkt niet gegarandeerd op elk toestel/elke browserversie — het is
  een hulpmiddel, geen vereiste voor de app om te functioneren.
- `azureKey`, `azureRegion`, `azureVoice` (alle optioneel, default `null`
  / `null` / `"nl-NL-FennaNeural"`) — als beide `azureKey` en
  `azureRegion` zijn ingevuld, gebruikt `speak()` Azure Cognitive
  Services Speech (neurale stem, natuurlijker dan de systeemstem) in
  plaats van de lokale Web Speech API. Zie "Azure-spraak" hieronder voor
  de afwegingen en installatie.

**`voortgang`** (per `oefeningId`)
- `niveau` — huidig niveau (verwijst naar `niveaus[].n`).
- `promotieTeller` — aantal opeenvolgende sessies waarin het maximum van
  het doelbereik is gehaald mét feedback anders dan "te zwaar" of
  "voelde niet lekker". Bij 2 verschijnt een promotievoorstel; na
  bevestigen of weigeren gaat de teller terug naar 0.
- `zwaarTeller` — aantal opeenvolgende sessies met feedback "te zwaar".
  Bij 2 verschijnt een aanpassingsvoorstel (niveau omlaag of doelbereik
  verlagen); gaat daarna terug naar 0.
- `nietLekkerTeller` — aantal opeenvolgende sessies met feedback
  "voelde niet lekker". Bij 2 wordt `vervangingsmarkering` op `true`
  gezet. De app vervangt de oefening zelf niet; de markering gaat mee in
  de export zodat een nieuw blok er rekening mee kan houden.
- `rangeAdjust` — optelling op `min`/`max` van het huidige niveau
  (gebruikt door "doelbereik verlagen").
- `geintroduceerd` — of de volledige uitleg voor deze oefening al eens
  automatisch is geopend (voorkomt dat dit bij elke set opnieuw gebeurt).

**`log.sessies[].sets[]`** — één rij per gelogde set (of per kant bij
bilaterale oefeningen). `reps` wordt gebruikt bij `doelType`
`"reps"`/`"holds"`, `seconden` bij `doelType` `"seconden"`, `holdSec` is
het (vaste) aantal seconden per herhaling indien van toepassing.

## Export en import

- **Export** downloadt het volledige state-object (`programma`,
  `voortgang`, `instellingen`, `log`) als één JSON-bestand.
- **Import** herkent twee soorten bestanden:
  1. **Volledig back-upbestand** (bevat `voortgang`, `instellingen` én
     `log` naast `programma`) — dit vervangt de volledige staat
     één-op-één. Hiermee geeft "exporteren en datzelfde bestand meteen
     weer importeren" een identieke staat terug.
  2. **Programma-alleen bestand** (`{ "schemaVersion": 1, "programma":
     {...} }`, zoals Claude dat oplevert na de analyse-prompt) —
     hiermee wordt alléén `programma` vervangen. `log`, `gewicht` en
     `instellingen` blijven ongewijzigd staan; per oefening wordt het
     niveau opnieuw bepaald via `startNiveau` (of 1 als dat veld
     ontbreekt) en gaan alle promotie-/degradatietellers en de
     vervangingsmarkering terug naar de startwaarde.
- Bij een ongeldig bestand (geen JSON, verkeerd `schemaVersion`, of een
  `programma` dat niet aan het schema voldoet) toont de app een
  duidelijke foutmelding en verandert er niets aan de bestaande data.

## Vaste analyse-prompt

De tekst die op het export-scherm staat (zie punt 9 van de briefing) is
de vaste instructie die je samen met het geëxporteerde JSON-bestand in
een Claude-chat plakt om het volgende programmablok te laten
samenstellen. Het antwoord van Claude moet een programma-alleen
JSON-bestand zijn zoals hierboven beschreven, dat direct te importeren
is.

## Azure-spraak (optioneel)

Standaard gebruikt de app de lokale Web Speech API van de telefoon/
browser (gratis, werkt altijd, kwaliteit hangt af van de TTS-engine die
op het toestel staat). Wie een natuurlijkere stem wil, kan in
Instellingen → Spraakbegeleiding → "Betere stem via Azure" een eigen
Azure Speech-sleutel en -regio invullen. Zodra beide zijn ingevuld,
gebruikt `speak()` (in `index.html`) Azure Cognitive Services Speech
in plaats van de systeemstem.

**Afwegingen, expliciet:**
- **Niet gratis boven een grens.** De gratis tier van Azure Speech geeft
  circa 500.000 tekens/maand aan neurale spraak — voor een paar
  workouts per week ruim voldoende, maar bij overschrijding wordt er
  gefactureerd op het Azure-account van de sleutelhouder.
- **De sleutel is niet geheim te houden in een statische site.** Er is
  geen server om 'm achter te verstoppen. De sleutel wordt daarom nooit
  in `index.html`/de repo opgeslagen, alleen in `localStorage` op het
  toestel van de gebruiker die 'm zelf invult. Wie devtools opent op
  dat toestel kan de sleutel wel zien.
- **Netwerkafhankelijk en iets trager.** Elke aankondiging is een
  HTTP-verzoek naar `https://{regio}.tts.speech.microsoft.com`. Korte,
  veelvoorkomende cues ("Vast.", "Los.", "Klaar.", "Rust, N seconden.",
  "Nog tien seconden.", "Ga verder.") worden per exacte tekst
  gecachet (`azureCache` in `index.html`) zodat ze na de eerste keer
  instant afspelen. Nieuwe/variabele tekst (oefeningnaam + doel) kost
  wel steeds een verzoek.
- **Automatische terugval.** Lukt het verzoek niet (geen internet,
  ongeldige sleutel, timeout na 4s, quotum op), dan valt de app
  stilzwijgend terug op de lokale systeemstem — spraak blijft dus altijd
  werken, ook zonder Azure.

**Eigen sleutel aanmaken:** een gratis Azure-account op
[azure.microsoft.com](https://azure.microsoft.com), daarna in de Azure
Portal een resource van het type "Speech" aanmaken (regio bijv.
`westeurope`), en de "Key 1" + regio uit die resource kopiëren naar de
instellingen van de app.
