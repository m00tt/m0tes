# Funzioni chiavi nel network-layer
Le funzioni del network-layer sono :
- Forwarding 
  - Muovere i pacchetti da router input a output appropriato
- Routing
  - Determinare la rotta da sorgente a destinazione
  - Algoritmi di routing

Questo due funzioni vengono fornite dai router :
- Data plane 
  - Funzione locale per-router che determina in che porta deve essere trasmesso in output il datagramma che arriva alla porta input del router (forwarding function) 
- Control plane
  - logica network-wide, determina come il datagramma è instradato tra i router lungo la rotta da host sorgente a destinazione (algoritmi di routing) 

Nota: i singoli componenti dell'algoritmo di instradamento (data plane) in ogni router interagiscono nel piano di controllo (algoritmi di routing)<br>
![Control Plane](/assets/sicurezza_informatica/controle-plane.png)<br>
Questo processo può essere compreso in miglior modo studiando l'architettura di un router: 
![Architettura Router](/assets/sicurezza_informatica/architettura-router.png)<br>
La forward table, routing table e la logica di instradamento costituiscono la funzione del piano dati: un datagramma passa dalla porta di ingresso del router a quella di uscita in base grazie al data plane che segue la logica del control plane: significa che è responsabile dello spostamento dei pacchetti da sorgente alla destinazione. 

Il control plane è responsabile del popolamento della routing table, del disegno della topologia di rete, della forwarding table e quindi permette tutte le funzioni del data plane: significa quindi che in queste fasi il router prende la sua decisione, è responsabile di come devono essere inoltrati i pacchetti. <br>
![Layers](/assets/sicurezza_informatica/layers.png)<br>

# Algoritmo di routing
Un algoritmo di routing ha l'obiettivo di trovare un buon path tra un router sorgente ed un router di destinazione, dato un insieme di router. <br>
Si richiede un path con costo/lunghezza minore. <br>
- Un global routing algorithm usa conoscenza globale sulla rete
- Un decentralized routing algorithm, calcola il  least-cost path con un approccio iterativo e distribuito. 

Internet Routing Protocol: 
- RIP
- OSPF
- BGP

Con Autonomous Systems (AS) si intende un gruppo di router sotto la stessa amministrazione, i router gateway inoltrano I pacchetti con una destinazione al di fuori di un AS: 
- intra-AS routing protocol: determinano come avviene il routing in un AS, anche detti interior gateway protocols. 
  - RIP
  - OSPF
- inter-AS routing protocol: informazioni dai vicini AS, propagano le info a tutti i router interni agli AS 
  - BGP

# Intra Routing : RIP
RIP sta per Routing Information Protocol e rientra nei Intra-AS routing Protocol<br>

## Funzionamento
I router vicini ogni 30 secondi si scambiano aggiornamenti sul routing, questi messaggi vengono inviati sulla porta 520 UDP e sono detti RIP response message o RIP advertisements, ogni messaggio contiene una lista fino a 25 destinazioni subnet in un AS che il mittente può raggiungere (subnet, router vicino, hop che consiste nella distanza).<br>
Tipica connessione RIPv1 : 
- Richiesta broadcast inziale del router per le rotte 
- Il router legge i pacchetti unicast che contengono le rotte inviare da altri router
- Update periodico inviato ogni 30 secondi di default (broadcast):  
Attacco su RIPv1 : 
RIPAttackV1, Reflection DDoS: attacco basato su IP spoofing, chiunque può falsificare i pacchetti, l'unica possibile mitigazione è attivare l'autenticazione o passare a RIPv2 che è stato sviluppato nel 1994 (RFC 2453) per garantire la sicurezza degli aggiornamenti, autenticazione semplice con testo in chiaro e MD5, (RFC 2082). 

## RIP Attacco 
Attacchi alle informazioni : 
- Identificare un router RIP con una scansione nmap –v –sU –p 520 
- Determinare la routing Table: se sei nello stesso segmento fisico sniff, altrimenti rprobe (sonda) + sniff.
- Un attaccante può mandare false informazioni di routing, questo permette di 
  - impersonare un host non attivo: dirige il traffico per quell’host verso la macchina dell’attaccante 
  - impersonare un host attivo: esaminare il traffico e ridirigerlo verso l’host, tutto il traffico può essere ispezionato, tra cui password e altri dati sensibili 

Difese: gateway per filtrare in base a sorgente e/o destinazione ed utilizzare un'autenticazione forte. Oppure disabilitare RIP e usare OSPF e limitare i pacchetti con porta TCP/UDP 520 ai border router. 

# Open Shortes Path First
OSPF rientra nei Intra-AS Routing Protocol <br>
OSPF v2: RFC2328 è un protocollo di routing link-state ampiamente utilizzato, sviluppato per IP dal gruppo di lavoro IGP (Interior Gateway Protocol) di IETF.  
- Si basa sul trasferimento di informazioni tra hop (principalmente router e reti) ed è destinato a essere eseguito internamente in un AS. 
- È distribuito tra i router nell'AS e consente loro di costruire la stessa rappresentazione della topologia di rete dell'AS attraverso la pubblicazione di Link-State Advertising (LSA) da parte dei router: ogni router quindi costruisce un albero del percorso più breve verso diverse destinazioni (subnet), utilizzando l'algoritmo di Dijkstra, con se stesso come radice. 
- Quindi, instrada pacchetti IP attraverso la rete, basati esclusivamente sui loro indirizzi IP; in caso di modifiche topologiche, le rotte verranno ricalcolate, utilizzando LSA aggiornati (o loro assenza); il protocollo genera quantità relativamente piccole di traffico per configurazione. 

Un router fa il broadcast di informazioni di routing a tutti gli altri router in un AS con un pacchetto detto LSA (link state advertisements) quando c’è un cambio di stato in un link e periodicamente (ogni 30 minuti). 

OSPF viaggiano incapsulati in IP (piuttosto che TCP o UDP) inoltre lo scambio di messaggi tra OSPF router (esempio LSA) 

Lo scambio tra OSPF routers ( esempio LSA) può essere autenticato (simple authentication, in cui la stessa password è usata su ogni router, o MD5 authentication). 

Simple Password: password (8 bytes) in chiaro nei messaggi LSA, facile da superare con sniffing. 

## Funzionamento spiegato in modo easy
OSPF semplicemente funziona attraverso la diffusione di annunci (LSA) che contengono informazioni di routing, è come se ogni router costruisse un albero per trovare il percorso più breve tra la varie destinazioni (avendo se stesso come radice).<br>
Sostanzialmente, in base alle informazioni che il router raccoglie dalla rete, esso costruisce la sua visione della rete e lo annuncia agli altri router creando i messaggi LSA<br>
Questi messaggi vengono di volta in volta pubblicizzati, e ogni volta che c'è un annuncio che ha un effetto sulla topologia della rete, ovviamente il router ricalcola perché cambierà la sua visione della rete e manderà fuori dei messaggi aggiornati<br>
Il router lancia l'algoritmo di dijasktra cercando di calcolare il percorso minimo rispetto tutte le sottoreti conosciute<br>
Dopodiche il router annuncia tutto quello che conosce ogni 30 minuti tramite i LSA<br>
Gli annunci viaggiano sottoforma di pacchetti IP<br>

## Possibili Attacchi OSPF
- Starvation 
  - Il traffico dati può essere reindirizzato a una parte della rete che non include la macchina di destinazione.
- Network congestion 
  - Grandi quantità di traffico vengono reindirizzate a una parte specifica della rete che non è progettata per gestire quel carico. Nei casi peggiori si può arrivare ad un "black hole" in cui un router non è in grado di gestire l'aumento del livello di traffico e cancella molti pacchetti. 
- Delay 
  - Il traffico dati destinato a un nodo viene inoltrato lungo un percorso più lungo. 
- Looping 
  - Il traffico dati viene inoltrato lungo un percorso con dei loop, in modo che i dati non vengano mai consegnati. 
- Eavesdropping 
  - Il traffico dati viene inoltrato attraverso un router o una rete che altrimenti non vedrebbe il traffico, offrendo l'opportunità di vedere i dati. 
- Partition 
  - Una parte della rete ritiene di essere partizionata dal resto della rete quando non lo è. 
- Churn 
  - Il forwarding nella rete cambia rapidamente, determinando grandi variazioni nei modelli di consegna dei dati (e influenzando negativamente le tecniche di controllo della congestione). 
- Instability
  - OSPF diventa instabile in modo che non venga raggiunta la convergenza su uno stato di inoltro globale.
- Overload
  - I messaggi OSPF stessi diventano una parte significativa del traffico trasportato dalla rete. 
- Resource exhaustion
  - I messaggi OSPF stessi causano l'esaurimento delle risorse critiche del router, come il tablespace e le code.

In modo più generale possiamo dire che le principali categorie di attacco sono: Eavesdropping, Message Replay, Message Insertion, Message Deletion, Message Modification, Man-In-The-Middle e Denial-of-Service. 

## Altri attacchi nel dettaglio 
<b>The Max Age attack:</b><br>
l'attaccante invia pacchetti LSA con il campo max age  impostato ad un'ora (3600), il router originale quindi non accetterà altri LSA fino al termine del max age, l'aggressore quindi continuerà ad interrompe continuamente il router di origine con questi  pacchetti. Questa tecnica può causare confusione nella rete e può contribuire a una condizione DoS. <br>

<b>The Sequence++ attack:</b><br>
L‘attaccante invia un LSA con un numero di sequenza maggiore, che significa che il percorso è stato analizzato più di recente, quindi è più affidabile. Il router di origine lo contesta nel processo di contrattacco inviando il proprio LSA con un numero di sequenza ancora più nuovo rispetto al numero di sequenza dell'attaccante. 
Ciò crea una rete instabile e potrebbe contribuire in modo simile a una condizione DoS.  <br>

<b>Max Sequence attack:</b><br>
Il numero di sequenza massimo 0x7FFFFFFF viene inserito da un utente malintenzionato, questo significa che non si potrà inserire un altro percorso più "aggiornato".  In alcuni casi, MaxSeq LSA non viene eliminato e rimane nel database dello stato dei collegamenti per un'ora, fornendo a un utente malintenzionato il controllo per quel periodo di tempo.  <br>

<b>The Bogus LSA attack:</b><br>
bug in un'implementazione del demone GateD, questo attacco ha causato l'arresto anomalo di gateD e ha richiesto che tutti i processi gateD venissero arrestati e riavviati per eliminare l'LSA non valido. Causa una condizione di DoS e viene utilizzato per forzare OSPF a modificare le rotte, modificando il costo del collegamento, reindirizzando così tutto il traffico di rete attraverso un host specifico. <br>

<b>Designer Router (DR) attack</b><br>
<b> Spiegazione : </b> Il router OSPF può eleggere un router come Designated Ruter (DR) e un router come Backup Designated Router (BDR). Ad esempio, sulle reti di trasmissione multicast (come le LAN) i router scelgono per impostazione predefinita un DR e un BDR, ed essi fungono da punto centrale per lo scambio di informazioni di routing OSPF. Ciascun router non DR o non BDR scambierà informazioni di routing solo con DR e BDR, invece di scambiare aggiornamenti con ogni router sul segmento di rete. Il DR distribuirà quindi le informazioni sulla topologia a ogni altro router all'interno della stessa area, riducendo notevolmente il traffico OSPF. 

<b> Attacco : </b> è possibile inviare informazioni false al DR con IP Spoofato, utilizzando il protocollo Hello, per ottenere uno dei seguenti scenari: 
- Rendere il nostro router controllato un DR / BDR. 
- Rendere un altro router (esistente) un DR / BDR falsificando i suoi messaggi Hello 
- Modificare un DR autentico in modo che non sia un DR. 
- Trasformare un router fantasma in un DR / BDR. 

L'attacco può essere sfruttato per creare un black hole: in un router fantasma viene diretto la maggior parte del traffico AS. 

Prima del fake hello : <br>
![Pre Fake Hello](/assets/sicurezza_informatica/dr-pre.png)<br>

Post del fake hello : <br>
![Post Fake Hello](/assets/sicurezza_informatica/dr-post.png)<br>
 

 