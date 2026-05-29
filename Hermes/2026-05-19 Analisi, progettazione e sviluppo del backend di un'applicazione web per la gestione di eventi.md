---
title: "Analisi, progettazione e sviluppo del backend di un'applicazione web per la gestione di eventi"
source: 
source_type: pdf
date: 2026-05-19
hermes_id: f9b2b7c3-3d05-495f-ba48-205c01c5dc66
tags:
  - backend
  - ruby on rails
  - api rest
  - gestione eventi
  - stage universitario
  - database
  - architettura software
  - multi-tenant
---

## Riassunto

La tesi descrive il lavoro di stage svolto da Alberto Lazari presso Moku S.r.l. (sede di Treviso) nel periodo aprile-luglio 2022, per una durata di circa 300 ore. L'obiettivo principale era la reimplementazione completa del backend della piattaforma Evvvents, un'applicazione web multi-tenant per la gestione di eventi e conferenze dal vivo, online e ibride. Lo stage si è svolto all'interno di un contesto lavorativo agile basato su Scrum, con sprint bisettimanali, stand-up meeting quotidiani e sprint review con il cliente.

L'analisi del backend preesistente ha evidenziato numerosi difetti strutturali e funzionali: entità con attributi superflui o mal distribuiti, informazioni sulle integrazioni sparse in tabelle non dedicate, attributi replicati nella tabella eventi e nomi poco intuitivi. Sulla base di questa analisi, sono stati ridisegnati i modelli del database, unificando le entità degli utenti, creando entità dedicate per le integrazioni, rifattorizzando i reminder degli eventi in una tabella separata e migliorando la coerenza generale dello schema.

Le tecnologie adottate rispecchiano le prassi aziendali di Moku: il linguaggio Ruby con il framework Ruby on Rails (pattern MVC e Active Record), il database PostgreSQL, la gemma Devise per l'autenticazione, Pundit per la gestione delle autorizzazioni e Active Storage per la gestione dei file su AWS S3. La scelta di sviluppare una API REST (anziché GraphQL, normalmente preferita da Moku) è stata motivata dalla necessità di limitare le modifiche al frontend Angular già esistente.

La progettazione dell'API ha definito endpoint standard per le operazioni CRUD su ogni risorsa (User, Plan, Platform, Organizer, Location), con supporto alla paginazione, gestione delle risorse annidate e un sistema di permessi granulare basato sui ruoli utente (super admin, admin, manager, staff, speaker, partecipante). Sono stati definiti in dettaglio i permessi richiesti per ogni azione e i parametri ammessi nelle richieste di creazione e modifica.

La fase di codifica ha seguito un processo strutturato: generazione delle migrazioni del database tramite il generatore Rails, definizione delle associazioni tra modelli tramite Active Record, implementazione delle validazioni e creazione di un concern condiviso per la gestione del campo creator. Sono stati inoltre implementati controller basati su APIController, policy Pundit per l'autorizzazione e test di unità con RSpec, anche se in quantità limitata per scelta del project manager.

## Citazioni chiave

> il backend presentava grossi limiti, funzionali e strutturali. Mantenerlo e modificarlo per raggiungere uno standard di qualità adeguato sarebbe costato più tempo e risorse di una riscrittura completa, quindi è stata presa la decisione di svilupparlo da zero

> L'entità e_events, associata al modello degli eventi, era la più estesa del sistema, com'è lecito aspettarsi, essendo questa la funzionalità fulcro dell'intera applicazione. Non è però giustificabile la complessità e la quantità di informazioni che venivano memorizzate nell'entità

> tutto questo portava l'entità ad avere un totale di ben 84 attributi, un numero decisamente troppo elevato per essere gestibile nel modello di una funzionalità simile

> Per il progetto di stage si è optato per sviluppare una API REST per limitare le modifiche necessarie da fare sul frontend, essendo che questo era già pronto e si basava sulla API precedente, sviluppata secondo lo stile REST.

> Rails si basa sul paradigma convention over configuration, quindi cerca di ridurre al minimo le decisioni che lo sviluppatore deve prendere, al fine di ridurre il codice boilerplate e favorire il rispetto del principio don't repeat yourself (DRY).

> autenticazione e autorizzazione, seppur simili, sono due concetti diversi e, come tali, devono essere separati anche strutturalmente.

> La scelta di unire i partecipanti al resto degli utenti è stata presa per ridurre la duplicazione dei dati, in questo caso rilevante, perché le due entità, una volta riviste, avrebbero avuto quasi tutti i campi in comune.

> I clienti che desiderano personalizzare le impostazioni SMTP per la loro piattaforma potranno chiedere all'azienda di configurarle, secondo le loro richieste. In questo modo si può garantire il funzionamento delle notifiche mail, essenziali anche solamente per registrare un nuovo utente.

> La metodologia adottata permette di avere una comunicazione regolare ed efficace tra il team di sviluppo e il cliente, che porta a una definizione più semplice e precisa dei requisiti che il prodotto deve rispettare
