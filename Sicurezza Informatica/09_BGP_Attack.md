# Introduzione BGP
Inter-AS Routing, BGP: Border Gateway Protocol version 4, RFC 4271,<br>
Il BGP serve per mettere in comunicazione i router che sono sul confine (router che connettono un AS all'altro)<br>
![Infrastruttura](/assets/sicurezza_informatica/infrastruttura.png)<br>
In questo esempio il BGP abbiamo AS1, AS2, AS3 e i router 1c e 3a per comunicare utilizzano il BGP (mettono in comunicazione AS3 e AS1), come 1b e 2a (mettono in comunicazione AS1 e AS2)<br>
Determina le coppie source-destination che si estendono su AS multipli. <br>
BGP fornisce per un AS come: 
- Ottenere informazioni per la raggiungibilità delle subnet dagli AS vicini
- Propagare queste informazioni a tutti i router interni al AS 
- Determinare “buone” rotte per le subnets basandosi sulla raggiungibilità e sulle politiche degli AS. 

# eBGP e iBGP
Coppie di routes si scambiano informazioni di routing su connessioni semipermanenti usando la porta TCP<br>
Esistono due tipi di BGP peers : 
- Internal BGP Session : iBGP : comunicazioni interne all'AS
- External BGP Session : eBGP : comunicazioni esterne all'AS utilizzate per mettere in comunicazione 2 AS
BGP permette ad ogni AS di conoscere quali destinazioni sono raggiungibili attraverso le AS vicini. <br>
![eBGP vs iBGP](/assets/sicurezza_informatica/eBGP-vs-iBGP.png)<br>
Le destinazioni sono espresse come prefissi in notazione CIDR<br>
Un router può avere info su due 2 prefissi, ad esempio : <br>
- 211.120.0.0/12
- 211.120.132.0/22
se ad esempio la destinazione è : 
- 211.120.132.37
Essa fa match sia con il primo prefisso che con il secondo ma il router preferisce instradare i pacchetti sul secondo prefisso in quando più preciso rispetto al primo<br>
Autonomous Systems hanno un numero assegnato dalla IANA (ASNs): i numeri di AS da 1 a 64511 sono pubblici, ogni numero corrisponde ad un singolo AS. A volte il provider di riferimento fornisce un ASN nel range 64512–65535. 

# BGP
BGP è un protocollo di tipo :
- Path Vector Protocol 
  - Il percorso viene gestito attraverso un vettore
- Incrementale
  - Manda un annuncio quando una nuova rotta essite e un altro quando una rotta viene ritirata

Registra la sequenza di AS per ogni destinazione, i nodi indirizzabili da un AS sono quelli con un determinato prefisso, un AS path è la lista degli AS da attraversare per raggiungere un nodo con un dato presso 
1. Un AS A annuncia (UPDATE) ai vicini quali prefissi x sa indirizzare (A x) 
2. Il vicino B annuncia (B A x) 
3. Chi riceve un path che contiene se stesso non lo riannuncia 
4. I path contengono anche attributi utilizzabili nelle policy 
Ogni router deciderà per ogni prefisso qual è la rotta migliore e se annunciarla ai vicini 

Ad esempio : 
![BGP esempio](/assets/sicurezza_informatica/BGP-es.png)<br>
AS2 gestisce il traffico per e da AS7<br>
AS2 annuncia ai vicini (AS1, AS3, AS6) che per raggiungere AS7 devono passare attraverso AS2<br>
Ad esempio AS1 saprà che se vuole arrivare a AS7 il percorso da fare sarà : AS2, AS7 (stessa cosa AS3 e AS6)

BGP update message: ogni volta che un messaggio di UPDATE è ricevuto, la route table di BGP è aggiornata e il numero di versione della BGP route table è incrementato di uno. 

# Cosa può essere attaccato?
- Availability 
  - Denial-of-service attack 
  - Reachability 
  - Degrade link quality
    - route flapping, link cutting attacks 
- Data Confidentiality 
- Data Integrity 
- Authentication (impersonation) 

# (In)Sicurezza BGP
I messaggi di BGP update non contengono nessun meccanismo di autenticazione o integrità<br>
Attaccante può falsificare le rotte annunciate
- Modificare i prefissi IP associati ad una rotta
  - blackhole
- Cambiare AS path
  - Attrarre traffico all'AS dell'attaccante o deviarlo
   - Incentivi economici : un ISP vuole riversare il suo traffico su un altro ISP senza dirigere il suo traffico in cambio
- Ri-annunciare o propagare AS path senza permesso
  - Ad esempio, un cliente multi-homed può annunciare capcità di transito tra due grandi ISPs

# Attacchi contro la confidenzialità
- Eavesdropping
  - Monitorare i messaggi su una sessione BGP
  - tapping della connessione tra I router vicini
- Rivela informazioni sensibili
  - Inferenza di relazioni di business
  - Analisi della stabilità della rete
- Difficoltà dell’attacco
  - Difficile fare il tap della connessione
    - Spesso le sessioni , eBGP attraversano solo un link
  - Il contenuto può essere cifrato
    - Router vicini BGP possono eseguire IPSec

# Attacchi contro l'integrità dei messaggi
- Tampering
  - Man-in-the-middle può modificare i messaggi
  - Inserimento, delete, modify, o replay dei messaggi
- Porta ad un comportamento di BGP non corretto
  - Delete: il vicino non impara la nuova rotta
  - Insert/modify:il vicino apprende delle rotte fasulle
- Difficoltà:
  - Mettersi in mezzo tra due routers può essere difficile
  - Uso di autenticazione (firme) or cifratura
  - Manipolare i pacchetti TCP può essere difficile
    - Generare il giusto TCP sequence number

# Attacchi Denial-of-Service
- Sovraccaricare il link tra i routers
  - Per provocare packet loss e ritardi
  - Compromettendo la performance della sessione BGP
- Relativamente facile da fare
  - Basta spedire traffico tra gli host finali
  - E lungo le connessioni attraversate dai pacchetti
  - La rotta può essere tracciata da traceroute

- Difese
  - Alta priorità ai pacchetti BGP
  - Es. Mettere i pacchetti in code separate

## Altri attacchi di Denial-of-Service
- Attaccante manda pacchetti bogus TCP
  - FIN/RST per chiudere la sessione
  - SYN flooding per overload del router
- Porta a malfunzionamenti in BGP
  - Session reset, transient routing changes
  - Route-flapping
- Difficoltà degli attacchi
  - Spoofing TCP può essere difficile
    - Bisogna mandare FIN/RST con giusto TCP header
  - Packet filter possono bloccare SYN flooding
    - Filtrano pacchetti a porte BGP port da sorgenti non autorizzate
    - O destinati a router da sorgenti non autorizzate

# Errata configurazione di BGP
il dominio annuncia buone rotte per indirizzi che non sa raggiungere, questo porta che i pacchetti vanno in un “black hole” 

# Obiettivi di un attacco
- Blackholing
  - Un prefisso è irraggiungibile da una porzione di Internet.
  - Annuncio di false rotte allo scopo di attrarre traffico verso un router e poi abbandonarlo
- Redirection
  - Traffico verso una particolare rete è diretto verso un path diverso per raggiungere una destinazione non corretta e/o compromessa
  - La destinazione compromessa può impersonare l’host allo scopo di ricevere informazioni confidenziali
  - Altro obiettivo: ridirigere traffico verso un nodo o sottorete per causare congestione
- Subversion
  - Caso special di redirezione 
  - L’attaccante forza il traffico a passare per un certo nodo o link allo scopo di intercettare o modificare I dati 
  - Il traffico raggiunge comunque la destinazione
    - Attacco più difficile da scoprire
- Instability
  - Annunci in successione con attributi differenti
  - Scopo: Distruzione delle rotte e interruzione della connettività 
  - Aumento del traffico BGP per aumentare I tempi di convergenza

  # Prefix hijacking
- AS annuncia una rotta che non ha
- AS origina un prefisso che non possiede
  - MOAS – Multiple Origin AS Quando diverse AS
originano (sono le prime AS ad annunciare) un
particolare prefisso

Esempio : 
![Prefix Hijacking](/assets/sicurezza_informatica/prefix-hijacking.png)<br>
- Router B vuole dirottare il traffico verso AS2
- Router B annuncia una falsa rotta direttamente connessa con AS2
- AS1, AS2, AS3 NON sono affetti, invece il resto (AS6 e AS5 si)

L’AS che vuole effettuare hijacking ha:
- router con session eBGP
- Configurato per originare un prefisso

Bisogna avere accesso al router 
- Operatore di rete effettua un errore di configurazione
- Operatore scontento lancia un attacco
- Un attaccante entre nel router e lo riconfigura

Bisogna che altri AS credano alla falsa rotta
- Gli AS vicini non filtrano le rotte
  - Filtrando i prefissi
  - Settare i prefissi

