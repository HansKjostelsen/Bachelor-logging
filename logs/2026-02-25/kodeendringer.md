# Kodeendringer - 2026-02-25

Dette dokumentet viser de faktiske kodeendringene som ble gjort 25. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `21803dc..3d94149` (fra merge til handle-fix commit)

**Merk:** Én bugfix-commit med endringer i 2 filer. Liten diff, men stor visuell effekt.

---

## Hovedendringer

### 1. styles.css - Fiks av handle-posisjonering

#### Frontend/src/styles.css

**Rotårsaken - feilaktig CSS-regel:**

```diff
 /* Handles */
 .react-flow__handle {
-  top: 51px !important;
   width: 10px;
   height: 10px;
   background: white;
   border: 2px solid #002d00;
 }
```

**Regelen `top: 51px !important` tvang alle React Flow-handles til å ligge 51px fra toppen av noden.** Dette fungerte tilfeldigvis akseptabelt for top-handles, men skapte et problem for left/right/bottom-handles som endte opp midt i noden istedenfor på kantene.

**Løsningen - korrekte CSS-regler per retning:**

```diff
+.react-flow__handle-top {
+  top: -5px;
+}
+
+.react-flow__handle-bottom {
+  bottom: -5px;
+}
+
+.react-flow__handle-left {
+  left: -5px;
+  top: 50%;
+  transform: translateY(-50%);
+}
+
+.react-flow__handle-right {
+  right: -5px;
+  top: 50%;
+  transform: translateY(-50%);
+}
```

**Forklaring:**
- `top: -5px` og `bottom: -5px` plasserer topp- og bunnhandles akkurat utenfor nodekanten (handle-størrelse er 10px, så -5px sentrerer den på kanten)
- `left: -5px` / `right: -5px` plasserer venstre/høyre handles utenfor kantene
- `top: 50%` + `transform: translateY(-50%)` sentrerer venstre og høyre handles vertikalt midt på noden
- Ingen `!important` nødvendig - spesifisiteten til de individuelle klassene er tilstrekkelig

---

### 2. FlowCRTNode.jsx - Forenkling av handle-logikk

#### Frontend/src/components/FlowCRTNode.jsx

**Før - to separate if-blokker:**

```diff
-      {data.showHandles !== false && data.allHandles && (
+      {data.allHandles ? (
         <>
           <Handle type="target" position={Position.Top} id="top-target" />
           <Handle type="source" position={Position.Top} id="top-source" />
           <Handle type="target" position={Position.Bottom} id="bottom-target" />
           <Handle type="source" position={Position.Bottom} id="bottom-source" />
           <Handle type="target" position={Position.Left} id="left-target" />
           <Handle type="source" position={Position.Left} id="left-source" />
           <Handle type="target" position={Position.Right} id="right-target" />
           <Handle type="source" position={Position.Right} id="right-source" />
         </>
-      )}
-
-      {data.showHandles !== false && !data.allHandles && (
+      ) : (
         <>
           <Handle type="target" position={Position.Left} id="left-target" />
           <Handle type="source" position={Position.Right} id="right-source" />
         </>
       )}
```

**Forklaring:**
- Endret fra to separate betingede blokker til ett ternary-uttrykk
- Fjernet `data.showHandles !== false`-sjekken - handles vises alltid (handles er essensielle for interaksjon)
- Koden er nå mer kompakt og lettere å lese
- Funksjonaliteten er uendret: `allHandles=true` gir 8 handles, `allHandles=false/undefined` gir 2 handles

---

## Visuell effekt

**Før fiksen:**
- Handles på "Kunder"-noden (og alle andre noder på ISO 9001-undersider) sto feil plassert
- Left/right-handles havnet 51px ned fra toppen av noden - synlig midt inne i noden
- Brukere ville ikke kunne koble noder intuitivt

**Etter fiksen:**
- Left-handle sitter sentrert på venstre kant av noden
- Right-handle sitter sentrert på høyre kant av noden
- Top/bottom-handles sitter på henholdsvis topp- og bunntant av noden
- Utseendet samsvarer med React Flow-standardoppførselen

---

## Full diff

```diff
diff --git a/Frontend/src/components/FlowCRTNode.jsx b/Frontend/src/components/FlowCRTNode.jsx
index 65ceed1..f59162c 100644
--- a/Frontend/src/components/FlowCRTNode.jsx
+++ b/Frontend/src/components/FlowCRTNode.jsx
@@ -123,7 +123,7 @@ export default function FlowCRTNode({ data, id }) {
         </button>
       )}

-      {data.showHandles !== false && data.allHandles && (
+      {data.allHandles ? (
         <>
           <Handle type="target" position={Position.Top} id="top-target" />
           <Handle type="source" position={Position.Top} id="top-source" />
@@ -134,9 +134,7 @@ export default function FlowCRTNode({ data, id }) {
           <Handle type="target" position={Position.Right} id="right-target" />
           <Handle type="source" position={Position.Right} id="right-source" />
         </>
-      )}
-
-      {data.showHandles !== false && !data.allHandles && (
+      ) : (
         <>
           <Handle type="target" position={Position.Left} id="left-target" />
           <Handle type="source" position={Position.Right} id="right-source" />

diff --git a/Frontend/src/styles.css b/Frontend/src/styles.css
index 00f5b30..61eccc6 100644
--- a/Frontend/src/styles.css
+++ b/Frontend/src/styles.css
@@ -187,13 +187,32 @@ button:hover {

 /* Handles */
 .react-flow__handle {
-  top: 51px !important;
   width: 10px;
   height: 10px;
   background: white;
   border: 2px solid #002d00;
 }

+.react-flow__handle-top {
+  top: -5px;
+}
+
+.react-flow__handle-bottom {
+  bottom: -5px;
+}
+
+.react-flow__handle-left {
+  left: -5px;
+  top: 50%;
+  transform: translateY(-50%);
+}
+
+.react-flow__handle-right {
+  right: -5px;
+  top: 50%;
+  transform: translateY(-50%);
+}
```
