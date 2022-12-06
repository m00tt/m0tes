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

//add Image

## TCP Packet
//add Image


## TCP Flag
| FLAG | Spiegazione |
| ---- | ----------- | 
| SYN  |richiesta di connesione, sempre il primo pacchetto di una comunicazione |
| FIN  | indica l'intenzione del mittente di terminare la sessione in maniera concordata |
| ACK  | conferma del pacchetto precedente |
| RST  | reset della sessione |
| PSH  | operazione di push, i dati che vengono inviati al destinatario non dovrebbero essere bufferizzati |
| URG | dati urgenti che vengono inviati con precedenza sugli altri |

## TCP Handshake
Le connessioni TCP vengono stabilite tramite un handshake a 3 vie.

//add Image (internet)

- Il server ha generalmente un listener passivo, in attesa di una richiesta di connessione
- Il client richiede una connessione inviando un pacchetto SYN
- Il server risponde inviando un paccheto SYN/ACK, indicando un riconoscimento per la connesione
- Il client risponde inviando un ACK al serer stabilendo così la connessione

2.23
