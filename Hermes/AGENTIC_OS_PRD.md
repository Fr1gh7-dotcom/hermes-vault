# Agentic OS — PRD v1.0

**Owner:** Nicolò
**Status:** Draft per kickoff
**Data:** Maggio 2026
**Stack target:** Next.js + Supabase + Claude API + Vercel (gemello di Zoroaster)

---

## 1. North Star (una frase)

La mia piattaforma personale digitale che ingerisce automaticamente tutto quello che produco e consumo, lo rende interrogabile da agenti AI specializzati, e restituisce output strutturati che alimentano il mio secondo cervello in Obsidian — accessibile sempre, ovunque, dal telefono.

**Obsidian = cervello (lettura, scrittura, connessioni).**
**Agentic OS = braccio operativo (esegue, ingerisce, monitora).**

---

## 2. Problema che risolvo

Oggi i miei dati e processi sono frammentati tra:
- Drive (PDF, libri, documenti)
- Gmail (informazioni importanti perse nel rumore)
- YouTube (video formativi che non sintetizzo mai)
- Note sparse (Apple Notes, Obsidian, draft vari)
- Strumenti AI scollegati (Claude, ChatGPT, Antigravity)
- Zoroaster (dominio finanza, isolato)

Non ho un punto di entrata unico. Ogni informazione richiede un'azione manuale per essere utilizzabile. L'AI non sa nulla di me tra una sessione e l'altra.

**Agentic OS unifica ingestion + processing + output in un sistema modulare con memoria persistente.**

---

## 3. Principi non negoziabili

1. **Fratello di Zoroaster**, non parente lontano. Stesso stack, stesso DB, stesso design system, stesso auth. Le due app condividono `user_profiles` e potranno chiamarsi a vicenda.
2. **Mobile-first nell'interfaccia.** Tu lavori in giro dal telefono. La web app è responsive prima che desktop.
3. **File-system + DB ibrido.** I file binari (PDF, audio, immagini) stanno su Supabase Storage. I metadata e i contenuti testuali estratti stanno su Postgres. Le note finali Markdown sono **anche** scaricate periodicamente in una cartella locale che Obsidian apre.
4. **Modulare per skill.** Ogni skill è una tabella + un endpoint + una pagina. Aggiungere/rimuovere = una migration + un file.
5. **Reversibile.** Esporto tutto in .md/.json in qualsiasi momento. Niente lock-in.
6. **Costruisco per me, ma architettura pulita per scalare.** Multi-tenant da subito (RLS Supabase). Quando un parente la userà, basta crearli un account.

---

## 4. Architettura

### 4.1 Stack tecnico (allineato a Zoroaster)

| Layer | Tecnologia | Note |
|-------|-----------|------|
| Web app | Next.js 15 (App Router) + TypeScript + Tailwind | Stesso di Zoroaster |
| Auth + DB | Supabase (Postgres + Auth + Storage + RLS) | **Stesso progetto Supabase di Zoroaster** o nuovo? → decisione in §9 |
| AI | Anthropic Claude (sonnet-4-6 default, opus per skill heavy) | Stesso di Zoroaster |
| Deploy | Vercel | Stesso account |
| Cron / job schedulati | Vercel Cron (gratis fino a un certo limite) | Per ingestion automatica |
| Storage file | Supabase Storage | PDF, audio, immagini |
| Vector search | pgvector (estensione Postgres) | NIENTE Pinecone/Qdrant, resta in casa Supabase |
| Local sync (per Obsidian) | Script Node.js che gira sul Mac + cartella iCloud | Vedi §6 |
| Mobile | Web app responsive + PWA installabile | NIENTE app Expo nativa subito |

**Perché PWA e non React Native:** una sola codebase, deploy in 30 secondi, accesso da iPhone come app installata. La parità con Zoroaster (che ha Expo) si valuta a Fase 3 se serve.

### 4.2 Modello dati (Postgres / Supabase)

Tabelle core (Fase 1):

```
user_profiles               (estende quella di Zoroaster)
  ↳ campi nuovi: agentic_os_settings JSONB, default_skill_prefs JSONB

documents                   archivio centrale di ogni cosa ingerita
  id, user_id, source_type (pdf|email|youtube|web|note|voice|image),
  source_url, source_ref, title, raw_content TEXT, summary TEXT,
  metadata JSONB, status (pending|processing|ready|error),
  embedding vector(1536), created_at, processed_at

skills                       catalogo skill installate per utente
  id, user_id, skill_key, enabled, config JSONB, created_at

runs                         log di ogni esecuzione skill (storico)
  id, user_id, skill_key, status, input JSONB, output JSONB,
  document_ids[], tokens_used, cost_cents, duration_ms,
  error TEXT, created_at

memory                       fatti persistenti che l'AI sa su di te
  id, user_id, key, value TEXT, source_run_id, confidence,
  last_used_at, created_at

ingestion_sources            fonti di ingestion configurate
  id, user_id, source_type, config JSONB (es. folder Drive ID),
  enabled, last_sync_at, sync_interval_minutes

usage_tracking               consumi API divisi per provider/skill
  id, user_id, provider (anthropic|openai|...), model, skill_key,
  tokens_in, tokens_out, cost_cents, created_at
```

Tutte con RLS attiva (`user_id = auth.uid()`). Stessa logica di Zoroaster.

### 4.3 Layer logici dell'app

```
┌──────────────────────────────────────────────────────────┐
│                    INTERFACCIA (PWA)                     │
│  Dashboard │ Documents │ Skills │ Runs │ Memory │ Costs  │
└────────────────────────┬─────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                  CORE / ORCHESTRATION                    │
│  - Skill runner (chiama Claude API con context giusto)   │
│  - Memory injector (aggiunge fatti rilevanti al prompt)  │
│  - Usage tracker (logga ogni chiamata API)               │
└────────────────────────┬─────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                       SKILLS                             │
│  pdf-digest │ yt-transcribe │ email-triage │ daily-brief │
│  vault-search │ usage-monitor │ zoroaster-bridge │ ...   │
└────────────────────────┬─────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                     INGESTION                            │
│  Drive watcher │ Gmail │ YouTube │ Web clipper │ Voice   │
└────────────────────────┬─────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│              STORAGE (Supabase + Vercel Cron)            │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Skill — catalogo Fase 1

**Regola Fase 1: massimo 4 skill, scelte per coprire i 4 verbi fondamentali del sistema.**

| Skill | Verbo | Cosa fa | Effort |
|-------|-------|---------|--------|
| `pdf-digest` | **Ingerire** | Carico PDF → estrazione testo + riassunto strutturato + key quotes + embedding | Bassa |
| `yt-transcribe` | **Catturare** | Incollo link YouTube → trascrizione completa + riassunto + .md scaricabile | Bassa |
| `vault-search` | **Trovare** | Ricerca semantica su tutto quello che ho ingerito | Media |
| `daily-brief` | **Sintetizzare** | Ogni mattina alle 7: digest di email + meteo + agenda + 1 documento recente del vault | Media |

**Cosa NON c'è in Fase 1 (esplicitamente):**
- Ingestion automatica Drive (manuale upload prima)
- Ingestion automatica Gmail (manuale forward prima)
- Memory adattiva avanzata (solo memoria esplicita)
- Pattern detection / anticipazione
- Zoroaster bridge (Fase 2)
- Dashboard consumi avanzata (Fase 2, ora basic)

---

## 6. Sync con Obsidian (la parte critica)

**Problema:** Obsidian Mobile non parla con Supabase. Obsidian vuole file .md in una cartella.

**Soluzione: sync unidirezionale Supabase → cartella locale → iCloud → Obsidian Mobile.**

Mac sempre acceso o quasi → script Node `obsidian-sync` (cron locale, ogni 5 min):
1. Pulla da Supabase tutti i `documents` con `status='ready'` non ancora sincronizzati
2. Scrive `.md` con frontmatter YAML in `~/iCloud/Obsidian/AgenticOS/`
3. iCloud sincronizza con iPhone
4. Obsidian Mobile vede i file

Struttura cartella Obsidian:
```
~/iCloud/Obsidian/AgenticOS/
  daily-briefs/
    2026-05-11.md
  documents/
    pdf-digest/
      libro-X.md
    youtube/
      video-Y.md
  memory/
    about-me.md
    facts.md
  runs/
    2026-05/
      run-abc123.md
```

Obsidian Mobile/Desktop apre questa cartella come vault. Tu pensi e linki lì dentro. L'app non interferisce con quello che scrivi tu manualmente — scrive solo nelle sue sottocartelle.

**Limite onesto:** se cancelli o modifichi un .md in Obsidian, l'app NON lo recepisce (per ora). Sync è one-way per evitare conflitti. Sufficiente per Fase 1.

---

## 7. Design system

**100% allineato a Zoroaster.** Nessuna deviazione.

- Background: `#0A0A0F`
- Accento: `#00C896` (uso <15% per schermata)
- Font: Geist
- Tutte le regole tipografiche, motion, blur già definite nel design system Zoroaster → riusa identico

**Aggiunte specifiche Agentic OS:**
- Icona/logo dedicato (da definire — proposta: glyph minimale tipo `[A+]`)
- Sidebar con sezioni: Home, Documents, Skills, Runs, Memory, Costs, Settings
- Mobile bottom tab bar (4 voci): Home / Documents / Skills / More

---

## 8. Roadmap a fasi con cancelli di uscita

### Fase 0 — Foundation (settimana 1)
- Repo `agentic-os-web` creato, deploy Vercel funzionante
- Schema Supabase migrato (8 tabelle, RLS attiva)
- Auth funzionante (riuso Supabase Zoroaster se decidi così, vedi §9)
- Layout app + sidebar + bottom tab mobile
- Pagina vuota Dashboard con metriche placeholder
- **Cancello:** apro l'app dal telefono, faccio login, vedo lo scheletro. Sì/no.

### Fase 1 — Le 4 skill core (settimane 2-5)
- `pdf-digest` (settimana 2)
- `yt-transcribe` (settimana 3)
- `vault-search` con pgvector (settimana 4)
- `daily-brief` con Vercel Cron (settimana 5)
- Sync Obsidian script Node (parallelo, settimana 4-5)
- **Cancello:** la uso davvero ogni giorno per 1 settimana? Se no, stop.

### Fase 2 — Automazione + integrazione Zoroaster (settimane 6-8)
- Watcher Drive automatico (Google Drive API + Vercel Cron)
- Gmail digest automatico
- `zoroaster-bridge` skill — leggi profilo + ultima analisi Zoroaster dentro Agentic OS
- Dashboard consumi vera (grafici Recharts con `usage_tracking`)
- **Cancello:** Zoroaster e Agentic OS si parlano? Vale lo sforzo?

### Fase 3 — Memoria + extra skill (settimane 9-12)
- Memory layer (estrazione fatti dalle run, iniezione nei prompt)
- 2-3 skill extra basate su cosa è emerso dall'uso reale (NON deciderle ora)
- Eventuale parità mobile Expo (se serve davvero)

### Fase 4 — Replicabilità per altri (open-ended)
- Onboarding flow per parenti
- Selezione skill installabili
- Documentazione
- (Eventuale) monetizzazione

---

## 9. Decisioni aperte (da chiudere prima del codice)

1. **Stesso progetto Supabase di Zoroaster o nuovo?**
   - **Stesso:** condivisione `user_profiles` immediata, single sign-on, costi più bassi. Rischio: schema esplode, se rompo una migration rompo entrambe le app.
   - **Nuovo:** isolamento totale. Costo: dovrai duplicare `user_profiles` o federare. → mia raccomandazione: **stesso progetto, schema separato (`agentic_os` schema in Postgres)**.

2. **Nome definitivo dell'app?** "Agentic OS" è generico. Pensa a un brand pulito allineato a Zoroaster (es. nome mitologico/storico).

3. **Domain?** Sottocartella di zoroaster.it o dominio proprio?

4. **Repo separato o monorepo con Zoroaster?** Raccomandazione: separato per Fase 1, eventuale monorepo dopo.

---

## 10. Cosa serve da te per partire (checklist)

- [ ] Conferma punto §9.1 (Supabase condiviso sì/no)
- [ ] Nome dell'app
- [ ] Dominio
- [ ] Accesso al progetto Supabase di Zoroaster (anon key + service role)
- [ ] Google Cloud project con Drive API + Gmail API abilitate (per Fase 2, non Fase 1)
- [ ] Account Anthropic con credit residui sufficienti
- [ ] Cartella iCloud dedicata a Obsidian creata sul Mac
- [ ] App Obsidian installata su iPhone, vault puntato a quella cartella

---

## 11. Stima costi runtime (mensile)

| Voce | Costo |
|------|-------|
| Vercel Pro (se serve, gratis basta per ora) | €0–20 |
| Supabase (Pro tier se cresci) | €0–25 |
| Anthropic API (uso personale stimato) | €30–80 |
| OpenAI Whisper (se attivi voice) | €0–10 |
| Domain | €1–2 |
| **Totale Fase 1-2** | **~€30–100/mese** |

---

## 12. Rischi e mitigazioni

| Rischio | Mitigazione |
|---------|-------------|
| Perdo focus su Zoroaster | Cancelli di uscita rigidi. Se Fase 1 di Agentic OS non viene usata, stop. |
| Costi API esplodono | `usage_tracking` da giorno 1 + soft cap mensile per skill |
| Sync Obsidian fragile | One-way per evitare conflitti. Backup automatico Supabase. |
| Schema Postgres esplode con Zoroaster | Schema separato Postgres (§9.1) |
| Non lo uso davvero perché complicato | Bottom tab mobile semplice. 4 skill, non 20. |

---

**Fine PRD v1.0.**
