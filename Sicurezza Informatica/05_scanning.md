# Network Scanning
Scanning : Ricognizione della rete<br>
Lo scanning è un attività di analisi preliminare che viene fatta per capire varie informazioni (ad esempio il tipo di macchina che c'è dall'altra parte, il sistema operativo, tipo di webserver ecc)<br>
Obiettivi : 
- Riconoscere i servizi UDP and TCP disponibili
  - Quando si effettua un scanning per vedere le porte attive su una determinata macchina si parla di port scanning
    - Quali servizi in esecuzione su quale porta
    - Quali utenti hanno accesso ai servizi
    - Esame di anonymous logins
    - Esame dei servizi di autenticazione
- Riconoscere i sistemi di filtraggio usati trga l'utente e l'host target
- Determinare il sistema operatvio esaminando le risposte IP
- Network e port scanning possono essere rilevati!
  - Ricadute legali

## Scenari
//add Image<br>
Lo scenario tipico è il seguente ovvero firweall davanti ai server e si vuole capire le caratteristiche dei server (schema a sinistra)<br>

## Scanning tipi e approci
Classificazione Port Scanning:
- Verticale : rispetto a una sottorete vado in profondità della rete
- Orizzontale : rispetto a una sottorete vado allo stesso livello per trovare le macchine attive
- Ibridi : Mischio entrambi gli approcci

Approcci :
- Scanning passivo : mi metto solamente in ascolto, monitoro il traffico, vedo quali sono le reti che orginano il traffico o che rispondono
- Scanning attivo : io stesso interrogo le diverse macchine e vedo se rispondono o meno 

Tipi di scanning : 
//add Image<br>


//add Image Schema<br>

Come fa il gestore della rete a capire se c'è un port scanning in atto?<br>
- Vedo se magari c'è un IP che sta interrogando più macchine

## Natura del Cyber Scanning
- Active scanning : trasmissione di pacchetti probe
  - Probe packets possono essere generici o specifici per un particolo protocollo o mirati per particolai applicazioni
- Passive scanning : osservazione del traffico generato da client e servers
  - con mezzi hw o tool sw
  - per TCP, c'è bisogno di catturare i messaggi di setup di connessione TCP
    - Il completamento di un 3 way handshake indica che un servizio è disponibile

//add Image

## Scopi dello scanning
- Wide Range Scanning
  - Scanning automatica e veloce di un blocco di indirizzi
  - Richiesta poca interazione umana
  - Esempio : 
    - auto-rooter
    - worm propagation
    - Fanno questa scansione perché vogliono capire le potenziali vittimi per la propogazione del worm o malware
- Target-Specific scanning
  - Stealthy
  - Indirect scanning
  - Botnet scanning
  - Low and slow scanning

## Metodo
- Single source scanning
  - operata da uno (source) verso molti (targets)
  - Tipi di scansione :
    - vertical scan : consiste in un port scan di alcune o tutte le porte di un singolo computer
    - horizontal scan : scansiona una singola porta fra molti indirizzi IP
    - strobe scan : scansone molte porte fra molti indirizzi IP
    - block scan : scansiona tutte le porte su molti indirizzi IP
- Distribuited scanning
  - Molteplici sistemi agiscono in unione strategica per scansionare una rete o un host
  - Riduce la traccia lasciata da uno scanning di un singolo sistema e diminuisce la possibilità di essere scoperti.

## Risultati di una scansione
Il risultati della scansione di una porta :
- Aperta (accept)
  - Il target ha risposto indicando che un servizio è in ascolto su quella porta
- Chiusa (denied)
  - Il target non ha risposto, le connessioni alla porta saranno rifiutate
- Bloccata/filtrata (dropped/filtered)
  - Un firewall o altro bloccano l'accesso alla porta impedendo di individuarne lo stato

In realtà ci posson essere più stati : 
- Open
- Closed
- Filtered
- Unfiltered
- Open | filtered
- Closed | filtered

# Scroprire gli host : ARP scan
Effetuare una scansione utilizzando le richieste ARP (chi è che ha questo indirizzo IP?)<br>
Ciclo for in cui effettuo la richiesta su tutte le macchine di una sottorete (funziona solo in sottorete locale e come dest si usa un indirizzo di broadcast)<br>
//add Image<br>

# Scoprire gli host : ICMP
Stessa cosa la possa fare con i ping<br>
Misura il tempo in ms perché i pacchetti ICMP raggiungoano un dispositivo di rete<br>
Pacchetto ICMP di tipo echo request e si rimane in attesa di un pacchetto ICMP di tipo echo reply in risposta.<br>
Spesso i pacchetti vengono filtrati<br>
Oltre al ping classico si possono sfruttare messaggi ICMP più esotici : 
- ICMP timestamp
- ICMP address mask

# Scoprire gli host con TCP/IP/UDP
Come riconoscere se l'host è up o down, quindi se è raggiungibile o meno<br>
- TCP SYN PING
  - Si manda un TCY SYN sulla porta 80 dell'host remoto, se risponde qualcosa significa che è il server up (se rispode SYN/ACK server attivo e porta attiva, se risponde RST server attivo e porta chiusa)
- TCP ACK Ping
  - Si manda un pacchetto ACK su una porta TCP (esempio 80)
  - Viene generato un RST di risposta
  - Vantaggio rispetto a usare il SYN ping : 
    - Se davanti all'host c'è un firewall risuciamo a superarlo (se non è un firewall stateful)
- UDP ping
  - Manda un pacchetto UDP vuoto ad una data porta (di default 31338, una porta a caso preferibilmente non "aperta")
  - L'host dovrebbe rimandare indietro un ICMP port unreachable rivelando il fatto di essere up
- IP protocol ping
  - In ipv4 un datagramma può contenere diversi protocolli di livello 4 : TCP, UDP, ICMP, ipsec, IGMP, IP-in-IP..
  - Il protocollo è indicato da un numero di protocollo (simile al numero di porta) nell'header IP
  - Per standard se mandiamo i pacchetti su un protocollo non abilitato, la risposta è : ICMP protocol unreachable
  - Quindi mandiamo richieste su protocolli non abilitati e otteniami risposte

# Scansione delle porte
Una connessione è identifica dalla quintupla : 
- SrcIp, SrcPort, DstIp, DstPort, Protocol <br>
Caratteristiche porte : 
- Le porte sono a 16 bit, vanno da 0 a 65535
- Porte note : 22,80,25..
- Porte registrare : da 1024 a 49151
- Porte dinamiche e private : da 49152 a 65535
//addImage<br>

# Specifiche TCP : RFC 793
1. Un segmento in arrivo con il flag RST viene sempre scartato senza alcuna risposta.
2. Se una porta è nello stato <b>closed </b>
  - Se il segmento in arrivo NON ha il flag RST attivo allora viene inviato come risposta al mittente un pacchetto con il flag RST
3. Se una porta è nello stato <b>listen </b>
  - Se il segmento in arrivo contiene un ACK allora viene risposto un RST
  - Se il segmento in arrivo contiene un SYN allora viene inviato come risposta al mittente un pacchetto con il flag SYN e ACK
  - Se nessuno dei precedenti è vero allora il segmento viene scartato senza risposta.

Rispetto a queste regole sono stati creati dei meccanismi di scansione : <br>
//add Image<br>

# TCP connect scan
Questo tipo di scan fa uso della syscall <b>connect()</b> da utente non privilegiato<br>
Il connect scan non solo impiega più tempo, ma rischia anche di far loggare la connessione<br>
//add Image<br>

## TCP connect (Open) scan
//add Image
Se sto cercando di stabile una connessione con il server, quello che faccio è :
- Inviare un SYN sulla porta 80
- La ragola vista prima dice che :
  - Se porta chiusa = risposta RST
  - Se porta aperta = risposta SYN/ACK
- Se non ricevo risposta significa che c'è un firewall che mi blocca la connessione
- Se ricevo ICMP unreachable significa che c'è un firewall che mi blocca la connessione

Dato che la connessione si conclude lascio sicuramente una traccia.

# TCP SYN (Half-Open) Scan
//add Image
Il funzionamento è uguale a prima MA se ricevo un SYN/ACK allora invio un pacchetto RST per non concludere la connessione e quindi non lasciare tracce

# SYN vs Connect Scan
SYN :
- port scanner generare pacchetti raw IP e analizza le risposte
- "half-open scanning", la connessione non è completamente "open" TCP
- SYN scan ha il vantaggio che la connessione al servizio non si conclude, approccio meno intrusivo

Connect()
- Usa le funzioni dell'OS
- Stabilita una full TCP connection
  - loggata

# Stealth Scan - SYN/ACK
Scan che usano il SYN flag sono facilmente rilevabili da IDS<br>
Il filtraggio si evita usando altri flags<br>
Si usa inverse mapping per determinare le porte open<br>
Una tecnica potrebbe essere anziché spedire SYN spediscon SYN/ACK<br>
Per una porta chiusa, il target risponde con RST flag<br>
Per una porta open, nessuna risposta
- ICMP richiesono solo SYN flag per iniziare una connessione<br>
A causa di packet loss, si possono avere falsi posiviti.<br>
Se il server è stato raggiunto: il meccanismo di filtraggio del server non è in grado di distinguere tra : 
- pacchetto di ACK isolato
- pacchetto di ACK inserito all'interno di una sessione TCP
Quindi firewall non maniente lo stato della connessione (stateless)<br>
//addImage<br>

# Stealth Scan - ACK
Non è mirato a verificare su una porta è open<br>
Si usa per determinare se c'è un firewall<br>
- Si manda un pacchetto TCP ACK alla porta specifica
- Lo scan pu essere invisibile quando combinato con altro traffico
  - Non si apre una connessione

- TCP RST Response -> unfiltered
- Nessuna risposta -> Potrebbe essere filtered / Potrebbe essere aperta
- ICMP unreachable -> filtered

//addImage <br>

# Window Scan
Windows scan è simile a un ACK scan (manda ACK comunque)<br>
- Sfrutta un dettaglio implementativo
- Per differenziare porte open da closed, si esami il campo window del pacchetto RST
- Su alcuni sistemi, porte open usano un valore positivo per la window size
- Porte closed hanno valore zero
- Invece di listare come unfiltered quando si riceve un RST
- Window scan restituisce open o closed a seconde del valore TCP Window

Sistemi che non supportano questo meccanismo restituiscono closed.

- TCP RST response con window non-zero -> open
- TCP RST response con window zero -> closed
- Nessuna risposta -> filtered
- ICMP unreachable error -> filterd

//add Image <br>

# TCP : FIN, NULL, Xmas
- Nel FIN scan lo scanner pacchetti isolati con il flag FIN a 1
- Nel Xmas scan lo scanner pacchetti isolati FIN, URG e PSH a 1
- Nel Null Scan lo scanner pacchetti isolati con tutti i flag a 0
- Se la porta è open i pacchetti vengono scartati senza alcuna risposta
- Se la porta è closed il server risponde con un RST

- Nessuna risposta ricevuta -> open | filtered
- TCP RST Packet -> closed
- ICMP unreachable -> filterd

scavalcano alcuni firewall non stateful che filtrano i SYN o ACK<br>
Alcuni OS(windows, cisco..) non rispettano lo standard<br>
Vantaggi : no TCP sessions, no application log
//add Image <br>

# Fragmentation Scan
Si modificano i TCP stealth scan (SYN,FIN,Xmas,NUll) per usare piccoli pacchetti frammentati (IP datagrams)<br>
- Vantaggi : si aumenta la difficoltà del detection e blocking del pacchetto
- Svantaggi : 
  - Non funziona su tutti gli OS, può mandare in crash alcuni firewall/sniffers
  - Lento
  - Non affidabile per packet loss

# Idle Scan
Tecniche che si appoggia su uno zombie (host che risponde alle richieste ma non particolarmente attivo).
Mandare dei pacchetti spoofati e utilizzare la macchina zombie per capire se un determinato servizio / porta è attivo su un server target.
Nell'IDLE Scan lo scanner non invia direttamente pacchetti alla vittima ma utilizza come intermediario un host (zombie)<br>
Usa indirizzi spoofed e il campo ID nell'header IP.<br>
- Campo ID è uguale per ogni frammento appartenente allo stesso pacchetto mentre è solitamente incrementato di uno per ogni diverso pacchetto.

- Requisiti per la macchina zombie:
  - Zombie deve essere idle
  - In modo da mantenere consistenti gli IP identification frames durante lo scan (IPID = sequence number)

//addIMage <br>

La sorgente (attaccante) manda un SYN/ACK allo zombie host e aspetta un RST come risposta con IPID, questa iterazione tra attaccante e zombie viene fatta l'attaccante riesce a capire il sequence number che viene rimandato indietro dallo zombie, importante perché riuscirò a capire cosa sta succedendo alla macchina vittima<br>
Esempio l'attaccante vuole vedere se c'è un determinato server attivo sulla macchina vittima, non sarà lui a testare la porta 80 della vittima ma manderà un pacchetto spoofato con la SYN sulla porta 80, quindi richiesta di inizio connessione, con l'indirizzo spoofato della macchina zombie<br>
Il sorgente esegue un Half-Open scan, usando l'indirizzo spoofed IP dello zombie, con target la vittima.<br>
Se la porta è open, la vittima risponderà allo zombie con SYN/ACK<br>
Lo zombie, non aspettando un SYN/ACK (non ha mandato un SYN, quindi non ha inziato una connessione), risponderà con un RST e aumenterà di 1 il IPID<br>
La sorgente rispedisce un SYN/ACK allo zombie<br>
Se l'IPID è stato aumentato, allora la sorgente deduce che la porta della vittima è open on the destination target, altrimenti la porta è closed.

//add Image

- Vantaggi :
  - Nessuna iterazione tra macchina attaccante e target
  - Attaccante conclude che guardando solo il seq number iniziale e finale ricevuti dalla macchina zombie, se una porta sulla vittima è attivo o meno (se il seq number è avanzato)

# UDP Scan
Nel UDP scan lo scanner genera pacchetti UDP con 0 byte di dati.
- Una qualcunque risposta UDP (Insolito) -> open
- Nessuna risposta ricevuta -> open | filtered
- ICMP port unreachable -> closed
- Altri errori ICMP -> filtered

Non essendoci quai masi nessuna risposta, è un tipo di scan quasi sempre lento<br>
Si spediscono pacchetti application-specific UDP, sperando di generare una risposta<br>
- esempio, send a dns query sulla porta 53, verrà resituta una risposta, se il server DNS è presente

La scansione è limitata a porte per cui una specifica applicazione è disponibile

# Protocollo FTP
//add Image <br>

## FTP Bounce Scan
Simile a IDLE Scan<br>
FTP server agisce come uno zombie<br>
In FTP active mode con il comando PORT si può indicare su quale IP e porta ricevere la risposta<br>
Se il server non riesce a collegarsi da un errore sulla connessione TCP
- Porta chiusa

Se il trasferimento riesce
- Porta aperta

//add Image

- Vantaggi 
  - Usa lo standard FTP per il suo compito
  - Stealth : IP sorgente nascosto, la vittima vede il server FTP
- Svantaggi 
  - Solo su porte TCP
  - Lento
  - Lascia tracce su server FTP

Possibili contromisure : 
- Impedire che l'indirizzo IP specificato dal comando PORT sia diverso da quello dell'FTP Client
- Impedire che il numero di porta sia < 1023
  - evitare attacchi a servizi standard quali SMTP, POP 3, ecc..


11.10


