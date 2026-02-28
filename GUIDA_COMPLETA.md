# 📘 GUIDA COMPLETA — Dashboard PNRR Molise

---

## 📋 INDICE
1. [Come funziona il sistema](#1-come-funziona-il-sistema)
2. [Primo deploy su GitHub Pages](#2-primo-deploy-su-github-pages)
3. [Aggiornamento settimanale](#3-aggiornamento-settimanale)
4. [Dominio personalizzato (nascondere il nome utente)](#4-dominio-personalizzato)
5. [Installare come App (PWA)](#5-installare-come-app-pwa)

---

## 1. COME FUNZIONA IL SISTEMA

### Architettura
```
pnrr_regis.db  ──→  genera_dashboard.py  ──→  site/
(database)          (script Python)            ├── index.html (dashboard completa)
                                               ├── manifest.json (PWA)
                                               ├── sw.js (offline)
                                               ├── icon-192.png
                                               └── icon-512.png
```

**Il principio è semplice:**
- Ogni settimana ricevi un nuovo database `pnrr_regis.db` aggiornato (dall'ETL)
- Esegui lo script `genera_dashboard.py` 
- Questo produce una cartella `site/` con tutti i file
- Carichi la cartella su GitHub → il sito si aggiorna automaticamente

**I dati sono DENTRO il file `index.html`** (non c'è nessun file `data.js` esterno da caricare separatamente). Questo evita errori come `PNRR_DATA is not defined`.

---

## 2. PRIMO DEPLOY SU GITHUB PAGES

### Passo 1: Crea un'organizzazione GitHub (per nascondere il nome personale)

> ⚠️ **IMPORTANTE:** Se crei il repository con il tuo account personale `griguoligiuseppe-gif`, il link sarà sempre `https://griguoligiuseppe-gif.github.io/pnrr-molise/`. Per evitarlo:

1. Vai su https://github.com/organizations/plan → clicca **"Create a free organization"**
2. Scegli un nome per l'organizzazione, ad esempio: `pnrr-molise` oppure `monitoraggio-pnrr` o qualsiasi nome istituzionale
3. Completa la creazione (è gratis)

Il link diventerà:
```
https://pnrr-molise.github.io/dashboard/
```
oppure
```
https://monitoraggio-pnrr.github.io/dashboard/
```

Se preferisci un dominio completamente personalizzato (es. `pnrr.regione.molise.it`), vedi la sezione 4.

### Passo 2: Crea il repository

1. Dalla pagina dell'organizzazione, clicca **"New repository"**
2. Nome: `dashboard` (o `pnrr-molise`)
3. Visibilità: **Public** (necessario per GitHub Pages gratuito)
4. ✅ Spunta "Add a README file"
5. Clicca **"Create repository"**

### Passo 3: Carica i file

1. Nella pagina del repository, clicca **"Add file"** → **"Upload files"**
2. Trascina TUTTI i file dalla cartella `site/`:
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `icon-192.png`
   - `icon-512.png`
3. In "Commit changes" scrivi: `Deploy iniziale dashboard PNRR`
4. Clicca **"Commit changes"**

### Passo 4: Attiva GitHub Pages

1. Vai su **Settings** (⚙️ in alto a destra)
2. Nel menu laterale sinistro, clicca **"Pages"**
3. Sotto "Source", seleziona:
   - Branch: **main**
   - Folder: **/ (root)**
4. Clicca **"Save"**
5. Aspetta 2-3 minuti

### Passo 5: Verifica

Torna su **Settings → Pages**. Vedrai:
```
Your site is live at https://[nome-organizzazione].github.io/dashboard/
```

Clicca il link per verificare che tutto funzioni.

---

## 3. AGGIORNAMENTO SETTIMANALE

### Metodo A: Manuale (senza installare nulla sul PC)

Questo è il metodo più semplice se non vuoi installare Python o Git.

#### Prerequisiti
- Avere il nuovo file `pnrr_regis.db` 

#### Procedura

1. **Chiedi a Claude di rigenerare la dashboard:**
   - Apri una nuova conversazione su claude.ai
   - Carica il file `pnrr_regis.db` aggiornato + i due loghi
   - Scrivi: *"Genera la dashboard PNRR Molise aggiornata da questo database. Usa lo stesso formato della versione precedente."*
   - Claude ti darà il nuovo `index.html`

2. **Aggiorna su GitHub:**
   - Vai sul tuo repository GitHub
   - Clicca su `index.html` nella lista dei file
   - Clicca l'icona ✏️ (matita) in alto a destra
   - Clicca i tre puntini ⋯ → **"Upload file"** (oppure cancella tutto e incolla il nuovo contenuto)
   - In alternativa: elimina il file vecchio, poi usa "Add file" → "Upload files" per caricare quello nuovo
   - Commit: `Aggiornamento dati [data]`
   - In 2-3 minuti il sito si aggiorna automaticamente

### Metodo B: Semi-automatico (con Python sul PC)

#### Prerequisiti
- Python 3.8+ installato
- Libreria Pillow: `pip install Pillow`

#### Primo setup (una volta sola)

```bash
# Crea una cartella di lavoro
mkdir pnrr-dashboard
cd pnrr-dashboard

# Metti qui dentro:
# - genera_dashboard.py
# - Logo_Regione_Molise__1_.png
# - Logo_PNRR.png
# - etl_regis_to_sqlite.py (opzionale, se fai anche l'ETL)
```

#### Ogni settimana

```bash
# 1. Copia qui il nuovo database
cp /percorso/al/nuovo/pnrr_regis.db .

# 2. Rigenera la dashboard
python3 genera_dashboard.py --db pnrr_regis.db

# 3. I file sono nella cartella site/
# Caricali su GitHub (vedi sotto)
```

#### Caricare su GitHub da terminale (opzionale)

Se installi Git:
```bash
# Prima volta: clona il repository
git clone https://github.com/[organizzazione]/dashboard.git
cd dashboard

# Ogni settimana:
# Copia i file generati
cp ../site/* .

# Carica
git add .
git commit -m "Aggiornamento dati $(date +%d/%m/%Y)"
git push
```

### Metodo C: Automatico con GitHub Actions (avanzato)

Puoi fare in modo che GitHub rigeneri la dashboard automaticamente quando carichi un nuovo database.

1. Nel repository, crea la cartella `.github/workflows/`
2. Crea il file `.github/workflows/aggiorna.yml`:

```yaml
name: Aggiorna Dashboard

on:
  push:
    paths:
      - 'pnrr_regis.db'  # Si attiva quando carichi un nuovo DB

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install Pillow
      
      - name: Genera dashboard
        run: python genera_dashboard.py --db pnrr_regis.db --output .
      
      - name: Commit e push
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add index.html sw.js
          git commit -m "Dashboard aggiornata automaticamente $(date +%d/%m/%Y)" || echo "Nessuna modifica"
          git push
```

3. Carica anche `genera_dashboard.py` e i loghi nel repository
4. **Da quel momento**: ogni volta che carichi un nuovo `pnrr_regis.db` su GitHub, la dashboard si rigenera da sola in 2-3 minuti!

#### Workflow con Drive/Cloud

Se il database è su Google Drive o OneDrive:
1. Scarica il file `pnrr_regis.db` dal cloud
2. Trascinalo nel repository GitHub (Upload files)
3. La GitHub Action fa il resto automaticamente

---

## 4. DOMINIO PERSONALIZZATO

### Opzione A: Organizzazione GitHub (GRATUITO, consigliato)

Come spiegato nel Passo 1, crea un'organizzazione con un nome istituzionale.
Il link sarà tipo: `https://monitoraggio-pnrr.github.io/dashboard/`

### Opzione B: Dominio personalizzato completo (richiede acquisto dominio)

Se vuoi un link tipo `https://pnrr-molise.it`:

1. **Acquista un dominio** da un registrar (es. Aruba, OVH, Google Domains) — circa €10-15/anno
2. **Configura DNS** presso il registrar:
   - Aggiungi 4 record A:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```
   - Aggiungi un CNAME: `www` → `[organizzazione].github.io`

3. **Su GitHub:** Settings → Pages → Custom domain → scrivi `pnrr-molise.it` → Save
4. ✅ Spunta "Enforce HTTPS"

Dopo qualche ora il sito sarà raggiungibile su `https://pnrr-molise.it`

### Opzione C: Usare Netlify con sottodominio personalizzato (GRATUITO)

1. Vai su https://app.netlify.com
2. Accedi con GitHub
3. Trascina la cartella `site/` nella pagina
4. Il sito va online in 30 secondi con un link tipo `random-name.netlify.app`
5. Vai su **Site settings → Domain management → Custom domains**
6. Cambia il nome in qualcosa tipo `pnrr-molise.netlify.app` (gratuito!)

---

## 5. INSTALLARE COME APP (PWA)

La dashboard include già il supporto PWA (Progressive Web App). 

### Su Android
1. Apri il link della dashboard in Chrome
2. Appare automaticamente un banner **"Aggiungi alla schermata Home"**
3. Se non appare: clicca ⋮ (tre puntini in alto a destra) → **"Installa app"** o **"Aggiungi a schermata Home"**
4. L'icona PNRR Molise appare sulla home come un'app normale

### Su iPhone/iPad (iOS)
1. Apri il link della dashboard in **Safari** (non Chrome!)
2. Tocca l'icona **condividi** (quadrato con freccia in alto)
3. Scorri e tocca **"Aggiungi alla schermata Home"**
4. Conferma il nome e tocca **"Aggiungi"**
5. L'app appare sulla schermata home

### Su PC/Mac
1. Apri in Chrome
2. Nella barra degli indirizzi appare un'icona ⊕ (installa)
3. Clicca → l'app si installa come applicazione desktop

### Funzionalità PWA
- ✅ Funziona anche offline (dopo il primo caricamento)
- ✅ Si aggiorna automaticamente quando il sito cambia
- ✅ Icona dedicata sulla home
- ✅ Apre senza barra del browser (modalità standalone)

---

## 🔄 RIEPILOGO WORKFLOW SETTIMANALE

```
Lunedì: Ricevi dati aggiornati → pnrr_regis.db

Opzione FACILE:
  → Carica DB su Claude → Scarica nuovo index.html → Upload su GitHub

Opzione MEDIA:
  → python3 genera_dashboard.py → Upload cartella site/ su GitHub

Opzione AUTOMATICA:
  → Upload pnrr_regis.db su GitHub → La Action rigenera tutto da sola
```

**Tempo richiesto: 5-10 minuti a settimana** (qualunque metodo)

---

## ❓ TROUBLESHOOTING

| Problema | Soluzione |
|----------|-----------|
| `PNRR_DATA is not defined` | Stai usando una versione vecchia con data.js separato. Usa il nuovo index.html all-in-one |
| Pagina bianca | Apri Console del browser (F12) e cerca errori. Di solito è un problema di cache: Ctrl+Shift+R |
| GitHub Pages non si aggiorna | Aspetta 5 minuti. Se persiste, vai su Settings → Pages e verifica che sia su branch main |
| PWA non si installa | Verifica che manifest.json e sw.js siano nella stessa cartella di index.html |
| Mappa non si vede | Serve connessione internet per le tile OpenStreetMap |
