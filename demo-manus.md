# Demo-manus – FlowCRT bachelorpresentasjon

Dette manuset brukes mens appen vises live for sensor og veileder. Hvert steg har en handling (hva du gjør) og en manustekst (hva du sier).

---

## Intro

*[ Handling: Vis startsiden eller landingssiden til appen i nettleseren. ]*

"FlowCRT er en nettapplikasjon der bedrifter kan visualisere og tilpasse prosessene sine opp mot ISO-standarder som ISO 9001. I demoen her skal vi gå gjennom hele brukerreisen fra start til slutt – vi registrerer en ny bruker, kjøper tilgang til en standard, redigerer et prosessdiagram og viser at endringene er lagret til akkurat den brukeren."

---

## Steg 1: Registrer en bruker

*[ Handling: Naviger til registreringssiden. Fyll inn navn, e-postadresse og passord, og klikk "Registrer". ]*

"Vi starter med å opprette en helt ny bruker. Jeg fyller inn et navn, en e-post og et passord – og trykker registrer. Det du ser nå er at vi automatisk blir logget inn med en gang. Brukeren er opprettet med rollen 'Viewer', som betyr at man kan se innholdet i appen, men ikke redigere noe ennå."

---

## Steg 2: Kjøp en ISO-standard

*[ Handling: Naviger til siden for kjøp av standard, for eksempel via en "Kjøp tilgang"-knapp eller i en profilmeny. Klikk på kjøp for ISO 9001. ]*

"For å få redigeringstilgang må brukeren kjøpe tilgang til en standard. Jeg klikker på 'Kjøp ISO 9001'. Det som skjer i bakgrunnen er at rollen til brukeren oppgraderes fra 'Viewer' til 'Flow Admin'. Vi trenger ikke å laste inn siden på nytt – endringen skjer umiddelbart og vi er klare til å begynne å redigere."

---

## Steg 3: Naviger til ISO 9001-oversikten

*[ Handling: Naviger til ISO 9001-seksjonen i appen. Vis oversiktskartet med prosessområdene. ]*

"Her er ISO 9001-oversikten. Vi ser et prosessdiagram med de forskjellige prosessområdene innenfor standarden – ledelse, planlegging, støtte, drift og så videre. Hvert av disse er en klikkbar node som tar oss videre til den tilhørende undersiden."

---

## Steg 4: Åpne en underside og rediger

*[ Handling: Klikk deg inn på en underside, for eksempel "Strategi og styring". Dra en node til en ny posisjon. Dobbeltklikk på en node for å endre navn. Klikk på et handle-punkt for å koble to noder med en ny kant. ]*

"Jeg åpner undersiden for Strategi og styring. Her ser vi et flowdiagram med noder og kanter som viser prosessflyten. Fordi vi nå er logget inn som Flow Admin, er diagrammet redigerbart. Jeg kan dra noder rundt, jeg kan dobbeltklikke på en node for å endre navnet, og jeg kan koble til en ny node ved å dra fra et av tilkoblingspunktene på kanten av noden. De åtte punktene – opp, ned, venstre, høyre og hjørnene – gir full frihet til å bygge opp den strukturen som passer for virksomheten."

---

## Steg 5: Lagre

*[ Handling: Pek på lagreknappen. Vis at den er aktiv (ikke grået ut) etter at endringer er gjort. Klikk lagre. Vis at knappen grås ut igjen etter lagring. ]*

"Legg merke til lagreknappen her oppe. Den ble aktiv i det øyeblikket jeg begynte å gjøre endringer – det er en visuell indikator på at det finnes ulagrede endringer. Nå klikker jeg lagre... og knappen er grået ut igjen. Det betyr at alt som er i diagrammet nå er lagret."

---

## Steg 6: Logg ut og inn igjen

*[ Handling: Logg ut. Logg inn med de samme innloggingsdetaljene. Naviger tilbake til samme underside. ]*

"For å vise at dette faktisk er persistert, logger jeg ut og inn igjen. Jeg navigerer tilbake til Strategi og styring... og her er diagrammet akkurat slik jeg la det, med de endringene jeg akkurat gjorde. Data er lagret per bruker, så hadde vi logget inn med en annen konto, ville vi sett den brukerens versjon av diagrammet."

---

## Avslutt

*[ Handling: Ingen spesiell handling – se ut mot publikum. ]*

"Det vi nettopp viste er altså tre ting i kombinasjon: persistens per bruker via localStorage, rollebasert tilgangsstyring der kun Flow Admin kan redigere, og et interaktivt diagramverktøy bygget på React Flow. Takk."
