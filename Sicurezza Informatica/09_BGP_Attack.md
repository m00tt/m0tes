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

# Traffic Attractor : De-aggregation
- Un AS illegittimamente origina dei “sub-prefix” di un blocco IP di un altro AS
  - “spezza” un blocco di indirizzi in diversi prefissi più specifici (i.e., più lunghi). 
- Il processo di selezione delle rotte dà preferenza più alta al prefisso più lungo che fa match: 
  - quindi l’attaccante usa de-aggregation per annunciare false rotte che in tutta Internet saranno preferite rispetto a quelle legittime. 

Esempio: (schema di prima)
- B può de-aggregare 
- Il prefisso annunciato da AS2 in 2 prefissi più lunghi di 1 bit, mentre l’AS-PATH per AS2 rimane lo stesso: il traffico in Internet per AS2 (tranne in AS2) viene forwardato a B. 

## Traffic Attractor, AS-Path Shortening
invece di sostenere di originare un prefisso si mantiene l’origine corretta ma si accorcia il resto del path per renderlo più attrattivo. 
Funzionamento : 
![Traffic Attractor](/assets/sicurezza_informatica/AS-Path-Shortening.png)<br>
- Rimuovi AS da un AS path
  - es. cambia “701 3715 88” in “701 88” 
- Motivazione:
  - Questo rende l'AS path più corto di quello che è ed attrae mittenti che normalmente cercano di evitare AS 3715. Fai sembrare AS 88 più vicino al core di internet. Può rendersi conto del fatto solo uno degli AS nel path: es. AS 88 si connette a AS 701 direttamente. 

 
# Annunci contradittori
Annunci differenti di routing spediti dalla stessa AS a differenti BGP. 
![Prefix Hijacking](/assets/sicurezza_informatica/prefix-hijacking.png)<br>
Esempio : <br>
- AS3 vuole usare B-M come connessione primaria ad Internet, e V-N come backup.
- AS-PATH per AS1 e AS2 ad AS4 saranno {AS1,AS3} e {AS2,AS3}.
- Invece, AS-PATH del UPDATE per AS5 saranno artificialmente paddate come {AS2,AS3,AS3,AS3} e {AS1,AS3,AS3,AS3}.
- Il path attraverso AS5 sarà più lungo e meno attrattivo per altre ASs.
- Un router malizioso può ridirigere il traffico verso se stesso o verso un’altra AS. 
- Il router B dovrebbe annunciare la rotta per AS1: {AS1,AS3,AS4}
- Invece, B può propagare quella rotta solo ad A indicando che non dovrebbe essere annunciata oltre, e annunciare la rotta paddata che va attraverso AS5 a R.
- Quindi parte di Internet (escluso AS4) sarà capace di raggiunger AS1 solo attraverso AS5.
- Lo scopo dell’attacco è creare congestione in AS5, o ridirigere il traffico verso AS1 attraverso un path subottimo

# Link Flapping
A livello Internet è perfettamente normale avere una topologia estremamente dinamica: BGP permette di scartare e annunciare nuove rotte con facilita.
Quindi con link flapping si intende che un link viene disattivato e poi riattivato (normale). Se succede spesso pero, crea instabilità nella rete perché gli instradamenti sono in continua variazione route dampening: la riattivazione di una rotta viene accettata con tempi crescentemente più lunghi. 
Esempio: dampening a R per le rotte verso AS1. B spedisce a R una sequenza di ritiri per le rotte {AS1,AS3,AS4}, e annunci {AS1,AS3,AS5,AS4}, e annunci per {AS1,AS3,AS4}. 

# Attacchi instability 
causare non disponibilità temporanea per alcune regioni di Internet, creare failure a cascata tra diversi domini tra diversi domini di routing. Alcuni attacchi spesso hanno come obiettivo le risorse limitate di un router. 
- Intentional Route-flapping 
- Route leaks (advertise many /24’s, overwhelm RIB, FIB memory) 
- BGP connection resets (CPU exhaustion, congestion, etc). 

## Contromisure 
- BGP TTL Security Hack
  - BGP in contatto sono di solito a distanza di un hop, quindi per difendersi da un attaccante, è possibile controllare che i pacchetti del messaggio BGP non hanno fatto un lungo viaggio in rete: 
  - IP Time-to-Live (TTL) : Decrementato una volta per ogni hop e si evita che i pacchetti stanno in rete per sempre
  - Generalized TTL Security Mechanism (RFC 3682): si manda un pacchetto BGP con TTL iniziale di 255, il BGP ricevente controlla che TTL è 254 ed elabora o scarta di conseguenza. 
- Diventa quindi difficile iniettare un pacchetto in una connessione punto-punto. 

- Authentication: 
  - RFC 2385 descrive una estensione di TCP definendo :
    - TCP OPTION (option-kind 19) MD5 digest.
  - Si tratta di un'estensione di TCP non di BGP, secondo la RFC, il digest MD5 digest è calcolato includendo diversi campi degli header IP e TCP. In questo BGP MD5 protegge contro: 
    - session hijacking
    - reply attacks
    - sesssioni non autorizzate BGP

- IPSEC
  - Usare IPSEC come meccanismo per rendere sicura una sessione BGP.
  - non è un protocollo specifico per BGP, ma una suite di protocolli che forniscono sicurezza al network layer. I protocolli definiscono metodi per cifrare, autenticare IP headers e payload, e forniscono servizi di key management. 

- Route Filterign
  - Filtering funziona creando delle Access Control Lists di prefissi o di ASs che sono usati da un router quando spedisce o riceve UPDATE.
  - I messaggi di UPDATEs in uscita 
    - Passano attraverso egress filters
    - Operatori controllano quali rotte sono annunciate da un peer
  - In entrata
    - Ingress filters sono applicati a messaggi UPDATE in ricezione
    - Usati per controllare la validità delle rotte ricevute
  - ISP devono conoscere il possessore di ogni blocco di indirizzi attraverso Internet Routing Registries (IRRs).

- Resource Public Key Infrastructure
  - AS ottengono un certificato Route Origin Authorizations (ROA) dalla autorità regionale (RIR) Regional Internet Registries (enti che assegnano blocchi IP agli AS)
  - RIR sono punti di trust
  - Router non fanno la validazione
    - Interrogano un remote validatot
  - ROA contengono: AS number (ASN), range di validità per le date, prefissi IP
  - ROA dice se un AS è autorizzata o meno ad annunciare un prefisso
  - AS attacca un ROA ad ogni annuncio di path.
    - Annunci senza un ROA valido sono ignorati
    - Difesa contro un AS maliziosa (ma non contro un attaccante)
 ![ROA](/assets/sicurezza_informatica/roa.png)<br>

- Versioni sicure di BGP

# S-BGP
estensione a BGP per proteggere BGP da UPDATE erroneo o malizioso, basata su public-key cryptography
- Address Attestation (AA): generate dal possessore di un prefisso, usate da S-BGP routers per verificare che la AS origine è autorizzata ad annunciare quel blocco di IP. 
- Route Attestationa (RA), autorizza una AS vicina a propagare la rotta contenuta in un UPDATE. 
 ![S-BGP](/assets/sicurezza_informatica/sbgp.png)<br>
Ogni S-BGP router lungo il path deve validare l’integrità di un UPDATE prima di firmarlo e di riannunciarlo. 
- A genera un RA per il prefisso P indicando B come il prossimo hop per quella rotta. 
- A spedisce UPDATE, includendo RA a B. 
- B valida la firma nel RA usando la chiave pubblica di A.  
- B verifica AA per P controllando che A è il vero possessore di quel prefisso. 
- B verifica che B è il prossimo hop in RA. 
- B genera due nuovi RA per i suoi peers C eD, include ogni RA in un differente UPDATE, e inoltra i due UPDATE a C e D. 

## Problemi di S-BGP
- ha bisogno di registri completi e accurati per determinare il possesso dei prefissi 
- public Key Infrastructure: per assegnare le chiavi pubbliche ad ogni AS 
- ha bisogno di eseguire le operazioni velocemente: per evitare risposte in ritardo a cambi di rotte 
- difficoltà ad un implementazione incrementale: difficile avere un giorno per far partire S-BGP per tutti i router, ha bisogno che tutti gli ISPs si accordino per aderire ad un nuovo protocollo, organizzazioni non sempre d’accordo e in competizione 
- incentivi economici: altrimenti difficile convincere i clienti a pagare per la sicurezza 
- nessun beneficio a usarlo singolarmente: tutto il path deve impiegarlo, come IPv6 

# Secure Origin BGP (soBGP)
maggiore flessibilità, trade off tra sicurezza e protocol overhead, la PKI gestisce tre tipi di certificati: 
- il primo tipo di certificato lega una chiave pubblica ad ogni soBGP-speaking router. 
- un secondo tipo di certificato fornisce dettagli sulla politica, configurazione dei parametri del protocollo e topologia della rete locale. 
  - Queste informazioni sono memorizzate dal soBGP router che riceve il certificato e usa quelle informazioni per costruire un database delle topologie che riflette la conoscenza del router sul network. 
- un terzo certificato è simile a S-BGP’s address attestations include possesso degli indirizzi 

# BGP is Vulnerabile
Ci sono state molte interruzioni di servizio di alto profilo e continuano ad esserci, esempio blackholing di un singolo prefisso e Hijacking.<br>
Non c’è molto interesse visto che la maggior parte delle interruzioni è dovuta a mal configurazioni; la maggior parte degli attori (anche quelli maliziosi) vogliono Internet funzionante per fare i loro affari(e.g., spam, identity theft, denial-of-service attacks, port scans, …). 
BGP difficile da "aggiustare" visto che è un sistema complesso: grande, con circa 30,000 AS, decentralizzato tra AS in competizione che forma il Core della infrastruttura Internet. <br>
Difficile raggiungere un accordo sulla soluzione giusta: S-BGP con public key infrastructure, registries, crypto…ma chi gestisce la PKI e i registri? <br>
Difficile sfruttare la soluzione scelta, difficile che gli ASes applichino filtri di rotta e difficile che facciano l’upgrade al protocollo tutti nello stesso momento. 

# Conclusioni
- Protocolli Internet progettati e basati sulla fiducia 
  - gli insiders sono tutti buoni 
  - tutti i cattivi sono fuori dalla rete 
- Border Gateway Protocol è molto vulnerabile 
  - difficile per un AS identificare localmente le rotte fasulle 
  - attacchi possono avere conseguenze molto serie a livello globale 
- Soluzioni proposte 
  - varianti sicure del Border Gateway Protocol 
  - schemi di rilevamento delle anomalie, con risposta automatica 

 
