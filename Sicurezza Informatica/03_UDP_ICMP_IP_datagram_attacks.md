# UDP
Tutti i discorsi visti su TCP possono essere traslati su UDP<br>
UDP : User Datagram Protocol<br>
Protocollo di trasposto minimo
- Senza connessione
- Senza stato
- Minimo overhead
 
 UDP invia semplicemente il datagramma al processo di applicazione alla porta specificata dell'indirizzo IP<br>
 Il numero di porta di origine fornisce l'indirizzo di ritorno<br>
 Applicazioni : streaming multimediale, trasmissione<br>
 Nessun rincoscimento, nessun controllo del flusso, nessuna continuazione del messaggio<br>
 ![UDP](/assets/sicurezza_informatica/udp.png)<br>
 
 ## NTP Amplification
 Network Time Protocol (NTP) è un esempio di meccanismo di amplificazione ovvero meccanismi che consentono di creare un grosso numero di pacchetti di traffico in risposta a un singolo pacchetto.<br>
 NTP è usato per sincronizzare i clock.<br>
 Servizi accessibili al pubblico che reagiscono a un comando, un esempio di comando famoso :
 - Monlist 
   - Restituisce la lista degli ultimi 600 host
Un esempio di utilizzo : attaccante spedisce un messaggio con IP spoofato e l'NTP server risponde con la lista (grosso numero di pacchetti) allo spoofed IP
 ![NTP](/assets/sicurezza_informatica/ntp-amplification-attack.png)<br>
 Attacco di tipo reflection e amplification

 # ICMP
 Utilizzato da host e router per comunica informazioni a livello di rete 
 - Segnalazione errori : host irrangiungibile, rete, porta, protocollo non raggiungibile
- Echo richiesta / risposta (usata da ping)
| Type | Code | Description |
| ---- | ---- | ----------- |
| 0 | 0 | echo reply (ping) |
| 3 | 0 | dest. network unreachable |
| 3 | 1 | dest. host unreachable |
| 3 | 2 | dest. protocol unreachable |
| 3 | 3 | dest. port unreachable |
| 3 | 6 | dest. network unknown  |
| 3 | 7 | dest. host unknown |
| 4 | 0 | source quench (congestion control - not used) |
| 8 | 0 | echo request  (ping) |
| 9 | 0 | route advertisement |
| 10 | 0 | route discovery |
| 11 | 0 | TTL expired |
| 12 | 0 | bad IP header |

Perché vengono fatti questi ping?
- Fornire un feedback about operazioni network
  - Errore nella configurazione della rete
  - Test su host se raggiungibile o meno
  - Controllo della congestione

## Smurf DoS Attack
![Smurf](/assets/sicurezza_informatica/smurf-dos-attack.png)<br>
Eseguire un ping con indirizzo spoofato e destinazinoe del ping un indirizzo broadcast
- Ogni host della sottorete risponderà alla richiesta di ping inviando un messaggio all'indirizzo della vittima (quello spoofato)
- Il flusso di risposta ping può sovraccaricare e mettere fuori uso la macchina 

# IP Datagram
 ![IP Datagram](/assets/sicurezza_informatica/ip-datagram.png)<br>
 ![IP Datagram 2 ](/assets/sicurezza_informatica/ip-datagram-2.png)<br>

## IP Datagram Fragmentation
I frame Ethernet possono trasportare fino a 1.500 byte di dati, i collegamenti wide-area non possono trasportare più di 576 byte <br>
Funzionamento frammentazione :
- Quando viene inviato un datagramma IP grande (>1500 byte), avviene la frammentazione, ovvero viene diviso in datagrammi più piccoli che vengono fatti viaggiare sulla rete
- Quando arrivano a destinazione avviene il "Riassembly" ovvero i datagrammi più piccoli vengono riassemblati in ordine per creare il datagramma più grande

## Teadrop Attack
L'attaccante invia una serie di datagrammi che non possono combaciare correttamente.<br>
Un datagramma potrebbe dire che è la posizione 0 per la lunghezza di 60 byte, un'altra posizione 30 per 90 byte e un'altra posizione 41 per 173 byte.
Questi tre pezzi si sovrappongono, quindi non possono essere rimontati correttamente. <br>
In un caso estremo, il sistema operativo si blocca con queste unità di dati parziali che non è possibile riassemblare, portando così alla negazione del servizio.

## Ping of Death
Mandare un frammeto in cui era settato l'offeset massimo, il payload deve saturare il numero massimo di byte gestibili creando un buffer-overflow (quando il ricevitore assembla tutti i frammenti IP ricevuti finirà con un paccheto IP che è più grande di 65535 byte quindi si crea un buffer overflow)<br>
Il problema non ha nulla a che fare con ICMP, è un problema nel processo di riassemblaggio di frammenti IP, che possono contenere qualsiasi tipo di protocollo<br>
Mitigazione : verifica nel processo di riassemblaggio
- La somma dei campi di fragment offeset e lunghezza totale nell'intestazione IP di ogni frammento IP deve essere inferiore a 65535, se maggiore il pacchetto viene considerato non valido e viene ignorato

# LAND (Local Area Network Denial)
Funzionamento : <br>
- Creare pacchetto TCP SYN contraffatto 
  - Indirizzo IP dell'host di destinazione su una porta aperta inserito sia come origine che come destinazione
  - Questo fa si che la macchina risponda a se stessa continuamente in un loop infinito

Dovuta a una non corretta implementazione dei protocolli

# DNS Amplification DDoS Attack

Funzionamento : <br>
- Query DNS trasmesse tramite UDP
  - Come le query ICMP utilizzate in un attacco SMURF
- Il DNS è anche in grado di generare una risposta molto più ampia della semplice query
  - Si utilizza il comando dif per interrogare il DNS che ha un fattore di amplificazione x50 

``` bash
dig ANY website @ x.x.x.x
```
![DNS-Amplification](/assets/sicurezza_informatica/dns-amplification.png)<br>

1.54

