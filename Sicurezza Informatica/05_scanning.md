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




