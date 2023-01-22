# Cloud computing
Principali servizi di Cloud Computing:
- Storage: inteso come spazio di memoria, scalabile.
- Computing power: potenza di calcolo

La maggior parte dei servizi, sono erogati in modalità <b>Pay-as-you-Go</b>, ovvero paghi esattamente per quanto consumi.

Modalità di erogazione dei servizi cloud:
- <b>SaaS</b>: Software as a Service
- <b>PaaS</b>: Platform as a Service
- <b>IaaS</b>: Infrastructure as a Service

## Caratteristiche del cloud computing
### Vantaggi
- Flessibilità e scalabilità
    - Uso on-demand delle risorse
    - Facilità di cambiamento del fornitore
    - Accesso real-time alle risorse
- Disponibilità di API: per permettere la comunicazione con l'infrastruttura cloud.
- Riduzione dei costi: la modalità Pay-as-you-Go permette di legare i costi all'effettivo utilizzo dell'infrastruttura (si passa da costi di investimento a costi operativi)
- Maggiore immediatezza: dato che l'infrastruttura è fornita da terze parti e non deve essere comprata
- Minor esigenza di competenze ICT interne.
- Indipendenza da dispositivi e dalla localizzazione
- Virtualizzazione delle risorse
- Semplificazione delle attività di manutenzione
- Incremento dell'affidabilità (sistemi di disaster recovery, backup e HA)
- Multi-tenancy, che permette di condividere risorse tra un grande numero di utenti, garantendo:
    - La centrallizzazione dell'infrastruttura
    - Gestione migliore dei picchi di carico
    - Aumento dell'efficienza

#### Svantaggi
- Rischio di perdita del controllo su alcuni dati sensibili
- Maggiore esposizione a possibili attacchi

## Tipologie di cloud
### Private cloud
L'infrastruttura cloud è ad uso esclusivo di una singola organizzazione. In questo modo si ha un pieno controllo sui vincoli contrattuali e pieno controllo sulla parte cloud, permettendo anche l'implementazione di proprie pratiche operative.

Al contrario, però, si ha una minore flessibilita, minore ridondanza dei dati ed una possibile minor resistenza ad attacchi.

### Community cloud
L'infrastruttura cloud è ad uso specifico di una comunità di consumatori che hanno obiettivi comuni.

I punti di forza sono:
- Condivisione dei requisiti, vincoli e rischi
- Ecosistema chiuso
- Possibilità di valutare l'inclusione di nuovi membri sulla base di relazioni di fiducia

I punti di debolezza sono:
- La community potrebbe crescere troppo rapidamente
- Difficile previsione dell'uso delle risorse
- Difficoltà dell'identificazione di una entità responsabile
- Necessità di indebolire i vincoli sul controllo degli accessi e autenticazione

### Public cloud
L'infrastruttura cloud è fornita in modo che sia di dominio pubblico.

Punti di forza:
- Sicurezza
- Velocità di ripristino
- Affidabilità e disponibilità
- Performance

Punti di debolezza:
- Rischio di non-conformità normativa
- Assenza di un controllo diretto sul controllo degli accessi
- Bersaglio privilegiato per attacchi
- Dipendenza dal provider

### Hybrid cloud
L'infrastruttura cloud è l'unione di 2 o più infrasttrutture cloud che rimangono entità separate, ma sono legate tra loro da tecnologia standardizzata o proprietaria che consente la portabilità dei dati e delle applicazioni.


# Cloud computing: nuovo concetto?
Il concetto che sta alla base del cloud computing era stato pensato negli anni '50: mainframe accessibili via thinclient.<br>
La vera novità è <b>Internet</b>.

# Cosa differenzia il Cloud Computing da ASP?
ASP = Application Service Provider

Quali sono le differenze tra cloud e ASP?
- Il cloud fornisce capacità di elaborazione come una utility, ASP fornisce applicazioni come utilities
- Il cloud è multi-tenants, mentre le applicazioni ASP vengono replicate per ogni singolo utente

# Cloud ed elettricità
Le aziende si affidano all'elettricità non meno che alle tecnologie informatiche. Eppure 
le aziende non hanno bisogno di un "Chief Electricity Officer" e di uno staff di professionisti altamente professionisti per gestire l'elettricità nelle loro aziende.

## Cosa differenzia il cloud e l'elettricità?
Nicholas Carr, blogger indipendente, ha scritto:

A livello puramente economico, le somiglianze tra l'energia elettrica e la tecnologia dell'informazione sono ancora più evidenti. Entrambe sono quelle che gli economisti chiamano tecnologie di uso generale.<br>Le tecnologie di uso generale, o GPT, sono considerate come piattaforme su cui strumenti o applicazioni possono essere costruiti.

Spostando i datacenter più vicini alla produzione di energia, il cloud computing crea ulteriori risparmi sui costi. Questi risparmi si ottengono quando 
i datacenter sono situati in prossimità di fonti di energia a basso costo, come le dighe idroelettriche.

Oltre ai suoi punti di forza, tuttavia, l'analogia con le utenze elettriche presenta anche tre debolezze tecniche e tre debolezze del modello di business.

- <b>Il ritmo dell'innovazione</b>: il ritmo dell'innovazione nella generazione e distribuzione di energia elettrica avviene su scala di decenni o secoli. Al contrario, la legge di Moore si misura in mesi.

- <b>Limiti di scalabilità</b><br>
La rapida disponibilità di istanze server aggiuntive è un vantaggio centrale del cloud computing, ma ha i suoi limiti.<br>
Uno di questi è che alcuni problemi e processi devono essere affrontati con altre architetture di elaborazione, memoria e archiviazione, quindi il semplice affitto di altri nodi non sarà d'aiuto.

Nel frattempo, le aziende di una certa dimensione possono ottenere il meglio di entrambi i mondi implementando cloud privati. Intel, ad esempio, sta consolidando i propri data center da oltre 100 a circa 10. Nel 2008 il totale è sceso a 75, con un risparmio di 95 milioni di dollari.

Anche se il modello di utilità viene propagandato per l'informatica, l'approccio altamente centralizzato sta diventando meno efficace per l'elettricità stessa. Inoltre, molte aziende generano la propria elettricità, per le stesse ragioni per cui continueranno a mantenere alcune classi di IT all'interno dell'azienda: affidabilità, vantaggio strategico o visibilità dei costi.

- <b>Latenza</b>
Una delle poche leggi immutabili della fisica è la velocità della luce. Di conseguenza, la latenza rimane una sfida formidabile.<br>
Per questo motivo è stato introdotto il modello di Edge computing, quindi collocare macchine in punti strategici in modo da abbattere le distanze che renderebbero la latenza un fattore determinante.

Però, per molte classi di applicazioni, le prestazioni, la convenienza e le considerazioni sulla sicurezza impongono che l'elaborazione sia locale. Allontanare i data center dai loro clienti può far risparmiare sui costi dell'elettricità, ma questi risparmi sono spesso superati dai costi della latenza.


Per quanto importanti siano le differenze tecniche tra elettricità e cloud computing, le differenze del modello di business sono ancora più profonde.
- <b>Complementarietà e co-invenzione</b>
Inizialmente, le catene di montaggio e i processi produttivi non sono stati riprogettati per sfruttare i vantaggi dell'elettricità: i grandi motori a vapore centrali sono stati semplicemente sostituiti da grandi motori elettrici e poi collegati agli stessi vecchi alberi a gomito e ingranaggi. Solo con la reinvenzione del processo produttivo è stato realizzato il potenziale dell'elettrificazione.

Le aziende che si limitano a sostituire le risorse aziendali con il cloud computing, senza cambiare nient'altro, sono destinate a non sfruttare appieno i vantaggi della nuova tecnologia.

- <b>Lock-in e interoperabilità</b>
I problemi di lock-in con l'elettricità sono stati affrontati molto tempo fa con la regolamentazione dei monopoli.

Affinché l'enterprise computing si comporti come la tensione di rete, sarà necessaria una gestione dei dati radicalmente diversa da quella prevista dalla roadmap tecnologica di chiunque. A seconda dell'applicazione, della sua progettazione e dell'uso che se ne intende fare, le offerte cloud non saranno intercambiabili tra i vari cloud provider.

- <b>Sicurezza</b>
I problemi di sicurezza legati al cloud computing non hanno analogie con l'elettricità. Nessun ente normativo o di polizia controllerà gli elettroni di un'azienda, ma i processi relativi ai dati dei clienti, ai segreti commerciali e alle informazioni governative classificate sono tutti soggetti a severi requisiti e standard di verificabilità. Le risorse tipicamente condivise e dinamiche del cloud computing (tra cui CPU, rete e così via) riducono il controllo per l'utente e pongono nuovi e gravi problemi di sicurezza non riscontrabili nell'elaborazione on-premise dietro firewall.

## Conclusioni
Se il modello delle utility fosse adeguato, le sfide del cloud computing potrebbero essere risolte con soluzioni simili a quelle dell'elettricità, ma non è così. La realtà è che il cloud computing non può raggiungere la semplicità plug-and-play dell'elettricità, almeno non finché il ritmo dell'innovazione, sia all'interno del cloud computing stesso, sia nella miriade di applicazioni e modelli di business che esso consente, continuerà a essere così rapido. 

Sebbene le aziende elettriche siano considerate un modello di semplicità e stabilità, anche questo settore non è immune dal potere di trasformazione dell'IT.