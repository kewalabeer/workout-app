# Briefing Claude Code: workout-app Bas

Bouw een Nederlandstalige workout-PWA voor thuistraining. Deze briefing is volledig: functionele eisen, het complete trainingsprogramma voor blok 1 inclusief uitlegteksten, het dataformaat en de vaste analyse-prompt voor programma-updates. Bouw exact wat hier staat; stel vragen bij tegenstrijdigheden in plaats van aannames te doen.

## 1. Context

Gebruiker: Bas, beginner in krachttraining, traint thuis zonder materiaal. Doel: een sterke core om rugklachten voor te zijn en algemeen sterker te worden. Speelt hockey (1x training, 1x wedstrijd per week in het seizoen). Toestel: OnePlus 13 (Android, Chrome, scherm 6,8 inch).

Kernprincipe: de app is de uitvoerder, Claude (via chat) is de programmeur van de inhoud. Elke 4 tot 6 weken exporteert Bas zijn data, laat Claude een nieuw programmablok maken en importeert dat. De app hoeft dus zelf geen programma's te genereren, alleen uitvoeren, loggen en promoveren binnen een blok.

## 2. Techniek

- Drie bestanden: `index.html` (alle HTML, CSS en JS inline), `manifest.webmanifest`, `sw.js` (service worker voor offline en installatie). Plus een `README.md` met deploy-instructie.
- Hosting: GitHub Pages. Geen build-stap, geen frameworks, geen externe requests na eerste load. Vanilla JS.
- Opslag: localStorage, alles in één JSON-object (zie schema, punt 8). Datavolume blijft klein.
- Oefeningsanimaties: inline SVG met twee afwisselende standen (CSS-animatie, stick figure met duidelijke gewrichtshoeken). Geen externe gifs, geen licentierisico, werkt offline. Kwaliteitseis: de twee standen moeten begin- en eindpositie van de beweging correct tonen.
- Mobiel-first. Tap-targets minimaal 48px. Dark mode via `prefers-color-scheme`. Rustig, eenvoudig design; UX gaat boven visuele franje.
- Taal: alles Nederlands, informeel (je-vorm).

## 3. Schermen

1. **Vandaag.** Toont de workout van vandaag (of eerstvolgende): naam, duur, aantal oefeningen. Eén grote startknop. Secundair: keuze micro-workout, knop "week overslaan", link naar voortgang en instellingen. Toont bloknummer en weeknummer binnen het blok.
2. **Workout-flow.** Eén oefening per scherm. Bovenaan: naam, niveau, SVG-animatie. Doel van de set (reps of seconden). Grote knoppen om reps te loggen (voorinvulling op doelaantal, plus/min). Ingebouwde timer voor holds en voor rust tussen sets (start automatisch na set-bevestiging, overslaan kan). Knop "volledige uitleg" altijd zichtbaar. Bij een oefening die de gebruiker nog nooit deed opent de volledige uitleg automatisch.
3. **Afsluitscherm.** Na de laatste oefening: alle oefeningen van de sessie onder elkaar, per oefening vier opties: te makkelijk / goed / te zwaar / voelde niet lekker. Alles staat standaard op "goed"; alleen afwijkingen aantikken. Eén bevestigknop. Daarna een korte samenvatting (sets, totaaltijd, eventuele promotievoorstellen).
4. **Oefeningdetail.** Animatie, korte uitleg, en uitklapbaar: uitvoering stap voor stap, veelgemaakte fouten, welke spieren, wanneer stoppen. Inhoud komt uit de programmadata (punt 7).
5. **Voortgang.** Per oefening: huidig niveau en verloop van reps/holds over de sessies (eenvoudige lijngrafiek of staafjes, geen dashboard). Optioneel tabblad gewicht (handmatige invoer, datum + kg, lijngrafiek).
6. **Blokreview.** Verschijnt aan het eind van het blok: per oefening algemeen gevoel plus vrij opmerkingenveld, en een algemeen opmerkingenveld. Daarna wordt de gebruiker naar export geleid.
7. **Instellingen.** Trainingsdag (standaard dinsdag), reminder aan/uit met tijd (standaard 20:00), seizoensmodus aan/uit, lengte (eenmalig, cm), export- en importknoppen, disclaimer.

## 4. Functionele eisen

- F1. Schema: in seizoensmodus 1 hoofdworkout per week als norm, micro-workouts optioneel. Buiten seizoensmodus: hoofdworkout plus 1 micro-workout als norm. De norm is zichtbaar maar dwingt nergens.
- F2. Week overslaan: één knop op het Vandaag-scherm. Het blok schuift een week op (einddatum verschuift mee). Geen streaks, geen schuldtaal. Overgeslagen weken worden gelogd.
- F3. Promotieregel per oefening: haalt de gebruiker twee sessies op rij in alle sets het maximum van het doelbereik, en was de feedback niet "te zwaar" of "voelde niet lekker", dan stelt de app promotie naar het volgende niveau voor. De gebruiker bevestigt of weigert. Nooit automatisch promoveren.
- F4. Degradatieregel: feedback "te zwaar" twee sessies op rij → app stelt voor een niveau terug te gaan of het doelbereik te verlagen.
- F5. "Voelde niet lekker" twee sessies op rij → oefening krijgt een markering die meegaat in de export; de app vervangt zelf niets.
- F6. Reminder: notificatie op de trainingsdag om de ingestelde tijd, best effort (Notification API plus wat er bij een geïnstalleerde PWA betrouwbaar werkt). Documenteer in de app-instellingen eerlijk dat dit niet gegarandeerd is en adviseer een alarm als backup.
- F7. Export: één JSON-bestand (download) met het volledige data-object. Import: accepteert een programma-object (punt 8), valideert tegen het schema, vervangt het programma, bewaart log en gewichtshistorie, reset promotietellers. Duidelijke foutmelding bij ongeldig bestand.
- F8. Backup-reminder: als de laatste export langer dan 31 dagen geleden is, toont het Vandaag-scherm een rustige melding.
- F9. Veiligheid: bij eerste gebruik en in instellingen een korte disclaimer: dit is geen medisch advies; pijn (anders dan spiervermoeidheid) is een reden om te stoppen en bij aanhouden een professional te raadplegen. Elke oefening heeft een eigen stopcriterium (punt 7).
- F10. Eerste gebruik: korte onboarding (naam blok, trainingsdag kiezen, seizoensmodus, lengte optioneel, disclaimer). Programma blok 1 zit ingebakken in de app als startdata.

## 5. Programma blok 1 (6 weken)

Onderbouwing voor de uitlegteksten: kern is de McGill Big 3 (curl-up, side plank, bird dog), aangevuld met de bewegingspatronen squat, push, pull, heupscharnier en anti-extensie. Vanwege hockey (gebogen houding, eenzijdige rotatie) relatief veel aandacht voor anti-rotatie en heupwerk. Hoofdworkout bij voorkeur niet op de dag voor de wedstrijd; dat is een tekstuele tip bij de instelling trainingsdag, geen harde blokkade.

### Hoofdworkout (35 tot 40 min, 1x per week)

Vaste warming-up (5 min): 8x cat-camel, 10x schouderrollen, 10x heupcirkels per kant, 8x lichte squat in rustig tempo.

Daarna, rust 45 tot 60 seconden tussen sets. Koppel B-oefeningen als superset (afwisselen zonder extra rust) om binnen 40 minuten te blijven:

| # | Oefening | Sets | Doel (niveau 1) |
|---|----------|------|-----------------|
| 1 | McGill curl-up | 3 | 4-6 holds van 8 sec per set |
| 2 | Side plank | 3 per kant | 10-20 sec |
| 3 | Bird dog | 3 | 5-8 per kant, 3 sec hold |
| 4a | Squat | 3 | 8-12 |
| 4b | Push-up progressie | 3 | 6-10 |
| 5a | Row progressie | 3 | 8-12 |
| 5b | Glute bridge | 3 | 10-15 |

### Micro-workout A: Core kort (10 min)

3x cat-camel als opwarming. Curl-up 2 sets, side plank 2 per kant, bird dog 2 sets. Doelen als hoofdworkout.

### Micro-workout B: Romp en heup (12 min)

3x cat-camel. Dead bug 2x6-10 per kant, front plank 2x doeltijd niveau, glute bridge 2x10-15.

## 6. Niveaus per oefening

Elk niveau heeft een eigen doelbereik. Promotie volgens F3.

- **McGill curl-up.** N1: basis, 4-6 holds van 8s. N2: ellebogen los van de vloer, 4-6 holds van 8s. N3: 5-8 holds van 10s.
- **Side plank.** N1: vanaf knieën, 10-20s. N2: vanaf knieën, 20-35s. N3: op voeten, 15-25s. N4: op voeten, 30-45s of met bovenste been geheven.
- **Bird dog.** N1: alleen arm óf been strekken, 6-10 per kant. N2: tegengestelde arm en been, 5-8 per kant met 3s hold. N3: 6-10 per kant met 5s hold en elleboog-knie aanraken tussen reps.
- **Squat.** N1: box squat naar stoel, 8-12. N2: vrije squat tot dijen horizontaal, 8-12. N3: squat met 3s pauze onderin, 8-10. N4: split squat, 6-10 per been.
- **Push-up progressie.** N1: tegen muur, 8-12. N2: handen op aanrecht of tafel, 8-12. N3: op knieën, 6-10. N4: volledige push-up, 5-10.
- **Row progressie.** N1: table row onder stevige tafel, knieën gebogen, 8-12. N2: towel row aan deurklinken, 8-12. N3: table row met gestrekte benen of towel row dieper achterover, 8-12.
- **Glute bridge.** N1: tweebenig, 10-15. N2: met 3s hold bovenin, 10-12. N3: marching, 6-10 per been. N4: single leg, 6-10 per been.
- **Dead bug.** N1: alleen benen, 8-12 per kant. N2: tegengestelde arm en been, 6-10 per kant. N3: traag tempo, hiel zweeft boven de vloer, 6-10 per kant.
- **Front plank.** N1: op knieën, 20-30s. N2: volledig, 20-35s. N3: volledig 40-60s of shoulder taps 8-12 per kant.

## 7. Uitlegteksten

Neem deze teksten letterlijk op in de programmadata. Structuur per oefening: kort (1 zin bij de animatie), uitvoering (stappen), fouten, spieren, stoppen.

**McGill curl-up.** Kort: til alleen hoofd en schouders een paar centimeter op, de onderrug beweegt niet. Uitvoering: lig op je rug, één been gestrekt, één knie gebogen. Leg je handen met de handpalmen onder de holte van je onderrug. Til hoofd en schouders een paar centimeter van de vloer zonder je rug te bewegen. Houd 8 tellen vast en blijf doorademen. Zak rustig terug. Wissel halverwege de set van been. Fouten: te hoog komen (dit is geen sit-up); kin naar de borst trekken; de onderrug plat in de vloer drukken. Spieren: rechte en schuine buikspieren, zonder belasting van de rugwervels. Stoppen: bij pijn in rug of nek.

**Side plank.** Kort: lichaam één rechte lijn op je zij, steunend op elleboog. Uitvoering: lig op je zij met je elleboog recht onder je schouder. Span je buik en billen aan en til je heup op tot je lichaam een rechte lijn vormt (niveau 1: vanaf de knieën). Houd vast en adem door. Zak rustig terug en wissel van kant. Fouten: heup laten zakken; schouder naar je oor optrekken; naar voren of achteren kantelen. Spieren: schuine buikspieren, diepe rompspieren aan de zijkant, schouderstabilisatoren. Stoppen: bij schouderpijn, of wanneer je de rechte lijn niet meer kunt houden.

**Bird dog.** Kort: op handen en knieën, strek arm en tegengesteld been zonder dat je romp beweegt. Uitvoering: handen onder je schouders, knieën onder je heupen. Span je buik licht aan. Strek één arm en het tegengestelde been tot horizontaal. Houd je rug stil en je bekken recht, alsof er een glas water op je onderrug staat. Houd 3 tellen vast en kom rustig terug. Wissel van kant. Fouten: doorzakken in de onderrug; heup laten meedraaien; arm of been hoger dan de romp tillen. Spieren: rugstrekkers, bilspieren, diepe core. Stoppen: bij rugpijn.

**Squat.** Kort: zak gecontroleerd alsof je gaat zitten en kom op kracht omhoog. Uitvoering: voeten op schouderbreedte, tenen iets naar buiten. Zak rustig naar beneden alsof je op een stoel gaat zitten (niveau 1: raak de stoel licht aan). Knieën in lijn met je tenen, hakken op de vloer, borst open. Kom vanuit je benen omhoog. Fouten: knieën naar binnen laten vallen; hakken van de vloer; rug bol maken onderin. Spieren: bovenbenen, bilspieren, core. Stoppen: bij kniepijn die tijdens de set toeneemt.

**Push-up progressie.** Kort: lichaam één rechte lijn, zak tot je ellebogen negentig graden maken en duw op. Uitvoering: plaats je handen iets breder dan schouderbreedte (niveau 1: tegen de muur; niveau 2: op het aanrecht). Maak je lichaam één rechte lijn van hoofd tot hakken (of knieën). Zak gecontroleerd tot je ellebogen ongeveer negentig graden maken, ellebogen schuin langs je lichaam. Duw jezelf terug omhoog. Fouten: heupen laten doorzakken; ellebogen wijd naar buiten; halve herhalingen. Spieren: borst, triceps, schouders, core. Stoppen: bij schouder- of polspijn.

**Row progressie.** Kort: hang achterover en trek je borst naar de tafel of deur, schouderbladen naar elkaar. Uitvoering (table row): ga onder een stevige tafel liggen en pak de rand vast. Trek je borst naar de tafelrand en knijp je schouderbladen naar elkaar. Zak gecontroleerd terug. Uitvoering (towel row): sla een stevige handdoek om beide klinken van een geopende deur, zet je voeten bij de deur en hang met gestrekte armen achterover. Trek je borst naar de deur. Test tafel, deur en handdoek eerst voorzichtig met weinig gewicht. Fouten: schouders optrekken; alleen met de armen trekken zonder schouderbladen te bewegen; heupen laten knikken. Spieren: bovenrug, biceps, grip. Stoppen: bij schouderpijn, of als het materiaal niet stabiel aanvoelt.

**Glute bridge.** Kort: duw je heupen omhoog vanuit je billen tot knie, heup en schouder een lijn vormen. Uitvoering: lig op je rug met gebogen knieën, voeten plat op heupbreedte, hakken dicht bij je billen. Span je billen aan en duw je heupen omhoog tot knie, heup en schouder een rechte lijn vormen. Houd kort vast en zak rustig terug. Fouten: doorduwen vanuit de onderrug (holle rug bovenin); voeten te ver van je billen; afzetten via je tenen. Spieren: bilspieren, hamstrings. Stoppen: hamstringkramp is onschuldig (zet je voeten dichterbij); rugpijn is een reden om te stoppen.

**Dead bug.** Kort: onderrug blijft in contact met de vloer terwijl arm en been langzaam strekken. Uitvoering: lig op je rug, armen recht omhoog, knieën gebogen boven je heupen. Druk je onderrug licht in de vloer en houd dat contact de hele oefening. Strek langzaam één been terwijl je de tegengestelde arm naar achteren brengt. Kom rustig terug en wissel. Fouten: onderrug komt los van de vloer; te hoog tempo; adem vasthouden. Spieren: diepe buikspieren. Stoppen: bij rugpijn of als je het vloercontact verliest.

**Front plank.** Kort: lichaam één rechte lijn op ellebogen en tenen, billen aangespannen. Uitvoering: ellebogen onder je schouders, lichaam één rechte lijn (niveau 1: vanaf de knieën). Span buik en billen aan en adem rustig door. Fouten: heupen te hoog of doorzakken; hoofd laten hangen. Spieren: de hele core. Stoppen: bij onderrugpijn.

## 8. Dataformaat

Eén JSON-object in localStorage, en identiek als exportbestand. Hoofdstructuur:

```json
{
  "schemaVersion": 1,
  "programma": {
    "blok": 1,
    "duurWeken": 6,
    "startDatum": "2026-07-14",
    "workouts": [
      {
        "id": "hoofd",
        "naam": "Hoofdworkout",
        "type": "hoofd",
        "oefeningen": [
          { "oefeningId": "curlup", "sets": 3, "superset": null, "rustSec": 60 }
        ]
      }
    ],
    "oefeningen": [
      {
        "id": "curlup",
        "naam": "McGill curl-up",
        "patroon": "anti-extensie",
        "niveaus": [
          { "n": 1, "naam": "Basis", "doelType": "holds", "min": 4, "max": 6, "holdSec": 8 }
        ],
        "uitleg": { "kort": "", "uitvoering": [], "fouten": [], "spieren": [], "stoppen": "" },
        "animatie": "curlup"
      }
    ]
  },
  "voortgang": { "curlup": { "niveau": 1, "promotieTeller": 0, "zwaarTeller": 0, "nietLekkerTeller": 0, "vervangingsmarkering": false } },
  "instellingen": { "trainingsdag": 2, "reminder": true, "reminderTijd": "20:00", "seizoen": true, "lengteCm": null, "laatsteExport": null },
  "log": {
    "sessies": [
      {
        "datum": "2026-07-14",
        "workoutId": "hoofd",
        "sets": [ { "oefeningId": "curlup", "set": 1, "niveau": 1, "reps": 5, "holdSec": 8 } ],
        "feedback": [ { "oefeningId": "curlup", "score": "goed" } ],
        "duurMin": 36
      }
    ],
    "gewicht": [ { "datum": "2026-07-14", "kg": 80.5 } ],
    "overgeslagenWeken": [],
    "blokreviews": [ { "blok": 1, "datum": "", "perOefening": [ { "oefeningId": "", "gevoel": "", "opmerking": "" } ], "algemeen": "" } ]
  }
}
```

Werk dit schema uit waar nodig, maar houd de veldnamen en de scheiding programma/voortgang/instellingen/log aan. Import accepteert een bestand met alleen `schemaVersion` en `programma`; de rest blijft staan, promotietellers gaan naar nul en het niveau per oefening wordt overgenomen als het nieuwe programma dat meegeeft (veld `startNiveau` per oefening is toegestaan). Documenteer het definitieve schema in de README, want toekomstige Claude-sessies moeten er exact op aansluiten.

## 9. Vaste analyse-prompt (in de app tonen bij export)

Toon deze prompt als kopieerbare tekst op het exportscherm:

> Hierbij de export van mijn workout-app (JSON). Analyseer mijn progressie en stel het volgende programmablok samen (4 tot 6 weken). Regels: houd de McGill Big 3 als vaste kern; feedback "te makkelijk" versnelt progressie, "te zwaar" vertraagt, oefeningen met vervangingsmarkering of herhaald "voelde niet lekker" vervang je door een alternatief binnen hetzelfde bewegingspatroon; houd rekening met de seizoensinstelling; ik train thuis zonder materiaal, adviseer alleen bij duidelijke meerwaarde een aanschaf (zoals een weerstandsband) en vraag dat eerst. Onderbouw je keuzes kort. Lever daarna het nieuwe blok als één JSON-codeblok met exact deze structuur: `{ "schemaVersion": 1, "programma": { ... } }`, inclusief volledige Nederlandse uitlegteksten per oefening (kort, uitvoering, fouten, spieren, stoppen) en `startNiveau` per oefening. Nieuwe oefeningen krijgen `"animatie": null`; die voeg ik later toe.

## 10. Acceptatiecriteria

1. Installeerbaar als PWA op Android (Chrome), werkt daarna volledig offline.
2. Volledige hoofdworkout doorlopen, loggen en afsluiten met feedback lukt zonder toetsenbordinvoer.
3. Promotievoorstel verschijnt na twee sessies op het maximum, en niet eerder.
4. Week overslaan verschuift het schema en logt de week.
5. Export gevolgd door import van hetzelfde bestand geeft een identieke staat; import van een los programma-object bewaart de historie.
6. Volledige uitleg opent automatisch bij een oefening die nog nooit is gedaan.
7. Alle teksten Nederlands, geen lorem ipsum, uitlegteksten uit punt 7 letterlijk overgenomen.
8. `index.html` bevat geen externe verwijzingen (fonts, CDN's, gifs).
