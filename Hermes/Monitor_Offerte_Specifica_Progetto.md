# Monitor Offerte — Specifica di progetto v2.0

> **Documento sorgente per lo sviluppo dell'applicazione Monitor Offerte, modulo della piattaforma "Monitor" per il settore ambientale (intermediazione rifiuti).**
> Versione 2.0 arricchita con analisi competitiva, requisiti normativi e mitigazioni dei rischi.

**Versione**: 2.0 — Maggio 2026
**Stato**: Specifica iniziale arricchita con ricerca competitiva e normativa

---

## 1. Contesto e obiettivo

### 1.1 Cosa è Monitor

"Monitor" è una piattaforma SaaS multi-tenant per aziende del settore ambientale (intermediari di rifiuti, gestori impianti). Funziona **come layer di intelligenza sopra ECOS** (il gestionale operativo che usano molte di queste aziende). Monitor non sostituisce ECOS: gli affianca strumenti di monitoraggio, controllo, decisione e produzione di KPI/report che ECOS non fornisce in forma sintetica e direzionale.

La piattaforma è composta da moduli verticali. Il primo modulo già in produzione è **Monitor Analisi** (gestione certificati di analisi chimiche con estrazione AI da PDF, scadenzario, vista 360° cliente, trend parametri). Stack: Next.js + Supabase + Claude AI, multi-tenant già predisposto.

Questo documento descrive il secondo modulo: **Monitor Offerte**.

### 1.2 Il business dell'intermediario rifiuti

L'azienda fa **intermediazione senza detenzione**: non tratta rifiuti, li piazza. È iscritta all'Albo Nazionale Gestori Ambientali nella **Categoria 8** (intermediazione e commercio di rifiuti senza detenzione). Il valore aggiunto è la rete di partner (impianti di smaltimento e trasportatori) e la capacità di costruire offerte competitive ai clienti produttori.

**Responsabilità legale dell'intermediario** (D.Lgs 152/2006 art. 183, Cassazione 15771/2018): l'intermediario ha un **onere di vigilanza e controllo** sul possesso, da parte di trasportatori e impianti, delle dovute autorizzazioni e qualifiche. È co-responsabile penalmente per la corretta gestione del rifiuto. Questo trasforma Monitor Offerte da strumento commerciale a **strumento di tutela legale**.

Il ciclo tipico di un'offerta:

1. Una richiesta arriva al commerciale (via email, telefono, WhatsApp, di solito frammentaria)
2. Il commerciale raccoglie informazioni base: EER del rifiuto, quantità, frequenza ritiro, modalità ritiro, attuali smaltitori/trasportatori del cliente, mezzo di trasporto necessario, prezzi attuali del cliente
3. Se cliente nuovo: si chiede compilazione scheda anagrafica
4. La pratica passa al **back office commerciale**, che coordina commerciale + cliente + trasportatori + impianti per:
   - Raccogliere quotazioni di smaltimento dagli impianti
   - Raccogliere quotazioni di trasporto dai vettori
   - Verificare le autorizzazioni di trasportatori e impianti per quei CER
   - Formulare un'offerta competitiva al cliente
5. L'offerta viene inviata via email (informale) o come PDF su carta intestata (formale)
6. Negoziazione, eventuale revisione, chiusura (accettata / rifiutata)
7. Se accettata: **attivazione del workflow di omologa rifiuto** (passaggio obbligatorio prima del primo conferimento)
8. Organizzazione operativa dei ritiri (questa parte continua in ECOS)

### 1.3 Strumento attuale e suoi limiti

L'azienda usa oggi un file Excel mensile ("Monitor Offerte"). Colonne: ID, Data Arrivo, Commerciale, Cliente, Urgenza, Stato Offerta, Note, Note Back office, Data Consegna, Incaricato Back Office.

**Problemi critici dello strumento attuale:**

- Una "riga" Excel deve contenere offerte multi-EER per lo stesso cliente: tutto incollato nella colonna Note come testo libero
- Gli aggiornamenti datati ("13/04: ...", "17/04: ...", "30/04: ...") finiscono dentro una cella come log conversazionale: impossibile cercare, filtrare, aggregare
- Prezzi di impianti e trasportatori sparsi nelle note in forma libera
- Confronti competitivi solo verbali ("85€/ton da altri", nomi competitor nelle note)
- Nessuno storico aggregato per CER, cliente, impianto, trasportatore
- Nessuna intelligence in fase di costruzione offerta: il backoffice deve ricordare a memoria prezzi passati
- Nessuna vista geografica della rete di partner
- Nessuna verifica automatica delle autorizzazioni dei partner (rischio legale)
- Nessun report direzionale automatico

Volume attuale: **60-77 pratiche/mese**, ~5-6 commerciali attivi, ~3-4 persone in back office.

### 1.4 Obiettivo del prodotto

Costruire un'applicazione web che:

1. **Sostituisca completamente** il file Excel mensile
2. **Strutturi** ogni pratica in pratica → linee EER → quotazioni separate (smaltimento, trasporto, prezzo cliente)
3. **Catturi le anagrafiche** di impianti e trasportatori con i dati operativi che servono (EER trattati, zone, mezzi, performance, **autorizzazioni con scadenze**)
4. **Fornisca intelligence in tempo reale** durante la costruzione di una nuova offerta (storico prezzi, suggerimenti, alternative)
5. **Mostri la rete su mappa geografica** per coprire ritiri in zone non abituali
6. **Misuri le performance dei partner**: chi è competitivo, chi risponde rapidamente, chi accetta più CER
7. **Tuteli legalmente l'azienda** verificando autorizzazioni dei partner prima di selezionarli
8. **Gestisca l'omologa rifiuto** post-accettazione offerta
9. **Produca report direzionali**: andamento mensile pratiche, tassi di chiusura, marginalità, trend prezzi CER
10. **Esporti dati** in Excel e PDF (offerta formale al cliente su carta intestata)
11. **Dialoghi con Monitor Analisi**: quando si costruisce un'offerta, sapere se il cliente ha analisi valide per il CER

---

## 2. Analisi competitiva e posizionamento

### 2.1 Mappa dei concorrenti

**Gestionali tradizionali** (WinWaste/Zucchetti con oltre 2800 clienti, PrometeoRifiuti con oltre 2000 clienti, GRIF/SIA, ECOS, AGON, Passepartout, EcoSolve, RifiutiGuru, Gruppo Santini, TSL Rifiuti):
- Focus su FIR, registri carico/scarico, MUD, vidimazione VIVIFIR, integrazione RENTRI
- Anagrafiche e scadenzario autorizzazioni
- Statistiche standard (volumi, giacenze)
- Interfacce in larga parte datate, desktop-first o web "vecchia scuola"
- **Nessuno copre la fase commerciale di costruzione offerta**
- Sono i sistemi che gestiscono l'operativo POST-accettazione

**Piattaforme di intermediazione/marketplace** (CER Manager, WasteTrade):
- Espongono le richieste su una piattaforma pubblica
- Matching domanda/offerta tra produttori e impianti via AI
- Database autorizzazioni aggiornato (CER Manager aggiorna settimanalmente i dati dell'Albo)
- Sono potenziali concorrenti per il **modello di business** dell'intermediario (in teoria potrebbero disintermediarlo)
- **Non sono strumenti interni** per gestire le offerte di una propria azienda

**Verticali AI specifici** (es. Estrella di Eco-Management per omologazione AI):
- Risolvono singoli problemi puntuali con AI
- Non sono piattaforme complete

### 2.2 Posizionamento di Monitor Offerte

**Monitor Offerte sta in un buco di mercato** che oggi le aziende riempiono con Excel e memoria delle persone. Non sostituisce i gestionali tradizionali (continueranno a gestire FIR e MUD), ma copre la fase commerciale che a loro manca. Non è un marketplace pubblico (l'intermediario non vuole esporsi). È uno strumento interno che potenzia la rete privata dell'azienda.

**Vantaggi competitivi previsti**:
- Intelligence sui prezzi storici interni (cosa che le piattaforme pubbliche non hanno)
- Workflow strutturato pratica → linee → quotazioni (cosa che gli Excel non danno)
- Verifica compliance automatica (cosa che la gestione manuale rischia di mancare)
- UX moderna, mobile-first per i commerciali
- AI nativa (estrazione da email, suggerimenti)

**Differenziazione rispetto ai gestionali tradizionali**: noi facciamo il *prima* (offerta), loro fanno il *dopo* (operativo). Convivono, non si sostituiscono.

### 2.3 Quadro normativo di riferimento

- **D.Lgs 152/2006 art. 183** definisce l'intermediario senza detenzione e le sue responsabilità
- **Cassazione 15771/2018**: confermata la co-responsabilità penale per l'intermediario
- **Albo Nazionale Gestori Ambientali — Categoria 8**: obbligo iscrizione per intermediari senza detenzione, rinnovo ogni 5 anni
- **RENTRI** (D.M. 59/2023): dal 13 febbraio 2026 iscrizione obbligatoria per intermediari di rifiuti pericolosi. FIR digitale obbligatorio dalla stessa data, con proroga del cartaceo fino al 15 settembre 2026 (DL 200/2025). Registri carico/scarico digitali obbligatori dall'iscrizione (no proroga).
- **Dal 30 giugno 2026**: obbligo di geolocalizzazione sui mezzi categorie 1 e 5 (trasporto rifiuti pericolosi conto terzi)
- **Omologa rifiuto**: procedura di caratterizzazione obbligatoria prima del primo conferimento, su modulistica dell'impianto destinatario

---

## 3. Modello dati

### 3.1 Gerarchia principale

```
Pratica (richiesta cliente)
  └─ Linee EER (una per codice rifiuto)
       └─ Quotazioni Impianto (preventivi smaltimento)
       └─ Quotazioni Trasportatore (preventivi trasporto)
       └─ Prezzo Cliente (composizione finale)
       └─ Omologa (post-accettazione, per linea EER)
```

**Pratica**: una richiesta da un cliente, contenitore logico. Può avere più linee EER se il cliente chiede preventivi per più rifiuti contemporaneamente.

**Linea EER**: una specifica richiesta per un codice EER. Ha quantità, frequenza, modalità ritiro, mezzo richiesto. Una linea può avere multiple quotazioni impianto (più impianti consultati) e multiple quotazioni trasportatore.

**Quotazione Impianto**: preventivo di smaltimento ricevuto da un impianto per una specifica linea EER. Comprende prezzo €/ton, operazione R/D, validità, eventuali condizioni.

**Quotazione Trasportatore**: preventivo di trasporto ricevuto da un vettore. Comprende prezzo (€/viaggio o €/ton), mezzo, zona, validità.

**Prezzo Cliente**: prezzo finale composto = miglior smaltimento + miglior trasporto + margine + oneri documentali. Il sistema suggerisce la combinazione ottimale ma l'operatore può scegliere diversamente.

**Omologa Rifiuto**: documento di caratterizzazione del rifiuto attivato dopo l'accettazione dell'offerta. Una per linea EER. Stato workflow: da compilare → in compilazione → inviata all'impianto → accettata/rifiutata. Senza omologa accettata non parte il conferimento.

### 3.2 Entità — schema concettuale

#### Anagrafiche di base

- **Cliente** (produttore): ragione sociale, P.IVA, indirizzo, sedi/stabilimenti, contatti, scheda anagrafica, iscrizione RENTRI (se applicabile)
- **Impianto di smaltimento**: ragione sociale, indirizzo, lat/lng (mappa), tipologia (D/R), referenti, modalità di lavoro, note operative
- **Autorizzazione impianto**: AIA/AUA con CER ammessi, capacità, operazioni R/D, scadenza
- **Trasportatore**: ragione sociale, indirizzo, lat/lng, mezzi disponibili (bilico, container, motrice, cassone stagno, IBC, compattatore...), zone abituali, referenti
- **Iscrizione Albo Gestori**: categoria (4 non pericolosi, 5 pericolosi, ecc.), CER autorizzati, scadenza
- **Codici EER**: catalogo europeo, descrizioni, pericolosità

#### Pratica e linee

- **Pratica**:
  - ID, codice progressivo
  - Cliente (FK)
  - Commerciale di riferimento (FK utente)
  - Back office assegnato (FK utente)
  - Data arrivo richiesta
  - Canale richiesta (email/telefono/WhatsApp/altro)
  - Urgenza (Urgente / Alta / Media / Bassa)
  - Stato pratica (Da fare, In lavorazione, In attesa info, Fatta, In attesa accettazione, Chiusa+, Chiusa-, Annullata)
  - Tipo cliente (Nuovo/Esistente)
  - Note generali

- **Linea EER**:
  - ID
  - Pratica (FK)
  - Codice EER (FK)
  - Descrizione rifiuto (libera)
  - Quantità prevista (kg o ton)
  - Frequenza (spot, mensile, settimanale, annuale)
  - Modalità ritiro (descrittiva)
  - Mezzo richiesto (tipo)
  - Stato fisico
  - Pericoloso (bool)
  - Sede ritiro (FK sede cliente)
  - Note linea
  - Stato linea (Da quotare, Quotato, Offerta inviata, Accettata, In omologa, Pronto per conferimento, Conferimenti attivi, Rifiutata, Annullata)
  - Analisi associata (FK Monitor Analisi, opzionale)

- **Quotazione Impianto**:
  - ID
  - Linea EER (FK)
  - Impianto (FK)
  - Prezzo €/ton smaltimento
  - Operazione R/D
  - Data ricezione preventivo
  - Validità (giorni)
  - Note e condizioni
  - Stato (Richiesta inviata, Ricevuta, Selezionata, Scartata)
  - Tempo risposta (giorni — calcolato)
  - **Compliance check**: autorizzazione AIA attiva per quel CER (calcolato)

- **Quotazione Trasportatore**:
  - ID
  - Linea EER (FK)
  - Trasportatore (FK)
  - Mezzo
  - Prezzo (€/viaggio o €/ton)
  - Modalità prezzo (per_viaggio | per_tonnellata)
  - Data ricezione preventivo
  - Validità (giorni)
  - Note
  - Stato (Richiesta inviata, Ricevuta, Selezionata, Scartata)
  - **Compliance check**: iscrizione Albo attiva per quel CER (calcolato)

- **Prezzo Cliente** (offerta finale, una per linea):
  - ID
  - Linea EER (FK)
  - Quotazione Impianto selezionata (FK)
  - Quotazione Trasportatore selezionata (FK)
  - Componente smaltimento €
  - Componente trasporto €
  - Oneri documentali €
  - Margine €
  - Prezzo finale €/ton
  - Validità offerta cliente
  - Versione (per revisioni)
  - Stato (Bozza, Inviata, Accettata, Rifiutata, Da rivedere)

- **Omologa Rifiuto** (post-accettazione):
  - ID
  - Linea EER (FK)
  - Impianto destinatario (FK)
  - Stato (Da compilare, In compilazione, Inviata, Accettata, Rifiutata, Revisione richiesta)
  - Data compilazione
  - Data invio
  - Data risposta
  - Note rifiuto/revisione
  - Allegato PDF compilato
  - Validità (tipicamente 12 mesi, da rinnovare)

#### Concorrenza e contesto

- **Concorrenti noti**: anagrafica competitor (altre intermediarie / impianti che il cliente già usa)
- **Prezzi concorrenza dichiarati**: per linea EER, il prezzo che il cliente dichiara di avere già da competitor X (input commerciale)

#### Timeline e collaborazione

- **Eventi pratica** (timeline): ogni evento datato sostituisce le note libere
  - Tipo evento (richiesta_prezzo_impianto, ricezione_prezzo_impianto, mail_cliente, decisione_direzione, offerta_inviata, contro_offerta, omologa_inviata, ...)
  - Pratica o Linea EER (FK polimorfico)
  - Autore (FK utente)
  - Data/ora
  - Testo
  - Allegati (rif. Supabase Storage)

- **Commenti**: simili agli eventi ma per discussione interna tra team

#### Performance e analytics

Tabelle calcolate / view materializzate per:

- **Performance Impianto**:
  - Tempo medio risposta a richiesta prezzo
  - Tasso accettazione (quotazioni selezionate / quotazioni richieste)
  - Prezzo medio per EER negli ultimi N mesi
  - Variazione prezzo trend (in salita/in discesa)
  - Numero EER coperti
  - Tasso "preventivo competitivo" (quante volte è risultato il più economico)
  - Tasso accettazione omologhe

- **Performance Trasportatore**: analoghe
- **Performance Commerciale / Back office**: pratiche gestite, tasso chiusura
- **Performance Cliente**: pratiche totali, valore, tasso accettazione, marginalità media

---

## 4. Funzionalità chiave

### 4.1 Schermate principali

1. **Dashboard pratiche**: vista tabellare con filtri (stato, urgenza, commerciale, back office, periodo). Sostituisce il file Excel.
2. **Nuova pratica**: form di creazione con compilazione progressiva. Auto-suggerimenti dall'inizio. AI parsing email opzionale.
3. **Dettaglio pratica**: vista completa con tutte le linee, quotazioni, timeline, allegati.
4. **Costruzione offerta linea EER**: la schermata operativa più importante. Tutto il valore differenziante del prodotto si vede qui (vedi 4.3).
5. **Workflow omologa**: post-accettazione, gestione del documento di omologa per ogni linea EER.
6. **Anagrafica Impianti**: lista, dettaglio singolo impianto con storico pratiche, performance, autorizzazioni e scadenze.
7. **Anagrafica Trasportatori**: lista, dettaglio, performance, iscrizioni Albo.
8. **Anagrafica Clienti**: lista, dettaglio, storico pratiche.
9. **Mappa rete**: vista cartografica con impianti e trasportatori, filtri per EER/zona/mezzo.
10. **Report direzionali**: dashboard analitica con KPI, trend, grafici.
11. **Esportazioni**: generatore di Excel, PDF offerte
12. **Scadenzario compliance**: tutte le scadenze critiche (autorizzazioni partner, omologhe, iscrizioni Albo) in vista unificata.

### 4.2 Stati e workflow

**Stati pratica**:
- `Da fare` — richiesta ricevuta, non ancora elaborata
- `In lavorazione` — back office sta lavorando
- `In attesa info` — mancano dati dal cliente o commerciale
- `Offerta da formulare` — ho tutti i prezzi, devo comporre l'offerta
- `Offerta inviata` — spedita al cliente
- `In attesa accettazione` — in attesa di risposta cliente
- `Chiusa positiva` — accettata (apre workflow omologa per ogni linea)
- `Chiusa negativa` — rifiutata
- `Annullata` — cancellata o non più rilevante

**Stati linea EER** (più granulari):
- `Da quotare` — manca quotazione
- `In quotazione` — richieste prezzi inviate a impianti/trasportatori
- `Quotata` — ho tutti i preventivi
- `Offerta inviata`
- `Accettata` — passa automaticamente in stato omologa
- `In omologa` — workflow omologa in corso
- `Pronto per conferimento` — omologa accettata, può partire l'operativo
- `Conferimenti attivi` — pratica viva, ritiri ricorrenti
- `Rifiutata`
- `Annullata`

### 4.3 Schermata "Costruzione offerta" — l'intelligenza del sistema

Quando l'operatore lavora una linea EER (es. 15.01.10 per cliente X), vede contestualmente:

**A) Storico prezzi rilevanti** (pannello laterale)
- Ultime 5 offerte fatte per lo stesso codice EER (qualsiasi cliente): prezzo finale, impianto usato, trasportatore, esito
- Ultime 3 offerte fatte allo stesso cliente (qualsiasi EER): per capire la relazione commerciale
- Prezzo medio impianto e trasporto per quel EER negli ultimi 6 mesi
- Trend prezzo CER (in salita/in discesa) ultimi 12 mesi

**B) Suggerimenti partner** (in base a zona ritiro + EER + mezzo)
- Top 5 impianti che trattano questo EER, ordinati per: prezzo storico, tempo risposta, distanza
- Top 5 trasportatori che coprono la zona con il mezzo richiesto
- Indicatori visivi: "prezzo in salita", "risponde in <24h", "spesso il più competitivo"
- **Filtro compliance attivo di default**: mostra solo partner con autorizzazioni valide per quel CER (vedi 4.7)

**C) Confronto competitivo**
- Se il cliente ha dichiarato prezzo concorrente: confronto visivo con quello che stai per offrire
- Margine attuale vs margine medio storico

**D) Composizione prezzo cliente**
- Componenti separate: smaltimento + trasporto + oneri + margine = prezzo finale
- Modificabile in tempo reale, mostra impatto su marginalità

**E) Collegamento Monitor Analisi**
- Se il cliente ha analisi valida per questo EER: ok, link cross-modulo
- Se scaduta o mancante: alert + azione "richiedi analisi"

### 4.4 Mappa rete (stile CER Manager)

Vista cartografica Italia (e oltre se rilevante):
- Marker per ogni impianto (colore per tipologia D/R)
- Marker per ogni trasportatore
- Filtri: EER specifico, mezzo, raggio km, stato attivo/inattivo, compliance ok
- Click su marker: scheda riassuntiva con performance
- Modalità "ricerca per zona": inserisci comune/CAP del ritiro, vedi impianti e trasportatori vicini ordinati per distanza
- Heatmap: zone più coperte / meno coperte

### 4.5 Performance impianti e trasportatori

Per ogni partner, schermata di dettaglio con:
- **Scheda anagrafica** (dati base)
- **Autorizzazioni e iscrizioni Albo** (sezione dedicata, vedi 4.7)
- **Performance commerciali** (KPI):
  - Tasso di risposta (quante richieste prezzo hanno risposto / inviate)
  - Tempo medio risposta
  - Tasso "competitività" (quante volte il loro preventivo è stato selezionato)
  - Trend prezzi ultimi 12 mesi (grafico per EER)
- **Storico pratiche**: tutte le pratiche in cui sono stati coinvolti
- **EER trattati / mezzi**: lista con prezzo medio per ognuno
- **Note operative** (modalità di lavoro, contatti, idiosincrasie)
- **Score qualitativo** (rating interno, da 1 a 5 stelle, manuale)

### 4.6 Report direzionali

Dashboard analitica con vista mensile/trimestrale/annuale:

**KPI principali**:
- Numero pratiche per stato
- Tasso conversione (pratiche chiuse positive / totali)
- Tempo medio chiusura
- Volume offerte (€ totali)
- Marginalità media
- Top 10 clienti per volume
- Top 10 EER per volume
- Top 5 impianti per uso
- Top 5 trasportatori per uso

**Grafici** (modello Monitor Analisi):
- Trend pratiche/mese
- Trend tasso conversione
- Trend prezzo per EER (selezionabili)
- Distribuzione geografica ritiri
- Marginalità per CER

**Esportazione**:
- Report PDF mensile auto-generato
- Excel personalizzato con filtri
- Snapshot direzionale settimanale via email

### 4.7 Compliance check automatico (feature critica, novità v2.0)

**Perché**: per legge l'intermediario senza detenzione è co-responsabile della corretta gestione del rifiuto. Se un trasportatore o un impianto perde autorizzazione e l'azienda continua a usarli, scattano sanzioni penali. Oggi questo controllo è manuale e basato sulla memoria delle persone. Va automatizzato.

**Cosa fa il sistema**:

1. Per ogni **trasportatore** mantiene: categoria Albo, CER autorizzati per categoria, scadenza iscrizione
2. Per ogni **impianto** mantiene: numero autorizzazione AIA/AUA, CER ammessi, operazioni R/D, scadenza
3. Quando l'operatore crea una linea EER e seleziona un partner:
   - **Verifica automatica**: l'autorizzazione copre questo CER? È valida alla data prevista del conferimento?
   - **Semaforo visivo**: verde / giallo (scadenza entro 30gg) / rosso (scaduta o CER non coperto)
   - **Blocco** sulla selezione di partner non in regola (override possibile solo da admin con motivazione registrata)
4. **Scadenzario unificato**: tutte le autorizzazioni in scadenza nei prossimi 90 giorni in vista dedicata
5. **Audit log**: ogni selezione di un partner viene loggata con stato compliance al momento della scelta

**Output**: report compliance per direzione, che mostra in qualunque momento se la rete attiva è 100% in regola.

### 4.8 Workflow omologa rifiuto (feature critica, novità v2.0)

**Perché**: dopo che il cliente accetta l'offerta, prima del primo conferimento parte la procedura di omologa. Ogni impianto ha il suo modulo, ma la sostanza è standardizzata: caratterizzazione rifiuto, processo di origine, analisi, dichiarazione di tracciabilità. Oggi questo passaggio è gestito via email/PDF in modo non strutturato. Spesso si perde tempo perché l'omologa torna indietro per dati mancanti.

**Cosa fa il sistema**:

1. Alla accettazione di una linea EER, attiva automaticamente lo stato `In omologa`
2. Pre-compila un template di omologa con i dati già nel sistema:
   - Anagrafica cliente
   - Descrizione rifiuto, CER, processo origine, stato fisico
   - Analisi associata (link a Monitor Analisi)
   - Quantità e frequenza dal preventivo
3. L'operatore completa i campi mancanti e genera il PDF
4. Invio all'impianto (via email integrata) e tracking dello stato:
   - Inviata (con timestamp)
   - In revisione
   - Accettata (omologa valida per N mesi)
   - Rifiutata / Da rivedere (con motivo)
5. **Scadenzario omologhe**: l'omologa tipicamente vale 12 mesi, va rinnovata. Alert prima della scadenza.
6. **Riusabilità**: se lo stesso cliente porta lo stesso CER allo stesso impianto, riusa l'omologa esistente.

### 4.9 Output PDF offerta cliente

Generazione di PDF formale su carta intestata:
- Logo e dati azienda
- Riferimenti pratica
- Dati cliente
- Tabella linee EER con prezzi
- Condizioni (validità, modalità ritiro, ecc.)
- Firme
- Allegati eventuali

Template configurabile, dati strutturati dal database.

### 4.10 Integrazione Monitor Analisi

Bidirezionale:
- Da Monitor Offerte: vedi se il cliente ha analisi valide per il CER della linea (link cross-modulo)
- Da Monitor Analisi: vedi se ci sono pratiche aperte che usano quell'analisi
- Cliente unico tra i due moduli (stessa entità Cliente in DB)

### 4.11 Mobile-first per commerciali (feature critica, novità v2.0)

**Perché**: i commerciali ricevono richieste via WhatsApp/telefono mentre sono in giro dai clienti. Se devono aspettare di tornare in ufficio per inserire la richiesta, perdono dettagli e tempo. Allo stesso modo, devono poter consultare stato pratiche in mobilità.

**Cosa serve**:
- PWA (Progressive Web App) o app responsive che funzioni bene su smartphone
- Inserimento rapido di nuova pratica: form ridotto, foto allegate, dettatura vocale per le note
- Notifiche push su aggiornamenti delle proprie pratiche
- Vista "le mie pratiche" con stato

**Approccio raccomandato**: PWA su Next.js. Senza app store, senza approvazioni, deploy istantaneo. Sufficiente per il 90% delle esigenze.

### 4.12 AI parsing email richieste (feature critica, novità v2.0)

**Perché**: la maggior parte delle richieste arriva via email, in formato libero. Il commerciale o back office deve leggerla, estrarre i dati, inserirli a mano. AI può fare questo in 2 secondi.

**Cosa fa**:
- L'utente fa forward dell'email di richiesta a un indirizzo dedicato (o la incolla nel form)
- Claude AI estrae: ragione sociale cliente, CER (se citato), quantità, frequenza, modalità ritiro, indirizzo, riferimenti
- Pre-compila il form della nuova pratica
- L'operatore verifica e conferma (come Monitor Analisi con i certificati)

**Confidence score**: come Monitor Analisi, se il sistema non è sicuro evidenzia i campi da rivedere.

---

## 5. Architettura tecnica

### 5.1 Stack

Coerente con Monitor Analisi:
- **Frontend**: Next.js (App Router), React, TypeScript
- **Backend**: API Routes Next.js + Server Actions
- **Database**: Supabase (PostgreSQL) con Row Level Security per multi-tenant
- **Auth**: Supabase Auth
- **Storage**: Supabase Storage (allegati, PDF offerte, omologhe)
- **AI**: Claude (Anthropic API) per assistente, estrazioni da email, suggerimenti
- **Mappe**: Mapbox o Leaflet+OpenStreetMap (preferire OSM per costi)
- **Deploy**: Vercel
- **Email**: Resend o servizio equivalente
- **Mobile**: PWA su Next.js (no app nativa nella prima fase)

### 5.2 Multi-tenancy

Già predisposto in Monitor Analisi: ogni tabella ha `org_id`, policies RLS filtrano per organizzazione dell'utente. Estendere lo stesso pattern.

### 5.3 Schema database (alto livello)

Tabelle nuove rispetto a Monitor Analisi:
- `pratiche`
- `linee_eer`
- `impianti` (se non già presente)
- `autorizzazioni_impianto`
- `trasportatori`
- `iscrizioni_albo_trasportatore`
- `quotazioni_impianto`
- `quotazioni_trasportatore`
- `prezzi_cliente`
- `omologhe`
- `concorrenti`
- `prezzi_concorrenza`
- `eventi_pratica` (timeline)
- `commenti_pratica`
- `audit_compliance` (log delle verifiche)
- View materializzate per performance: `mv_performance_impianto`, `mv_performance_trasportatore`, `mv_prezzi_storici_eer`

Tabelle condivise con Monitor Analisi:
- `organizzazioni`
- `org_members`
- `clienti`
- `sedi`
- `codici_cer`

### 5.4 Permessi e ruoli

Ruoli previsti dentro un'organizzazione:
- **Owner / Admin**: tutto, incluso override compliance
- **Commerciale**: vede e modifica le proprie pratiche, vede anagrafiche, vede performance senza dati sensibili
- **Back office**: vede e modifica tutte le pratiche, gestisce quotazioni e omologhe, modifica anagrafiche
- **Direzione**: read-only su tutto + accesso ai report direzionali e analytics avanzate
- **Viewer / Cliente esterno** (se mai esposto): vede solo proprie pratiche

### 5.5 Performance e scalabilità

- 60-77 pratiche/mese × 12 mesi = ~900 pratiche/anno per organizzazione
- ogni pratica con 1-5 linee → ~3000 linee/anno
- ogni linea con 2-5 quotazioni → ~12000 quotazioni/anno
- Volume gestibile senza ottimizzazioni particolari nei primi 2-3 anni
- Indici su FK e su campi di filtro frequenti (stato, urgenza, data, EER, cliente, commerciale)
- View materializzate per analytics, refresh notturno

---

## 6. Roadmap di sviluppo proposta (rivista v2.0)

### Fase 1 — Foundation + Compliance (3-4 settimane)
- Schema database completo (con migrazioni Supabase)
- Auth e RLS multi-tenant
- Anagrafiche: Clienti, Impianti, Trasportatori (CRUD completi)
- **Autorizzazioni e iscrizioni Albo con scadenze e CER coperti**
- **Compliance check di base nella selezione partner**
- Import dati esistenti (dall'Excel attuale, almeno parziale)

### Fase 2 — Core pratiche (3-4 settimane)
- Pratica con linee EER
- Quotazioni impianto e trasportatore
- Composizione prezzo cliente
- Timeline eventi
- Dashboard pratiche con filtri
- Stati e workflow base
- **AI parsing email per pre-compilazione nuova pratica**

### Fase 3 — Intelligence (2-3 settimane)
- Schermata "Costruzione offerta" con suggerimenti
- Storico prezzi e statistiche per EER
- Performance impianti e trasportatori (view + dashboard)
- Trend e grafici
- Filtri compliance integrati nei suggerimenti

### Fase 4 — Workflow omologa + Mappa (2-3 settimane)
- **Workflow omologa rifiuto** completo
- Template PDF omologa con pre-compilazione
- Scadenzario omologhe
- Mappa geografica rete partner
- Ricerca per zona
- Generazione PDF offerta su carta intestata

### Fase 5 — Report + Integrazione (2 settimane)
- Dashboard direzionale con KPI
- Report mensili auto-generati
- Esportazione Excel report
- Integrazione con Monitor Analisi (link cross-modulo, dati cliente condivisi)

### Fase 6 — Mobile + AI avanzata (2 settimane)
- Ottimizzazione PWA per uso mobile da parte dei commerciali
- Notifiche push
- Esperto AI: interroga i dati in linguaggio naturale
- Suggerimenti predittivi avanzati per costruzione offerta

**Totale stimato**: 14-18 settimane per MVP completo. MVP minimale (Fasi 1+2): 6-8 settimane.

---

## 7. Rischi e mitigazioni (novità v2.0)

Analisi critica delle feature ad alto rischio, con motivi per cui potrebbero non funzionare e rimedi anticipati.

### 7.1 Verifica compliance automatica

**Rischio**: l'Albo Nazionale Gestori Ambientali non ha API pubbliche moderne. Il database autorizzazioni AIA è frammentato tra province/regioni. CER Manager mantiene il suo database ma è un servizio commerciale.

**Mitigazione**:
- Fase 1: inserimento manuale dei dati autorizzativi al momento del setup del partner + reminder periodici di aggiornamento (mensile/trimestrale)
- Fase 2: scraping periodico del portale Albo (rischio: termini d'uso, fragilità tecnica)
- Fase 3 (se realmente serve): valutare licenza dati da CER Manager o servizi simili

**Importante**: anche la versione "manuale" è infinitamente meglio del nulla attuale. Il valore è che il sistema *forza* il controllo, non che lo automatizza al 100%.

### 7.2 AI parsing email

**Rischio**: le email di richiesta sono molto variabili, con linguaggio gergale, abbreviazioni, allegati con info chiave. Claude potrebbe fare errori (allucinare quantità, scambiare cliente con destinatario). Se gli operatori si fidano ciecamente, finiscono nel sistema dati sbagliati.

**Mitigazione**:
- Confidence score per campo, come Monitor Analisi
- Sotto soglia 95% → revisione manuale obbligatoria (non si salva niente in automatico)
- Inizio: usare l'AI parsing come "aiuto compilazione", non automatismo
- Feedback loop: ogni correzione manuale alimenta prompt engineering successivo

### 7.3 Storico prezzi per intelligence

**Rischio**: il sistema parte vuoto. Senza dati storici, l'intelligence sui suggerimenti non funziona. Per i primi 3-6 mesi non c'è valore differenziante.

**Mitigazione**:
- Parsing AI delle note del file Excel attuale (60-77 pratiche/mese × 12 mesi = ~900 record da cui estrarre prezzi). Estrazione "best effort", non garantita al 100%.
- Inserimento progressivo: ogni nuova quotazione popola lo storico
- Importazione manuale "guidata" dei prezzi più importanti per i clienti top
- Comunicare chiaramente all'azienda che l'intelligence "matura" col tempo

### 7.4 Performance partner — interpretazione

**Rischio**: la metrica "tempo risposta" o "tasso competitività" può essere fuorviante. Un impianto può rispondere lento ma essere strategico. Un trasportatore può essere caro ma affidabile sull'urgenza.

**Mitigazione**:
- Mostrare KPI con contesto, non come ranking puro
- Permettere annotazioni qualitative ("strategico", "affidabile", "ultima scelta")
- Non automatizzare la decisione (mai un "selezione automatica del miglior partner"). Il sistema suggerisce, l'umano decide.

### 7.5 Adozione interna (rischio non tecnico, ma cruciale)

**Rischio**: il back office usa Excel da anni. Il nuovo sistema, anche se migliore, ha curva di apprendimento. Resistenza al cambiamento. Se l'adozione è parziale, si finisce a usare entrambi (Excel + Monitor), peggio del solo Excel.

**Mitigazione**:
- Parallel run di 1-2 mesi con confronto risultati
- Import file Excel storico per dare valore immediato (vedi anche 7.4)
- Identificare 1-2 "champions" interni che adottino per primi e diventino formatori
- Onboarding ridotto: massimo 2 ore di formazione per ruolo
- Iterazione settimanale con il team nei primi 3 mesi per aggiustare quello che non funziona

### 7.6 Sovrapposizione con CER Manager (concorrenza disruptive)

**Rischio**: CER Manager o piattaforme analoghe potrebbero crescere abbastanza da disintermediare il modello di business dell'intermediario stesso. Se i produttori iniziano a usare direttamente queste piattaforme, l'intermediario perde valore.

**Mitigazione**:
- Monitor Offerte si posiziona come strumento *interno* per l'intermediario, non come marketplace
- Valore aggiunto: relazione personale, conoscenza del cliente, intelligence proprietaria sui prezzi
- Possibile evoluzione: integrare CER Manager come "fonte di lead aggiuntiva" senza renderla l'unico canale
- Investire nel valore della rete privata e nella qualità del servizio commerciale

---

## 8. Criteri di successo

Il prodotto è considerato riuscito se:

1. **Sostituisce completamente l'Excel mensile** — il back office non apre più il file Excel per gestire le pratiche
2. **Riduce il tempo medio di costruzione di un'offerta** del 40-50% grazie ai suggerimenti automatici
3. **Aumenta il tasso di chiusura positiva** (offerte accettate) misurabilmente, grazie a prezzi più competitivi informati dallo storico
4. **Riduce le richieste di prezzo "alla cieca"** agli impianti: si interpella prima chi ha più probabilità di essere competitivo
5. **Azzera gli errori di compliance**: nessun rifiuto inviato a un partner con autorizzazione scaduta o non coperto sul CER
6. **Riduce il tempo morto dell'omologa**: da media attuale a target -50% sui tempi di completamento
7. **Produce un report direzionale mensile** che la direzione effettivamente usa per le decisioni
8. **Estende la copertura geografica**: l'azienda accetta richieste in zone "nuove" perché vede subito chi può servirle

---

## 9. Considerazioni aperte e da definire

Punti su cui serve allineamento prima/durante lo sviluppo:

- **Origine dati prezzi storici di partenza**: AI parsing del file Excel attuale? Inserimento progressivo? Mix?
- **Verifica Albo automatica**: si paga per servizio terzo o si parte manuale? Decidere in funzione del budget.
- **Gestione contratti pluriennali**: alcune pratiche diventano contratti full-service annuali. Modello dati per gestirli (oggi nelle note del file).
- **Mezzi disponibili**: catalogo standard di mezzi trasporto da pre-popolare (bilico, container 6m³, container 30m³, motrice, cassone stagno, IBC 1000lt, compattatore, ecc.).
- **Categorie Albo Gestori dei trasportatori**: categorie 1, 4, 5, 8, 9, 10. Per la Cat. 5 dal 30/06/2026 obbligo geolocalizzazione mezzi.
- **Esperto AI**: quale fonte di verità per le domande in linguaggio naturale? Solo i dati del modulo o anche normativa?
- **Notifiche**: email/in-app/push per scadenze, urgenze, decisioni richieste — strategia anti-spam.
- **Template omologa**: standardizzare un template "universale" o creare varianti per gli impianti più usati?
- **Integrazione ECOS** (evoluzione futura, non in scope iniziale): a oggi non è proponibile o praticabile. In una fase successiva, quando l'adozione di Monitor Offerte sarà consolidata, potrà essere valutata un'integrazione con il gestionale ECOS (export dati, eventuali API se disponibili) per evitare doppi data entry post-accettazione offerta. **Per ora il flusso si interrompe alla consegna della pratica al team operativo, che continua il lavoro su ECOS in modo autonomo**.

---

## 10. Note per chi sviluppa

- Coerenza visiva con Monitor Analisi (stesso design system, stessa palette, stessa navigazione)
- Tutto in italiano (UI, dati, documentazione utente)
- Multi-tenant rigoroso: nessun dato deve mai filtrare tra organizzazioni
- Audit log su modifiche critiche (chi ha cambiato cosa quando) — minimo su stato pratica, prezzi, override compliance
- Soft delete su entità principali (pratiche, quotazioni): nulla si cancella davvero
- Versionamento offerte cliente: ogni revisione è una versione tracciabile
- Performance: la dashboard pratiche deve aprirsi in <1s anche con 5000 pratiche storiche
- Mobile-first per le schermate dei commerciali (inserimento rapido)
- Tutte le scadenze normative (autorizzazioni, omologhe) hanno almeno 3 livelli di alert: 90gg, 30gg, 7gg

---

*Documento da aggiornare iterativamente man mano che lo sviluppo procede e nuovi requisiti emergono.*
