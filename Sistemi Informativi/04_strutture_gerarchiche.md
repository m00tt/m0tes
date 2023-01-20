# Struttura gerarchica
In condizioni di bassa incertezza, le prestazioni migliori sono conseguite con una <b>struttura gerarchica</b>.

![Piramide Di Anthony](/assets/sistemi_informativi/piramide_anthony.png)

- Maggiore stabilità e robustezza in ambienti ostili
- Necessità di minore veicolazione delle informazioni
- Maggiore scalabilità

## Vantaggi: minore veicolazione d'informazione

<b>STRUTTURA PIATTA</b>
Il numero di connessioni tra componenti è pari a $\frac{n * (n-1)}{2}$

Dove _n_ è il numero di componenti dell'organizzazione<br>
Dove _e_ è il numero di connessioni necessarie

Di principio, in una struttura piatta, tutti parlano con tutti.

Quindi, se $n = 6$, allora $e = \frac{6 * (6-1)}{2} = 15$

![Struttura piatta](/assets/sistemi_informativi/flat_struct.png)


<b>STRUTTURA GERARCHICA</b>
Numero cilomatico:<br>

$N(G) = e - n + c$

- e: archi
- n: nodi
- c: componenti connesse

Il numero ciclomatico di un grafo ne definiscce la complessità.

![Struttura gerarchica](/assets/sistemi_informativi/gerarchica1.png)

$N(G) = 5 - 6 + 1 = 0$

Andando ad analizzare, invece, la struttura piatta precedente avremo:<br>
$N(G) = 15 - 6 + 1 = 10$

Solamente l'aumento di flussi comunicativi (magari superflui) incide sulla complessità di una struttura gerarchica:

![Struttura gerarchica](/assets/sistemi_informativi/gerarchica2.png)
 
$N(G) = 12 - 6 + 1 = 7$


# Organizzazione verticale/orizzontale

![Org Verticale/Orizzontale](/assets/sistemi_informativi/struttura_vert_hor.png)

- Flussi informativi verticali: destinati a realizzare principalmente il controllo della struttura e delle attività
- Flussi informativi orizzontali: favoriscono la collaborazione tra le sue componenti

## Collegamenti verticali
I sistemi informativi verticali possono assumere forme come:
- Report periodici
- Comunicazioni interne che vengono diffuse dai manager
- Sistemi automatizzati di monitoraggio delle performance
- Sistemi informativi automatizzati

Il principale meccanismo di coordinamento è la gerarchia.

Il livello superiore prende decisioni che vengono distribuite ai manager, che a loro volta le riportano al piano sotto e così via.

![Organigramma](/assets/sistemi_informativi/organigramma.png)

PROBLEMA: quando aumentano le situazioni di incertezza, i livelli gerarchici superiori vengono sovraccaricati. Il sovraccarico decisionale è determinato dalla capacità elaborativa dell'unità gerarchica più alta.

## Collegamenti orizzontali
Per risolvere i problemi di sovraccarico decisionale, si deve aumentare la capacità elaborativa attraverso comunicazioni laterali.

Tipologie di collegamenti orizzontali:
- <b>Contatti diretti</b> fra coloro che condividono uno stesso problema
- <b>Ruoli di collegamento</b> fra unità organizzative diverse
- <b>Task force</b> create per risolveere problemi comuni
- <b>Ruoli manageriali</b> di integrazione (come project manager, team leader) istituiti per gestire le problematiche dedicate ad un determinato prodotto o servizio
- <b>Team permanenti</b>
- <b>Organizzazioni a matrice</b>, ovvero organizzazioni in cui ogni entità ha una doppia dipendenza gerarchica (per esempio, è possibile che un'unità dipenda dalla propria responsabilità funzionale e, allo stesso tempo, da una direzione di una particolare area geografica)