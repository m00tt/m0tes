# Introduzione

## Protocolli
<b>Definzione : </b>  un protocollo definisce il formato e l'ordine dei messaggi scambiati tra due o più entità comunicanti, nonché le azioni intraprese sulla trasmissione e/o la ricezione di un messaggio.
<b>Esempio : </b> Un protocollo di comunicazione descrive come i computer trasmettono i dati attraverso la rete.

## Layering
La rete è organizzata tramite una struttura layered (a strati).<br>
<b> Layering Architecture </b>: per far fronte alle complessità, le funzioni sono suddivise in livelli.

<b> Network Stack </b>: i protocolli sono organizzati in uno stack di livelli, i dati vengono passati dal livello più alto a quello più basso.

<b> Data Encapsulation </b>: i protocolli e gli standard di ogni livello eseguono funzioni specifiche e inseriscono queste informazioni nei dati, prima di passare al livello sottostante.


### Motivi del layering
- La <b>struttura esplicita</b> consente l'identificazione, la relazione tra i pezzi del sistema complesso
  - <b>Modello di riferimento a layer</b>
- La modularizzazione facilita la manuntenzione, l'aggiornamento del sistema
  - Cambio di implementazione del servizio di livello trasparente al resto del sistema (ad esempio, la modifica della procedura di accesso non influisce sul resto del sistema)

# Modelli ISO/OSI e TCP/IP

## ISO/OSI Model VS TCP/IP Model
![ISO/OSI vs TCP/IP](/assets/sicurezza_informatica/model.png)

### OSI Protocol Stack
![OSI Protocol Stack](/assets/sicurezza_informatica/osi-protocol-stack.png)

### Formato dei dati
![Formato dei dati](/assets/sicurezza_informatica/data-format.png)

### TCP/IP Model Layer
| Layer   | Layer Name           | Function                                                         | Protocols-Standards |
| ------- | -------------------- | ---------------------------------------------------------------- | --------------- |
| Layer 4 | Application Layer    | Equivalente ai livelli Application, Presentation e Session del modello OSI | SMTP, POP, HTTP, FTP |
| Layer 3 | Transport Layer      | Simile al livello di trasporto nel modello OSI                   | TCP, UPD            |
| Layer 2 | Internet Layer       | Svolge le stesse funzioni del livello Network del modello OSI    | IP, ARP, RARP, ICMP |
| Layer 1 | Network Access Layer | Combina il livello Data-Link e il livello Fisico del modello OSI | IEEE 802.3, IEEE 802.11b, IEEE 802.11g |

### Data Encapsulation
Il processo di incapsulamento si verifica nel momento in cui un Computer (Host A) vuole inviare dati all’Host B, prima deve impacchettarli attraverso il cosiddetto processo di incapsulamento, aggiungendo Header, Trailer ed altre informazioni<br>
![ISO/OSI Encapsulation](/assets/sicurezza_informatica/iso-osi-encapsulation.png)<br>
![TCP/IP Encapsulation](/assets/sicurezza_informatica/tcp-ip-encapsulation.png)

### Network Stack Security
![Network Stack Security](/assets/sicurezza_informatica/network-stack-security.png)

## Link Layer : Ethernet
Caratteristiche del protocollo:
- Carrier Sense, Multiple Access with collision detection
- Indirizzi da 48 bit
- MTU (Maximum Transmission Unit) : 1500 bytes
- Dimensione minima : 46 byte (padding)
Dal punto di vista della sicurezza : 
- Tutti i nodi connessi (LAN) ricevono tutti i frame ma scartano quelli non diretti a loro.

## Network Interface
Come per esempio le Ethernet Card e i WIFI Adapter.
Caratteristiche:
- Un computer può avere più interfacce di rete
- I pacchetti vengono trasmessi tra le interfacce di rete
- La maggior parte delle rete locali (comprese ETH e WIFI) trasmettono frame
- In modalità normale, ogni interfaccia di rete riceve i frame previsti
- Lo sniffing del traffico può essere eseguito configurando l'interfaccia di rete per leggere tutti i frame(Modalità Promiscua)

## Sicurezza con gli switch
Gli switch utilizzati per creare sottoreti e smistano i frame, a seconda dell'indirizzo gestiscono il traffico di diversi pacchetti
- in entrata andando a leggere il singolo frame
- in uscita andando a rilanciare il frame sulla particolare sottorete al quale è designato
Uno switch è un disposito di rete che opera al Link Layer, esso è caratterizzato da più porte, ciascuna collegata a un dispositivo.
Le operazioni che effettua sono le seguenti:
- Memorizza l'indirzzo MAC di ogni computer ad esso connesso e inoltra i frame solo al computer di destinazione
- La separazione dei collision domain è solo logica
  - La sepazione è ottenuta tramite una CAM (Content Addressable Memory) che contiene le associazioni MAC-Porta dello switch.
MAC Flooding :
Il MAC flooding consiste nel mandare tante associazioni nuove tra MAC e porta uscita dello switch in modo tale da saturare lo switch stesso.
Se non ci sono controlli sullo spazio utilizzato dalla tabella CAM essa cresce occupando tutto lo spazio disponibile, quando succede diventa satura e non viene utilizzata, lo switch va in modalità HUB, e il pacchetto non viene smistato su una sola porta (quella designata) ma su tutte le porte, in parole povere non c'è più controllo sul traffico.
- Se la tabella è generata dinamicamente (basta attaccare i nodi allo switch e l'amministratore divide i collision domain per porta) è possibile saturarla.
- Permette di violare i collision domain impostati dallo switch (pacchetto diffuso su tutte le sottoreti)
- La tabella satura non viene utilizzata

## MAC Address e Spoofing
Le schede di rete sono identificate da un numero seriale, che viene utilizzato come indirizzo all'interno della rete LAN, formato da 48 bit(scritti usando una notazione esadecimale)di cui i primi 24 identificano il produttore.

### Affidabilità del MAC Address
- L'indirizzo MAC può essere riconfigurato dal software del driver dell'interfaccia di rete
- Avendo i privilegi adeguati, e quasi sempre possibile (e facile) cambiare l'indirizzo MAC usato nella produzione dei frame
- Quindi conoscendo l'indirizzo MAC di una macchina assente, è immediato impersonarla.

### Attacco Spoofing MAC Address
Un attacco di spoofing MAC semplicemente consiste nell'impersonare un'altra macchina.
- Scoprire l'indirizzo MAC di un'altra macchina (macchina vittima)
- Riconfigurare l'indirizzo MAC della macchina attaccante
- Spegnere o scollegare la macchina di destinazione (macchina vittima)

### Contromisure
- Bloccare la porta dello switch quando la macchina è spenta o scollegata
- Disabilitare gli indirizzi MAC duplicati

## IP (Internet Protocol)
Caratteristiche Internet Protocol : 
- Senza connessioni
- Protoccolo inaffidabile, lavora in "best effort"
- Utilizza indirizzi numerici per il routing
- Ogni nodo è identificato da un numero IP da 32 bit (IPV4), tradizionalmente scritto come 4 ottetti (notazione in base 256)
- L'instradamento (routing) avviene tramite nodi gateway che si interfacciano con due o più LAN






