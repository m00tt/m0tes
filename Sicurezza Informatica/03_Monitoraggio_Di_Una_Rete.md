# Attività di monitoraggio di una rete
L'attività di monitoraggio di una rete è essenziale per la gestione della rete stessa.<br>
Monitoraggio significa supervisionare cosa sta accadendo alla nostra rete.<br>
I tool di monitoraggio della rete sono essenziali in quanto ci aiutano a capire ad esempio :
- Se ce qualche anomalia nella rete 
- Se ci sono errori d'instradamento dei pacchetti
- Se c'è qualche attacco perche in quanto vengono registrate molte richieste 
- Controllare le richieste di autorizzazioni

Dal punto di vista della sicurezza sono utili per :
- Capire se sta succedendo qualcosa di strano, rilevando traffico normale 
- Tracciare un log per capire ad esempio come si è svolto un attacco, qual'è stata la sorgente ecc

# Tool Overview
Ci sono molti tool che permettono di fare monitoraggio, i primi tool sono tool a riga di comando.<br>
Questi tool si mettono in ascolto sull'interfaccia di rete, catturano ogni singolo pacchetto che arriva e permettono di analizzarne il contenuto
## Tcpdump
- Tool a riga di comando basato su unix utilizzato per intercettare pacchetti
- Include il filtraggio : è possibile filtrare i pacchetti d'interesse

Command:
``` bash
tcpdump 
```
Opzioni più utilizzate:
- -i interface : utilizzata per selezionare l'interfaccia che si vuole monitorare
``` bash
tcpdump -i eth0
```
- -w nomefile: utilizzato per salvare l'output in un file
``` bash
tcpdump -i eth0 -w log.txt
```

## TShark
- Tool molto simile a Tcpdump  in quanto oltre a effettuare monitoraggio anche il comportamento i flag sono molto simili
- Anche TShark è un tool da riga di comando
Command:
``` bash
tshark
```
Opzioni più utilizzate:
- -D : visualizza le interfacce disponibili da monitorare
``` bash
tshark -D 
```
- -i interface: utilizzata per selezionare l'interfaccia che si vuole monitorare
``` bash
tshark -i eth0
```
- -f port: utilizzata per filtrare una porta specifica
``` bash
tshark -i eth0 -f "port 80"
```

## Wireshark
- Wireshark è semplicemente l'interfaccia grafica di TShark
- Wireshark è un packet sniffer, è uno strumento open-source<br>

![Packet Sniffer](/assets/sicurezza_informatica/packet-sniffer.png)<br>
Wireshark nella foto è il packet sniffer formato dai tool :
- Packet Analyzer
- Packet Capture (pcap)
Il compito di questi tool è :
- Catturare ogni frame che arriva dal livello link (ogni frame che esce dalla macchina che si sta analizzando)
- Lo gestisce in formato pcap, quindi le tracce (insieme di pacchetti catturati) sono messi insieme nel formato pcap
  - Il file pcap contiene una sequenza di pacchetti che sono stati catturati al livello link
- Vengono infine analizzati, quindi decodificato il contenuto rendendo più leggibili i dati contenuti nel pacchetto

### Interfaccia di Wireshark
![Interfaccia Wireshark](/assets/sicurezza_informatica/wireshark-interface.png)<br>
Esempi di filtri utilizzabili : 
| Filtro | Spiegazione |
| ------ | ----------- | 
| ip.src == <address 1> | Monitorare per una certa sorgente |
| ip.addr == <address 1> &&  ip.addr == <address 2> | Filtrare pacchetti che hanno determinati indirizzi IP |
| tcp.port == <numero 1> \|\| tcp.port == <numero 2> | filtrare per una porta o per un altra |
| tcp.dstport == <numero 1> | Filtrare porta di destinazione |

Grazie ai filtri è possbile rendere più semplice l'analisi del traffico rete in live o a posteriori su un file di log salvato

### TCP Stream
Wireshark dà la possibilità di seguire TCP Stream.<br>
Quando due host si parlano, all'inizio avviene il TCP Handshake e dopo avviene lo scambio dati, questo in Wireshark viene denominato TCP Stream (dall'handshake fino allo scambio di dati)

# Esercizio
<b>Spiegazione testuale: </b>

- Inserisco un URL nel browser
  - Voglio iniziare una conversazione HTTP con la porta 80 dell'URL che ho messo
  - Prima cosa da capire è qual'è l'IP -> il browser prepara una richiesta al DNS per capire l'IP del sito a cui mi sono collegato
  - Una volta recuperato l'IP avviene il TCP Handshake 
    - Il mio PC manda un SYN
    - Ricevo un SYN/ACK dal sito
    - Il mio PC manda un ACK
  - Dopo che ho fatto l'handshake il mio PC manda la richiesta HTTP
  - E il server risponde

<b>Passaggi Wireshark: </b>

1. Faccio partire il monitoraggio su Wireshark filtrando per il protocollo HTTP
2. Inserisco il link nel browser
3. Fermo il monitoraggio
4. Vedremo una richiesta GET da parte del mio Browser al sito
   - All'interno dei dettagli di questa richiesta troviamo :
     - A livello HTTP vedremo : 
      - La richiesta GET
      - Host destinatario della mia richiesta
      - User-Agent
      - In quale frame si trova la risposta
      - Ecc.
    - A livello TCP vedremo : 
      - Com'è composto il pacchetto TCP
      - Porta origine, Porta di destinazione
      - Sequence Number
      - ACK Number
      - i FLAG impostati
      - Ecc.
    - A livello IP (internet) vedremo : 
      - IP Sorgente, IP destinatario
      - Ecc.
5. E sotto vedremo una risposta del sito (che è la pagina html mostrata dopo aver inserito il link)
   - Oltre a tutti i livelli descritti sopra avremo il livello chiamato : "Line-based text data":
     - Dove al suo interno vedremo il payload del pacchetto (vedremo l'html del sito)
   - A livello HTTP :
     - Troviamo la risposta HTTP ad esempio 200 che significa che è andata a buon fine o 404 not found
     - Server : dove troviamo informazioni riguardo al server come versione, cosa gira sopra, moduli attivi, ecc, <b>IMPORTANTI</B> perché un attaccante potrebbe andare a vedere se la versione dei moduli caricati sul server, ad esempio SSL, se è affetto da qualche bug (va a vederlo all'interno di CVE vulnerability)

50.01





