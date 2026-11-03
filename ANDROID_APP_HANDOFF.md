# Android-app handoff — Javaanse Kalender

## Doel

Gebruik deze handoff als startpunt om de huidige Javaanse kalender als Android-app te bouwen. De huidige webprototype is [index6.html](index6.html). Deze handoff beschrijft wat is afgesproken, wat nog onzeker is en hoe de Android-versie kan worden opgezet.

## Aanbevolen aanpak

Bouw de Android-app met **Kotlin en Jetpack Compose**. Houd kalenderberekeningen in een aparte, pure Kotlin-module zodat ze los van de schermen getest kunnen worden.

Voorgestelde onderdelen:

- **CalendarCore**: omzetting tussen Javaanse en Gregoriaanse datums, maandlengtes, weekdag, Pasaran en bijzondere-dagregels.
- **UI**: Compose-schermen voor maandkalender, vorige/volgende maand, datum kiezen en bijzondere dagen van de maand.
- **Offline**: voor de huidige functionaliteit is geen server of account nodig; kalendergegevens en rekenregels kunnen lokaal worden opgenomen.
- **Tijd en datums**: gebruik java.time.LocalDate voor kalenderdagen. Vermijd tijdstippen en impliciete tijdzoneconversies bij datumrekenen.

Een WebView kan als snelle demonstratie dienen, maar voor een onderhoudbare Android-app verdient een native Compose-interface met gedeelde, geteste kalenderlogica de voorkeur.

## Bevestigde kalenderregels

1. Het Javaanse jaar heeft **10 maanden**.
2. **Oneven maanden hebben 36 dagen**; **even maanden hebben 35 dagen**. Daarmee telt het jaar 355 dagen.
3. De Pasaran-volgorde is **Legi → Pahing → Pon → Wage → Kliwon**.
4. Dag 1 van ieder Javaans jaar is **Pon**. De Pasaran-index kan vanaf de eerste dag van het jaar worden berekend; 355 is deelbaar door 5, zodat opeenvolgende jaarstarts ook op Pon uitkomen.
5. **HOD + Kliwon** heet *Srengat Gusti*. Het is alleen in **maand 7** een benoemde bijzondere dag.
6. De twee lijsten met pechdagen zijn **vaste kalendergegevens**, niet een afgeleide formule:
   - vaste Javaanse dagnummers per maand;
   - vaste combinaties van weekdag en Pasaran per maand.
7. Benoemde vaste dagen die in de prototypecode staan:
   - maand 1, dag 1: HOSORO;
   - maand 1, dag 36: PANGENEP;
   - maand 4, dag 8: PANGAPES;
   - maand 5, dag 15: PANJEPI;
   - maand 7, dag 21: PROLOJO MODJO;
   - maand 8, dag 7: PAMUKAS; dag 12: PAMUKAS II; dag 16: KAHURIPAN.

## Huidige prototype-aannames die eerst moeten worden gevalideerd

index6.html heeft de volgende overgenomen of tijdelijke aannames:

- Gregoriaanse ankers: **5 augustus 2023 = Javaans jaar 6449, dag 1** en **20 december 1975 = Javaans jaar 6400, dag 1**. Controleer of beide ankers met dezelfde 355-dagenregels kloppen. Gebruik één gevalideerde bron of een expliciete tabel met jaarstarts als de regels historische uitzonderingen kennen.
- Weekdagberekening gebruikt een vaste verschuiving van UTC+7. Bevestig dat dit de juiste daggrens voor deze kalender is.
- De namen DOT, GOT, GOS, JOS, JOH, HOD en WOR worden in de koppen in de huidige volgorde aan zondag tot en met zaterdag gekoppeld. Bevestig die koppeling aan de hand van de papieren kalender.
- De pechdagen en bijzondere dagen zijn uit de bestaande HTML overgenomen. Controleer de spelling, afkortingen en alle maandlijsten met de kalenderbron.
- De documentpassage die 35 dagen voor een Javaanse maand noemt, wijkt af van de bevestigde kalenderindeling van 36 dagen in oneven maanden en 35 dagen in even maanden. Voor de prototypeberekening geldt voorlopig de kalenderindeling die de gebruiker heeft bevestigd.

Laat de app onzekerheden niet stilzwijgend als historische feiten presenteren. Leg de gevalideerde bron en eventuele uitzonderingen vast.

## Functionele scope voor eerste Android-versie

- Open de kalender op de huidige lokale datum.
- Toon één Javaanse maand tegelijk.
- Navigeer vooruit en terug tussen maanden en Javaanse jaren.
- Toon per dag: Javaans dagnummer, Pasaran plus waarde, Gregoriaanse datum.
- Markeer vandaag, vaste pechdagen en benoemde bijzondere dagen visueel verschillend.
- Toon onder de maand een lijst met bijzondere dagen en Gregoriaanse datums.
- Laat een Gregoriaanse datum kiezen en open de bijbehorende Javaanse maand.
- Werk volledig offline.

Bouw geen gebruikersaccounts, synchronisatie, notificaties of kalenderdeling in de eerste versie tenzij dit later expliciet wordt gevraagd.

## Rekenmodel voor implementatie

Definieer één CalendarRules-bron met maandnamen, Pasaran-namen/afkortingen/waarden, pechdagtabellen en benoemde dagen.

Conceptueel:

- monthLength(month) = 36 als maand oneven is, anders 35.
- dayOffset(month, day) is day - 1 plus de lengtes van de voorafgaande maanden.
- Pasaran-index vanaf Javaans jaar dag 1 (Pon): ((2 + dayOffset) mod 5), met volgorde-indexen Legi=0, Pahing=1, Pon=2, Wage=3, Kliwon=4.
- Bij een Gregoriaanse conversie: bepaal eerst het Javaanse jaar en de dag-offset ten opzichte van een gevalideerde jaarstart; loop daarna door de maandlengtes om maand en dag te bepalen.
- Bereken de weekdag met een expliciete, gedocumenteerde daggrens en tijdzone. Laat maandweergave en lokale “vandaag”-markering niet afhangen van UTC-toevalligheden.

De formule voor Javaans jaar en Gregoriaans jaarstart mag pas als definitief gelden na controle van de ankers en historische kalenderbron.

## Testgevallen vóór release

Maak tests voor:

- alle maandlengtes en de som van 355 dagen;
- de eerste en laatste dag van elke maand;
- overgang van maand 10 naar maand 1;
- Pon op dag 1 en de vijfdaagse Pasaran-cyclus;
- HOD + Kliwon als Srengat Gusti, met speciale markering uitsluitend in maand 7;
- alle vaste pechdag- en Pasaran-combinaties;
- heen-en-terugconversie voor meerdere gevalideerde Gregoriaanse datums;
- jaargrenzen en datums vóór/na ieder anker;
- gedrag rond lokale middernacht en Android-tijdzones;
- toegankelijkheid en layout op kleine schermen.

Verzamel eerst een kleine set datums uit de papieren kalender met zowel Javaanse als Gregoriaanse datum en Pasaran. Gebruik die als gezaghebbende regressietestdata.

## Prompt om later de Android-bouw te starten

> Bouw een native Android-app voor de Javaanse Kalender op basis van ANDROID_APP_HANDOFF.md en de gevalideerde regels. Gebruik Kotlin en Jetpack Compose. Scheid de kalenderberekeningen van de UI, werk offline en behoud de maandnavigatie, Gregoriaanse datumkeuze, Pasaran, pechdagmarkeringen en lijst met bijzondere dagen. Begin met het controleren van de ankers en de meegeleverde testgevallen; wijzig de kalenderregels niet zonder de bronkalender te raadplegen.
