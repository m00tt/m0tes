# Remote Method Invocation
RMI è una tecnologia del mondo Java che permette a processi distribuiti di comunicare attraverso la rete.<br>
Un processo client può effettuare chaiamate di un metodo ad un oggetto remoto (server) come se l'oggetto fosse sulla stessa macchina.

Tipicamente, quando si realizza un'architettura client-server con RMI:
- il server crea gli oggetti remoti, li rende visibili e aspetta che i client invochino metodi su di essi
- il client ottiene dei riferimenti agli oggetti remoti ed invoca metodi su di essi

RMI fornisce il meccanismo attraverso il quale server e client comunicano per costruire un'applicazione distribuita.

# Registry
Affinché un server possa esporre un oggetto remoto attraverso RMI, questo oggetto deve essere pubblicato su un <b>registry</b>.
- Il server crea l'oggetto e lo pubblica sul registry
- Il client cerca presso il registry il servizio di cui ha bisogno, e gli viene restituito un riferimento all'oggetto remoto sul server
- Il client usa il riferimento per poter ottenere il servizio

# Message passing
Con RMI, l'interazione tra client e server è quella tipica dei sistemi object-oriented: <b>il message passing</b>

L'invocazione di un metodo di un oggetto remoto avviene mediante l'invio di un messaggio che contiene il nome del metodo e gli eventuali argomenti.
L'oggetto remoto, a sua volta, invia un messaggio al chiamante con il risultato del metodo.

La costruzione, spedizione, ricezione e ricostruzione dei messaggi sono gesite da RMI, e sono trasparenti al programmatore.

## Passaggio di parametri
Consideriamo la chiamata:

```java
String result = server.sayHello(obj);
```
`server` è un oggetto remoto.
In questo caso si ha un comportamento diverso a seconda della natura dell'argomento `obj`:
- Se è un riferimento ad un oggetto remoto, al metodo remoto viene passato tale riferimento
- Se è un oggetto locale, viene inviata al metodo remoto una copia dell'oggetto mediante serializzazione. In particolare quello che avviene è una _deep copy_: se l'oggetto contiene riferimenti ad altri oggetti, anche questi vengono serializzati e passati al server
- Se è di tipo primitivo, viene passato per valore

Potrebbe essere che l'oggetto remoto non conosca la classe di `obj`. In questo caso, è possibile passare al server anche il codice di tale classe (<b>dynamic class loading</b>).


# Implementazione di RMI
Un'invocazione di un metodo di un oggetto remoto è fornita localmente da uno <b>stub</b>: esso si occupa della clareazione del messaggio contenente il nome e gli argomenti del metodo (parameter marshalling), e l'invio verso il server.

Lato server, il messaggio è ricevuto da uno <b>skeleton</b>: esso ricostruisce i parametri (unmarshalling) e chiama il metodo sull'oggetto vero e proprio. Successivamente prepara il risultato e lo invia allo stub.

## Layer

![RMI Layers](/assets//programmazione_concorrente_e_distribuita/rmi_layer.png)

- Transport Layer: TCP
- Remote Reference Layer (RRL): crea e gestisce i riferimenti remoti
- stub e skeleton comunicano tra di loro utilizzando i riferimenti remoti e fanno da intermediari a client e server

Tutta questa struttura è praticamente trasparente al programmatore.