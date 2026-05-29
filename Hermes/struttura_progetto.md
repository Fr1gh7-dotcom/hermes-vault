# Struttura Progetto — MonitorAnalisi

> Documento di riferimento per replicare il progetto su un nuovo dominio (es. fatture).

---

## Stack tecnologico

| Layer | Tecnologia | Versione |
|---|---|---|
| Framework | Next.js (App Router) | 16.2.4 |
| UI | React | 19 |
| Language | TypeScript | 5 |
| Database + Auth + Storage | Supabase | latest |
| Styling | Tailwind CSS | v4 |
| Component library | shadcn/ui (Radix) | latest |
| AI / PDF parsing | Anthropic Claude SDK | ^0.91.1 |
| Email | Resend + React Email | latest |
| Forms | React Hook Form + Zod | latest |
| Charts | Recharts | latest |
| Test | Vitest | latest |
| Package manager | pnpm | — |
| Deploy | Vercel | — |

---

## Struttura cartelle

```
/
├── app/
│   ├── layout.tsx                    # Root layout (font, provider globali)
│   ├── page.tsx                      # Landing page pubblica
│   ├── globals.css
│   │
│   ├── (app)/                        # Gruppo: app autenticata (operatori interni)
│   │   ├── layout.tsx                # Sidebar + auth guard
│   │   ├── page.tsx                  # Redirect → dashboard
│   │   ├── dashboard/page.tsx        # KPI, scadenze, attività recente
│   │   ├── clienti/
│   │   │   ├── page.tsx              # Lista clienti
│   │   │   └── [id]/page.tsx         # Dettaglio cliente + storico
│   │   ├── analisi/                  # ← sostituire con "fatture/"
│   │   │   ├── page.tsx              # Lista analisi
│   │   │   ├── nuova/page.tsx        # Form inserimento manuale
│   │   │   ├── import/page.tsx       # Bulk import da PDF
│   │   │   ├── da-controllare/page.tsx # Coda validazione
│   │   │   └── [id]/
│   │   │       ├── page.tsx          # Dettaglio analisi
│   │   │       └── confronto/page.tsx # Confronto tra analisi
│   │   ├── esperto/page.tsx          # Chat AI contestuale
│   │   └── settings/page.tsx         # Impostazioni account
│   │
│   ├── (auth)/                       # Gruppo: autenticazione
│   │   ├── login/page.tsx
│   │   └── set-password/page.tsx     # Primo accesso / reset
│   │
│   ├── (client)/                     # Gruppo: portale clienti (read-only)
│   │   ├── layout.tsx
│   │   └── portal/
│   │       ├── page.tsx              # Dashboard cliente
│   │       └── analisi/[id]/page.tsx  # Dettaglio per il cliente
│   │
│   ├── actions/
│   │   └── analisi.ts                # Server Actions (CRUD principale)
│   │
│   ├── api/
│   │   ├── estrai-pdf/route.ts       # Estrae dati da PDF via Claude
│   │   ├── ingest-pdf/route.ts       # Upload PDF → Supabase Storage
│   │   ├── import-analisi/route.ts   # Bulk import batch
│   │   ├── chat-esperto/route.ts     # Streaming chat AI
│   │   └── export/cliente/[id]/route.ts  # Export Excel/CSV per cliente
│   │
│   └── auth/
│       ├── callback/route.ts         # OAuth callback Supabase
│       └── signout/route.ts
│
├── components/
│   ├── ui/                           # Primitivi shadcn/ui (non toccare)
│   │   ├── button.tsx, card.tsx, dialog.tsx, form.tsx ...
│   │
│   ├── shared/                       # Componenti trasversali
│   │   ├── app-sidebar.tsx           # Navigazione laterale
│   │   ├── stato-scadenza-badge.tsx  # Badge stato (es. In scadenza, Scaduta)
│   │   └── trend-chart.tsx           # Grafico andamento
│   │
│   ├── clienti/                      # Componenti dominio clienti
│   │   ├── cliente-autocomplete.tsx
│   │   ├── crea-cliente-button.tsx
│   │   └── modifica-cliente-button.tsx
│   │
│   └── analisi/                      # ← rinominare in "fatture/"
│       ├── form-analisi.tsx          # Form inserimento/modifica
│       ├── lista-analisi-client.tsx  # Tabella con filtri (client component)
│       ├── filtri-analisi.tsx        # Pannello filtri
│       ├── parametri-table.tsx       # Tabella parametri (specifico dominio)
│       ├── parametri-input.tsx       # Input parametri dinamici
│       ├── bulk-import-client.tsx    # UI upload multiplo PDF
│       ├── approvazione-rapida.tsx   # Quick action validazione
│       ├── archivia-button.tsx
│       ├── ritiro-button.tsx
│       ├── azioni-analisi.tsx        # Menu azioni su riga
│       ├── confronto-report.tsx      # Confronto 2 record
│       ├── confronto-multi-report.tsx
│       ├── confronto-toggle.tsx
│       └── selettore-confronto.tsx
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts                 # createBrowserClient
│   │   ├── server.ts                 # createServerClient (cookie)
│   │   └── service.ts                # createServiceClient (service role key)
│   │
│   ├── services/                     # Business logic pura (no UI)
│   │   ├── analisi-service.ts        # CRUD analisi + query complesse
│   │   ├── clienti-service.ts        # CRUD clienti
│   │   └── compliance-service.ts     # Logica validazione/conformità
│   │
│   ├── config.ts                     # Costanti globali (env vars typed)
│   ├── utils.ts                      # cn(), formatDate(), ecc.
│   ├── validators.ts                 # Schemi Zod condivisi
│   ├── scadenze.ts                   # Logica calcolo scadenze
│   ├── normalize-cer.ts              # Normalizzazione codici (specifico dominio)
│   ├── format-cer.ts
│   ├── confronto.ts                  # Logica confronto record
│   └── confronto-multi.ts
│
├── types/
│   └── database.ts                   # Tipi generati da Supabase (o manuali)
│
├── supabase/
│   └── migrations/                   # SQL migrations in ordine numerico
│       ├── 00001_initial_schema.sql  # Tabelle base
│       ├── 00002_*.sql
│       └── ...
│
├── scripts/                          # Utility standalone (Node.js / mjs)
│   ├── bulk-import.js                # Import massivo da cartella PDF
│   ├── watcher.js                    # Cron locale per notifiche
│   ├── check_reports.mjs
│   └── *.sql                         # SQL di setup/seed
│
├── watcher/                          # Servizio watcher separato (deploy autonomo)
│   ├── watcher.mjs                   # Polling Supabase → invio email
│   └── package.json
│
├── docs/                             # Documentazione interna
│   ├── struttura_progetto.md         # ← questo file
│   └── sessions/                     # Note di sessione
│
├── .env.local                        # Variabili d'ambiente (non committare)
├── .env.example                      # Template variabili
├── next.config.ts
├── tailwind.config (embedded in CSS)
├── components.json                   # Config shadcn/ui
├── tsconfig.json
├── vitest.config.ts
└── vercel.json                       # Config deploy
```

---

## Schema database (pattern generale)

```sql
-- Tabella utenti interni
users (id, email, nome_completo, attivo, created_at)

-- Tabella clienti (entità principale)
clienti (id, ragione_sociale, partita_iva, indirizzo, referente_*, note, created_*)

-- Tabella dominio principale (es. analisi → fatture)
[entità] (
  id uuid PK,
  cliente_id FK → clienti,
  -- campi dominio specifici --
  pdf_path text,               -- path su Supabase Storage
  stato text,                  -- workflow: bozza | in_revisione | approvata | archiviata
  created_by FK → users,
  updated_by FK → users,
  created_at, updated_at
)

-- Tabella dettaglio (righe fattura, parametri analisi, ecc.)
[entità]_righe (
  id uuid PK,
  [entità]_id FK ON DELETE CASCADE,
  -- campi riga --
)

-- RLS: ogni tabella ha policy per ruolo (admin / operatore / client)
```

---

## Pattern architetturali chiave

### Auth & Multi-tenant
- Supabase Auth con RLS
- Ruoli: `admin`, `operatore`, `client` (via `app_metadata`)
- Ogni query filtrata automaticamente da RLS policy

### PDF → Dati strutturati
1. Upload PDF → Supabase Storage (`[bucket]-pdf`)
2. `POST /api/estrai-pdf` → Claude legge PDF, restituisce JSON strutturato
3. Form pre-compilato per revisione umana
4. Server Action salva in DB

### Workflow documento
```
bozza → in_revisione → approvata → archiviata
                     ↘ rifiutata
```

### Notifiche email
- Watcher separato (Node.js) fa polling Supabase
- Invia email via Resend + React Email template
- Oppure: Supabase Edge Functions + pg_cron

### Portale clienti (read-only)
- Route group `(client)` separato
- Auth separata o token monouso via email
- Vede solo i propri documenti

---

## Variabili d'ambiente necessarie

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
ANTHROPIC_API_KEY=
RESEND_API_KEY=
NEXT_PUBLIC_APP_URL=
```

---

## Cosa cambia per il progetto Fatture

| MonitorAnalisi | MonitorFatture (nuovo) |
|---|---|
| `analisi/` | `fatture/` |
| `parametri_analisi` table | `righe_fattura` table |
| Codice CER | Numero fattura / codice articolo |
| Data scadenza analisi | Data scadenza pagamento |
| Laboratorio | Fornitore |
| Score conformità | Stato pagamento / scaduto |
| Export compliance | Export contabilità |
| `compliance-service.ts` | `pagamenti-service.ts` |

Stack identico. Migrazioni, RLS pattern, PDF extraction, portale client → tutti riusabili.
