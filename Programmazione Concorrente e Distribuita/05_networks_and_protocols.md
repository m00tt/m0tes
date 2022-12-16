# Modello client-server
Il principale modello di interazione utilizzato nelle applicazioni di rete è il modello <b>client-server</b>.

- Server: è una macchina esposta in rete (pubblica o privata) in grado di fornire un servizio
- Client: è un applicativo che vuole usufruire del servizio offerto dal server

Un client, che vuole richiedere il servizio deve connettersi al server.
Per farlo deve essere a conoscenza dell'indirizzo di destinazione e della porta da contattare (che il server deve aver messo a disposizione).

1. Il server deve, quindi, dichiarare di essere disposto a ricevere richieste di connessione da parte dei client, cioè apre la connessione in modo passivo
2. Il cliente, attraverso indirizzo IP e porta, chiede l'apertura della connessione in modo attivo

# Protocollo di comunicazione
Per <b>protocollo di comunicazione</b> si intede un insieme di regole di comunicazione che devono essere seguite in modo che due interlocutori possano comprendersi.

I protocolli per le reti di calcolatori sono organizzati secondo uno <b>stack</b>, una gerarchia: più si sale lo stack, maggiore è il livello di astrazione dei servizi offerti dal protocollo.

# Modello ISO/OSI
L'<b>Open System Interconnection</b> (stack ISO/OSI) è un protocollo di comunicazione che si suddivide in 7 layer:

| N°| Layer         | Data type | Protocol                     | 
| - | ------------- | --------- | ---------------------------- | 
| 7 | Applicativo   | data      | HTTP, FTP, DNS, SNMP, Telnet |
| 6 | Presentazione | data      | SSL, TLS                     |
| 5 | Sessione      | data      | NetBIOS, PPTP                |
| 4 | Trasporto     | segment   | TCP, UDP                     |
| 3 | Rete          | packets   | IP, ARP, ICMP, IPSec         |
| 2 | Data link     | frame     | PPP, ATM, Ethernet           |
| 1 | Fisico        | bit       | Ethernet, USB, Bluetooth     |

La comunicazione dal sender avviene scendendo lo stack ed arrivando al livello fisico, mentre il destinatario effettua il procedimento inverso.

# Protocollo IP
Ogni host collegato ad una rete basata su TCP/IP è identificato tramite un <b>indirizzo IP</b> univoco.
Esso è composto da 4 byte, suddivisi tra _netid_ e _hostid_.

Il servizio realizzato da IP (Internet Protocol) è la consegna di un datagramma, che è un pacchetto di bit contenente:
- dati principali
- metadati

IP fornisce un servizio _connectionless_ e _non affidabile_. Esso non assicura:
- la consegna
- l'integrità
- la non-duplicazione
- l'ordine di consegna
dei datagrammi.

Il protocollo IP specifica il formato esatto di tutti i dati, l’insieme di regole che inglobano l’idea di consegna non affidabile, e fornisce le funzioni di instradamento (routing). Queste ultime sono realizzate in base agli indirizzi IP: ogni gateway dispone di opportune tabelle di routing che, se il destinatario non è direttamente connesso al gateway, permettono di determinare verso quale altra macchina instradare il messaggio in modo da farlo avvicinare alla destinazione, che raggiungerà mediante una serie di salti (hop).

## Pacchetto IP

![IP packet](/assets/programmazione_concorrente_e_distribuita/ip_packet.png)

(dimensioni in bit)

- Versione: 4 per IPv4
- Lunghezza totale: dimensione complessiva del pacchetto (in byte)
- Id del datagramma: un identificatore univoco di ciascun pacchetto, che permette al ricevente di distinguerlo dagli altri
- TTL: numero massimo di hop che il pacchetto può fare prima di essere considerato perso
- Protocollo: protocollo di livello superiore incapsulato nella parte di dati del pacchetto (es. TCP)
- Checksum dell'header: codice che consente di rilevare errori di trasmissione

Come detto precedentemente, questo datagramma viene poi incapsulato in un frame del protocollo di livello inferiore.

![IP into](/assets/programmazione_concorrente_e_distribuita/ip_into.png)


# Protocollo UDP
<b>UDP (User Datagram Protocol)</b> si appoggia al protocollo IP, dal quale eredità le caratteristiche _connectionless_ e _non affidabile_.<br>
In aggiunta, però, UDP fornisce:

- Un servizio (facoltativo) di protezione dagli errori di trasmissione
- Il concetto di <b>porta</b>, che permette di distinguere più sorgenti/destinazioni per uno stesso indirizzo IP

## Pacchetto UDP

![IP into](/assets/programmazione_concorrente_e_distribuita/udp_packet.png)

Oltre ai numeri di porta (che sono di 16 bit ciascuno), l’header UDP contiene solo le seguenti informazioni di servizio:
- lunghezza del messaggio (16 bit), misurata in byte
- checksum (16 bit), per il rilevamento di errori nella trasmissione (facoltativa: se non usata, questo campo contiene il valore 0)

Non è invece presente l’indirizzo IP, dato che esso è già presente nell’header del datagramma IP, all’interno del quale i pacchetti UDP vengono incapsulati per essere trasmessi.

# Protocollo TCP

TCP (Transmission Control Protocol) è un protocollo <b>connection-oriented</b>: esso realizza una connessione <b>full duplex e affidabile</b> tra due applicazioni, identificate da un indirizzo IP e una porta.

## Pacchetto TCP

![TCP Packet](/assets/programmazione_concorrente_e_distribuita/tcp_packet.png)

Le informazioni principali contenute nell'header sono:
- Porta sorgente e destinazione
- Numero di sequenza: indica la posizione dei dati del pacchetto nello stream di comunicazione
- Numero di acknowledgement: indica il numero del byte che il mittente di questo pacchetto si aspetta di ricevere dal destinatario nel prossimo pacchetto che quest'ultimo invierà
- HLEN: lunchezza dell'header
- Flag: usati per gestire le fasi di apertura e chiusura delle connessioni e contrassegnare i messaggi che fanno parte della comunicazione vera e propria. Ogni flag ha un nome (SYN, ACK, FIN, ...)

## Sessione TCP
Una sessione di comunicazione TCP prevede tre fasi:
- setup
- scambio dati
- shutdown

### Setup
Un server, in ascolto su una determinata porta, riceve una richiesta di connessione da un client. Tale richiesta è composta da un segmento TCP con _flag_ SYN e contiene un numero casuale _c_ come numero di sequenza.

![TCP Packet SYN](/assets/programmazione_concorrente_e_distribuita/syn_client_to_srv.png)

Il server risponde con un segmento con i flags SYN e ACK a 1, nel quale:

- il numero di sequenza è un altro numero casuale, _s_
- il numero ack è il sequence number passato dal client incrementato di 1 (_c_ + 1)

![TCP Packet SYN](/assets/programmazione_concorrente_e_distribuita/syn_srv_to_client.png)

Infine, il client manda un segmento con numero di sequenza uguale a _c_ + 1 e numero di ack _s_ + 1, con il flag ACK a 1

![TCP Packet SYN](/assets/programmazione_concorrente_e_distribuita/ack_client_to_srv.png)


### Scambio dati
Una volta instaurata la connessione, avviene il vero e proprio scambio di dati.

In ogni pacchetto, il client e il server inseriscono:
- il proprio numero di sequenza incrementato del numero di byte trasmessi in precedenza
- l'ack del pacchetto precedente, dato dal numero di sequenza di tale pacchetto incrementato del numero di byte che esso conteneva

![TCP Packet Data Exch](/assets/programmazione_concorrente_e_distribuita/eg_data_exch.png)

Inoltre, il ricevente indica una finestra di ricezione, cioè quanti pacchetti il mittente può spedire prima di dover aspettare di ricevere un acknowledgement. Questo serve a evitare che il ricevente venga “inondato” di una quantità di messaggi che non è in grado di gestire.

### Shutdown
Il client (o il server) può indicare la fine della trasmissione con un pacchetto marcato dal bit di FIN. Il server (o il client) risponderà con un segmento di acknowledgement, e, prima o poi, indicherà che anch’esso ha finito di trasmettere, mandando a sua volta un pacchetto FIN, e ricevendo come risposta un ultimo ACK. Allora, il circuito virtuale verrà interrotto.

![TCP Packet Shutdown](/assets/programmazione_concorrente_e_distribuita/tcp_shutdown.png)
