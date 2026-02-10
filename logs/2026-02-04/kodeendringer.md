# Kodeendringer - 2026-02-04

Dette dokumentet viser de faktiske kodeendringene som ble gjort 4. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `1f44ec7..940ebe8` (fra forrige dags siste commit til dagens siste commit)

**Merk:** I tillegg til kildekoden vist nedenfor, ble også `package-lock.json` (2996+ linjer) lagt til med alle npm-avhengigheter.

---

## Nye konfigurasjonsfiler

### Frontend/.gitignore

```diff
+# Logs
+logs
+*.log
+npm-debug.log*
+yarn-debug.log*
+yarn-error.log*
+pnpm-debug.log*
+lerna-debug.log*
+
+node_modules
+dist
+dist-ssr
+*.local
+
+# Editor directories and files
+.vscode/*
+!.vscode/extensions.json
+.idea
+.DS_Store
+*.suo
+*.ntvs*
+*.njsproj
+*.sln
+*.sw?
```

### Frontend/README.md & Frontend/Readme.md

```diff
+Her ligger nettside filene
```

### Frontend/package.json

```diff
+{
+  "name": "flowcrt-nettside",
+  "private": true,
+  "version": "0.0.0",
+  "type": "module",
+  "scripts": {
+    "dev": "vite",
+    "build": "vite build",
+    "lint": "eslint .",
+    "preview": "vite preview"
+  },
+  "dependencies": {
+    "react": "^19.2.0",
+    "react-dom": "^19.2.0",
+    "react-router-dom": "^7.13.0"
+  },
+  "devDependencies": {
+    "@eslint/js": "^9.39.1",
+    "@types/react": "^19.2.5",
+    "@types/react-dom": "^19.2.3",
+    "@vitejs/plugin-react": "^5.1.1",
+    "babel-plugin-react-compiler": "^1.0.0",
+    "eslint": "^9.39.1",
+    "eslint-plugin-react-hooks": "^7.0.1",
+    "eslint-plugin-react-refresh": "^0.4.24",
+    "globals": "^16.5.0",
+    "vite": "^7.2.4"
+  }
+}
```

### Frontend/eslint.config.js

```diff
+import js from '@eslint/js'
+import globals from 'globals'
+import reactHooks from 'eslint-plugin-react-hooks'
+import reactRefresh from 'eslint-plugin-react-refresh'
+import { defineConfig, globalIgnores } from 'eslint/config'
+
+export default defineConfig([
+  globalIgnores(['dist']),
+  {
+    files: ['**/*.{js,jsx}'],
+    extends: [
+      js.configs.recommended,
+      reactHooks.configs.flat.recommended,
+      reactRefresh.configs.vite,
+    ],
+    languageOptions: {
+      ecmaVersion: 2020,
+      globals: globals.browser,
+      parserOptions: {
+        ecmaVersion: 'latest',
+        ecmaFeatures: { jsx: true },
+        sourceType: 'module',
+      },
+    },
+    rules: {
+      'no-unused-vars': ['error', { varsIgnorePattern: '^[A-Z_]' }],
+    },
+  },
+])
```

### Frontend/vite.config.js

```diff
+import { defineConfig } from 'vite'
+import react from '@vitejs/plugin-react'
+
+// https://vite.dev/config/
+export default defineConfig({
+  plugins: [
+    react({
+      babel: {
+        plugins: [['babel-plugin-react-compiler']],
+      },
+    }),
+  ],
+})
```

### Frontend/index.html

```diff
+<!doctype html>
+<html lang="en">
+  <head>
+    <meta charset="UTF-8" />
+    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
+    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
+    <title>flowcrt-nettside</title>
+    <link rel="stylesheet" href="/src/styles.css" />
+  </head>
+  <body>
+    <div id="root"></div>
+    <script type="module" src="/src/main.jsx"></script>
+  </body>
+</html>
```

---

## React-applikasjon (src/)

### Frontend/src/main.jsx

```diff
+import { StrictMode } from 'react'
+import { createRoot } from 'react-dom/client'
+import './index.css'
+import App from './App.jsx'
+
+createRoot(document.getElementById('root')).render(
+  <StrictMode>
+    <App />
+  </StrictMode>,
+)
```

### Frontend/src/App.jsx

```diff
+import React from "react";
+import Header from "./layout/Header";
+import Footer from "./layout/Footer";
+import Home from "./pages/Home";
+import FAQ from "./pages/FAQ";
+import About from "./pages/about";
+import Contact from "./pages/Contact";
+import ISO9001 from "./pages/ISO9001";
+import ISO14001 from "./pages/ISO14001";
+import ISO27001 from "./pages/ISO27001";
+
+import { BrowserRouter, Routes, Route } from "react-router-dom";
+
+function App() {
+  return (
+    <BrowserRouter>
+      <div className="App">
+        <Header />
+        <Routes>
+          <Route path="/" element={<Home />} />
+          <Route path="/about" element={<About />} />
+            <Route path="/faq" element={<FAQ />} />
+            <Route path="/contact" element={<Contact />} />
+            <Route path="/iso9001" element={<ISO9001 />} />
+            <Route path="/iso14001" element={<ISO14001 />} />
+            <Route path="/iso27001" element={<ISO27001 />} />
+        </Routes>
+        <Footer />
+      </div>
+    </BrowserRouter>
+  );
+}
+
+export default App;
```

### Frontend/src/App.css

```diff
+(tom fil)
```

### Frontend/src/index.css

```diff
+/* Tom index.css for å unngå import-feil. Du kan legge til global CSS her om ønskelig. */
```

---

## Layout-komponenter

### Frontend/src/layout/Header.jsx

```diff
+import React from 'react';
+
+function Header() {
+  return (
+    <header className="header">
+      <nav className="navbar">
+        <div className="logo">FlowCRT Eksempel</div>
+        <ul className="nav-links">
+          <li><a href="/">Hjem</a></li>
+          <li><a href="/about">Om</a></li>
+          <li><a href="/contact">Kontakt</a></li>
+          <li><a href="/faq">FAQ</a></li>
+        </ul>
+      </nav>
+    </header>
+  );
+}
+
+export default Header;
```

### Frontend/src/layout/Footer.jsx

```diff
+import React from 'react';
+
+function Footer() {
+  return (
+    <footer className="footer">
+      <div className="footer-content">
+        <p>&copy; 2026 FlowCRT. Alle rettigheter reservert.</p>
+        <div className="footer-links">
+          <a href="/about">About</a> | <a href="/contact">Kontakt</a>
+        </div>
+      </div>
+    </footer>
+  );
+}
+
+export default Footer;
```

---

## Side-komponenter (pages/)

### Frontend/src/pages/home.jsx

```diff
+import React from 'react';
+
+function Home() {
+  return (
+    <main>
+      <h2>Velkommen!</h2>
+      <p>Dette er startsiden for ditt FlowCRT-prosjekt. Bygg videre her!</p>
+      <div className="card-container">
+        <a href="/iso9001" className="card iso9">ISO 9001</a>
+        <a href="/iso14001" className="card iso14">ISO 14001</a>
+        <a href="/iso27001" className="card iso27">ISO 27001</a>
+      </div>
+    </main>
+  );
+}
+
+export default Home;
```

### Frontend/src/pages/about.jsx

```diff
+import React from "react";
+
+function About() {
+  return (
+    <main>
+      <h2>About us</h2>
+      <ul>
+        Vi er et team dedikert til å bygge eksempler for FlowCRT.
+        <li>Vårt mål er å hjelpe utviklere med å komme i gang raskt.</li>
+        <li>Vi tilbyr ressurser og veiledninger for å lette læringsprosessen.</li>
+        <li>Kontakt oss for mer informasjon!</li>
+      </ul>
+    </main>
+  );
+}
+
+export default About;
```

### Frontend/src/pages/Contact.jsx

```diff
+import React from "react";
+
+function Contact() {
+  return (
+    <main>
+      <h2>Kontakt oss</h2>
+      <p>Du kan kontakte oss på e-post: <a href="mailto:kontakt@flowcrt.no">kontakt@flowcrt.no</a></p>
+      <p>Vi svarer vanligvis innen 24 timer.</p>
+    </main>
+  );
+}
+
+export default Contact;
```

### Frontend/src/pages/FAQ.jsx

```diff
+import React from "react";
+
+function FAQ() {
+  return (
+    <main>
+      <h2>FAQ</h2>
+      <ul>
+        <li><strong>Hva er FlowCRT?</strong> FlowCRT er et eksempelprosjekt for nettside.</li>
+        <li><strong>Hvordan kan jeg kontakte dere?</strong> Bruk kontaktlenken i footeren.</li>
+      </ul>
+    </main>
+  );
+}
+
+export default FAQ;
```

### Frontend/src/pages/ISO9001.jsx

```diff
+import React from "react";
+
+function ISO9001() {
+  return (
+    <main>
+      <h2>ISO 9001</h2>
+      <p>Dette er siden for ISO 9001 sertifisering.</p>
+    </main>
+  );
+}
+
+export default ISO9001;
```

### Frontend/src/pages/ISO14001.jsx

```diff
+import React from "react";
+
+function ISO14001() {
+  return (
+    <main>
+      <h2>ISO 14001</h2>
+      <p>Dette er siden for ISO 14001 sertifisering.</p>
+    </main>
+  );
+}
+
+export default ISO14001;
```

### Frontend/src/pages/ISO27001.jsx

```diff
+import React from "react";
+
+function ISO27001() {
+  return (
+    <main>
+      <h2>ISO 27001</h2>
+      <p>Dette er siden for ISO 27001 sertifisering.</p>
+    </main>
+  );
+}
+
+export default ISO27001;
```

---

## Styling

### Frontend/src/styles.css

```diff
+body {
+  margin: 0;
+  font-family: 'Segoe UI', Arial, sans-serif;
+  background: #f6f8fa;
+  color: #222;
+  min-height: 100vh;
+  display: flex;
+  flex-direction: column;
+}
+/* Main app wrapper for sticky footer */
+.App {
+  flex: 1 0 auto;
+  display: flex;
+  flex-direction: column;
+  min-height: 100vh;
+}
+
+main {
+  flex: 1 0 auto;
+}
+
+footer, .footer {
+  flex-shrink: 0;
+}
+
+header, .header {
+  background: #013220; /* Endret farge */
+  color: #fff;
+  padding: 0 0 1rem 0;
+  margin-top: 0;
+}
+
+footer, .footer {
+  background: #013220; /* Endret farge */
+  color: #fff;
+  padding: 1rem 0;
+  margin-bottom: 0;
+  text-align: center;
+  font-size: 1rem;
+  padding: 1.5rem 0 1rem 0;
+}
+
+.navbar {
+  display: flex;
+  align-items: center;
+  justify-content: space-between;
+  max-width: 1000px;
+  margin: 0 auto;
+  padding: 0 2rem;
+}
+
+.logo {
+  font-size: 1.5rem;
+  font-weight: bold;
+}
+
+.nav-links {
+  list-style: none;
+  display: flex;
+  gap: 2rem;
+  margin: 0;
+  padding: 0;
+}
+ .nav-links li a {
+  color: #fff;
+  text-decoration: none;
+  font-size: 1rem;
+  font-weight: 500;
+  padding: 0.3em 0.8em;
+  border-radius: 4px;
+  transition: background 0.2s, color 0.2s;
+}
+footer a:visited, .footer a:visited {
+  color: #1CEEEE;
+}
+.nav-links li a:hover, .nav-links li a:focus {
+  background: #123223;
+  color: #1CEEEE;
+}
+
+
+.card-container {
+  display: flex;
+  gap: 2rem;
+  justify-content: center;
+  margin-top: 2rem;
+  flex-wrap: wrap;
+}
+.card-container {
+  display: flex;
+  flex-wrap: wrap;
+  justify-content: center;
+  align-items: center;
+  gap: 2rem;
+  margin-top: 2rem;
+}
+
+.card {
+  background: #123223;
+  border-radius: 12px;
+  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
+  padding: 2rem 3rem;
+  font-size: 2rem;
+  font-weight: 600;
+  color: #fff;
+  cursor: pointer;
+  transition: transform 0.2s, box-shadow 0.2s, background 0.2s;
+  outline: none;
+  font-family: 'Segoe UI', Arial, sans-serif;
+  letter-spacing: 0.5px;
+}
+.card:hover, .card:focus {
+  background: #145c25;
+  color: #1CEEEE;
+  transform: translateY(-8px) scale(1.05);
+  box-shadow: 0 8px 24px rgba(20,92,37,0.15);
+  outline: 3px solid #1CEEEE;
+}
+
+
+button {
+  border-radius: 4px;
+  padding: 0.5rem 1.5rem;
+  font-size: 1rem;
+  cursor: pointer;
+  margin-top: 1rem;
+}
+
+button:hover {
+  background: #123223;
+}
```

**Viktig fargeendring i siste commit (940ebe8):**
- Header og footer bakgrunnsfarge ble endret fra `#123223` til `#013220` (mørkere grønn)

---

## Public assets

### Frontend/public/vite.svg

```diff
+(SVG-fil med Vite logo - binært innhold)
```

---

## Oppsummering av endringer

- **24 filer endret**, 3417 nye linjer (+), 1 linje slettet (-)
- Opprettet komplett React + Vite applikasjon med routing
- 7 side-komponenter (Home, About, Contact, FAQ, 3x ISO-sider)
- 2 layout-komponenter (Header, Footer)
- Fullstendig CSS-styling med grønn fargepalett
- Alle npm-avhengigheter installert via package-lock.json
