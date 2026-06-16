---
title: "Agent Harness: cos'è e come ancorare un LLM alla realtà"
source: https://www.instagram.com/reel/DYe2KwvNPhM/?igsh=MWFuaWpydTR4djdkNw==
source_type: instagram
date: 2026-06-16
hermes_id: 3b69616c-2dc7-4258-89e8-f5ca7666d615
tags:
  - agenti AI
  - LLM
  - harness
  - sviluppo agenti
  - architettura AI
---

## Riassunto

L'harness è una struttura che serve ad ancorare un LLM alla realtà, suddivisa in Eval Harness e Agent Harness. È composta da elementi chiave come il registro dei tool, la scelta del modello, la gestione del contesto, le guardrails e l'agentic loop. È importante non confondere l'harness con l'agentic loop: possono esistere loop annidati con fasi di verifica esterne all'harness. L'harness non è permanente: dipende dal modello usato e va rimossa o adattata quando il modello evolve e supporta nativamente certe funzionalità. Un errore comune è sostituire la logica dell'LLM con codice rigido: bisogna distinguere quando serve stabilità (codice) e quando serve creatività (LLM). La scelta giusta dell'harness può far performare ottimamente anche modelli piccoli, mentre un'harness troppo restrittiva penalizza modelli grandi.

## Contenuto

Se stai creando un'agente hai sicuramente sentito parlare di harness, ma cosa è e perché ci serve? Questa è un'imbraccatura, o in inglese, harness. Serve a legare l'essere umano a una cosa solida, la montagna. Anche questo è un harness. Quindi usiamo l'harness perché è qualcosa di affidabile. Ma l'harness si divide in due tipi. La Eval Harness e la Agent Harness. Noi andremo a parlare di quest'ultimo. A noi l'harness serve per ancorare l'LLM alla realtà. E lo facciamo con una serie di cose. Il registro dei tool, come i comandi bash. La scelta del modello, che può anche intercambiare. La gestione del contesto. Ormai quasi tutti gli agenti compattano la chat per mantenere più contesto. Le guardrails, come per esempio i tools tra cui dopo 5 passi esce. L'agentic loop, che attenzione non è da confondere. L'harness non va tutta attorno all'agentic loop. Per esempio potremmo avere un agentic loop sull'agentic loop e poi il verify. E tutto questo serve a dare una direzione al nostro agente di quello che vogliamo che faccia. Ma non si limita solo alle cose di coding. Il verify potrebbe essere un fact check. Ma attenzione! L'harness non è per sempre. Vi faccio un esempio. Volete mandare un audio al vostro agente. E quindi convertite l'audio in testo. Il giorno che il modello supporterà l'audio, tutta quest'harness va tolta. L'harness dipende tantissimo dal modello che si usa. Un modello molto grande, con un harness troppo restrittiva, performerà male. Contemporaneamente, un modello da pochi bilioni può performare benissimo con l'harness adatta. Un errore che vedo molto spesso nelle persone che stanno iniziando a fare agenti è che, invece che dare la possibilità alla gente di fare le cose, lo fanno loro per la gente. Dovete dividere quello che è codice da quello che è LLM. A volte vi serve stabilità, a volte vi serve creatività. Per esempio, mettiamo che volete che il vostro agente ogni mattina vi dia un riassunto di quello che ha fatto la notte. Adesso avete due modi. O ogni mattina forzate quel messaggio, oppure passate il contesto alla gente e decide lui cosa farci. Come presentarlo e se presentarlo. Se vi piacciono questi video, ne trovate un sacco nel mio profilo.