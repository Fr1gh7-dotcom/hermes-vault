---
title: "Agent Harness: cos'è e come usarla correttamente"
source: https://www.instagram.com/reel/DYe2KwvNPhM/?igsh=MWFuaWpydTR4djdkNw==
source_type: instagram
date: 2026-05-20
hermes_id: 6dfd45b4-7476-4933-8773-deac08d803fb
tags:
  - Agenti AI
  - LLM
  - Sviluppo AI
  - Architettura agenti
  - Prompt engineering
---

## Riassunto

L'harness è un sistema che serve ad ancorare un LLM alla realtà, fornendo struttura e direzione a un agente AI. Si compone di elementi come il registro dei tool, la gestione del contesto, le guardrail e l'agentic loop. È importante sottolineare che l'harness non è permanente: dipende dal modello utilizzato e va rimossa o adattata quando il modello evolve e acquisisce nuove capacità native. Un errore comune è sovra-controllare l'agente invece di lasciargli autonomia decisionale, confondendo il codice deterministico con ciò che può essere delegato all'LLM. La scelta giusta dell'harness in base alla dimensione del modello è cruciale: un'harness troppo restrittiva penalizza i modelli grandi, mentre una ben calibrata può far performare ottimamente anche modelli piccoli.

## Contenuto

Se stai creando un'agente hai sicuramente sentito parlare di harness, ma cosa è e perché ci serve? Questa è un'imbraccatura, o in inglese, harness. Serve a legare l'essere umano a una cosa solida, la montagna. Anche questo è un harness. Quindi usiamo l'harness perché è qualcosa di affidabile. Ma l'harness si divide in due tipi. La Eval Harness e la Agent Harness. Noi andremo a parlare di quest'ultimo. A noi l'harness serve per ancorare l'LLM alla realtà. E lo facciamo con una serie di cose. Il registro dei tool, come i comandi bash. La scelta del modello, che può anche intercambiare. La gestione del contesto. Ormai quasi tutti gli agenti compattano la chat per mantenere più contesto. Le guardrails, come per esempio i tools tra cui dopo 5 passi esce. L'agentic loop, che attenzione non è da confondere. L'harness non va tutta attorno all'agentic loop. Per esempio potremmo avere un agentic loop sull'agentic loop e poi il verify. E tutto questo serve a dare una direzione al nostro agente di quello che vogliamo che faccia. Ma non si limita solo alle cose di coding. Il verify potrebbe essere un fact check. Ma attenzione! L'harness non è per sempre. Vi faccio un esempio. Volete mandare un audio al vostro agente. E quindi convertite l'audio in testo. Il giorno che il modello supporterà l'audio, tutta quest'harness va tolta. L'harness dipende tantissimo dal modello che si usa. Un modello molto grande, con un harness troppo restrittiva, performerà male. Contemporaneamente, un modello da pochi bilioni può performare benissimo con l'harness adatta. Un errore che vedo molto spesso nelle persone che stanno iniziando a fare agenti è che, invece che dare la possibilità alla gente di fare le cose, lo fanno loro per la gente. Dovete dividere quello che è codice da quello che è LLM. A volte vi serve stabilità, a volte vi serve creatività. Per esempio, mettiamo che volete che il vostro agente ogni mattina vi dia un riassunto di quello che ha fatto la notte. Adesso avete due modi. O ogni mattina forzate quel messaggio, oppure passate il contesto alla gente e decide lui cosa farci. Come presentarlo e se presentarlo. Se vi piacciono questi video, ne trovate un sacco nel mio profilo.