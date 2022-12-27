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
![Risoluzione Nomi](/assets/sicurezza_informatica/risoluzione_nomi.png)<br>

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

![Pacchetto DNS](/assets/sicurezza_informatica/pack_dns.png)<br>

## Resolver to NS Request
![Resolver to NS Request](/assets/sicurezza_informatica/resolver_to_ns.png)<br>

## Response to resolver
![Response to Resolver](/assets/sicurezza_informatica/response_to_resolver.png)<br>

## Client to Server
![Client to Server](/assets/sicurezza_informatica/client_to_server.png)<br>

## Server to Client
![Server to Client](/assets/sicurezza_informatica/server_to_client.png)<br>

# Caching
Le risponse dei DNS vengono messe in cache in modo tale da velocizzare le richieste ripetute, anche i record per i nameserver sono messi in cache<br>
Anche le query con esito negative sono messe in cache in modo tale da risparmiare tempo, anche in caso di errore <br>
I dati in cache dopo un pò espirano
- Lifetime (TTL) dei dati controllati dal possessore dei dati
- Il valore TTL è passato con ogni record

# DNS Vulnerabilities : Summary
![DNS Vulnerabilities](/assets/sicurezza_informatica/vulnerabilities_dns.png)<br>


# Cache poisoning
Se viene scoperto il query ID è possibile effettuare un attacco di cache poisoning
- Scopo : 
  - Inserire false informazioni nella cache del nameserver ricorsivo
- Tutti i suoi "client" riceveranno falsi IP per un certo nome inconsapevolmente
- Il DNS accetta solo risposte a query in attesa:
  - Risposte non desiderate vengono ignorate
- La risposta deve arrivare sulla stessa porta UDP

La parte Question è duplicata nella risposta e deve fare match con la richiesta ed il Query ID fa match con la query in attesa. Le parti Authority e Additional contengono nomi nello stesso dominio della richiesta "bailiwick checking". 

# Attacco : indovinare il query ID
Nei vecchi nameserver Query ID è incrementato di 1, quindi è facile indovinare il numero, basta fare una richiesta su un dominio controllato dall’attaccante (test.badguy.com). Il nameserver della vittima riceve la richiesta e inizia la risoluzione ed alla fine viene diretto al nameserver dell’attaccante (authoritative per badguy.com.) e basta monitorare la richiesta a test.badguy.com o sniffare il traffico IP per scoprire la porta e il Query ID.  
![Query ID](/assets/sicurezza_informatica/query_id.png)<br>

## Version 1 query ID
![Versione 1 Query ID](/assets/sicurezza_informatica/version_1_query_id.png)<br>

## Punti essenziali
In questo caso l'unica misura di sicurezza è il QID che è poco funzionale, affinché l'attacco abbia successo il nome non deve giá essere nella cache (altrimenti si deve attendere il TTL), bisogna indovinare il QID ed il pacchetto manomesso deve arrivare prima di quello legittimo. 

# Attaccare authority record
Configurazione di un nameserver authoritative per bankofsteve.com zone
1. L'attaccante richiede un nome a caso nel dominio target
  - Non deve essere in cache
2. L'attaccante manda un flusso di pacchetti
  - Ma delega a un altro nameserver via Authority Records.
I dati contengono il vero nameserver per bankofsteve.com nameserver ma la glue punta agli IP dell’attaccante.<br>
La vittima crede che il nameserver dell’attaccante è authoritative per bankofsteve.com e adesso “possiede” tutta la zona<br>
![Attaccare authority record](/assets/sicurezza_informatica/authority_record_attack.png)<br>

## Punti essenziali
- Tutti possono configurare un proprio nameserver
  - Authoritative per qualsiasi dominio
  - Inutile perché non verrà puntato
- Attacco Pericolosissimo 
  - Controllando il dominio, l’attaccante controlla tutta la risoluzione del nameserver
    - Ridirige I visitatori del web (es google.com)
    - Dirige email ai propri server con falsi MX records
  - L’attaccante setta un TTL molto alto
    - Falsi dati in cache più a lungo. 
  - Si può attaccare un dominio di alto livello
    - .com, .net, ecc
  - Tutti I nomi sotto il top level, sono dirottati
    - Sforzo per soddisfare tutte le richieste

# DNS Vulnerabilità
- Sia gli utenti che gli host si fidano del mapping host-address restituito dal DNS:
  - Si usa per politiche di sicurezz
  - Browser same origin policy, URL address bar

- Problemi : 
  - Intercettare richieste o compromissione del server DNS in riposte sbagliate o maliziose
    - e.g. un access point malizioso in un internet point
  - Soluzione : Autenticazione delle richieste e delle risposte
    - DNSsec (non molto in uso)

# Serie di query
- Si può dirottare una singola query
  - Ma a causa del Query ID randomization difficile che l'attaccante indovini su 64k IDs
- Per ovviare a questo problmea l'attaccante genera una serie di query, ognuna per un nome casuale sotto lo stesso dominio
  - la prima richiesta avvia la risoluzione e la risposta finisce in cache (insieme al record ns1.bankofsteve.com) 
  - la richiesta successiva per un nome casuale (che non è sicuramente nella cache) causa una richiesta a ns1 server. L’attaccante invia un flusso di dati falsi verso la vittima. 

Esempio: se genera 50 false risposte per ogni nome casuale prima che la vera risposta potrebbe arrivare un successo (si ha una finestra di circa 10 secondi). 

# Triggering a Race
Ogni link, ogni immagine , ogni ad, può provocare un DNS lookup (non c’è bisogno di JavaScript) anche i mail server possono iniziare una richiesta. 

La difesa consiste nell'aumentare la taglia delle Query ID; aggiunta di una porta casuale (11 bit in piú) questo porta ad un attacco che dura diverse ore e richiedere ogni query due volte (l’attaccante deve indovinare il QueryID due volte, anche se causa troppo overhead). 

# Reverse DNS Spoofing
- Un accesso fidato spesso si basa sul nome dell’ host 
  - es. accesso permesso a tutti gli host in .rhosts di lanciare una remote shell. 
- Se le richieste rsh o rlogin arrivano da indirizzi sorgente numerici il sistema esegue un reverse DNS lookup per determinare l’host name del richiedente e controllare se è in.rhosts. 

- Se un attaccante può cambiare la risposta ad una reverse DNS query, può ingannare la macchina target, visto che non è presente nessuna autenticazione per le risposte DNS e nessun double-checking (numeric -> symbolic -> numeric)

# Altri attacchi contro name server

## “Exploit To Fail” DOS Attack
Sfrutta una vulnerabilità in alcuni elementi di un'infrastruttura di server dei nomi per causare l'interruzione del servizio di risoluzione dei nomi. Esempio: iniezione di messaggi DNS dannosi (CVE-2002-0400). 
![“Exploit To Fail” DOS Attack](/assets/sicurezza_informatica/exploit_to_fail.png)<br>

## “Exploit To Own” DOS Attack
Sfrutta una vulnerabilità in alcuni elementi di un'infrastruttura di server dei nomi per ottenere privilegi amministrativi di sistema. Esempio: RCE tramite BOF. 
![“Exploit To Own” DOS Attack](/assets/sicurezza_informatica/exploit_to_own.png)<br>

## Reflection Attack
L'attaccante falsifica l'indirizzo IP dell'host target ed invia messaggi DNS al recursor che invierà la risposta all'host mirato (host target)
![Reflection Attack](/assets/sicurezza_informatica/reflection_attack.png)<br>

## Reflection And Amplification Attack
L'attaccante falsifica l'indirizzo IP dell'host target ed invia messaggi DNS al recursor che sollecita una risposta LARGE. Il Recursor invia risposte LARGE all'host target, queste risposte occupano tante risorse. 
![Reflection And Amplification Attack](/assets/sicurezza_informatica/reflection_amplification.png)<br>

## Distributed Reflection And Amplification Attack
Lancia reflection and amplification attack da migliaia di bot. 
![Distributed Reflection And Amplification Attack](/assets/sicurezza_informatica/distributed_reflection_amplification.png)<br>

## Resource Depletion DOS Attack
L'aggressore invia un flusso di messaggi DNS su TCP dall'indirizzo IP contraffatto del bersaglio, il nameserver alloca le risorse per le connessioni finché le risorse non sono esaurite, la risoluzione dei nomi è ridotta o interrotta
![Resource Depletion DOS Attack](/assets/sicurezza_informatica/resource_depletion_dos.png)<br>

## NXDOMAIN Cache Exhaustion
L'aggressore inonda il ricorsore con query DNS per nomi di dominio inesistenti, il recursor tenta di risolvere le query e aggiunge ogni risposta NXDOMAIN alla cache. Questo causa che la cache di Recursor si riempie di risposte inutili e quindi l'elaborazione di query DNS legittime è ridotta. 
![NXDOMAIN Cache Exhaustion](/assets/sicurezza_informatica/nxdomain_cache_exhaustion.png)<br>

# Pharming
- La parola deriva da farming, esternalizzazione, sul modello di phishing/fishing
  - Moltre difese anti-phishing si basano su DNS
- Si possono avere
  - entry non corrette in un host file di un computer
  - compromissione di un router di un local network
- Uno scenario che include JavaScript per cambiare il DNS server del router
- Le difese di bypassano avvelenando la DNS cache e/o falsificando le risposte
- Dynamic pharming
  - Si fornisce un falso DNS Mapping per un server fidato, download di uno script malizioso
  - Si forza l'utente a fare il download dal vero server, fornendo temporaneament un DNS mapping corretto
  - Lo script, e il contenuto hanno la stessa origine

# DNS rebinding
I web browser in genere aderiscono alla same origin policy, un applet si può connettere solo al server dal quale è stato scaricato. <br>
Per eseguire la connessione il browser ha bisogno dell’ IP del server<br>
L'authoritative DNS server associa i nomi nel DNS a indirizzi IP.<br> 
Il browser si fida del DNS server (soprattutto quando aderisce alla same origin policy) 
1. L'attaccante crea il dominio attacker.org
2. Lega il nome a due indirizzi IP
  - Il proprio e l'indirizzo della vittima
3. Il client scarica l'applet da attacker.org
  - Applet si connette al target
  - Permesso da same origin policy
![Rebinding](/assets/sicurezza_informatica/rebinding.png)<br>

## Attacco con rebinding
Esempio pratico: <br>
- Sfruttando l'attacco di rebinding, la vittima si connette a una pagina esterna gestita dall'attaccante
- In questa pagina esterna è presente un codice javascript malizioso, che in qualche modo riesce a capire l'indirizzo IP locale della vittima
- L'attaccante potrà accedere tramite js alla configurazione di network (browser non si accorge di nulla)

L’attaccante usa il DNS rebinding per accedere alle macchine che sono dietro un firewall, si tratta di ip hijacking<br>
L’attaccante usa il DNS rebinding per accedere pubblicamente a server usando l’ IP del client. Il beneficio si ha nella fiducia che proviene dall’IP del client.

- Spidering the intranet
  - L'attacante indovina il nome di una macchina interna (usa CNAME per rebind), il resolver ritorna l'IP
  - Evita lo scan degli IP interni
  - Si può connettere al server (web server non protetto) 
- Compromising Unpatched Machines
  - Macchine interne sono meno protette
  - Patching non aggiornato, exploit di una macchina dietro al firewall
- Abusing Internal Open Service
  - Alcuni servizi sono aperti per uso interno, come stampanti, ftp server, ecc
  - Router e servizi con pw di default

## IP Hijacking
Utile per, ad esempio : <br>
- Committing Click Fraud
  - Si generano falsi click con IP diversi
- Sending Spam
  - Alcuni indirizzi IP sono in blacklist per lo spam
  - Si usa IP del client per inviare spam
- Defeating IP-Based Authentication
  - Accesso a biblioteche ecc
- Farming Clients
  - Si attacca usando l'IP del client
    - La colpa ricade sul suo IP
  - Se si usa HTTPS non si lascia traccia

# DNS Rebingind Defenses
- Browser Mitigation : DNS Pinning
  - Rifiuta di passare a un nuovo IP
  - Interagisce male con proxy, VPN, DNS dinamico
  - Non implementato in modo coerente
- Difese lato server
  - Controlla l'intestazione host per domini non riconosciuti
  - Autentica gli utenti con qualcosa di diverso dall'ip
- Difese firewall
  - I nomi esterni non possono essere risolti in indirizzi interni
  - Protegge i browser all'interno dell'organizzazione

# DNSSEC
IETF (Internet Engineering Task Force) ha definito DNSSEC, versione sicurea del DNS, si basa su public key cryptography per la verifica dei dati DNS che garantisce autenticazione e integrità. Si stabilisce una chain of trust che parte dalla radice attraverso la gerarchia di risoluzione: ad ogni livello nel DNS, la firma del livello superiore è verificata usando una chiave pubblica associata

# DNSSEC - Protezioni offerte
Nessun cambiamento al formato del pacchetto, il goal è la sicurezza dei dati DNS, non la sicurezza del canale
- Goal è la sicurezza dei dati DNS, non la sicurezza del canale
![DNSSEC](/assets/sicurezza_informatica/dnssec.png)<br>
Ogni zona ha una coppia di chiavi pubblica e privata:  
- Il proprietario della zona utilizza la chiave privata per firmare i dati della zona, producendo firme digitali per ogni set di record di risorse
- La chiave pubblica viene utilizzata da altri (risolutori DNS) per convalidare le firme (prova di autenticità)
- La chiave pubblica è pubblicata nella zona stessa in modo che i resolver possano trovarla
- Le chiavi pubbliche della zona sono organizzate in una catena di fiduzia che segue il normale percorso di delega DNS
- I risolutori DNS autenticano le firme DNS dalla radice alla zona foglia contenente il nome.

Nuovi Resource Records (RR): 
![DNSSEC Record](/assets/sicurezza_informatica/dnssec_record.png)<br>

# Extended DNS
I messaggi DNS più grandi di 512 byte richiedono: 
- Utilizzo di TCP (in genere risposta UDP troncata seguita da un nuovo tentativo TCP)
- EDNS0 - un meccanismo di estensione DNS che consente la negoziazione di dimensioni maggiori Buffer dei messaggi UDP
- RFC 6891 "Meccanismi di estensione per DNS (EDNS0) 
Per DNSSEC, EDNS0: negoziazione di carichi utili UDP più grandi, flag per indicare che l'interrogante è in grado di elaborare i record DNSSEC: il Bit "DNSSEC OK" o "DO" 

# Nuovi Flag

## Flag AD
AD - “Authenticated Data”<br>
Resolver imposta questo flag nelle risposte quando il record interrogato è firmato con una firma valida e non scaduta e una catena di fiducia autenticata fino a un ancoraggio di fiducia configurato (che potrebbe essere la chiave radice preconfigurata / tracciata) <br>
Tutti i dati nelle sezioni risposta e autorità incluse sono stati adeguatamente autenticati dal risolutore <br>
Può anche essere impostato in una query DNS - per indicare che l'interrogante comprende le risposte con il bit AD (ad es. Se desidera uno stato autenticato ma non necessariamente DNSSEC RR)

## Flag CD
CD - “Checking Disabled”
Si imposta il flag CD per indicare che "in sospeso" (dati non autenticati) è accettabile, ad es. perché è disposto a fare la propria convalida crittografica delle firme. <br>
I server abilitati DNSSEC non devono tuttavia restituire dati "cattivi" (ad esempio con firme errate)<br>

# Pacchetto DNS
![DNS Pacchetto](/assets/sicurezza_informatica/dnssec_pack.png)<br>

# Chiavi
In genere, viene utilizzata una gerarchia a 2 livelli di DNSKEY 
- KSK: chiave di firma della chiave, firma altre chiavi (può essere più grande, ad es. più forte e mantenuto offline; utilizzato come punto di ancoraggio della fiducia e certificato dalla zona padre nel DS) 
- ZSK: chiave di firma della zona, firma tutti i dati nella zona (può avere una forza inferiore e imporre un sovraccarico di calcolo inferiore; può essere modificato senza coordinamento con la zona genitore)

# Secure Delegation
Indicato dal record DS (Delegation Signer), appare nella zona di delega (ovvero genitore), contiene un hash della chiave pubblica della zona figlio. La convalida dei resolver utilizza la presenza del record DS e la relativa firma (RRSIG) per autenticare in modo sicuro la delega. 

# Authenticated Denial of Existence : Risposte Negative
Risposta negativa che indica un errore nei record NSEC or NSEC3 (e le loro firme). <br>
È necessario un ordinamento canonico dei nomi nelle zone firmate (RFC 4034, sezione 6.1), questo avviene a causa del modello di firma precalcolato di DNSSEC (problemi di calcolo e sicurezza della chiave di firma). <br>
Ordina i nomi DNS in ordine di più significativo (più a destra) prima le etichette, quindi, all'interno di ciascuna etichetta, ordinale come ottetto stringhe, da lettere ASCII a maiuscole e minuscole.<br>
Esempio : <br>
- a.example.com 
- a.example.com 
- blah.a.example.com 
- Z.a.example.com 
- zABC.a.EXAMPLE.com 
- z.example.com 
- \001.z.example.com 
- *.z.example.com 
- \200.z.example.com 

# Adozione di DNSSEC
È considerato un elemento fondamentale nelle strategie globali della cosiddetta trusted Internet. <br>
In realtà la sua adozione non è semplice: 
- Complessità delle configurazioni 
- Aumento del traffico
- Perplessità di una parte della comunità sull'efficacia

# Limiti di DNSSEC
Secondo D. J. Bernstein, autore di djbdns e di una proposta alternativa (DNSCurve), è mal progettato:
- L'assunzione di base e che non è pensabile usare la crittografia in ogni pacchetto, per ragioni di efficienza.  
- Non c'è crittografia dei dati ma solo integrità
- Le firme sono precalcolate (e quindi occorre ruotare le chiavi per limitare i replay)
- Tutti i tool per la modifica dei dati DNS devono essere `signature aware'
- Non c'è protezione contro DoS. 