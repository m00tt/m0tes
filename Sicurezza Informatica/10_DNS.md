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

|



 