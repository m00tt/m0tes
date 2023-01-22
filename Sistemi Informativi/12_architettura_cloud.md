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