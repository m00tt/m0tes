# DHCP
DHCP sta per Dynamic Host Configuration Protocol<br>
Obiettivo : 
- Permettere di ottener dinamicamente un indirizzo IP dal network server quando un client si unisce alla rete
  - Può rinnovare il permesso di un indirizzo in uso 
  - Permette il riuso di indirizzi
  - Supporta mobile users che vogliono unirsi alla rete (per tempi brevi)

DHCP overview:
- Host broadcast "DHCP discover" msg
- DHCP server responds with "DHCP offer" msg
- Host requests IP address : "DHCP request" msg
- DHCP server sends address : "DHCP ack" msg

![DHCP Scenario](/assets/sicurezza_informatica/dhcp.png)<br>

## DHCP : more than IP addressess
DHCP oltre a restituire un indirizzo IP sulla sottorete può assegnare : 
- indirizzo del router più vicino per il client
- Nome e indirizzo del DSN server
- Network mask (indica la parte di rete dell'indirizzo)

## DHCP : esempio
Se un dispostivo vuole collegarsi alla rete ha bisogno di un indirizzo IP
![DHCP Esempio](/assets/sicurezza_informatica/dhcp-es.png)<br>

- Richiesta DHCP incapsulata in UDP, incapsulata in IP, incapsulata in 802.1 Ethernet
- Ethernet frame broadcast on LAN, ricevuto dal router che dove viene eseguito il DHCP server
- Ethernet demuxed to IP demuxed, UDP demuex to DHCP
- Incapsulamento del DHCP server,frame inoltrato al client, demuxing dal DHCP al client
- Client ora conosce l'IP address, nome e IP address del DNS server, IP address del router

# Vulnerabilità DHCP
Le vulnerabilità di questo protocollo dipendono dal fatto che lavora a livello di rete locale in cui i nodi condividono il mezzo trasmissivo e sono «identificati» dal MAC (facilmente modificabile). 

Il client avvia il processo inviando un messaggio Discovery agli indirizzi di broadcast.  

Source IP: 0.0.0.0 

Destination IP: 255.255.255.255 

Source MAC: The MAC Address of the sender 

Destination MAC: FF:FF:FF:FF:FF:FF 

<b>Primo problema:</b> se un utente malintenzionato è collegato alla rete, il PC dell'aggressore riceverà la trasmissione. 

Lo switch riceve il broadcast e la inoltra a ogni porta della stessa VLAN. 

Il server DHCP riceverà il messaggio Discovery e risponderà con un'offerta, se il server ha un IP disponibile. Il body dell'offerta comprende l'ip assegnato con tutti i relativi dati (es. 10.0.0.1 con Mask 255.255.255.0 per un periodo di leasing di 8 giorni) e il suoi indirizzo eventualmente per rinnovare il lease (es. 10.0.0.254)  

- Source IP: 10.0.0.254 
- Destination IP: 255.255.255.255 
- Source MAC Address: The MAC Address of the server. 
- Destination MAC Address: The MAC Address of the client 

<b>Secondo problema:</b> cosa succede quando il server DHCP invia l'offerta, ma il client non risponde? 

l'IP elencato nell'offerta viene bloccato non lo rilascerà a nessun altro host. Il timer non è esattamente chiaro, ma nei test sembra essere di circa 10 minuti. 

<b>Terzo problema:</b> come viene assicurato? Come sappiamo che il server DHCP è legittimo?  

Una volta che il cliente riceve l'offerta (possono essercene più di una), inizierà il processo di DHCP request. Il primo passo è DAD (Duplicate Address Detection), ma supponendo che l'IP sia univoco sulla rete, invia la richiesta senza problemi. La richiesta dice fondamentalmente: "Ho ricevuto la tua offerta e accetto". 

- Source IP: 0.0.0.0 
- Destination IP: 255.255.255.255 
- Source MAC Address: The MAC Address of the client. 
- Destination MAC Address: FF:FF:FF:FF:FF:FF

va al broadcast, quindi se esistono più server DHCP sul segmento, tutti sapranno quale offerta del server è stata accettata. Gli altri server DHCP "perdenti" restituiranno i loro IP offerti ai loro pool. 

Ci sono alcuni strumenti che richiedono credenziali per ottenere un IP, ma per impostazione predefinita, non c'è sicurezza.  

Nella fase finale del processo, il server invia al client un riconoscimento. Il riconoscimento dice: "L'IP è ora tuo per il prossimo x periodo di tempo, ed ecco alcune altre cose (DNS, gateway predefinito, ecc ...) 

# DHCP Starvation Attack
Non essendo prevista nessuna forma di autenticazione, un attaccante : 
- manda molte richieste con MAC differenti
- Il pool si esaurisce
- Client legittimi non riescono a ottenere una configurazione

Inoltre un attaccante può settare un roughe DHCP server che risponde alle nuove richieste DHCP

# Roughe DHCP Server
A Roughe DHCP server è un dhcp server che non è sottocontrollo dell'amministratore di rete
- A volte incidentalmente si attacca un modem o home wireless router con DHCP capabilities

Un attaccante tramite un roughe DHCP server:
- può dare default gateway and Domain Name System (DNS) server
- man in the middle attack

Se l'attaccante diventa default gateway : 
- Tutti i client con indirizzi dal roughe DHCP Server fanno il forward dei pacchetti alla macchina dell'attaccante
- A sua volta i pacchetti possono essere consegnati o dirottati

Se l'attaccante ha anche il suo roughe DNS server
- possono essere resi phishing websites
  - per ottenere informazioni conidenziali : credit card details e passwords


# Mitigation : DHCP Spoofing
Si filtrano messaggi da untrusted DHCP messages
- Si costruisce un DHCP snooping binding database
  - Detta DHCP snooping binding table
  - Ogni entry contiene client MAC address, IP addressess, lease time, binding type, VLAN number, and port ID

Porte Trusted possono originare tutti i tipi di messaggi DHCP (DHCPOFFER, DHCPACK OR DHCPNAK)<br>
Porte untrusted solo richieste<br>
Se un rogue device su una porta untrusted tenta di mandare un pacchetto DHCP in risposta
- la porta viene chiusa
- si usa DHCP option 82, DHCP relay agent information option
- lo switch inserisce