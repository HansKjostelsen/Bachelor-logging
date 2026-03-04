# Oppsummering - 10. til 12. februar 2026

## Periode: Mandag 10. - Onsdag 12. februar

Den uka jobbet jeg med å utvide FlowCRT-systemet med fokus på ISO 9001-visualisering. Jeg fikset responsivitet i React Flow, la fundamentet for ISO 9001-modulen med klikkbare noder og navigasjon, og bygde ut redigerbare noder med multi-directional handles. Totalt ble 14 prosessområder etablert med realistiske flowdiagrammer og over 71 nye noder.

## Begrunnelse

Valgene som ble tatt i denne perioden var tett koblet til konkrete brukerbehov og tekniske begrensninger som oppsto underveis.

**Responsivitet med resize-lytter og fitView** ble prioritert fordi nodene i React Flow ved oppstart ikke tilpasset seg vindusstørrelsen. En hardkodet zoom på 0.7 blokkerte `fitView` fra å fungere dynamisk, og uten en resize-lytter ville layouten brytes hver gang brukeren endret vindusstørrelse. Løsningen med `useEffect` og `window.addEventListener('resize', ...)` er en standardtilnærming i React og gjør komponenten selvkorrigerende. `translateExtent` ble lagt til i forlengelsen av dette for å hindre at brukere panorer seg vekk fra innholdet — et lite men viktig UX-tiltak.

**Slug-basert routing** for undersidene var et bevisst valg for skalerbarhet. Å koble noder til URL-er via slug-felt gjør det enkelt å legge til nye prosessområder uten å endre navigasjonslogikken. Det holder også URL-ene lesbare og vedlikeholdbare.

**Redigerbare noder med dobbeltklikk og slette-knapper** ble implementert fordi FlowCRT skal kunne brukes til å bygge egne prosessflytdiagrammer, ikke bare vise statiske eksempler. `window.prompt()` og `window.confirm()` ble valgt som raske løsninger for redigering og sletting — de er funksjonelle uten å kreve egne modal-komponenter, noe som holdt kompleksiteten nede på dette stadiet.

**Multi-directional handles** ble innført fordi prosessflyt sjelden er strengt venstre-til-høyre. Med 8 handles (alle fire kanter, både source og target) kan brukere koble noder i alle retninger, noe som er nødvendig for å modellere reelle ISO-prosesser med tilbakekoblinger og parallelle steg.

**LabelNode** ble introdusert som en enkel separat komponent for å holde visuell informasjon adskilt fra interaktive noder. Det er et prinsipp om single responsibility — en komponent som kun viser tekst trenger ikke handles eller klikkhåndtering, og holder koden ryddigere.

**Realistiske prosessnoder i undersidene** (fremfor placeholders) ble valgt for å gjøre applikasjonen meningsfull allerede i denne fasen, slik at den kan brukes som demo og grunnlag for videre diskusjon med veileder og brukere.
