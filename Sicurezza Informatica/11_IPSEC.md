# Network Protocol Stack
![Network Protocol Stack](/assets/sicurezza_informatica/network_protocol_stack.png)<br>

# Problemi di sicurezza
Quando un host riceve un pacchetto IP, non ha nessuna certezza: autenticazione e integrità dei dati, confidenzialità e non-ripudio. <br>
Problemi: source spoofing, replay packets, etc. 

# Protezione a diversi livelli
Livello 1,2 : 
- Protezione fisica del cavo
- Wireless
  - WEP (Wired Equivalent Privacy) Layer 2 per le connessioni wireless ormai superato, fornisce:
    - Autenticazione (weak)
    - Data confidenzialità (weak)
    - Data integrità (weak)
  - 802.1x (port-based Network Access Control) basata sulla porta LAN (non TCP/UDP) layer 2
    - Access Control

IPsec layer 3 fornisce:
- Autenticazione (mutua)
- Access control
- Confidenzialità dati ed integrità

SSL/TLS over TCP layer 4+, fornisce :
- Autenticazione (mutua / one way)
- Confidenzialità dati ed integrità

![Livelli di protezione](/assets/sicurezza_informatica/lv_protezione.png)<br>

# Goal di IPSec
IPSec vuoe garantire : 
- <b>autenticazione</b> della sorgente e <b>integrità</b> dei dati: non è possibile mandare un IP datagram con un indirizzo sorgente IP mascherato senza che il destinatario se ne accorga, non è possibile modificare un IP datagram in transito senza che il ricevente se ne accorga. Dati “firmati” dal sender e verificati (verifica della firma) dal ricevente, l'eventuale modifica dei dati può essere rilevata dalla “verifica” della firma poiché la “firma” si basa su un segreto condiviso si ha autenticazione della sorgente. (Evita IP Spoofing)
- <b>replay protection:</b> non è possibile mandare nuovamente un pacchetto IP senza che il ricevente se ne accorga, è opzionale, il sender può fornirla ma può essere ignorata dal ricevente
- <b>confidenzialità:</b> non è possibile esaminare il contenuto di un datagramma IP, opzionale. 
- <b>Key management:</b> negoziazione e stabilimento di sessioni, le sessioni possono essere rigenerate o cancellate automaticamente, le chiavi segrete sono stabilite in maniera sicura e autenticata (ci sono diverse opzioni per la autenticazione degli host). 
- <b>Politiche di sicurezza</b> : il mittente e il ricevente possono determinare il livello di protezione per un pacchetto IP secondo una politica di sicurezza locale. i nodi Intermedi e il ricevente ignoreranno i pacchetti IP che non soddisfano questi requisiti. 

# IPSec Suite
Estensione sicura di IPv4 e IPv6 formata da una collezione di protocolli <br>
- Internet Key Exchange (IKE) protocol: per negoziare i parametri di sicurezza e stabilire chiavi di autenticazione, usa UDP porta 500 for ISAKMP 
- Encapsulating Security Payload (ESP) protocol: per cifrare, autenticare, e rendere sicuri i dati; IP protocol 50.
- Authentication Header (AH) protocol: per autenticare e rendere sicuri i dati; IP protocol 51. 
- Architettura (RFC 2401): AH (RFC 2402), ESP (RFC 2406), IKE (RFC 2409) e IPcomp (RFC 3137) 

# Beneficidi IPSec
- Garantisce confidenzialità, Integrità , e autenticazione
  - Data firmati dal sender e firma verificata dal ricevente
  - Modifica dei dati può essere rilevata dalla verifica della firma
  - Poieché la firma si basa su un segreto condiviso si ha autenticazione della sorgente
- Anti-replay Protection 
  - Opzionale, il sender può fornirla ma può essere ignorata dal ricevente
- Key management
  - IKE - negozazione e stabilimento delle sessioni (internet key exchange protocol)
  - Le sessioni possono essere rigenerate o cancellate automaticamente
  - Le chiavi segrete sono stabilite in maniera sicura e autenticata
  - Ci sono diverse opzioni per la autenticazione degli host

# IPSec Modalità
- Tunnel Mode: 
  - Creato un nuovo pacchetto incapsulando il vecchio pacchetto IP
  - Pacchetti IP incapsulati, il pacchetto IP è cifrato e diventa il componente data di un nuovo pacchetto IP, si usa spesso in IPsec site-to-site VPN. 
  - Si usa quando almeno un “cryptographic endpoint” non è un “communication endpoint” dei pacchetti IP, permette ai gateway di rendere sicuro il traffico IP in vece di altre entità. Con cryptographic endpoints si intende le entità che generano o processano un IPSec header, mentre con communication endpoints si intende mittente e destinatario di un pacchetto IP (possono anche non essere host classici, es. un gateway gestito in remote via SNMP da una workstation). 
- Transport Mode:
  - Al pacchetto IP vengono aggiunte informazioni addizionali -> IPSec header -> contiene le informazioni che ci consento di cifrare/decifrare
  - IPsec header viene inserito in un pacchetto IP, non si crea un nuovo pacchetto, funziona bene in reti dove la dimensione del pacchetto è un problema. 
  - Si usa tra end-points di una comunicazione e per accesso remoto ad una VPN. Si può usare quando “cryptographic endpoints” coincidono con “communication endpoints” dei pacchetti IP. 

![IPSec Modalità](/assets/sicurezza_informatica/ipsec_mod.png)<br>
![Tunnel Mode VS Transport Mode](/assets/sicurezza_informatica/tunnel_vs_transport.png)<br>

# Architettura IPSec
![Architettura IPSec](/assets/sicurezza_informatica/architettura_ipsec.png)<br>

## Authentication Header (AH)
- Fornisce autenticazione della sorgente, protegge contro source spoofing 
- Fornisce integrità dei dati: usa strong hash (96-bit) con crittografia simmetrica (HMAC-SHA-96, HMAC-MD5-96)
- Protegge contro replay attack: usa in maniera crescente un sequence number di 32 bit e protegge contro denial of service attack 
- Non fornisce confidenzialità

### Formato Pacchetto con AH
![Formato pacchetto AH](/assets/sicurezza_informatica/pack_with_ah.png)<br>

### AH in trasport mode
Quando il pacchetto arriva a destinazione se passa il controllo di autenticazione 
- si rimuove AH header
- il campo Proto=AH viene rimpiazzato con "Next Protocol"
- Riporta IP datagram al suo stato originale e lo consegna al processo in attesa. 
![AH transport mode](/assets/sicurezza_informatica/ah_transport_mode.png)<br>

### AH in tunnel mode
Come nel Transport mode, il pacchetto è sigillato con un valore di controllo di Integrità
- Per autenticare il mittente e prevenire modifiche
- A differenza del Transport mode, incapsula tutto l’IP header e il suo payload
- Quindi sia mittente che destinazione possono essere differenti da quelli del pacchetto che li comprende
- Si permette un “tunnel”
Il modo Tunnel si usa di solito fra gateways (routers, firewalls, o VPN devices) per formare una Virtual Private Network (VPN). 
![AH tunnel mode](/assets/sicurezza_informatica/ah_tunnel_mode.png)<br>

### Transport or Tunnel?
- se il valore next-header value è IP: questo pacchetto incapsula un intero IP datagram (inclusi gli indirizzi sorgente e destinazione che permettono un routing diverso dopo la de-encapsulation) quindi è in tunnel mode. 
- ogni altro valore (TCP, UDP, ICMP, etc.) significa che è in transport mode: si ha una connessione endpoint-to-endpoint sicura. 
AH è incompatibile con NAT (Network Address Translation) come tra l'altro ESP, in quel caso autenticazione e cifratura non incorporano l’header IP modificato da NAT. 

# ESP : Encapsulating Security Payload
In aggiunta a quello che fornisce AH, ESP fornisce anche :
- Confidenzialità dei dati usando cifratura a chiave simmetrica
![ESP](/assets/sicurezza_informatica/pack_with_esp.png)<br>

## ESP con o senza autenticazione
![ESP autenticazione o senza autenticazione](/assets/sicurezza_informatica/esp_auth_noauth.png)<br>

## ESP in transport mode
Come in AH, Transport Mode incapsula solo il payload del datagram, progettato per host-to-host communication. L’IP header originale è lasciato, indirizzo IP sorgente e destinazione non sono cambiati.  
![ESP transport Mode](/assets/sicurezza_informatica/esp_transport_mode.png)<br>

## ESP in tunnel mode
A differenza di AH, la distinzione tra Tunnel o Transport mode, non è visibile visto che in tunnel mode (via next=IP) è parte del payload cifrato, non accessibile a chi ispeziona il pacchetto. 
![ESP tunnel Mode](/assets/sicurezza_informatica/esp_tunnel_mode.png)<br>

# Riassunto di tutte le modalità
![Riassunto modalità](/assets/sicurezza_informatica/riassunto_mod.png)<br>

# IPSec : Protezione contro Replay
Sia i pacchetti protetti con AH e ESP hanno un sequence number, è una protezione contro replay, quando si inizializza una SA il numero è zero, aumento per ogni pacchetto IP spedito. <br>
Il ricevente controlla che il sequence number sia contenuto in una finestra accettabile. 
![IPSec : Protezione contro Replay](/assets/sicurezza_informatica/protezione_contro_replay.png)<br>
Se un pacchetto ricevuto ha un sequence number :
- alla sinistra della finestra attuale: il ricevente rigetta il pacchetto 
- dentro la finestra attuale: il ricevente accetta il pacchetto 
- alla destra della finestra attuale: il ricevente accetta il pacchetto e scorre la finestra 
I pacchetti IP sono accettati se passano i controlli, la dimensione della finestra è minimo 32 pacchetti (64 il default).
![IPSec : Protezione contro Replay2](/assets/sicurezza_informatica/protezione_contro_replay2.png)<br>

# Sequence Number
IP Header non contiene un sequence number
- Quindi un replay di un pacchetto automatico è possibile
  - con un sequence number di 32 bit counter, ipotizzando un primo pack con SN=1, massimo SN = 2^32-1
- L'anti-replay è opzionale (ma attiva di default)

# IPSec acronimi
- Security Association (SA) 
  - Collezione di attributi associati ad una connessione, se l'host a e b devono parlarsi scrivo nel dettaglio i meccanismi di sicurezza utilizzati. E' asimmetrica: una SA per traffico in ingresso, un’altra per traffico in uscita. Simile ai cipher-suite in SSL. E' un concetto fondamentale per IPSEC, formato da tra parametri: SPI, Ip destinazione, identificatore del protocollo di sicurezza (AH o ESP). Coinvolge sia host to host, che host to intermediate router (security gateway), che security gateway to security gateway.
  - Unidirezionalità : SA deve essere nel SADB in entrambe le parti
    - ![Unidirezionalità SA](/assets/sicurezza_informatica/unidirection_SA.png)<br>
- Security Association Database (SADB)
  - Un database di SA, memorizza i tipi di protocollo di sicurezza per ogni SA, con i relativi parametri (es. quale cifratura; quali chiavi, durata della SA, Sequence number counter, etc.) 
- Security Parameter Index (SPI):
  - Un indice unico da 32 bit per ogni entry nel SADB, identifica la SA associata ad un pacchetto. Permette di avere multiple SA tra gli stessi due host, usato per cercare nel SADB a destinazione (la ricerca usa anche l’indirizzo delle destinazione e della sorgente
- Security Policy Database (SPD):
  - Memorizza le politiche usate per stabilire le SA, quali servizi di sicurezza sono forniti a quali pacchetti IP e in che modo.

## SPD - SABD example
![SPD - SABD example](/assets/sicurezza_informatica/spd_sadb_example.png)<br>

# Stabilire le Security Associations
Prima di processare ogni pacchetto bisogna stabilire una SA tra i due “cryptographic endpoints” SA può essere stabilita: 
- Manualmente
  - si suppone solo in configurazioni molto ristrette (es. due firewall in una VPN), deve essere svolta su ogni nodo: nodi partecipanti (i.e. traffic selector), AH o ESP [tunnel o transport mode] e algoritmi per la crittografia e le chiavi.
- Dinamicamente
  - Con un metodo standard di autenticazione e key management. 
Internet Key Exchange (IKE) definisce IPSec standard come protocollo di autenticazione e scambio di chiavi

# ISAKMP
Internet Security Association and Key Management Protocol: 
- definito in RFC 2408, definisce protocolli e procedure per la negoziazione dei parametri di sicurezza, 
- usato per stabilire Security Associations (SA) e chiavi crittografiche. 
- Fornisce solo il framework per l’autenticazione e lo scambio delle chiavi
  - i principali protocolli per lo scambio delle chiavi sono Internet Key Exchange (IKE) e Kerberized Internet Negotiation of Keys (KINK). 

# IKE
Internet Key Exchange:
- componente IPsec usato per avere mutua autenticazione e stabilire una SA (RFC 5996)
- permette di stabilire le sessioni IPSec grazie al suo meccanismo di scambio delle chiavi, usa la porta UDP 500. 
- Ci sono cinque modi per una negoziazione IKE: 
  - Due modi (aggressive e main modes)
  - Tre metodi di autenticazione (pre-shared, public key encryption, and public key signature)
IKE fornisce un modo per negoziare in modo automatico i parametri di sicurezza e derivare e apposite chiavi necessarie 
- Fornisce il processo per ri-creare, fare il refresh frequente delle chiavi per assicurare confidenzialità dei dati tra peer.
Le operazioni base si dividono in due fasi:
- IKE FASE 1
  - negozia i parametri ed il materiale per derivare le chiavi richieste per stabilire una ISAKMP Security Association (ISAKMPSA), ISAKMPSA viene usate per gli scambi futuri IKE e per settare un canale sicuro per negoziare IPsec SA nella fase 2. 
- IKE FASE 2
  - negozia i parametri e il materiale per derivare le chiavi richieste per stabilire due SA unidirezionali, le IPSEC SA sono usate per proteggere il traffico di rete durante a fase di Data transfer. 

## IKE FASE 1
-  Ha l'obiettivo di stabilire un canale sicuro tra due end point (canale che fornisce: autenticazione della sorgente, integrità, confidenzialità e protezione contro replay attack)
- Ogni applicazione ha diversi requisiti di sicurezza
  - in ogni caso si deve negoziare le politiche e scambiare le chiavi
  - si forniscono servizi base e si permette alle applicazioni di iniziare una sessione.
Esempio: i pacchetti spediti a mybank.com devono essere cifrati con AES e HMAC-SHA1 o tutti i pacchetti spediti to address www.forum.com devono usare come controllo di integrità HMAC-SHA1 (senza cifratura). 

Esistono due modi: Main Mode o Aggressive Mode, la prima protegge l’identità dei peer al contrario della seconda. 

## IKE FASE 1 : MAIN MODE
Main mode negozia una ISAKMP SA che sarà usata per creare IPsec SAs<br>
L'iniziatore manda una o più proposte all’altro peer (responder), il responder seleziona una proposta. Ci sono tre scambi di informazioni tra i peer IPSec per negoziare una ISAKMP SA che sarà usata per creare delle IPSec SA: 
1. ci si accorda per gli algoritmi e gli hash da usare per rendere sicuri le comunicazioni IKE facendo il match con ISAKMP SA in ogni peer. 
2. si usa Diffie-Hellman per generare un segreto condiviso per generare le chiavi e si passano dei nonce all’altra parte, firmati e ritornati per controllare l’identità. 
3. verifica dell’identità dell’altra parte (il valore di identità è IP address, FQDN, email address, DNS o un KEY ID form cifrato). 
Alla fine si ha il matching di ISAKMP SA tra i peer per fornire una connessione protetta per successivi scambi ISAKMP tra i peer IKE, l'ISAKMP SA è bi-direzionale e specifica valori per lo scambio IKE: il metodo di autenticazione, gli algoritmi di hash e cifratura, il gruppo Diffie-Hellman, durata della ISAKMP SA in secondi o kb, valori per la chiave condivisa per gli algoritmi di cifratura. 

![IKE FASE 1 : MAIN MODE](/assets/sicurezza_informatica/ike_main_mode.png)<br>

## IKE FASE 1 : AGGRESSIVE MODE
ottiene lo stesso risultato usando solo 3 pacchetti : 
1. primo pacchetto mandato dall’iniziatore contiene tutte le info per stabilire una ISAKMP SA: la chiave pubblica Diffie-Hellman (un nonce che l’altra parte firma) un pacchetto per l’identità 
2. il secondo pacchetto dal responder con tutti i parametri di sicurezza selezionati
3. il terzo pacchetto conclude l’autenticazione della sessione ISAKMP (conferma) 
Ha minori scambi, la debolezza è che alcune info nel aggressive mode sono scambiate fra le parti prima di avere un canale sicuro: è possibile sniffare la connessione e scoprire chi ha formato la SA, comunque più veloce del main mode. 

## IKE FASE 2
Ha l'obiettivo di stabilire una canale sicuro tra due end point, identificati da [IP, port] (es. [www.mybank.com, 8000]) o da pacchetti (es. tutti i pacchetti che vanno ad un determinato INDIRIZZO IP). <br>

Usa il canale sicuro stabilito nella fase 1, ha solo un modo: <br>
Quick Mode: negozia I parametri per la sessione Ipsec, all’interno della protezione offerta dalla sessione ISAKMP, tutto il traffico è cifrato secondo ISAKMP SA e genera SA per i due end points. 

![IKE FASE 2](/assets/sicurezza_informatica/ike_fase_2.png)<br>

## IKE Overview
![IKE Overview](/assets/sicurezza_informatica/ike_all_phase.png)<br>

# IPSec Policy
Le politiche in fase 1 sono definite in termini suite: algoritmo di cifratura, algoritmo Hash, metodo di autenticazione, gruppo per Diffie-Hellman, ecc. Può contenere anche altre info quali: durata della politica, ecc. <br>
Le politiche in fase 2 sono definite in termini di proposte: AH, ESP, IPComp con tutti gli attributi necessari (lunghezza della chiave durata ecc.) 

 ## Problemi con IPSec
troppo complicato, sono presenti molti modi per configurarlo => può essere configurato in modo insicuro, la sicurezza dei client è un problema e performance nell'implementazione IPv4. 