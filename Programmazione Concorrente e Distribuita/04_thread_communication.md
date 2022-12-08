# Intro
Esistono varie tipologie di comunicazione tra threads. Tutte le funzioni sono implemtabili manualmente tramite i meccanismi già visti in precedenza.

I principali metodi sono:
- Signal
- Buffer
- Blackboard
- Broadcast
- Barrier

# Signal
Serve principalmente per inviare segnali tra thread e non contiene dati particolari.

Ci sono 2 tipologie di segnali:
1. Persistent signal: rimane impostato fino a quando un thread non lo riceve
2. Transient signal: rilascia uno o più thread in attesa, ma si perde se invece non ci sono thread in attesa

Dato che, spesso, i thread che ricevono i segnali sono diversi da quelli che lo ricevono è utile andare a separare le interfacce:

```java
public interface SignalSender {
    void send();
}

public interface SignalWaiter {
    void waits() throws InterruptedException;
}
```

<b>! </b>: Se si volesse evitare un’attesa infinita, si potrebbe aggiungere un metodo `waits(t)` con un timeout.

È anche utile definire una classe astratta Signal, dalla quale si potranno poi derivare le classi concrete che realizzeranno i segnali persistenti e transienti:

```java
public abstract class Signal implements SignalSender, SignalWaiter {
    protected boolean arrived = false;
    
    public abstract void waits() throws InterruptedException;
    
    public synchronized void send() {
        arrived = true;
        notify();
    }
}
```

## Persistent signal
Siccome la classe PersistentSignal realizza un segnale persistente, la sua implementazione del metodo waits() è tale che:
- se il segnale è già impostato, viene azzerato immediatamente, senza bloccare il
thread chiamante;
- se il segnale non è ancora arrivato, il thread chiamante si blocca;
- quando ci sono uno o più thread bloccati, e il segnale arriva, il primo thread che si sveglia azzera (“consuma”) il segnale, quindi gli eventuali altri thread in attesa dovranno aspettare l’arrivo di ulteriori segnali per sbloccarsi.

Inoltre, questa classe mette anche a disposizione un metodo non bloccante (ma comunque `synchronized`) `watch()` (definito dall’interfaccia `SignalWaiterOrWatcher`), che indica al chiamante se il segnale è arrivato, e al tempo stesso lo azzera, in modo che solo un thread possa trovarlo impostato. Esso può essere utile per fare polling, invece di sospendersi in attesa dell’evento.

```java
//Interfaccia per il metodo watch()
public interface SignalWaiterOrWatcher extends SignalWaiter {
    boolean watch();
}

public class PersistentSignal extends Signal implements SignalWaiterOrWatcher {
    
    public synchronized void waits() throws InterruptedException {
        while (!arrived) {
            wait();
        }
        arrived = false;
    }

    public synchronized boolean watch() {
        if (!arrived) {
            return false;
        }
        arrived = false;
        return true;
    }
}
```

### Esempio: accesso al disco
Si ipotizza un'operazione di scrittura sul disco asincrona. Al termine dell'operazione viene inviato un segnale al thread di lettura.

```java
public class DiskController {
    // Varie operazioni, compresa:
    public SignalWaiterOrWatcher asyncWrite(/* argomenti... */) {
        SignalWaiterOrWatcher reply = new PersistentSignal();
        // Avvia l'operazione di scrittura
        // (ma senza aspettare che termini)
        return reply;
    }
}
```
Il metodo `asyncWrite` permette di effettuare scritture asincrone: esso crea un’istanza di PersistentSignal, e la restituisce dopo aver iniziato la scrittura, ma prima che essa sia completata.

Quando la scrittura sarà completata, verrà invocato il metodo send del segnale. Inoltre, in caso di fallimento della scrittura, sarebbe opportuno interrompere l’attesa di eventuali thread bloccati sul segnale (mediante interrupt), in modo che questi possano gestire tale fallimento.

Il chiamante riceve un riferimento al segnale, e può usarlo per capire quando la scrittura sia completata:
- mettendosi in attesa

```java
public static void main(String[] args) {
    DiskController controller = new DiskController();
    // Preparazione dei dati da scrivere...
    SignalWaiterOrWatcher outputDone = controller.asyncWrite(/* ... */);
    // ...
    // Quando è il momento di controllare
    // che l'output sia completato:
    try {
        outputDone.waits();
    } catch (InterruptedException e) {
        // La scrittura non è andata a buon fine,
        // quindi si avvia un'operazione di recovery.
    }
}
```

- mediante polling

```java
public static void main(String[] args) {
    DiskController controller = new DiskController();
    // Preparazione dei dati da scrivere...
    SignalWaiterOrWatcher outputDone = controller.asyncWrite(/* ... */);
    // ...
    // Quando è il momento di controllare
    // che l'output sia completato:
    if (!outputDone.watch()) {
        // La scrittura non è completata,
        // quindi si avvia un'operazione di recovery.
    }
}
```
<b>! </b>: ques'ultimo esempio effettuta un solo controllo sul termine della scrittura, e se essa non è terminata procede con il recovery. Si può, invece, utilizzare un ciclo per continuare a fare polling ed aspettare che la scrittura termini effettivamente.

### `waits(t)`
Posso implementare anche un metodo per attendere un determinato periodo di tempo massimo.

`waits(t)`, che riceve come argomento un timeout, e restituisce un valore booleano per indicare se è arrivato il segnale (true) o se è scaduto il timeout (false):

```java
//aggiungo in metodo nell'interfaccia
public interface SignalWaiter {
    void waits() throws InterruptedException;
    boolean waits(long timeout) throws InterruptedException;
}

//Definisco il funzionamento nella classe PersistentSignal
public class PersistentSignal extends Signal implements SignalWaiterOrWatcher {
    public synchronized boolean waits(long timeout) throws InterruptedException {
        long callTime = System.currentTimeMillis();
        long elapsed = 0;
        while (!arrived && elapsed < timeout) {
            wait(timeout - elapsed); //richiamo la primitiva
            if (arrived) {
                arrived = false;
                return true;
            } else {
                elapsed = System.currentTimeMillis() - callTime;
            }
        }
        return false;
    }
}
```

## Transient signal
Nell’implementare i segnali transienti, bisogna decidere cosa succede quando arriva un segnale e ci sono in attesa più thread: si può scegliere di sbloccarne solo uno, oppure di sbloccarli tutti.

1. Singolo sblocco

```java
public class TransientSignal extends Signal {
    protected int waiting = 0;
    public synchronized void send() {
        if (waiting > 0) {
        super.send(); //richiamo send() dalla classe Signal
        }
    }

    public synchronized void waits() throws InterruptedException {
        waiting++;
        try {
            while (!arrived) {
                wait();
            }
            arrived = false;
        } finally {
            waiting--;
        }
    }
}
```

- Il metodo `send` implementato dalla superclasse `Signal` (che contiene una notify, quindi sveglia un singolo thread) viene chiamato solo se ci sono effettivamente thread in attesa, per fare in modo che il segnale si perda se invece non ce ne sono. Altrimenti, in assenza di thread da svegliare, il flag arrived rimarrebbe impostato a true, e quindi il segnale si comporterebbe in modo persistente.
- Nell’implementazione di `waits`, è importante decrementare il numero di thread
in attesa anche in caso di InterruptedException, quindi quest’operazione viene
messa in un `finally`. Non è invece necessario impostare arrived a false, perché
se l’attesa termina a causa di una interrupt significa che il segnale non è arrivato.
- `TransientSignal` non fornisce un metodo `watch`, perché non deve essere possibile controllare a posteriori se il segnale sia arrivato: come già detto, l’unico scopo di un segnale transiente è svegliare gli eventuali thread che sono in attesa al momento del suo arrivo.

2. Unlock all
Si definisce una classe `Pulse`, la quale estende `TransientSignal`, mettendo a disposizione un metodo aggiuntivo `sendAll()` che sveglia tutti i thread in attesa:

```java
public class Pulse extends TransientSignal {
    public synchronized void sendAll() {
        if (waiting > 0) {
        arrived = true;
        notifyAll();
        }
    }
    public synchronized void waits() throws InterruptedException {
        waiting++;
        try {
            while (!arrived) {
                wait();
            }
        } finally {
            waiting--;
            if (waiting == 0) {
                arrived = false;
            }
        }
    }
}
```
<b>! </b>: `Pulse` ridefinisce anche il metodo `waits`, perché solo l’ultimo thread svegliato deve resettare il flag arrived: altrimenti, una volta resettato, i thread usciti dalla wait penserebbero che il segnale non sia arrivato, e si rimetterebbero in attesa.

### Esempio: apertura negozio
Durante la stagione dei saldi, un negozio decide di aprire le porte solo a intervalli di tempo di N secondi:
1. i clienti che arrivano quando le porte sono chiuse aspettano;
2. quando le porte vengono aperte, i clienti in attesa entrano;
3. le porte si richiudono subito dopo che i clienti sono entrati.
Nella simulazione di questa situazione, l’apertura delle porte è rappresentata da un segnale transiente, e i clienti sono thread che attendono tale segnale:

```java
//Creazione del thread cliente
public class Cliente extends Thread {
    private Pulse portaNegozio;
    public Cliente(Pulse portaNegozio) {
        this.portaNegozio = portaNegozio;
        setName("Cliente " + getName());
    }
    public void run() {
        System.out.println(getName() + ": mi metto in coda");
        try {
            portaNegozio.waits();
            System.out.println(getName() + ": entrato nel negozio");
        } catch (InterruptedException e) {}
    }
}

//Creazione del thread negozio
public class Negozio extends Thread {
    private Pulse portaNegozio;
    public Negozio(Pulse portaNegozio) {
        this.portaNegozio = portaNegozio;
        setName("Negozio " + getName());
    }
    public void run() {
        try {
            while (true) {
                Thread.sleep(1000);
                System.out.println(getName() + ": apro le porte");
                portaNegozio.sendAll();
                System.out.println(getName() + ": chiudo le porte");
            }
        } catch (InterruptedException e) {}
    }
}

//Main
public class Saldi {
    public static void main(String[] args) throws InterruptedException {
        Pulse portaNegozio = new Pulse(); //Creazione di un pulse condiviso (signal)
        new Negozio(portaNegozio).start();
        Random rnd = new Random();
        while (true) {
            int numClienti = rnd.nextInt(5); //Creo un numero di utenti random
            for (int i = 0; i < numClienti; i++) {
                new Cliente(portaNegozio).start();
            }
            Thread.sleep(300);
        }
    }
}
```

# Buffer
Un buffer contiene dati che, una volta letti, verranno distrutti.
i seguito, viene mostrata un’implementazione di un BoundedBuffer generico (in particolare, una coda con capacità limitata), di fatto uguale a quella già vista nell’ambito del problema del produttore-consumatore:

```java
public class BoundedBuffer<Data> {
    private final int size;
    private final Data[] buffer;
    private int numItems = 0;
    private int first = 0;
    private int last = 0;

    public BoundedBuffer(int size) {
        this.size = size;
        buffer = (Data[]) new Object[size];
    }
    public synchronized void put(Data item) throws InterruptedException {
        while (numItems == size) {
            wait();
        }
        buffer[last] = item;
        last = (last + 1) % size;
        numItems++;
        notifyAll();
    }
    public synchronized Data get() throws InterruptedException {
        while (numItems == 0) {
            wait();
        }
        Data item = buffer[first];
        first = (first + 1) % size;
        numItems--;
        notifyAll();
        return item;
    }
}
```

# Blackboard
Molto simile a `Buffer`, ma:
- la lettura dei dati è non distruttiva, cioè i dati letti rimangono disponibili, e possono quindi essere letti più volte;
- i messaggi lasciati sulla blackboard vengono preservati finché non sono sovrascritti, cancellati o invalidati;
- la scrittura è solitamente non bloccante.

Metodi principali di una `Blackboard`:

| Metodo          | Descrizione                                                                                           |
| --------------- | ----------------------------------------------------------------------------------------------------- |
| write()         | crea un messaggio                                                                                     |
| clear()         | cancella i messaggi                                                                                   |
| read()          | legge un messaggio, bloccando il chiamante finché non ci sono dati validi sulla blackboard            |
| dataAvailable() | indica se la blackboard ha dati disponibili, e può essere usato dal lettore per evitare di bloccarsi. |

```java
public class Blackboard<Data> {
    private Data message;
    private boolean valid;
    
    public Blackboard() {
        valid = false;
    }
    public Blackboard(Data initial) {
        message = initial;
        valid = true;
    }
    public synchronized void write(Data message) {
        this.message = message;
        valid = true;
        notifyAll();
    }
    public synchronized void clear() {
        valid = false;
    }
    public synchronized Data read() throws InterruptedException {
        while (!valid) {
            wait();
        }
        return message;
    }
    public boolean dataAvailable() {
        return valid;
    }
}
```

- Sono disponibili due costruttori: uno senza argomenti, che crea una blackboard
“vuota”, e uno che permette invece di specificare un messaggio iniziale (quindi la
blackboard conterrà subito dei dati validi).
- I metodi write e clear non sono bloccanti: il messaggio corrente viene sovrascritto,
che sia stato letto o meno.
- Invece, il metodo read è bloccante: se non è già presente un messaggio valido, si
aspetta che ne venga scritto uno.

## Esempio: digitale terrestre
Un canale digitale terrestre propone ogni giorno una sequenza di programmi. Un utente
vuole vedere le informazioni su cosa stiano trasmettendo in qualsiasi momento. Per
farlo, si sintonizza e legge le informazioni sul programma che stanno trasmettendo in un
determinato istante.

```java
//Creo la classe programma
public class Programma {
    private final String nome;
    private final String oraInizio;
    private final int durataInMin;

    public Programma(String nome, String oraInizio, int durataInMin) {
        this.nome = nome;
        this.oraInizio = oraInizio;
        this.durataInMin = durataInMin;
    }

    public String toString() {
        return nome + " (inizia alle " + oraInizio + ")";
    }

    public int durataInMin() {
        return durataInMin;
    }
}

//Gli utenti ricevono tutti una blackboard comune dove leggere le info
public class Utente extends Thread {
    private Blackboard<Programma> blackboard;
    
    public Utente(Blackboard<Programma> blackboard) {
        setName("Utente " + getName());
        this.blackboard = blackboard;
    }
    public void run() {
        try {
            System.out.println(getName() + ": attendo programma");
            Programma msg = blackboard.read();
            System.out.println(getName() + ": sto guardando " + msg);
        } catch (InterruptedException e) {}
    }
}

//Anche CanaleTV riceve la blackboard sulla quale scrive le programmazioni
public class CanaleTv extends Thread {
    private final String nome;
    private Blackboard<Programma> blackboard;
    private Programma[] programmazione;
    
    public CanaleTv(String nome, Blackboard<Programma> blackboard, Programma[] programmazione) {
        this.nome = nome;
        this.blackboard = blackboard;
        this.programmazione = programmazione;
    }

    public void run() {
        for (Programma programma : programmazione) {
            System.out.println(nome + " trasmette " + programma);
            blackboard.write(programma);
            try {
                Thread.sleep(programma.durataInMin()); //Simula la durata del programma
            } catch (InterruptedException e) { return; }
        }
    }
}

//Main
public class BlackboardExample {
    public static void main(String[] args) throws InterruptedException {
        Blackboard<Programma> bbRai3 = new Blackboard<>(); //Crea la blackboard condivisa
        
        Programma[] progs = new Programma[] { //Crea la programmazione
            new Programma("Meteo 3", "20:05", 5),
            new Programma("Blob", "20:10", 15),
            new Programma("TG3", "20:25", 40),
            new Programma("Dott. Stranamore", "21:05", 120),
            new Programma("Notturno", "23:05", 420)
        };

        new CanaleTv("Rai3", bbRai3, progs).start();
        
        for (int i = 0; i < 8; i++) {
            Thread.sleep(ThreadLocalRandom.current().nextInt(30));
            new Utente(bbRai3).start();
        }
    }
}
```

# Broadcast
Una comunicazione broadcast permette di inviare dati a più thread contemporaneamente. Solo i thread in attesa al momento dell’invio ricevono i dati.

Sono possibili delle varianti, nelle quali si cambia ciò che succede se, al momento dell’invio:
- non ci sono thread in attesa;
- un broadcast precedente è ancora in corso, cioè non tutti i thread in attesa hanno ricevuto i dati.

Nell’implementazione presentata in seguito, si sceglie di eseguire l’invio solo se ci sono
thread in attesa e un eventuale messaggio precedente è stato ricevuto da tutti, altrimenti
la chiamata send non fa niente (in particolare, non blocca il chiamante).

```java
public class Broadcast<Data> {
    private Data message;
    // sending indica se c'è un messaggio non ancora ricevuto da tutti
    private boolean sending = false;
    private int waiting = 0;

    public synchronized void send(Data message) {
        if (waiting > 0 && !sending) {
            this.message = message;
            sending = true;
            notifyAll();
        }
    }

    public synchronized Data receive() throws InterruptedException {
        waiting++;
        try {
            while (!sending) {
                wait();
            }
        } finally {
            waiting--;
            if (waiting == 0) {
                // L'ultimo thread che riceve il messaggio
                // resetta il flag
                sending = false;
            }
        }
        return message;
    }
}
```

## Esempio: agenzia di stampa
Un’agenzia di stampa, ANSA.it, fornisce le prime notizie su tutto quello che accade in
Italia ai giornali che si abbonano al servizio. Per questo esempio, si suppone che ci siano 4
giornali interessati al servizio: IlCorriere.it, IlFatto.it, LaRepubblica.it e IlMattino.it.

L’invio di una notizia (messaggio) in broadcast è svolto da un apposito thread:

```java
public class AnsaMessage extends Thread {
    private Broadcast<String> broadcast;
    private String message;
    
    public AnsaMessage(Broadcast<String> broadcast, String message) {
        this.broadcast = broadcast;
        this.message = message;
    }
    
    public void run() {
        System.out.println("News from ANSA.it: " + message);
        broadcast.send(message);
    }
}
```

Invece, i thread corrispondenti ai giornali aspettano di ricevere alcuni messaggi:

```java
public class Newspaper extends Thread {
    private final String name;
    Broadcast<String> broadcast;

    public Newspaper(String name, Broadcast<String> broadcast) {
        this.name = name;
        this.broadcast = broadcast;
    }

    public void run() {
        for (int i = 0; i < 4; i++) {
            System.out.println(name + ": waiting for news...");
            try {
                String msg = broadcast.receive();
                System.out.println(name + ": " + msg);
            } catch (InterruptedException e) { break; }
        }
    }
}
```

Nel main viene creata un'istanza di Broadcast condivisa e vengono creati i vari thread per ricevere ed inviare le notizie:

```java
public class BroadcastExample {
    public static void main(String[] args) throws InterruptedException {
        Broadcast<String> broadcast = new Broadcast<>();
        new AnsaMessage(
            broadcast, "Diminuiscono le immatricolazioni"
        ).start();
        
        Thread.sleep(100);
        
        new Newspaper("IlCorriere.it", broadcast).start();
        new Newspaper("IlFatto.it", broadcast).start();
        new Newspaper("LaRepubblica.it", broadcast).start();
        new Newspaper("IlMattino.it", broadcast).start();
        
        for (int i = 0; i < 5; i++) {
            Thread.sleep(10);
            
            new AnsaMessage(
                broadcast, "Aumentano le immatricolazioni"
            ).start();
            
            Thread.sleep(10);
            
            new AnsaMessage(
                broadcast, "Diminuiscono le immatricolazioni"
            ).start();
        }
    }
}
```

# Barrier
`barrier` blocca un insieme di thread fino a quando non raggiungono tutti tale barriera.
1. i thread che desiderano bloccarsi a una barriera devono chiamare il metodo waitB;
2. tutti i thread che arrivano, tranne l’ultimo, vengono trattenuti;
3. quando l’ultimo thread arriva, tutti i thread vengono rilasciati (possono procedere).

Questo è un costrutto che serve puramente per la sincronizzazione, senza alcuna
comunicazione di dati.
In particolare, esso è utile quando bisogna essere certi che il sistema sia arrivato in una
situazione nota.

```java
public class Barrier {
    // Numero di thread che devono arrivare perché la barriera si apra
    private final int need;
    // Numero di thread attualmente presenti alla barriera
    private int arrived = 0;
    // Stato della barriera (true significa aperta, false indica chiusa)
    private boolean releasing = false;
    
    public Barrier(int need) {
        this.need = need;
    }
    
    public synchronized int getArrived() {
        return arrived;
    }
    
    public int capacity() {
        return need;
    }
    
    public synchronized void waitB() throws InterruptedException {

        while (releasing) {
            // La barriera è ancora aperta (per far passare un gruppo
            // precedente di thread).
            // Bisogna aspettare che si richiuda, altrimenti potrebbero
            // passare più thread del numero previsto.
            wait();
        }
        arrived++;
        try {
                while (arrived < need && !releasing) {
                wait();
            }
            if (arrived == need) {
                releasing = true;
                System.out.println("All " + need + " parties arrived at barrier");
                notifyAll();
            }
        } finally {
            arrived--;
            if (arrived == 0) {
                // L'ultimo thread che lascia la barriera la richiude,
                releasing = false;
                // e sveglia i nuovi thread che stavano aspettando la
                // chiusura (i quali andranno alla seconda wait).
                notifyAll();
            }
        }
    }
}
```

### Esempio: corsa di cavalli
Durante una corsa di cavalli, ogni cavallo deve entrare nella barriera (che ha una postazione per ogni cavallo) prima di partire. Inoltre, a ogni giro:
- i cavalli devono fermarsi alla barriera e aspettare tutti gli altri;
- un cronista annuncia il numero di giri fatti.

Ci sono N cavalli/postazioni, e bisogna fare M giri.

Il main crea semplicemente la barriera e i thread cavalli e cronista:

```java
public class CorsaDiCavalli {
    private static final int NUM_CAVALLI = 3;
    private static final int NUM_GIRI = 2;
    
    public static void main(String[] args) throws InterruptedException {
        Barrier barriera = new Barrier(NUM_CAVALLI);
        for (int i = 0; i < NUM_CAVALLI; i++) {
            new Cavallo(barriera, NUM_GIRI).start();
        }
        new Cronista(barriera, NUM_GIRI).start();
    }
}
```
Il thread Cronista usa un apposito metodo waitCycle della barriera per attendere ogni
volta l’inizio del giro successivo

```java
public class Cronista extends Thread {
    private final Barrier barriera;
    private final int numGiri;

    public Cronista(Barrier barriera, int numGiri) {
        this.barriera = barriera;
        this.numGiri = numGiri;
    }

    public void run() {
        for (int i = 0; i < numGiri; i++) {
            try {
                int g = barriera.waitCycle(i + 1);
                System.out.println("Siamo al giro numero " + g);
            } catch (InterruptedException e) { return; }
        }
        System.out.println("Siamo all'ultimo giro!");
    }
}
```

Per ogni giro, il thread Cavallo esegue una sleep (che simula il tempo necessario ad
arrivare alla barriera), e poi si blocca per aspettare gli altri:

```java
public class Cavallo extends Thread {
    private final Barrier barriera;
    private final int numGiri;
    
    public Cavallo(Barrier barriera, int numGiri) {
        this.barriera = barriera;
        this.numGiri = numGiri;
        setName("Cavallo " + getName());
    }
    
    public void run() {
        for (int i = 0; i < numGiri; i++) {
            try {
                System.out.println(getName() + " sta per entrare nella barriera");
                Thread.sleep(ThreadLocalRandom.current().nextInt(100));
                barriera.waitB();
                System.out.println(getName() + " è partito!");
            } catch (InterruptedException e) {}
        }
    }
}
```

Implemento ora la barrier:

```java
public class Barrier {
    private final int need;
    private int arrived = 0;
    private int numCycles = 0;
    private boolean releasing = false;
    
    public Barrier(int need) {
        this.need = need;
    }
    
    public synchronized int getArrived() {
        return arrived;
    }
    
    public int capacity() {
        return need;
    }
    
    public synchronized int waitCycle(int cycleNum) throws InterruptedException {
        while (cycleNum != numCycles) {
            wait();
        }
        return numCycles;
    }
    
    public synchronized void waitB() throws InterruptedException {
        while (releasing) {
            wait();
        }
        arrived++;

        try {
            while (arrived < need && !releasing) {
                wait();
            }
            if (arrived == need) {
                releasing = true;
                System.out.println("All " + need + " parties arrived at barrier");
                numCycles++;
                notifyAll();
            }
        } finally {
            arrived--;
            if (arrived == 0) {
                releasing = false;
                notifyAll();
            }
        }
    }
}
```

## CyclicBarrier
Il funzionamento di questa barriera è analogo a quello descritto finora, ma con un elemento in più: opzionalmente, è possibile associare alla barriera un comando Runnable, che viene eseguito quando sono arrivati tutti i thread, subito prima che la barriera si apra. Questo permette di eseguire delle operazioni nell’istante in cui tutti i thread sono fermi.

### Costruttori
I costruttori di CyclicBarrier sono due: quello “classico”, nel quale si specifica solo il
numero di thread che devono essere attesi prima che la barriera si apra:

```java
public CyclicBarrier(int parties);
```

e uno con il quale si specifica anche il comando Runnable:

```java
public CyclicBarrier(int parties, Runnable barrierAction);
```

### Metodi

| Metodo          | Descrizione                                                                                           |
| --------------- | ----------------------------------------------------------------------------------------------------- |
| public int await() | Il metodo di attesa (analogo al waitB delle classi Barrier di prima)                                  |
| public int await(long timeout, TimeUnit unit) | Una versione di await che permette di specificare un tempo di attesa massimo, allo
scadere del quale il thread viene sbloccato anche se la barriera non si è aperta |
| public int getNumberWaiting() | Restituisce il numero di thread attualmente in attesa alla barriera |
| public int getParties() | Restituisce il numero totale di thread necessari per aprire la barriera |
| public boolean isBroken() | indica se la barriera è in uno stato inconsistente |
| public void reset() | Resetta la barriera allo stato iniziale |

### Esempio: corsa dei cavalli
L’esempio della corsa di cavalli può essere riscritto usando la classe `CyclicBarrier`.
La principale differenza è che il cronista, invece di essere un thread, diventa il comando `Runnable` associato alla barriera, garantendo così una sincronizzazione ottimale: il cronista dirà che inizia l’(n + 1)-esimo giro esattamente quando tutti i cavalli hanno completato l’n-esimo, ma non sono ancora ripartiti per l’(n + 1)-esimo.


```java
//Main
public class CorsaDiCavalli {
    private static final int NUM_CAVALLI = 3;
    private static final int NUM_GIRI = 2;
    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier barriera = new CyclicBarrier(NUM_CAVALLI, new Cronista());
        for (int i = 0; i < NUM_CAVALLI; i++) {
            new Cavallo(barriera, NUM_GIRI).start();
        }
    }
}

//Thread cavallo
public class Cavallo extends Thread {
    private final CyclicBarrier barriera;
    private final int numGiri;
    
    public Cavallo(CyclicBarrier barriera, int numGiri) {
        this.barriera = barriera;
        this.numGiri = numGiri;
        setName("Cavallo " + getName());
    }
    
    public void run() {
        for (int i = 0; i < numGiri; i++) {
            try {
                System.out.println(getName() + " sta per entrare nella barriera");
                Thread.sleep(ThreadLocalRandom.current().nextInt(3000));
                barriera.await();
                System.out.println(getName() + " è partito!");
            } catch (InterruptedException | BrokenBarrierException e) {}
        }
    }
}

/*Il codice del Cronista si semplifica notevolmente, dato che, come già detto, esso non è più
un thread a sé stante, bensì solo un comando Runnable associato alla barriera. In particolare, il Cronista non ha più bisogno di preoccuparsi della sincronizzazione.*/
public class Cronista implements Runnable {
    private int giro = 0;
    
    public void run() {
        System.out.println("I cavalli partono per il giro " + giro);
        giro++;
    }
}

```