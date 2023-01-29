# Infrastruttura IT
Insieme delle risorse tecnologiche condivise che fornisce la piattaforma per le applicazioni utilizzate dal sistema informativo di un'organizzazione.

## Cosa comprende?
- <b>Piattaforme hadware</b>: utilizzate per vari scopi, dalla gestione e salvataggio dei dati al collegamento tra clienti e fornitori

- <b>Applicazioni software</b>: forniscono supporto alle varie attività dell'organizzazione

- <b>Servizi di telecomunicazione</b>: che forniscono la connettività

- <b>Servizi di gestione dati</b>: che memorizzano e gestiscono i dati dell'organizzazione e che consentono di analizzarli

- <b>Servizi di gestione dell'infrastruttura</b>: 
    - Gestione degli aggiornamenti fisici e non
    - Servizi di management della funzione IT (ICT) che pianificano e sviluppano l'infrastruttura IT

- <b>Servizi IT standard</b>: che definiscono le politiche di uso delle tecnologie IT

- <b>Servizi di fomazione IT</b>: che si occupano della formazione del personale e dei manager

- <b>Servizi di ricerca e sviluppo IT</b>: che forniscono all'organizzazione indicazioni su progetti IT futuri e su quali investimenti potrebbero aiutare l'organizzazione a differenziarsi rispetto ai competitors

# Portafoglio applicativo
È l'insieme delle applicazioni informatiche in uso nell'organizzazione.

E comprende:
- Portafoglio direzionale
- Portafoglio istituzionale
- Portafoglio operativo

## Struttura

![Portafoglio applicativo](/assets/sistemi_informativi/portafoglio_applicativo.png)

### Portafoglio istituzionale
Comprende le applicazioni per i processi di supporto, che includono:
- Amministrazione
    - Contabilità
    - Gestione finanziaria
- Gestione delle risorse
    - Risorse umane
- Gestione delle infrastrutture
    - Impianti
    - Immobili


### Portafoglio direzionale
I sistemi informativi direzionali hanno una duplice finalità:
- Dare un feedback informativo ai decisori su una serie di indicatori
- Assistere i decisori durante il processo decisionale (DSS)

Si parla anche di <b>Business Intelligence</b>.

![Portafoglio Direzionale](/assets/sistemi_informativi/portafoglio_direzionale.png)

#### Pianificazione e controllo strategico
In questa fase ci si preoccupa di effettuare analisi di mercato, studi sulla concorrenza, valutazione di nuovi investimenti. (pianificazione)
Il tutto tenuto sotto controllo dal monitoraggio delle prestazioni dell'azienda, monitoraggio del _cash flow_ e della creazione di valore. (controllo)

#### Controllo direzionale
I sistemi informativi di controlo direzionale hanno lo scopo di confrontare periodicamente gli obiettivi ed i risultati e svolgere analisi dei relativi scostamenti.

#### Controllo operativo
I sistemi di controllo operativo hanno lo scopo di monitorare i processi aziendali.<br>
Per farlo, tengono conto di:
- Volumi di produzione
- Tempi di attraversamento
- Qualità ottenuta


### Portafoglio Operativo
Si occupa di:
- <b>Elaborazione delle transazioni</b>: informazioni anagrafiche, movimenti, informazioni di stato
- <b>Pianificaizone delle operazioni</b>:
    - Piano dei progetti
    - Piano degli acquisti
    - Piano della produzione
    - Previsioni di vendita
- <b>Acquisizione e organizzazione della conoscenza</b>: in questo caso, l'informazione non è usata per documentare una transazione ma per accumulare informazioni prodotte da diversi attori 


# Valorizzazione del portafoglio applicativo
## Punti funzione
Tipologie di funzioni:
- Funzione di input
- Funzione di output
- Funzioni di interrogazione
- Archivi interni
- Archivi esterni

## Calcolo del punteggio grezzo

$$\sum_{i=1}^{5} \sum_{j=1}^{3} n_{ij} * p_{ij}$$

![Punti funzione](/assets/sistemi_informativi/punti_funzione.png)

## Fattori di aggiustamento
È possibile avere un aggiustamento del valore in base a dei fattori di aggiustamento, come:
- Trasmissione dati
- Prestazioni
- Volume delle transazioni
- Transazioni online
- Aggiornamenti degli archivi online
- Facilità di operazione, modifica, installazione

$P_e$ = peso del fattore di aggiustamento.<br>
Il peso va da 0 (inifluente) a 5 (fortemente influente).

## Calcolo punteggio corretto
$$F_c = F * A_g$$

$$A_g = 0,65 + \sum_{e=1}^{14} P_e / 100$$
