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

//add Image
//add Image

## DHCP : more than IP addressess
DHCP oltre a restituire un indirizzo IP sulla sottorete può assegnare : 
- indirizzo del router più vicino per il client
- Nome e indirizzo del DSN server
- Network mask (indica la parte di rete dell'indirizzo)

## DHCP : esempio
Se un dispostivo vuole collegarsi alla rete ha bisogno di un indirizzo IP
//add Image

- Richiesta DHCP incapsulata in UDP, incapsulata in IP, incapsulata in 802.1 Ethernet
- Ethernet frame broadcast on LAN, ricevuto dal router che dove viene eseguito il DHCP server
- Ethernet demuxed to IP demuxed, UDP demuex to DHCP
- Incapsulamento del DHCP server,frame inoltrato al client, demuxing dal DHCP al client
- Client ora conosce l'IP address, nome e IP address del DNS server, IP address del router

# Vulnerabilità DHCP
Il protocollo lavora a livello di rete locale in cui i nodi : 
- condividono il mezzo trasmissivo
- sono identificati dal MAC

# DHCP Starvation Attack
Non essendo prevista nessuna forma di autenticazione, un attaccante : 
- manda molte richieste con MAC differenti
- Il pool si esaurisce
- Client legittimi non riescono a ottenere una configurazione

Inoltre un attaccante può settare un roughe DHCP server che risponde alle nuove richieste DHCP

# Roughe DHCP Server
A Roughe DHCP server non è sottocontrollo dell'amministratore di rete
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
//addImage

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