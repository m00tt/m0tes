# Introduzione

## Classi di indirizzo
![Classi di indirizzo](/assets/sicurezza_informatica/classi-indirizzo.png)<br>
Gli indirizzi IP vengono assegnati a classi di indirizzi IP.
| Class | Netowrk Address | Host Address | Example Address |
| ----- | --------------- | ------------ | --------------- | 
|<b>Classe A</b> <br>Address Range = 1.0.0.1 to 126.255.255.254 | I primi 8 bit definiscono il network address. L'indirizzo binario del primo ottetto inzia sempre con 0. L'indirizzo decimale ha un range da 1 a 126 (127 netowrk) | I rimanenti 24 bit definiscono l'host address | 110.160.212.156 <br> Network = 110<br> Host = 162.212.156 |
|<b>Classe B</b> <br>Address Range = 128.1.0.1 to 191.255.255.254 | I primi 16 bit definiscono il network address. L'indirizzo binario del primo ottetto inizia sempre con 10. L'indirizzo decimale ha un range che va da 128 a 191 (16000 network). 127 è riservato per il loopback testing su localhost | I 16 bit rimanenti definiscono l'host address (65000 host) | 168.110.226.155<br> Network = 168.110 <br> Host = 226.155 |
|<b>Classe C</b> <br>Address Range = 192.0.1.1 to 223.255.254.254 | I primi 24 bit definiscono il network address. L'indirizzo binario del primo ottetto inizia sempre con 110. L'indirizzo decimale ha un range che va da 192 a 223 (2 milioni di network)| Gli 8 bit rimanenti definiscono l'host address (254 host) | 200.168.198.156<br> Network = 200.168.198 <br> Host = 156 |
|<b>Classe D</b> <br>Address Range = 224.0.0.0 to 239.255.255.255 | L'indirizzo binario del primo ottetto inizia sempre con 1110. L'indirizzo decimale ha un range che va da 224 a 239 | Riservati per il multicasting | |

## Netmask
La netmask è una sequenza di 32 bit che identifica quali bit sono comuni negli IP all'interno di una LAN(sottorete).<br>
È molto comoda la notazione CIDR(Classless InterDomain Routing)
- Esempio: 159.149.30.0/24 
  - 24 bit per le sottoreti
  - 32-24 = 8 bit per gli host
In generale il numero di host della sottorete è pari a 2^y - 2 (dove y è il numero di bit per host)<br>
<b>N.B.</b>= - 2 perché il primo e ultimo indirizzo di ogni rete non sono assegnabili ad alcun host, in quanto come indirizzo broadcast<br>

- Altri esempi : 
  - 10/8 : 10.0.0.0 - 10.255.255.255
  - 172.16/12 : 172.16.0.0 - 172.16.255.255

![Netmask](/assets/sicurezza_informatica/netmask.png)<br>

Indirizzo da 32 bit in notazione /23 -> i primi 23 bit di questo indirizzo sono fissi e gli altri 9 bit sono liberi e indirizzabili per smistare il traffico a ciascun host contenuto nella  particolare sottorete<br>
Con la gestione delle maschere di sottorete possiamo più agevolmente smistare i pacchetti.<br>
Ci sono questi due livelli IP e Link che permettono in una stessa sottorete di gestire il passaggio di un pacchetto IP a un frame a livello DataLink.

# ARP
ARP è il protocollo che gestisce l'associazione tra IP e MAC Address.<br>
Il protocollo ARP(Address Resolution Protocol) connette il livello di rete con il livello dati converteno un indirizzo IP in un indirizzo MAC.<br>
Ogni nodo mantiene una tabella (ARP Cache) in cui ci sono le associazioni già note, altrimenti bisogna chiedere a tutti i nodi della rete locale chi ha un certo numero IP.<br>
Cosa succede nel momento in cui un nodo locale ha necessità di contattare un altro nodo della sottorete?
- ARP funziona inviando n messaggi Broadcast e memorizzando in cache le risposte per usi futuri. 
  - ARP Request - effettuata in Broadcast
    - Pacchetto che ha come Destination Address : 255.255.255.255
    - "Chi ha l'indirizzo IP <IP address 1>? Dillo a <IP address 2> (dillo alla macchina che ha fatto la richiesta)
  - ARP Reply - effettuata in Unicast
    - <IP address 1> is <MAC address>
Il comando per visualizzare la ARP Table (sia in Linux, sia in Windows) : 
```bash
arp -a
```

## ARP Spoofing / Cache Poisoning
Il problema del protocollo ARP è che non ci sono controlli di sicurezza ovvero:
- Chi ha l'indirizzo IP 192.168.0.2?
- Sono io 00:23:a2:d6:f2:15 (qualsiasi altra macchina può rispondere dando il proprio MAC, a questo punto verrà scritta una entry nella tabella ARP per associare IP al MAC)
- Le comunicazioni dirette a 192.168.0.2 vanno a chi riceve i frame destinati a 00:23:a2:d6:f2:15

La tabella ARP viene aggiornata ogni volta che vene ricevuta una risposta ARP
- Questo è possibile perché le richieste non vengono tracciate
- Gli annunci ARP non vengono autenticati
- Le macchine si fidano l'una dell'altra
- Una macchina maliziosa può ingannare le altre macchine

Questo cambiamento delle tabella ARP (inteso come l'inserimento di entry false nella tabella ARP)si chiama <b>ARP Poisoning</b><br>
Secono lo standard, quasi tutte le implementazioni ARP sono senza stato.
- Una cache ARP si aggiorna ogni volta che riceve una risposta ARP anche se non ha inviato alcuna
richiesta ARP!
- È possibile "avvelenare" una cache ARP inviando rispost ARP gratuite
- L'uso di voci statiche(alcune entry siano statiche ovvero non cambiabili da messaggi ARP che viaggiano nella sottorete) risolve il problema ma è quasi impossibile da gestire (in rete dinamiche non risolve il problema e non è facile da gestire in quanto renderebbe statica la rete)

![ARP Poisoning](/assets/sicurezza_informatica/ARP-Poisoning.png)<br>

Da un lato abbiamo la macchina A :
- con IP : 192.168.1.1 
- e MAC Address : 00:11:22:33:44:01
A destra abbiamo la macchina B : 
- con IP : 192.168.1.105
- e MAC Address : 00:11:22:33:44:02
Queste due macchine devono parlarsi tra di loro, nel momento in cui hanno bisogno di scambiare i dati, ciascuna macchina nella propria Cache manterrà l'associazione.<br>
Cosa può fare un attaccante?

![ARP Poisoning2](/assets/sicurezza_informatica/ARP-Poisoning-2.png)<br>

L'attacante è la macchina C :
- con indirizzo IP : 162.166.1.106
- e MAC Address : 00:11:22:33:44:03
Avvelena la cache andando a diffondere nella sottorete locale delle finte risposte ARP cosicche le cache si aggiornino, nell'esempio effettua delle risposte ARP dicendo che la macchina 105 ha come MAC quello dell'attaccante (04) e la macchina 11 ha l'indirizzo MAC dell'attaccante (04).<br>
Nel momento in cui la macchina A vuole inviare dati alla macchina B, nella cache ci sarà scritto che l'indirizzo della macchina B è associato al MAC Address dell'attaccante, quindi i dati arriveranno alla Macchina C e viceversa, e tutto questo può avvenire senza nessun controllo.

# MAC Address Flooding
A volte chiamati anche attacchi di overflow della tabella degli indirizzi MAC.<br>
Gli switch hanno una tabella dove si associa l'indirizzo MAC alla porta in cui devono smistare il pacchetto.<br>
Quando i frame arrivano sulle porte dello switch, gli indirizzi MAC di origine vengono appresi dall'intestazione del pacchetto di livello e registrati nella tabella delgi indirizzi MAC
- Se lo switch conosce già l'indirizzo MAC, esiste già una voce per l'indirizzo MAC; quindi il frame viene inoltrato alla porta dell'indirizzo MAC
- Se l'indirizzo MAC non esiste, lo switch funge da hub e inoltra il frame su ogni altra porta dello switch mentre segna e memorizza il MAC d'origine nella tabella associato alla porta

## Attacco
Le tabelle degli indirizzi MAC hanno dimensioni limitate.<br>
Il MAC flooding fa uso di questa limitazione per inviare allo swich un intero gruppo di indirizzi MAC di orgine falsi fino a quando la tabella degli indirizzi MAC dello switch non sia completamente satura e quindi non può salvare altri indirizzi MAC.
- Lo switch quindi entra in modalità fail-open, il che significa che inzia a fungere da hub
- Lo switch trasmetterà tutti i pacchetti ricevuti a tutte le macchine sulla rete.
- L'attaccante può vedere tutti i frame inviati da un host vittima a un altro host senza una voce nella tabella degli indirizzi MAC
I tool di attacco alla rete generarno circa 160000 voci MAC su uno switch al minuto



