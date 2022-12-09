# TCP (Transmission Control Protocol)
TCP permette la trasmissione dati attraverso una connesione e da anche prerogative diverse riguardo la consegna dei dati stessi.<br>
Come funziona?
- Mittente : suddivide i dati da inviare in pacchetti
  - Ad ogni pacchetto è allegato un numero di sequenza  
- Destinatario : riassembla i pacchetti nell'ordine corretto grazie al numero di sequenza
  - Riconosce la ricezione, i pacchetti persi vengono rispediti
Lo stato di connessione viene mantenuto su entrambi i lati.

## Il livello di trasporto
Processi a livelli più alti possano comunicare in maniera corretta senza alcun problema.<br>
La responsabilità pricipale del livello di trasporto è la consegna dei dati dai processo a processo.<br>
Tra due host ci possono essere piu canali di comunicazione in contemporanea, i singoli processi in esecuzione su un dato host sono identificati dai loro numeri di porta, che acquisiscono tramite l'API socket.<br>
Un Socket è un identificato da una combinazione di numeri IP e numeri di porta.<br>
Un segmento di scambio fra due processi necessita di 4 numeri (IPsorgente;NPsorgente : IPdestinazione,NPdestinazione)

## Porte
Un numero di porta è un numero intero a 16 bit che assume un valore compreso tra 0 e 65535.<br>
Esiste un meccanismo standard che permette di assegnare il numero di porta ai servizi offerti/ai diversi processi -> IANA (Internet Assigned Number Authority), essa è l'autorità ufficiale che riserca i numeri di prota a varie applicazioni.<br>
I numeri di porta che vanno da 0 a 1023 ("Numeri di porta Noti") sono riservati ad applicazioni come FTP (trasferimento file), HTTP, ecc.

## FAQ su Porte e Processi
Un numero di porta di destinazione identifica un processo su un host di destinazione.<br>
- È sempre vero che ogni processo ha un suo numero di porta univoco?
  - No, due processi possono essere in ascolto sullo stesso numero di porta.
- Esiste un singolo docket di rete per processo o un processo può utilizzare più di un socket?
  - Può esserci più di un socket per processo, ciascuno con un identificatore univoco.
- Un processo di destinazione può ricevere pacchetti da più host di origine su un singolo socket?
  - Si, questo è possibile con UDP, TCP non lo consente
- In che modo IANA impone l'uso esplicito di numeri di porta noti da parte di determinati tipi di applicazioni?
  - Il fatto che i numeri di porta siano "riservati" non significa che sia impossibile utilizzarli in altre applicazioni
  - Vietare l'uso della porta di destinazione 22 non significa vietare SSH, ma impedire che client e server possano accordarsi sull'uso di tale porta.

# TCP and UDP
I due protocolli del livello di trasporto utilizzati in internet sono :
- TCP : Transmission Control Protocol 
- UDP : User Datagram Protocol : Maggiore velocità ma non ha le garanzie del TCP, si potrebbero perdere pacchetti

| Application | Application Layer Protocol | Protocollo di trasposrto sottostante |
| -----------  | ------------------------- | ------------------------- |
| e-mail | SMTP | TCP |
| remote terminal access | Telnet | TCP |
| Web | HTTP | TCP |
| File Transfer | FTP | TCP |
| Straming Multimedia | HTTP, RTP | TCP or UDP |
| Internete Telephony | SIP, RTP | tipically UDP |

## TCP Packet
![TCP Packet](/assets/sicurezza_informatica/TCP-Packet.png)<br>


## TCP Flag
| FLAG | Spiegazione |
| ---- | ----------- | 
| SYN  | Richiesta di connesione, sempre il primo pacchetto di una comunicazione |
| FIN  | Indica l'intenzione del mittente di terminare la sessione in maniera concordata |
| ACK  | Conferma del pacchetto precedente |
| RST  | Reset della sessione |
| PSH  | Operazione di push, i dati che vengono inviati al destinatario non dovrebbero essere bufferizzati |
| URG | Dati urgenti che vengono inviati con precedenza sugli altri |

## TCP Handshake
Le connessioni TCP vengono stabilite tramite un handshake a 3 vie.

![3 Way Handshake](/assets/sicurezza_informatica/3-Way-Handshake.png)<br>


- Il server ha generalmente un listener passivo, in attesa di una richiesta di connessione
- Il client richiede una connessione inviando un pacchetto SYN
- Il server risponde inviando un paccheto SYN/ACK, indicando un riconoscimento per la connesione
- Il client risponde inviando un ACK al serer stabilendo così la connessione

### Spedizione Dati

Dopo aver fatto l'handshake comincia la spedizione dei dati:<br>
![Spedizione Dati](/assets/sicurezza_informatica/Spedizione-Dati-After-3Way.png)<br>
1. A spedisce a B 100 byte
   1. Primo byte ha come numero di sequenza 1000
   2. B conferma la ricezione dei byte ricevuti in precedenza fino al 4999 ed indica che si aspetta che il prossimo byte trasmesso sia quello con numero di sequenza 5000, inoltre incremente il numero di ack di 100 byte (dati ricevuti) : seq-num precedente + dati = 1000+100
2. B spedisce ad A 250 byte
   1. Primo byte ha come numero di sequenza 5000
   2. A conferma la ricezione dei byte ricevuti in precedenza fino al 1099 ed indica che si aspetta che il prossimo byte trasmesso sia quello con numero di sequenza 1100, inoltre incrementa il numero di ack di 250 = seq-num precedente + dati = 5000 + 250
3. A spedisce a B 150 byte
   1. Primo byte ha come numero di sequenza 1100
   2. B conferma la ricezione dei byte ricevuti in precedenza fino
al 5249 si aspetta che il prossimo byte trasmesso sia quello con numero di sequenza 5250.
4. B non ha dati da spedire, si limita solo a confermare i
dati ricevuti. Questo pacchetto non verrà confermato
in quanto non contiene dati

### TCP Data Transfer
Durante l'inizializzazione della connesione utilizzando l'handshake a tre vie, vengono scambiati i numeri di sequenza inziali.<br>
L'intestazione TCP include un checksum a 16 bit dei dati e di parti dell'intestazione, incluse l'origine e la destinazione.
- Il riconoscimento o la sua mancanza viene utilizzato da TCP per tenere traccia della confestione della rete e del flusso di controllo.
- Le connessioni TCP vengono terminate in modo pulito con un handshake a 4 vie
  - Il client : che desidera terminare la connessinoe invia un messaggio di FIN al server
  - Il server : risponde inviando un ACK
  - Il server : invia un FIN
  - Il client : invia un ACK e la connessione viene terminata

![Chiusura Connesione TCP](/assets/sicurezza_informatica/Chiusura-Connessione-TCP.png)<br>

## Problemi intrinsici in TCP/IP
- Non c'è autenticatione fra le parti, chiunque può fingersi Alice o Bob e intromettersi nel protocollo dato che chiede solo l'indirizzo IP del destinatario e del mittente
- I controlli d'integrità sono banali
- Si difende la disponibilità della rete dalla congestione, ma non la possibilità di connettersi ad un determinato noto

# IP Spoofing
L'IP Spooging è un tentativo da parte di un intruso di inviare pacchetti da un indirizzo IP che sembrano provenire da un altro. 
- Il campo SRC dello header IP è falsificabile senza particolare difficoltà.<br>

Le autenticazioni locali su indirizzi IP sono insicure, sopratutto all'interno di una rete locale.<br>
Fra l'altro la presenza di indirizzi IP duplicati può causare Denial Of Service.<br>
Dal punto di vista dell'attaccante : 
- Se l'IP sorgente è falso le risposte andranno al vero noto titolare (Attaccante spoofa IP di Alice, invia un pacchetto, ma la risposta arriva ad Alice)
- Spoofare l'IP non è sufficiente per iniziare una connessione TCP

## TCP Connection Spoofing
Ogni connessione TCP ha uno stato associato
- Numero di sequenza, numero di porta

Lo stato del TCP è facile da indovinare
- Numeri di porta sono standard e i numeri di sequenza prevedibili

Si possono iniettare pacchetti in connessioni esistenti
- Se l'aggressore conosce il numero di sequenza iniziale e la quantità di traffico, può indovinare probabilmente il numero corrente
- Indovinare un numero di seuqenza a 32 bit non è pratico MA
  - La maggior parte dei sistemi accetta grandi finestre di numeri di sequenza (per gestire le perdite dei pacchetti), quindi invia un flusso di pacchetti con proabili numeri di sequenza

## Initial Sequence Number
Intorno agli anni 80/90 si è visto che nei documenti standard RFC793 l'ISN (Initial Sequence Number) va incrementato circa 1 volta ogni 4 microsecondi per evitare confusioni con connessioni duplicate.<br>
In realtà i sistemi operativo non implenetarono questo standard correttamente e quindi prendevano dei sequence number abbastanza prevedibili (prendevano un vecchio sequence number e lo implementava di una certa quantità) = punto debole che l'attaccante poteva sfruttare.<br>
Quindi sono state rilasciate nuovi standard dove:
- ISN = M + FS dove:
  - M : contatore incrementato ogni 4 microsecondi
  - FS : funzione Hash che prende in Input (Indirizzo IP Mittente, Porta Mittente ; Indirizzo IP Destinazione, Porta Destinazione )

## Forme di IP spoofing
Esistono due forme base di IP spoofing:
- Blind Spoofing
  - Attacca da qualsiasi fonte - non sono in grado di vedere le risposte che il destinatario sta rilasciando
- Non-Blind Spoofing
  - Attacca dalla stessa sottorete - vedo in maniera dettagliata cosa succede, posso intercettare tutti i pacchetti

### Blind Spoofing
<b>Goal:</b> cercare d'impersonare un host qualcunque su internet, non facente parte della sottorete in cui si trova.<br>
<b>Blind:</b> l'attaccante non ha nessuna possibilità di vedere i pacchetti mandati in risposta ai pacchetti spoofati che ha spedito<br>
- Questi utlimi infatti saranno indirizzati all'host che egli sta impersonando, impedendogli quindi di venire a conoscenza di ACK number e Sequence number corretti per continuare la connesione

Alcuni server danno un accesso privilegiato a degli host fidati
- Esempio : rlogin senza password o servizi simili
- Controllo fatto sul numero IP sorgente della connessione

Se un attaccante riuscisse ad aprire una connessione spoofando il proprio IP potrebbe lanciare dei comandi al server facendogli credere di essere l'host dato

#### Attacco Blind
L'attacci su sviluppa in più fasi :
1. L'attaccante avrà una fase di ricognizione per cercare di capire tutte le caratteristiche della macchina che si vuole spoofare.<br>
Alice e bob stanno aprendo una connesione potenzialmente attaccabile, l'attacante può capire come si comporta Alice, quindi quali sono le caratteristiche con la quale vengono gestite ACK number e Sequence number
2. L'attaccante provocherà un attacco di SYN flood verso Bob mettendolo fuori uso
3. L'attaccante s'intrometterà nella connessione creato paccheti spoofati con l'indirizzo di Bob (quindi impersonando Bob)

Il protocollo TCP/IP richiede che i numeri di "Acknowledgement" vengano inviati attraverso le sessioni
- Si assicura che il client riceva i pacchetti del server e viceversa
- È necessario disposrre della giusta sequenza di numeri di riconoscimento per falsificare un'identità IP

#### IP Spoofing
Lo spoofing IP consiste in diversi passaggi e utilizza sia
lo spoofing dell'indirizzo IP che la previsione del numero di sequenza TCP.<br>
Con l'aiuto della tecnica di previsione del numero di sequenza TCP, un hacker potrebbe parlare con un demone rlogin anche senza dover ricevere il numero di sequenza del server restituito con una stretta di mano a tre vie.

<b> Normal Remote Login Session : </b>
| Route | Operation |
| ----- | --------- |
| C->S  | SYN (ISNc)|
| S->C  | SYN (ISNc), ACK (ISNc) |
| C->S  | ACK (ISNc)|
| C->S  | data      |
| S->C  | data      |

<b> Spoofed Attack : </b>
| Route | Operation | Spiegazione |
| ----- | --------- | ----------- |
| X->S  | SYN (ISNx), SRC = C| L'attaccante per intromettersi deve creare il pacchetto spoofato nel quale il sorgente è il client (SRC = C) e deve inserire anche il giusto sequence number |
| S->C  | SYN (ISNc), ACK (ISNx) | Quando il server vede la richiesta, il server produrrà un pacchetto destinato al client, <b> NON VISIBILE DALL'ATTACCANTE </b> | 
| X->S  | ACK (ISNc), SRC = C| L'attaccante deve creare una risposta ACK da inviare al server per la confermare la ricezione del pacchetto prima (<b> L'attaccante non ha visto quel pacchetto </b>) |
| X->S  | ACK (ISNc), SRC = C, nasty-data | Attaccante continua la comunicazione inviando codice malevolo |

#### IP Spoofing Steps
1. Viene scelto un host vittima e viene studiato il comportamente, come ad esempio di quali host la vittima si fida
2. Dopo aver individuato l'host attendibile, esso viene disabilitato, probabilmente da SYN flooding per arrestare la macchina o bloccare tutto il normale traffico di rete, questo perché se la macchina fidata riceve dei pacchetti non voluti essa risponderò con pacchetti RST (reset)
3. Successivamente,  un normale pacchetto di richiesta di connessione TCP viene inviato all'host vittima e si ottiene un numero di sequenza valido dall'host vittima, un attaccante potrebbe indovinare qual'è il prossimo numero di sequenza
4. L'attaccante invia quindi un pacchetto SYN con l'host attendibile come indirizzo IP di origine.
Anche se il pacchetto SYN_ACK restituito dall'host vittima non è
riuscito a raggiungere l'attaccante, può comunque continuare la connessione inviando il messaggio ACK finale con il numero di sequenza corretto generato dal passaggio precedente.

<b> Se l'attaccante non è riuscito a mettere fuori uso la macchina fidata : </b>

5. Il pacchetto SYN_ACK restituito andrà al vero host C
6. l'host C risponderà immediatamente con un pacchetto RST e l'attacco fallirà perché la connessione prevista verrà interrotta non appena l'host vittima avrà ricevuto questo pacchetto.

Pertanto, un attacco SYN flooding viene sempre eseguito come primo
passaggio in "IP Spoofing"

#### ISN Prediction
In questo tipo di attacco l'attaccante non si trova nella stessa rete della vittima quindi anche se può inviare pacchetti spoofati (si può tentare di instaurare connessioni con il server) apparendo la vittima, tutti i pacchetti di risposta saranno inviati alla vittima e l'attaccante non può intercettarli facilmente. Inoltre non conoscendo le risposte dal server non si possono calcolare i sequence number e quindi proseguire il dialogo. Per aggirare questo problema si può tentare di predire il sequence number del server (TCP sequence prediction attack).
L'attaccante manda qualche pacchetto SYN non spoofato per vedere e analizzare le risposte del server, come ad esempio
- Regola dei 64k :  che consiste nell'aumentare ad ogni secondo di una certa costante il sequence number (rispetto a quello utilizzato per la connessione precedente) e aggiungere un altro numero costante se si cambia la connessione
- Campionamenti su una serie di pacchetti
- Aumentando di una costante il sequence number a ogni intervallo di tempo prefissato
- Differenza tra i vari tempi registrati ottenendo l'intervallo di tempo trascorso tra due generazioni del ISN
- Divisione tra il risultato ottenuto e la differenza tra ISN per ottenere l'incremento per ogni unità di tempo

L'attaccante non ha la sicurezza matematica di essere riuscito ad aprire la connesione<br>
Esempio se stava cercando di aprire una connessione con il servizio rlogin non gli sarà richiesta nessuna password per entrare e con tutta probabilità quindi avrà accesso ad una shell.<br>
Il passo successivo solitamente sarà quello di inviare il comando : 

``` bash
echo "+ +" > .rhosts
```
che consente di aggiugere una riga nel file .rhosts e questo significa che rende possibile l'accesso al sistema vittima senza password a chiunque faccia un tentativo di rlogin

### Non-Blind IP Spoofing
IP Spoofing senza conoscere intrinsecamente lo schema della sequenza di riconscimento
- Fatto nella stessa sottorete
- Utilizzare uno sniffer di pacchetti per analizzare lo schema della sequenza

Basta utilizzare uno sniffer per vedere cosa sta succedendo, e quindi capire quali sono i sequence number, si ha una visione aperta sul traffico, grazie a questo il creare pacchetti spoofati risulta molto più facile.<br>
Si utilizzano sniffer di pacchetti per analizzare lo schema della sequenza, i packet sniffer intercettano i pacchetti di rete, alla fine decodifica e analizza i pacchetti inviati attraverso la rete e determinare il modello della sequenza di riconoscimento dai pacchetti. Con queste informazioni è possibile inviare messaggi al server con l'indirizzo IP del client effettivo e con un numero di riconoscimento sequenziato validamente. 

#### Sniffer
I packet sniffer "leggono" le informazioni che attraversano una rete
- I packet sniffer intercettano i pacchetti di rete, possibilmente utilizzando il poisoning della cache ARP
- Possono essere utilizzati come strumento legittimo per analizzare la rete

I packet sniffer permettono di :
- Monitorare l'utilizzo della rete
- Filtrare il traffico di rete
- Analizzare i problemi di rete

Può anche essere utilizzano in modo dannoso : 
- Rubare informazioni (come ad esempio password, conversazioni, ecc)
- Analizzare informazioni di rete per prepare un attacco

I packet sniffer possono essere sia software sia hardware

# DoS by Connection Reset
Oltre all'IP Spoofing, durante gli scambi di dati del TCP è possibile effetuare un DoS (Denial Of Service) creando un pacchetto ad hoc di reset.<br>
Non ci sono limitazioni se non creare il giusto pacchetto di reset, bisogna usare il sequence number valido e bisogna intromettersi nella comunicazione tra Alice e Bob e mandare un pacchetto di reset<br>
Se un utente malintenzionato può indovinare il numero di sequenza corrente per unaconnessione esistente, può inviare il pacchetto di ripristino per chiuderla, particolarmente efficace contro la connessione di lunga durata, basta inviare pacchetto FIN.

# TCP Session Hijacking
Un'attaccante può fare un Session Hijacking ovverò può intereccettare e dirottare la comunicazione, fingendosi un'altra persone (IP SPOOFING) può inserirsi in una comunicazione lecita e provocare malfunzionamenti.<br>
- L'attaccante contatta il bersaglio (il server) con un SYN e quest'ultimo risponde con un SYN/ACK : metodo utilizzato per capire le caratteristiche del server come ad esempio il server crea il Sequence Number.
- Hijacking : attaccante manda un indirizzo spoofato al server (manda un pacchetto con un indirizzo che non è il suo, quindi impersonando A) il server risponderà con SYN/ACK ad A.
- Se l'attaccante è in grado di capire qual'è l'evenetuale risposta del server (l'attaccante non la vede perché va verso l'host spoofato) può creare un messaggio di risposta da parte dell'indirizzo spoofato verso il server e poi inviare payload dannoso in quanto è stato concluso il 3-Way-Handshake

![TCP Session Hijacking](/assets/sicurezza_informatica/tcp-session-hijacking.png)

# ACK Storm
Gli attacchi ACK Storm si basano sul fatto che nel specifiche del procotollo tcp è scritto 
- Quando si riceve un pacchetto con ACK più grande di quello inviato dal client ricevente
- Il client deve inviare nuovamente l'ultimo pacchetto ACK inviato all'altro lato e scartare quello ricevuto.

L'attacco base consiste in:
- Preleva un pacchetto da una connessione TCP tra un client e un server
- Genera due pacchetti, ciascuno indirizzato a una parte e con l'indirizzo del mittente dell'altra parte (cioè contraffatto)
  - I pacchetti devono essere all'interno delle finestre TCP di entra,bi i lati
  - I pacchetti dovrebbero avere almeno un byte di dati
- Inviare i pacchetti contemporanemante al client e al server

La connessione entrerà quindi in un ciclo infinito di invio di pacchetti ack avanti e indietro tra entrambe le parti

![ACK Storm](/assets/sicurezza_informatica/ack-storm.png)

- A accetta pacchetti con SEQ = 1000 e ACK = 2000
- B accetta pacchetti con SEQ = 2000  e ACK = 1000
- Eve (il nemico) manda un pacchetto sbagliato sia ad Alice che a Bob di 10 byte (impersonando Alice e Bob)
- Il contatore di ACK viene incrementato di 10 in quanto il pacchetto contiene 10 byte
- Alice e Bob si disallinenao, in quanto quando i pacchetti arrivano a destinazione contengono un ACK superiore al sequence number (B riceve ack 2010 ma ha seq 200, A riceve 1010 ma ha seq 1000)
- Alie e Bob secondo lo standard inviano quidni l'ultimo ACK inviato da loro, creando cosi un loop

Punti chiave:
- L'iniezione di pacchetti avviene in una sessione attiva
- I pacchetti falsificati inviati devono essere ricevuti sia dal client che dal server, rientrando nella finestra di congestione TCP di entrambi
- Quando la connesione tra client e server è inattiva, i numeri di sequenze rimangono gli stess
- Quando si trasferiscono file di grandi dimensioni su TCP, la finestra di congestione TCP diventa grande
- L'attaccante deve conoscere l'IP esterno e la porta che il NAT assegna al computer interno quando accede a Internet

Contromisure
- Per prevenire attacchi di ACK Storm è necessaria una piccola modifica nell'implementazione (o standard) TCP
- Quando si riceve un pacchetto contente un campo ACK superiore al numero di sequenza del destinatario il pacchetto deve essere scartato e non deve essere inviata alcuna risposta
- L'utilizzo di una rete wirless crittografata come standard ridurrà in modo significativo le capacità di intercettazione
- Il filtraggio degli ACK duplicati TCP può neutralizzare i due pacchetti

# DoS Attack
Goal : Escludere un nodo o un servizio (con il minimo sforzo possibile)<br>
- "Amplification Attack"
  - La quantità di dati generati dall'attaccante è inferiore a quella che colpisce la vittima.<br>
  - In genere sono due i tipi di attacchi basati sull'amplificazione:
  - DoS Bug
    - Difetto di progettazione
  - DoS Flood
    - Si usano delle bot-net
- "Reflection attack"
  - Un attaccante, invece di colpire diretamente la vittima, dirige il suo traffico verso un host intermedio(reflector) e poi questo lo dirige verso la vittima.
  - Nell'IP Spoofing l'attaccante genera un pacchetto con l'indirizzo sorgente della vittima e lo manda al server che agisce da reflector
    - A causa dello spoofing, la risposta andrà alla vittima
    - La vittima quindi riceverà pacchetti provenienti dal reflector e non riuscirà a risalire all'attaccante vero

## ACK Reflection Attack
![ACK Reflection Attack](/assets/sicurezza_informatica/reflection-attack.png)


- Il pacchetto TCP SYN verso il reflector con IP source della vittima
- Il reflector risponde con SYN/ACK
- La vittima verrà quindi inondata di apcchetti TCP fuori sequenza provenienti da un server web pulito
  - Non c'è modo di distinguere i SYN spoofati dai SYN reali e quindi non c'è modo, per il reflector, di proteggersi
- Semplicemente il SYN generator genera tanti pacchetti SYN Spoofati(tutti con l'IP della vittima) diretti a diversi server web, quando il server vede arrivare un SYN risponde con SYN/ACK alla vittima, quindi, quest'ultima viene inondata di SYN/ACK, questo provoca un backscatter<br>
Backscatter(radiazione di ritorno)
- La vittima riceve TCP SYN con IP sorgenti diversi (spoofati)
- La vittima risponde normalmente con dei pacchetti, ciascuo ad ogni IP sorgente ricevuto
  - Backscatter (anche nelle mail indica la ricezione, talvolta massiva, di messaggi di rimbalzo a seguito di eventi connessi allo spam)


## TCP SYN Flood
Un attacco di SYN Flood è un inondazione di pacchetti SYN verso un determinato target, ovvero un servizio di rete che deve essere disponibile ma che dopo questo attacco inizierà a rifiutare nuove connessioni e quindi passa in uno stato di "non-disponibile".<br>
Un server web sta in ascolto sulla porta 80 in attesa di connessioni, l'attaccante genera tantissimi SYN con indirizzo spoofato (casuale) e tutti questi pacchetti hanno come destinazione la porta 80 di quel server che sto attaccando.<br>
La cosa di backlog del server si riempe e a una certa esaurisce lo spazio disponibile per raccogliere i vari SYN e non risponde più, quindi rifiuta altre connessioni<br>
Questo è un esempio di DoS Bug

### Scenari di attacco e parametri
Gli attacchi di SYN Flood sono di diversa natura
![Scenari di attacco e parametri](/assets/sicurezza_informatica/scenari-attacco-parametri.png)
- Attacchi diretti : Attaccante attacca direttamente la vittima mandano pacchetti SYN e SYN/ACK
- Attacchi con indirizzo spoofato : attaccante crea un certo numero di pacchetti con indirizzi cambiati, causali
- Attacchi distribuiti : Grazie all'utilizzo di BotNet o dei server che possono essere controllati dall'attaccando facendo attacchi di SYN Flood verso la vittima

Affinché l'attacco sia efficace devo conoscere il sistema operativo della vittima e le sue caratteristiche (esempio la dimensione della backlog utilizzata, ecc)<br>

### Contro Misure
- Aumentare il TCP della Backlog
- Ridurre il tempo di time-out dei SYN-RECEIVED
- SYN Cache
- SYN Cookies (Soluzione Migliore)

## Prevenzione dell'attacco DoS
DoS è causato da uno stato di allocazione simmetrico
- Il server apre un nuovo stato ad ogni richiesta di connessione.

Idea : postporre lo stato<br>
Cookies permetto al ricevente di rimanere stateless finche l'initiator produce almeno due messaggi
- Lo stato è memorizzato in un cookie (IP Addresses e Porte) e mandato all'initiator
- Se l'initiator risponde, il cookie viene rigenerato e comparato col cookie restituito dall'initiator

## SYN Cookies
![SYN Cookie](/assets/sicurezza_informatica/syn-cookie.png)<br>
Client manda SYN <br>
Il server quando manda SYN e ACK calcola un determinato valore tramite una funzione Hash (quella verde nella foto) se la connessione è legittima (ovvero se c'è un client legittimo) mi risponderà Cookie + 1 <br>
A quel punto il server farà i suoi controlli sul cookie e solo in questo caso allora la connesione è valida.<br>

Idea : usare una chiave segreta 
- Il server risponde al client con il cookie SYN/ACK che ha i seguenti campi:
  - T = Contatore a 5 bit incrementato ogni 64 secondi
  - L = MAC(key) (Source Addr, Source Port, Destination Addr, Destination Port, Sequence Number Client, T)
- Client onesto risponderà :
  - ACK(AN = Sequence Number Server, SN = Sequence Number Client +1)
- Tramite questo ACK il server può ricostruire tutte le informazioni che contiene il SYN/ACK che ha mandato prima al client e vedo se tutto corrisponde, se positivo allora il server alloca lo spazio

## Altra contromisura
Se la SYN queue è piena, si può cancellare a caso una delle query
- Le connessioni legettime hanno una chance di essere completate
- Fake Addr satanno eventualmente cancellati

## Prolexic Proxy
Esempio di un altra contromisura, il compito del Prolexic Proxy è effettuare un filtraggio sui SYN e SYN/ACK, immagazzinava questi SYN calcolando il syn/ack e effettuava il forwarding solo quando la connesione era ritenuta legittima.<br>
![Prolexic Proxy](/assets/sicurezza_informatica/prolexic-proxy.png)

## Attacco robusto : TCP con flood
Esempio una botnet con 20000 macchine che ricevono il comando di aprire una connessione con un server web sono effettivamente tutte richieste legittime (l'handshake viene completato normalmente) è come se 20000 utenti si connettessero al server web.
Questa tecnica aggirerà il proxy di protezione dal SYN Flooding











