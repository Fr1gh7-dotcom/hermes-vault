---
title: "Progettazione e implementazione del backend di un'applicazione di prospecting"
source: 
source_type: pdf
date: 2026-05-19
hermes_id: ad7db1bb-c57f-4275-839c-b4dbc87aa4dc
tags:
  - microservizi
  - backend
  - cloud computing
  - microsoft azure
  - architettura software
  - prospecting
  - metodologia agile
  - ingegneria informatica
---

## Riassunto

La tesi, sviluppata presso l'azienda di consulenza Akkodis nell'anno accademico 2023-2024, descrive la progettazione e l'implementazione del backend di un'applicazione aziendale di prospecting. Il software è destinato ai business manager responsabili di identificare, valutare e approcciare potenziali aziende clienti, con funzionalità quali l'invio automatico di email tramite template dinamici, il salvataggio di ricerche e la generazione di statistiche con granularità temporale personalizzabile. L'integrazione con Apollo.io (scelta in sostituzione delle API di LinkedIn, rimosse dalla piattaforma) consente la ricerca di prospect e dei relativi dipendenti da contattare.

Dal punto di vista architetturale, il progetto adotta un'architettura a microservizi con componenti ospitate su Microsoft Azure. La tesi approfondisce i concetti teorici alla base di questa scelta, confrontando l'architettura monolitica con quella a microservizi, analizzandone vantaggi e svantaggi. Viene descritto l'algoritmo 3-step microservices decomposition di Chris Richardson, utilizzato per identificare i microservizi, le loro system operation e le relative API.

Un capitolo dedicato al Cloud Computing esplora l'evoluzione del cloud, il ruolo dei Cloud Service Provider e i servizi Microsoft Azure impiegati nel progetto, tra cui Azure SQL Server, Cosmos DB, Blob Storage, Service Bus, Functions, Kubernetes Service, API Management e DevOps. Viene inoltre trattata la gestione dei dati in architetture distribuite, con particolare attenzione alle sfide legate alla data consistency, alle transazioni distribuite e all'eventual consistency tramite il pattern SAGA.

Il processo di sviluppo è stato condotto seguendo la metodologia agile con il framework Scrum, partendo dalla definizione di user story e acceptance criteria fino alla progettazione dell'architettura cloud. La tesi dedica inoltre spazio alle tecnologie e agli strumenti utilizzati (Visual Studio Code, Docker, Prisma, SendGrid, Postman, Git) e alle tipologie di test adottate, incluso il Test-Driven Development. Le conclusioni evidenziano i risultati raggiunti e le future possibilità di miglioramento, con particolare attenzione alla conformità GDPR nella gestione dei dati sensibili dei dipendenti ricercati.

## Citazioni chiave

> L'architettura riguarda le cose importanti. Qualsiasi esse siano

> un'architettura a microservizi può essere descritta come un'architettura che decompone un dominio di business in piccole unità, ovvero i microservizi, delimitate in modo coerente e il più indipendenti tra loro.

> risulta fondamentale aggiornare il software per gestire attentamente la memorizzazione dei dati sensibili relativi ai dipendenti cercati, al fine di garantire la conformità con le disposizioni del GDPR.

> Per separation of concerns si intende l'abilità di decomporre e organizzare un sistema in moduli logicamente coesi (cohesive) e poco dipendenti tra di loro (loosely-coupled), i quali nascondono la propria implementazione l'uno dall'altro e espongono dei servizi tramite delle interfacce ben definite.

> In un sistema che adotta l'eventual consistency si accetta che i dati possano essere non allineati per un breve periodo di tempo, finché, appunto, tutte le repliche non convergono ad uno stato in cui condividono lo stesso valore.

> ogni microservizio deve avere il proprio meccanismo di storage, non condiviso con altri servizi.

> Lo scopo di Richardson non è fornire un processo da seguire meccanicamente, piuttosto suggerire delle linee guida generiche; difatti questo è un algoritmo che richiede anche un pizzico di creatività e flessibilità.

> La comunicazione asincrona porta con se diversi vantaggi dato che non vi è dipendenza temporale tra i due microservizi.

> un'applicazione aziendale simile, soprattutto quando indirizzata ai singoli manager aziendali, potrebbe notevolmente semplificare e accelerare il processo di ricerca di potenziali clienti.
