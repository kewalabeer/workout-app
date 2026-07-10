# Outline workout-app Bas

Versie 3, 9 juli 2026.
Uitgangspunten: beginner in krachttraining, alleen lichaamsgewicht (blok 1 zonder investeringen), preventief (geen actuele rugklachten), PWA op OnePlus 13, taal Nederlands. Daarnaast hockey: 1x training en 1x wedstrijd per week in het seizoen. Hoofdworkout standaard op dinsdag, dag instelbaar.

## 1. Doel

Een sterke core opbouwen om rugklachten voor te zijn en algemeen sterker te worden. Minimaal 1 hoofdworkout per week, met de optie een week over te slaan en de optie voor kortere extra workouts. Het programma is onderbouwd en ontwikkelt mee met je progressie.

## 2. Haalbaarheidscheck vooraf

Drie punten waar je aannames bijstelling nodig hebben.

**Frequentie.** Met hockey (training plus wedstrijd) zit je in het seizoen al op twee intensieve momenten per week. Dan is 1 hoofdworkout per week met optionele micro-workouts een verdedigbare opzet; meer plannen gaat ten koste van herstel. Buiten het seizoen ligt dat anders: dan is 2x per week het advies. De app krijgt daarom een seizoensinstelling die de norm aanpast. Hockey zelf traint je core overigens nauwelijks gericht, dus de hoofdworkout blijft in het seizoen gewoon staan, bij voorkeur niet op de dag voor de wedstrijd.

**Watch-integratie.** De OnePlus Watch 2 schrijft data naar Health Connect (Android). Een PWA kan Health Connect niet uitlezen; dat kan alleen een native Android-app. Belangrijker: hartslag en stappen zeggen weinig over krachtprogressie. Reps, sets en oefeningsniveau zijn de maat die telt. Advies: watch buiten scope. Wil je het later toch, dan is een handmatige of geëxporteerde koppeling vanuit de OHealth-app een optie, maar verwacht daar weinig van.

**Gewicht en lengte.** Gewicht is voor dit doel geen goede voortgangsmaat. Het wordt een optioneel logveld, meer niet. Lengte nemen we eenmalig op voor context.

**Pull-oefeningen.** Met alleen lichaamsgewicht is trekkracht (belangrijk voor rug en houding) lastig te trainen. V1 lost dit op met towel rows aan een deur en table rows. Keuze: blok 1 zonder investeringen. Bij de blokupdate na blok 1 komt de weerstandsband (circa 15 euro) opnieuw op tafel.

**Reminder.** Gewenst: notificatie om 20:00 op de trainingsdag. Eerlijke check: een PWA zonder server kan op Android niet gegarandeerd op een exact tijdstip een notificatie afleveren. De app probeert het via de beschikbare browser-API's (best effort bij geïnstalleerde PWA), maar de betrouwbare route is een terugkerend agenda-item of alarm op de telefoon. Beide worden in de briefing opgenomen: notificatie als feature, alarm als aanbevolen backup.

## 3. Programma-opzet

**Onderbouwing.** Kern is de McGill Big 3 (curl-up, side plank, bird dog), de best onderzochte oefeningen voor rugpreventie. Extra relevant voor jou: hockey speel je in een gebogen houding met veel eenzijdige rotatie, precies het belastingsprofiel waar lage rugklachten bij horen. Anti-rotatie en heupwerk krijgen daarom relatief veel gewicht in het programma. Daaromheen een volledig beginnersprogramma met vijf bewegingspatronen: anti-extensie en anti-rotatie (plank-varianten, dead bug), heupscharnier (glute bridge, later single leg), squat-patroon, push (knee push-up naar push-up) en pull (towel row). Elke oefening in het programma krijgt een korte bronvermelding in de uitleg.

**Progressiemodel.** Elke oefening heeft 3 tot 5 niveaus, van makkelijk naar zwaar. Je stroomt door als je twee sessies op rij 3 sets met het doelaantal reps haalt met goede vorm. De app stelt de promotie voor; jij bevestigt. Zo blijft vorm leidend, niet het getal.

**Blokken.** Het programma loopt in blokken van 4 tot 6 weken. Aan het eind van een blok exporteer je je resultaten en laat je Claude een nieuw blok samenstellen (zie punt 5).

**Veiligheid.** Bij elke oefening staat wanneer je moet stoppen (pijn, uitstralende klachten). De app geeft de standaardwaarschuwing dat pijn een reden is om een professional te raadplegen, niet om door te trainen.

## 4. App-functionaliteit v1

- **Vandaag-scherm.** Toont de workout van vandaag: hoofdworkout of korte variant, met keuze. Eén tik om te starten.
- **Workout-flow.** Oefening voor oefening. Per set reps loggen met grote knoppen. Ingebouwde timer voor planks en rustpauzes.
- **Oefeningdetail.** Simpele gif als basisuitleg. Daaronder altijd een knop "volledige uitleg": uitvoering stap voor stap, veelgemaakte fouten, welke spieren, wanneer stoppen. Bij een oefening die nieuw voor je is opent de volledige uitleg automatisch.
- **Oefening-feedback.** Aan het eind van elke training een afsluitscherm met alle oefeningen van die sessie onder elkaar. Per oefening één tik: te makkelijk / goed / te zwaar / voelde niet lekker. Standaard staat alles op "goed", dus je past alleen aan wat afweek. Tien seconden werk, geen telefoongedoe tijdens de training. Gaat mee in de export en stuurt de blokupdate. "Voelde niet lekker" twee sessies op rij markeert de oefening voor vervanging.
- **Blokreview.** Aan het eind van een blok een kort reviewscherm: per oefening je algemene gevoel plus een vrij opmerkingenveld. Aanvulling op de feedback per sessie.
- **Week overslaan.** Eén knop. Schema schuift op, geen streaks, geen schuldgevoel-mechaniek.
- **Trainingsdag en reminder.** Trainingsdag instelbaar (standaard dinsdag), verplaatsbaar als je schema wijzigt. Notificatie om 20:00 op de trainingsdag, best effort (zie punt 2).
- **Seizoensinstelling.** Schakelaar hockeyseizoen aan/uit. In het seizoen: 1 hoofdworkout als norm, micro-workouts optioneel. Buiten het seizoen: 2 workouts als norm.
- **Voortgang.** Per oefening: huidig niveau en repsverloop. Simpele grafiek, geen dashboard-toestanden. Optioneel gewichtslog.
- **Export en import.** Export van programma plus volledige log als één JSON-bestand. Import van een nieuw programmabestand vervangt het programma en bewaart de historie. Export dient ook als backup.

## 5. Claude-updatecyclus

Belangrijk onderscheid: een blokupdate is een inhoudsupdate, geen code-update. Claude Code is alleen nodig als de app zelf verandert (bugs, features, design).

Per blok (4 tot 6 weken):

1. Exporteer JSON uit de app (programma, log, oefening-feedback, blokreview).
2. Plak of upload in een gewone Claude-chat of Cowork, met de vaste analyse-prompt (wordt meegeleverd in de briefing).
3. Claude beoordeelt progressie en feedback, stelt nieuw blok samen, levert nieuw JSON-programmabestand.
4. Importeer in de app.

Het JSON-formaat wordt in de briefing vastgelegd zodat elke toekomstige Claude-sessie hetzelfde formaat oplevert. De oefening-feedback weegt mee: te makkelijk versnelt promotie, te zwaar vertraagt, "voelde niet lekker" leidt tot een alternatieve oefening voor hetzelfde bewegingspatroon.

## 6. Techniek

- Single-file PWA: HTML, CSS en JavaScript in één bestand. Installeerbaar als app-icoon via Chrome op je OnePlus 13, werkt offline.
- Data lokaal op de telefoon (IndexedDB). Geen server, geen account, niets verlaat je toestel.
- Hosting: één statische pagina, gratis via GitHub Pages. Nodig omdat een PWA alleen installeerbaar is vanaf HTTPS. Update van de app is een nieuw bestand uploaden.
- Risico: data verdwijnt als je Chrome-sitedata wist. Daarom een maandelijkse export-herinnering in de app.
- Gifs: uit een vrij bruikbare oefenbibliotheek (bijvoorbeeld free-exercise-db of wger, licentie wordt in de bouwfase gecheckt), lokaal opgeslagen zodat alles offline werkt.

## 7. Buiten scope v1

- Watch en Health Connect (zie punt 2).
- Betrouwbare pushnotificaties op exact tijdstip; vereist een server. De best effort-notificatie zit wel in v1.
- Voeding, slaap, cardio.

## 8. Vervolgstappen

1. Akkoord op deze outline.
2. Ik schrijf de briefing voor Claude Code: functionele eisen, programmablok 1 volledig uitgewerkt (oefeningen, niveaus, reps, uitlegteksten), JSON-schema en de vaste analyse-prompt voor de updatecyclus.
3. Jij schakelt naar Claude Code en laat de app bouwen.
