# Demo-manus – FlowCRT bachelorpresentasjon

Dette manuset brukes mens appen vises live for sensor og veileder. Hvert steg har en handling (hva du gjør) og en manustekst (hva du sier).

---

## Teknisk grunnlag – React og React Flow-standarder

*[ Handling: Ingen spesiell handling – stå rolig og snakk til publikum. ]*

"Før vi går inn i selve demoen vil jeg si litt om hvilke tekniske valg som ligger bak applikasjonen, og hvorfor vi tok dem.

Hele frontenden er bygget i React, og vi har lagt vekt på å følge de anbefalingene som React selv gir i dokumentasjonen sin. Et sentralt prinsipp der er det som kalles separation of concerns – at hver del av koden skal ha ett tydelig ansvar. React sin egen dokumentasjon formulerer det slik: 'a component should ideally only be concerned with one thing.' Det har vi fulgt ved å skille logikk fra brukergrensesnitt.

Konkret betyr det at vi bruker custom hooks. React beskriver custom hooks som en måte å skjule komplekse detaljer og la komponentkoden uttrykke hensikten, ikke implementasjonen. Sitatet fra dokumentasjonen er: 'The code of your components expresses your intent, not the implementation.' Vi har to egne hooks: `useFlowPage`, som håndterer all logikk rundt lasting, lagring og tilstand for flowdiagrammene, og `useAuth`, som holder styr på hvem som er innlogget og hvilken rolle de har.
[Kilde: react.dev/learn/reusing-logic-with-custom-hooks]

For global auth-tilstand bruker vi Context API. `useContext` lar komponenter hente informasjon fra overordnede komponenter uten å sende props gjennom hele treet. Det passer perfekt til innloggingsstatus, der mange komponenter langt nede i hierarkiet trenger å vite om brukeren er Flow Admin eller ikke.
[Kilde: react.dev/reference/react/hooks]

Vi bruker også `useCallback` for å cache funksjoner som sendes ned til underkomponenter, og `useState` på riktig nivå slik at tilstanden sitter der den naturlig hører hjemme.
[Kilde: react.dev/reference/react/hooks]

Når det gjelder React Flow, som er biblioteket vi bruker for selve flowdiagrammene, har vi fulgt de anbefalingene de gir i sin dokumentasjon. En ting de er tydelige på er at `nodeTypes`-objektet – altså registreringen av hvilke node-typer som finnes – skal defineres utenfor React-komponenten. Grunnen er at React Flow sammenligner dette objektet mellom renders, og hvis det lages på nytt hver gang vil alle noder re-rendres unødvendig. Det er en liten detalj, men den har stor effekt på ytelsen.
[Kilde: reactflow.dev/examples/nodes/custom-node]

Vi bruker kontrollert tilstand for noder og kanter via `applyNodeChanges` og `applyEdgeChanges`. Det betyr at vi selv eier staten og forteller React Flow hva som har skjedd, i stedet for å la biblioteket styre alt internt. Mønsteret ser slik ut i koden: `onNodesChange` tar imot en liste med endringer og bruker `applyNodeChanges` for å oppdatere staten – immutabelt og kontrollert.
[Kilde: reactflow.dev/learn]

Selve nodene vi bruker er egendefinerte komponenter – custom nodes. React Flow sin dokumentasjon sier: 'Creating custom nodes is as easy as building a regular React component and passing it to nodeTypes.' Vår `FlowCRTNode`-komponent støtter inline redigering av navn, sletting, og åtte retningsstyrt tilkoblingspunkter – det React Flow kaller handles. Disse handle-IDene er eksplisitte, slik at vi kan styre nøyaktig hvilken retning en kant kan kobles i hvilken ende.
[Kilde: reactflow.dev/learn/concepts/introduction]

Til slutt bruker vi `fitView` med en resize-lytter, slik at diagrammet alltid tilpasser seg vindusstørrelsen og viser alle noder uansett skjermstørrelse.

Så oppsummert: vi har fulgt etablerte mønstre for begge bibliotekene – ikke fordi vi måtte, men fordi det gir en applikasjon som er lettere å vedlikeholde, som yter bedre, og som er enklere å forstå for noen som leser koden for første gang.

La oss nå gå over til selve demoen."

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
