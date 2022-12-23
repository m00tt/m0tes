# DNS
- Il DNS (Domain Name System) consente di mappare un IP all'URL

# DNS Root Name Servers
![Gerarchia](/assets/sicurezza_informatica/gerarchia.png)<br>
Domain Name System è un servizio gerarchico: 
- Root Name Server per domini ai Top-Level : in internet ci sono 13 server DNS principali ed una rete di server replicati
- Top-Leve domain (TLD) server : responsabili dei domini di primo livello come com, org, net, edu e gov e di tutti i domini di primo livello del paese come uk,fr,ca e jp
- Authoritative Name Server per i sottodomini : per una organizzazione contiene i DNS record. Implementare il proprio autoritative server DNS o in alternativa avere questi record archiviati in un autoritative server DNS autorevole di qualche fornitore di servizi. La maggior parte delle università e delle grandi aziende implementano e mantengono il proprio autoritative server DNS autorevole primario e secondario (backup). 
- Local name resolver: contattano authoritative server quando non conoscono un nome 

## Terminologia
- Zone: una collezione di coppie hostnames/IP gestite insieme. All'interno di un dominio ci possono essere diverse zone se è molto grande. Esempio: unixwiz.net, e tutti I record DNS simili — www.unixwiz.net, mvp.unixwiz.net, cs.unixwiz.net, etc. — fanno parte della “unixwiz.net zone” 
- Nameserver: software server che risponde a domande DNS, alcune volte il nameserver conosce la risposta direttamente (è "authoritative" per la zona), altre volte ridirige la domanda (recursive nameserver). Es. software che eseguono il servizio: BIND, PowerDNS, djbdns 
- Authoritative Nameserver: per ogni zona bisogna mantenere un file delle associazioni hostnames e indirizzi IP (linux.unixwiz.net is 64.170.162.98", etc). 
- Resolver: parte client del Sistema DNS client/server, libreria compilata in un programma che richiede il servizio DNS sa come interrogare un nameserver sulla porta 53 (es. funzione getHostByName in linux) 
- Resource Record: sostanzialmente il DNS è un database distribuito di "resource records" che contiene i campi: (Name, Value, Type, TTL). Il tipo comune è un IP Address (an "A" record), ma esistono altri tipi di records : NS (nameserver), MX (mail exchanger), SOA (Start of Authority), ecc. 

# Risoluzione dei nomi
- Ogni computer ha una librearia per la risoluzione dei nomi detta resolver, inoltre conosce un nome di un DNS server locale
- Il resolver manda una richiesta DNS al server, il DNS server o dà la risposta o la inoltra ad un altro server, o da un referente, ossia il server successico al quale la richiesta deve essere spedita. I Resolver usano UDP (single name) o TCP (whole group of names)
- Sapere l'indirizzo del Root Server è sufficiente?
  - Recursive Query : ritorna una risposta (non un referente) usato dai resolver (Usato dai resolver)
  - Iterative Query : ritorna una risposta o un riferimento al prossimo server, usato dai DNS server tra di loro (Usato dai server)

Esempio di DNS Lookup<br>
![DNS Lookup](/assets/sicurezza_informatica/dns-lookup.png)<br>
Il client ogni volta che ha bisogno di traduttore un URL in IP contatta il Local DNS resolver<br>
Il Local DNS Resolver, se non conosce il nome che il client sta cercando allora si prende il dominio, il dominio in questo caso è ".edu" quindi fa una richiesta al Top Level Domain, e quest'ultimo risponde che deve mettersi in comunicazione con il Name Server "stanford.edu" e cosi via fino a scoprire tutto l'indirizzo.

# Tipi di DNS entry : Resource Records
DNS è usato non solo per l'associazione nome-IP<br>
Ma anche per trovare un mail server, pop server ecc<br>
DNS Record Types:
- NS : name server (punta ad un altro server)
- A : address record (contiene l'indirizzo IP)
- MX : indirizzo usato per la gestione della posta elettronica
- CNAME : mappa un nome alias su un nome di dominio vero
- SOA : Start of Authorities : restituisce informazioni autorevoli sulla zona DNS, come il contatto per admin
- TXT : generic test

//insert esempi from documento DNS

# Risoluzione dei nomi
- Ogni computer ha una libreria per la risoluzione dei nomi (es. gethostbyname in UNIX) detta resolver, inoltre conosce un nome di un DNS server locale. 
- Il Resolver manda una richiesta DNS al server, il DNS server o dà la risposta o la inoltra ad un altro server, o da un referente, ossia il server successivo al quale la richiesta deve essere spedita. I resolver usano UDP (single name) o TCP (whole group of names). 
- Sapere l’indirizzo del root server è sufficiente? 
  - Recursive Query: ritorna una risposta (non un referente), usato dai resolver 
  - Iterative Query: ritorna una risposta o un riferimento al prossimo server, usato dai DNS server tra di loro
// add image onenote

- Il client("User's PC") richiede un sitoweb.net, indirizzato a nameserver fornito dal ISP. 
- Se il nameserver non è authoritative per il sitoweb.net richiesto 
  - guarda nel local zone database. 
  - non c’è alcun nome salvato nella cache 
  - richiesta indirizzata su internet 
- Tutti I nameserver ricorsivi hanno una lista con 13 root server che hanno gli IP quasi fissi: 
  - A.ROOT-SERVERS.NET. IN A 198.41.0.4 
  - B.ROOT-SERVERS.NET. IN A 192.228.79.201 
  - C.ROOT-SERVERS.NET. IN A 192.33.4.12 
  - ... 
  - M.ROOT-SERVERS.NET. IN A 202.12.27.33 
- Il nameserver ne sceglie uno a caso e manda la query per il record A del sitoweb.net richiesto; 
  - Es: b.root-servers.net 
- Se il root server non conosce il sitoweb.net richiesto ridirige al Global Top Level Domain (GTLD) responsabile per I domini .net utilizzando i NS records più qualificati per rispondere alla query. 
  - Il nameserver sceglie uno degli authoritative server a caso, es. c.gtld-servers.net, ed invia la richiesta per A record del sitoweb.net richiesto. Il GTLD server non conosce la risposta, quindi ridirige ad un referral (un insieme di NS record che hanno piú informazioni).
  - Il recursive nameserver sceglie uno dei nameserver a caso e manda una terza query, a questo punto otteniamo una risposta. Alcuni flag indicano "This is an authoritative response", la risposta proviene da una delle sorgenti fidate per quel dominio (e non da una cache). Il recursive nameserver del ISP riporta la risposta al client e memorizza nella cache.   

# Risoluzione Inversa
DNS è usato anche per mappare un indirizzo IP in un host name.  <br>
Nota: gli IP sono usati in ordine inverso es. ip 1.2.3.4 si trova il relativo nome associato con 4.3.2.1.in-addr.arpa.  <br>
Attenzione: un nameserver potrebbe dichiarare qualsiasi zona, anche quelle che non possiede. Un attaccante potrebbe settare un nameserver come authoritative zone per bancaitalia.it ma questo non avrebbe nessun effetto: nessun nameserver ad alto livello lo delega. <br>

# Ottimizzazione dei DNS
- Spatial Locality: I computer locali sono interrogati più di quelli remoti 
- Temporal Locality: lo stesso set di domini viene interrogato ripetutamente, si usa Caching, ogni entry ha un time to live (TTL)
- Replication: Multipli server,  multiple radici, si contatta il server geograficamente più vicino

# Messaggi DNS
Il DNS ha un tipo di messaggio che contiene 5 parti : 
- Header Section 
  - Tipo del messaggio e contenuto
- Question Section
  - Informazioni sull'oggetto della query
- Answer Section
  - RR relativi alla rispota
- Autority Section
  - SOA o NS record
- Additional Section
  - Informazioni aggiuntive

# Pacchetti DNS
DNS server sono in ascolto sulla porta 53/udp
- Query ID
  - Identificatore unico, permette al server di associa la richiesta alla risposta
  - Chiamato anche Transaction ID (TXID)
- FLAGS
  - QR (Query / Response) : 0 query, 1 response
  - Opcode 0 for a standard query
  - AA (Authoritative Answer)
  - RD (Recursion Desired)
  - RA (Recursion Available)
  - Z - reserved must be zero
  - rcode success or failure
- Question record count 
  - Cosa sta cercano : nome,tipo,classe
- Question count quasi sempre 1
- Answer / authority / additional recoun count : 
  - Includono diverse risposte addizionali dal client
- Question / Answer Data
  - Contiene la richiesta o la risposta

  //add Image



 