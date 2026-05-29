# Zoroaster — App Overview

## Cos'è

Zoroaster è un'app di finanza personale AI-powered per italiani digitalizzati (18-65 anni). L'utente carica i propri movimenti bancari (testo o PDF), l'AI li analizza, li categorizza, identifica pattern comportamentali e fornisce coaching personalizzato. L'obiettivo: chiudere l'app sapendo esattamente dove va il denaro e cosa fare la settimana successiva — senza dover studiare finanza.

**Posizionamento:** non una banca, non un'app wellness, non crypto. Il consulente delle 23:00 — strumento professionale costruito per sé stessi, preciso, scuro, senza ornamenti.

---

## Stack tecnico

| Layer | Tecnologia |
|-------|-----------|
| Web app | Next.js (App Router), TypeScript, Tailwind CSS |
| Mobile app | Expo / React Native, TypeScript |
| Backend / DB | Supabase (Postgres + Auth + RLS) |
| AI | Anthropic Claude (claude-sonnet-4-6) |
| Deploy web | Vercel (https://zoroaster-web.vercel.app) |
| Animazioni web | Framer Motion |
| Grafici web | Recharts |

---

## Architettura

```
zoroaster-web/          Next.js — app principale
  app/
    page.tsx            Landing page pubblica
    login/              Auth (Supabase email/password)
    dashboard/
      page.tsx          Overview — metriche aggregate
      analisi/          Upload + analisi AI movimenti
      coach/            Chat AI multi-conversazione
      accademia/        Piattaforma didattica finanziaria
      profilo/          Profilo comportamentale utente
      impostazioni/     Account settings
      radar/            (legacy — assorbito da accademia)
  api/
    analyze/            Parsing testo bancario → AnalysisResult
    analyze-pdf/        Estrazione testo da PDF
    analyze-extract/    Estrazione profile update da analisi
    analyze-news/       News finanziarie contestualizzate
    coach/              Stream conversazione Claude
    coach/summarize/    Titolo + sommario automatico conversazione
    coach/extract/      Estrazione trigger/strengths dal coach
    news/               Fetch news economiche italiane
    accademia/
      daily-tip/        Pillola giornaliera AI (cache DB)
      complete-lesson/  Segna lezione completata + XP

zoroaster-app/          Expo — mobile companion
  app/(tabs)/
    index.tsx           Home / dashboard
    analisi.tsx         Analisi movimenti
    radar.tsx           News + decisioni pre-acquisto
    profilo.tsx         Profilo comportamentale
    impostazioni.tsx    Settings
```

---

## Funzionalità attuali

### 1. Analisi AI dei movimenti bancari
- Input: testo libero (copia/incolla estratto conto) o PDF
- Output strutturato (`AnalysisResult`):
  - Lista movimenti categorizzati (9 categorie: Supermercato, Ristoranti & Bar, Trasporti, Shopping, Salute, Svago, Casa & Bollette, Viaggi, Altro)
  - Flag comportamentali per transazione: `alta` (spesa elevata), `impulsiva`, `ricorrente`
  - Riepilogo spese per categoria con percentuali
  - Analisi comportamentale testuale
  - 3 micro-obiettivi personalizzati con stima risparmio
  - Domanda di riflessione finale
- Storico analisi salvato in Supabase (`analyses`)

### 2. Coach AI
- Chat conversazionale con Claude
- Contesto iniettato automaticamente: profilo comportamentale utente + ultima analisi
- Multi-conversazione: ogni sessione ha titolo e sommario generati automaticamente dall'AI
- Estrazione automatica di `triggers`, `strengths`, `risk_categories` dal dialogo → aggiornamento profilo
- Stream token (risposta in tempo reale)
- Rate limiting server-side per sicurezza

### 3. Profilo comportamentale
- Tabella `user_profiles` in Supabase
- Campi: `triggers[]`, `strengths[]`, `risk_categories[]`, `budget_targets{}`, `goals_history[]`
- Aggiornato incrementalmente dopo ogni analisi e ogni sessione coach
- Usato come contesto permanente per personalizzare ogni risposta AI

### 4. Accademia (piattaforma didattica)
- 3 corsi con 5 lezioni ciascuno (15 lezioni totali):
  - **Fondamentali del Budget** (beginner, 📊, 120 XP): flusso di cassa, uscite invisibili, envelope budgeting, fondo emergenza, automazione risparmio
  - **Psicologia del Denaro** (beginner, 🧠, 130 XP): bias cognitivi, trigger emotivi, mentalità abbondanza/scarsità, abitudini durature
  - **Investire i Primi €5.000** (intermediate, 📈, 150 XP): prerequisiti, rischio/rendimento, ETF, fiscalità italiana, primo portafoglio
- Sistema XP per lezione completata
- Quiz a risposta multipla per ogni lezione (con spiegazione della risposta corretta)
- Progresso utente persistito (`academy_progress`)
- Pillola giornaliera AI personalizzata (`daily_tips`) — generata una volta al giorno, cachata in DB

### 5. Radar economico (web + mobile)
- Fetch notizie economiche italiane rilevanti
- AI le processa in `NewsAlert` con: categoria, urgenza, impatto stimato in €, pillola educativa, azione concreta
- Filtro per chi ha investimenti attivi (`per_investitori`)
- Urgenza codificata: `alta` / `media` / `info`
- Categorie: ENERGIA, BENZINA, INFLAZIONE, ABITAZIONE, ALIMENTARI, TASSI, MERCATI, LAVORO

### 6. Pre-Decision Coaching (mobile)
- L'utente descrive un acquisto che sta valutando
- L'AI risponde tenendo conto del profilo comportamentale
- Decisione finale registrata (`purchase_decisions`): `bought` / `avoided`
- Costruisce storia delle decisioni per analisi pattern

### 7. Auth + sicurezza
- Supabase Auth (email/password)
- RLS su tutte le tabelle — ogni utente vede solo i propri dati
- API routes server-side (la chiave Anthropic non è mai esposta al client)
- Rate limiting su `/api/coach` e `/api/analyze`
- Middleware Next.js per protezione route dashboard

---

## Design system

**North Star creativa:** "Il Consulente delle 23:00" — strumento professionale, scuro, preciso.

| Token | Valore |
|-------|--------|
| Background base | `#0A0A0F` (near-black con tint blue subpercettivo) |
| Accento unico | `#00C896` (verde smeraldo-teal) |
| Testo primario | `#F8FAFC` |
| Testo secondario | `#94A3B8` |
| Testo muted | `#4B5563` |
| Surface 1 | `rgba(255,255,255,0.025)` |
| Surface 2 | `rgba(255,255,255,0.045)` |
| Danger | `#EF4444` |
| Warning | `#F59E0B` |

**Font:** Geist (unico, no serif, no decorativi). Titoli con letter-spacing negativo (−0.03/−0.04em).

**Regole chiave:**
- Verde `#00C896` su meno del 15% di ogni schermata — rarità = significato
- Nessuna superficie con colore solido opaco (tutto in rgba/trasparenza)
- Blur solo su sidebar e modal, mai su card ordinarie
- Motion: solo `transform` e `opacity`, mai width/height; spring physics Apple

**Anti-references espliciti:** banche tradizionali (navy/gold), app wellness (pastelli), SaaS generico (hero + 3 colonne), crypto (neon su nero).

---

## Database schema (Supabase)

| Tabella | Scopo |
|---------|-------|
| `user_profiles` | Profilo comportamentale — triggers, strengths, risk_categories, budget_targets, goals_history |
| `analyses` | Storico analisi bancarie (raw_input + result JSONB) |
| `coach_conversations` | Conversazioni coach (messages JSONB, title, summary) |
| `purchase_decisions` | Decisioni pre-acquisto con esito |
| `academy_courses` | Catalogo corsi (contenuto statico) |
| `academy_lessons` | Lezioni con contenuto Markdown e quiz JSONB |
| `academy_progress` | Progresso utente per lezione (XP, quiz_score, quiz_answers) |
| `daily_tips` | Pillole giornaliere AI (cache per user + date) |

---

## Sviluppi futuri possibili

### Priorità alta — completamento funzionale

| Feature | Descrizione |
|---------|-------------|
| **Copy upgrade** | Applicare tone of voice premium a tutte le pagine ancora con copy generico (dashboard, analisi, profilo, impostazioni, Sidebar) |
| **Design upgrade** | Completare CSS design system in globals.css, Sidebar, login, tutte le pagine |
| **Sidebar aggiornamento** | Label "Radar" → "Accademia", icon Radio → GraduationCap |
| **Nuovi corsi Accademia** | Es: Pensione integrativa, FIRE movement, Debiti e credito, Tasse e dichiarazione, Assicurazioni |

### Crescita prodotto — analisi

| Feature | Descrizione |
|---------|-------------|
| **Trend multi-mese** | Grafico andamento spese su 3-12 mesi comparando analisi multiple nel tempo |
| **Budget tracking** | Definizione budget mensile per categoria + alerting in-app quando si avvicina al limite |
| **Anomaly detection** | Flag automatico per spese anomale rispetto alla media storica dell'utente |
| **Pattern settimanale** | Analisi comportamento per giorno della settimana / ora del giorno |
| **Import bancario diretto** | Integrazione con API PSD2 (open banking) per fetch automatico movimenti senza copia/incolla |
| **Export report** | PDF mensile con riepilogo spese + insights AI |

### Crescita prodotto — coaching

| Feature | Descrizione |
|---------|-------------|
| **Obiettivi strutturati** | Goal tracking con milestone, progresso visuale, reminder |
| **Notifiche push** | Alert settimanali con insight chiave ("questa settimana hai speso il 40% in più in ristoranti") |
| **Coaching proattivo** | Il coach avvia conversazioni su base di anomalie o milestone raggiunte |
| **Modalità crisi finanziaria** | Flusso dedicato per utenti in situazione di difficoltà economica |
| **Comparazione con peer** | Benchmark anonimi per fascia d'età / reddito stimato |

### Monetizzazione

| Modello | Descrizione |
|---------|-------------|
| **Freemium** | Piano base (3 analisi/mese, accesso limitato Accademia) + Piano Pro (illimitato + coach avanzato) |
| **Subscription mensile** | €5-8/mese — costo marginale basso, valore percepito alto |
| **One-time corsi premium** | Corsi avanzati a pagamento (es. "Investimenti avanzati", "Pianificazione fiscale") |
| **B2B** | White label per HR/welfare aziendale — literacy finanziaria come benefit ai dipendenti |

### Tech debt e infrastruttura

| Item | Descrizione |
|---------|-------------|
| **Prompt caching** | Aggiungere `cache_control` su system prompt e profilo utente nelle chiamate Claude per ridurre costi ~80% |
| **Edge functions** | Spostare le route più semplici su Edge per ridurre cold start |
| **Web Analytics** | Attivare Vercel Analytics + Speed Insights |
| **Test E2E** | Playwright per flussi critici: login, upload analisi, quiz lezione |
| **Mobile parity** | Accademia non ancora presente nell'app mobile — da costruire |
| **Offline mode** | Cache locale analisi e corsi su mobile per uso offline |
| **Internazionalizzazione** | Struttura i18n per eventuale espansione oltre il mercato italiano |

---

## Stato attuale (maggio 2026)

- Web app funzionante in produzione su Vercel
- Tutte le feature core implementate (analisi, coach, accademia, radar, profilo)
- Mobile app (Expo) con parità funzionale parziale — manca Accademia
- Copy e design ancora non uniformi al design system premium su tutte le pagine
- Nessun sistema di monetizzazione attivo
- Nessun test automatizzato
