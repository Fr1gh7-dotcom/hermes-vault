---
title: "Retrieval-Augmented Generation (RAG)"
source: https://en.wikipedia.org/wiki/Retrieval-augmented_generation
source_type: web
date: 2026-08-05
hermes_id: 253e6517-a5eb-49d7-abf1-b73d8fe5f795
tags:
  - intelligenza artificiale
  - modelli linguistici
  - recupero informazioni
  - allucinazioni AI
  - embeddings vettoriali
---

## Riassunto

RAG è una tecnica che permette ai modelli linguistici di grandi dimensioni (LLM) di recuperare informazioni da fonti esterne prima di generare risposte, riducendo le allucinazioni e la necessità di riaddestramento. Introdotta in un paper del 2020, combina un modello linguistico parametrico con una memoria esterna non parametrica accessibile tramite retrieval durante l'inferenza. Viene applicata in ambiti come l'healthcare e i chatbot aziendali per garantire risposte aggiornate e verificabili.

## Contenuto

Retrieval-Augmented Generation (RAG) è una tecnica che consente agli LLM di recuperare e incorporare nuove informazioni da sorgenti dati esterne. Gli LLM prima consultano un insieme specificato di documenti, poi rispondono alle query degli utenti. Questi documenti integrano le informazioni già presenti nei dati di addestramento dell'LLM, permettendo l'uso di informazioni di dominio specifico o aggiornate non disponibili nel training set. Ad esempio, abilita chatbot basati su LLM ad accedere a dati aziendali interni o generare risposte basate su fonti autorevoli. La tecnica è stata proposta per la prima volta nel 2020 ed è diventata un approccio ampiamente adottato nei moderni sistemi AI.

RAG migliora gli LLM incorporando il recupero delle informazioni prima della generazione delle risposte. A differenza degli LLM che si affidano a dati di addestramento statici, RAG recupera testi rilevanti da database, documenti caricati o fonti web. Secondo Ars Technica, 'RAG è un modo per migliorare le prestazioni degli LLM, essenzialmente fondendo il processo LLM con una ricerca web o un altro processo di ricerca documenti per aiutare gli LLM a rispettare i fatti.' Questo metodo aiuta a ridurre le allucinazioni dell'AI, che hanno causato la descrizione di politiche inesistenti o la raccomandazione a avvocati di casi legali inventati come citazioni. RAG riduce anche la necessità di riaddestrare gli LLM con nuovi dati, risparmiando costi computazionali e finanziari. Permette inoltre agli LLM di includere fonti nelle risposte, così gli utenti possono verificare le fonti citate, fornendo maggiore trasparenza.

Il termine RAG è stato introdotto in un paper del 2020 che descriveva la combinazione di un modello linguistico parametrico con una memoria esterna non parametrica accessibile tramite retrieval durante il tempo di inferenza.

Limitazioni degli LLM e RAG:
Gli LLM possono fornire informazioni errate. Ad esempio, quando Google ha dimostrato per la prima volta il suo strumento LLM 'Google Bard' (successivamente rinominato Gemini), l'LLM ha fornito informazioni errate sul James Webb Space Telescope. Questo errore ha contribuito a un calo di 100 miliardi di dollari nel valore delle azioni di Google. RAG viene utilizzato per prevenire questi errori, ma non risolve tutti i problemi. Ad esempio, gli LLM possono generare disinformazione anche quando attingono da fonti fattuali corrette se interpretano male il contesto. Il MIT Technology Review fornisce l'esempio di una risposta generata dall'AI che afferma: 'Gli Stati Uniti hanno avuto un presidente musulmano, Barack Hussein Obama.' Il modello ha recuperato questo dal titolo retorico del capitolo 'Barack Hussein Obama: America's First Muslim President?' nel libro Faith in the New Millennium: The Future of Religion and American Politics. L'LLM non 'sapeva' o 'capiva' il contesto del titolo, generando un'affermazione falsa.

Gli LLM con RAG sono programmati per dare priorità alle nuove informazioni. Questa tecnica è stata chiamata 'prompt stuffing.' Senza il prompt stuffing, l'input dell'LLM è generato da un utente; con il prompt stuffing, un contesto rilevante aggiuntivo viene aggiunto a questo input per guidare la risposta del modello. Questo approccio fornisce all'LLM informazioni chiave all'inizio del prompt, incoraggiandolo a dare priorità ai dati forniti rispetto alle conoscenze di addestramento preesistenti.

Processo RAG:
RAG migliora gli LLM incorporando un meccanismo di recupero delle informazioni che consente ai modelli di accedere e utilizzare dati aggiuntivi oltre al loro set di addestramento originale. Ars Technica nota che 'quando nuove informazioni diventano disponibili, invece di dover riaddestrare il modello, è sufficiente aumentare la knowledge base esterna del modello con le informazioni aggiornate' (augmentation). IBM afferma che 'nella fase generativa, l'LLM attinge dal prompt aumentato e dalla sua rappresentazione interna dei dati di addestramento per sintetizzare una risposta.'

Fasi chiave del processo RAG:
1. I dati da riferire vengono convertiti in embeddings LLM, rappresentazioni numeriche sotto forma di un grande spazio vettoriale. RAG può essere utilizzato su dati non strutturati (solitamente testo), semi-strutturati o strutturati (ad esempio knowledge graph). Questi embeddings vengono poi memorizzati in un database vettoriale per consentire il recupero dei documenti.
2. Data una query dell'utente, viene prima chiamato un document retriever per selezionare i documenti più rilevanti che verranno utilizzati per aumentare la query. Questo confronto può essere fatto utilizzando vari metodi, che dipendono in parte dal tipo di indicizzazione utilizzata.
3. Il modello inserisce queste informazioni rilevanti recuperate nell'LLM tramite prompt engineering della query originale dell'utente. Le implementazioni più recenti (a partire dal 2023) possono anche incorporare moduli di augmentation specifici con capacità come l'espansione delle query in più domini e l'uso della memoria e del miglioramento automatico per apprendere dai recuperi precedenti.
4. Infine, l'LLM può generare output basato sia sulla query che sui documenti recuperati. Alcuni modelli incorporano passaggi extra per migliorare l'output, come il re-ranking delle informazioni recuperate, la selezione del contesto e il fine-tuning.

Applicazioni:
RAG viene utilizzato in applicazioni in cui le risposte generate devono essere ancorate a informazioni esterne o aggiornate frequentemente. In ambito sanitario, RAG è stato studiato come un modo per ancorare gli output degli LLM in fonti di conoscenza medica esterne, sebbene le revisioni abbiano notato sfide continue riguardo alla valutazione, all'etica e all'affidabilità clinica.

Miglioramenti:
I miglioramenti al processo di base possono essere applicati in diverse fasi del flusso RAG.

Encoder: Questi metodi si concentrano sulla codifica del testo come vettori densi o sparsi. I vettori sparsi, che codificano l'identità di una parola, sono tipicamente lunghi quanto un dizionario e contengono principalmente zeri. I vettori densi, che codificano il significato, sono più compatti e contengono meno zeri. Vari miglioramenti possono ottimizzare il modo in cui le similarità vengono calcolate nei vector store (database). Le prestazioni migliorano ottimizzando il calcolo delle similarità vettoriali. I dot product migliorano il punteggio di similarità, mentre le ricerche approximate nearest neighbor (ANN) migliorano l'efficienza del recupero rispetto alle ricerche K-nearest neighbors (KNN). L'accuratezza può essere migliorata con Late Interaction.

Metodi centrati sul retriever: Riguardano l'ottimizzazione del componente di recupero.

Modello linguistico: Miglioramenti applicati direttamente all'LLM.

Chunking: Tecnica di suddivisione dei documenti in segmenti per migliorare il recupero.

Hybrid search: Combinazione di metodi di ricerca per ottimizzare i risultati.

Sfide - RAG Poisoning: Una delle sfide identificate è il cosiddetto RAG poisoning, una forma di attacco ai sistemi RAG.