# Problemi della programmazione concorrente e come evitarli

## Nondeterminismo
Il programmatore non può determinare l'ordine assoluto di esecuzione delle istruzioni appartenenti a thread diversi. Questo rende anche la fase di testing di un programma più difficoltosa.

## Race conditions
Sono situazioni in cui thread diversi condividono una risorsa, ed il risultato dipende dall'ordine in cui essi effettuano le loro operazioni.

```java
//Esempio
public class RaceExample extends Thread {
    Counter myCounter;
    public RaceExample(Counter c){
        myCounter=c;
    }
    public void run() {
        for(int i=0; i<10000; i++)
            myCounter.add(1);
    }
    public static void main(String args[]) throws InterruptedException {
        Counter counter = new Counter();
        RaceExample p1 = new RaceExample(counter);
        RaceExample p2 = new RaceExample(counter);
        p1.start(); p2.start();
        p1.join(); p2.join();
        System.out.println("Counter = " + counter.count);
    }
}

public class Counter {
    long count = 0;
    public void add (long value) {
        long tmp = this.count;
        tmp = tmp + value;
        this.count = tmp ;
    }
}
```
Al termine dell'esecuzione, ci si aspetterebbe di avere sempre il counter a 2000 ma questo non sempre accade.
Questo perché ci troviamo in una race condition dove un thread potrebbe essere interrotto subito dopo aver effettuato l'operazione `long tmp = this.count;`.
Successivamente, l'altro thread incrementa il contatore, ma quando il thread interrotto viene risvegliato, non avrà il valore aggiornato in `tmp`.

## Quando si verifica una Race Condition?
1. Una risorsa deve essere condivisa tra 2 thread
2. Deve esistere almeno un percorso di esecuzione tra i thread in cui una risorsa è condivisa in modo non sicuro

![RaceCondition](/assets/programmazione_concorrente_e_distribuita/race_condition1.png)

Anche nel caso in cui si andasse a modificare il codice in questo modo, il problema resta comunque presente

```java
public class Counter {
    long count = 0;
    public void add (long value) {
        this.count += value;
    }
}
```

Questo perché i thread vengono interrotti a livello di byte code, e non di istruzioni Java. <br>
Abbiamo quindi bisogno di un'istruzione atomica.

## Come si assicura la correttezza di un software concorrente?
Esistono 2 modalità:
1. Dimostrare che il programma soddisfa tutte le condizioni sufficienti perché possa essere considerato sicuro (rischioso e costoso).
2. Sviluppare il programma utilizzando strategie note che permettano di evitare a priori situazioni di Race Conditions.

Per rendere sicuro l'accesso ad una corsa critica, bisogna fare in modo che possa essere utilizzata solamente da un thread per volta. Quindi bloccare l'accesso quando è in uso da un thread e sbloccare l'accesso quando il thread ha finito di utilizzarla.

### Semaphores
Tramite un semaforo:
- Il primo thread che accede alla risorsa condivisa blocca l'accesso mediante il semaforo.
- Quando termina di utilizzare la risorsa, sblocca l'accesso

Il semaforo viene utilizzato tramite i metodi:
 - `acquire`: acquisisce l'accesso alla risorsa, mettendo in attesa gli altri thread che vorrebbero accedervi
 - `release`: rilascia l'utilizzo della risorsa

Java fornisce un generico semaforo n-ario tramite la classe `java.util.concurrent.Semaphore`. <br>
- Il semaforo viene inizializzato ad un valore intero positivo
- Viene decrementato ad ogni `acquire`
- Viene incrementato ad ogni `release`
- Solo quando il semaforo è 0, il thread che fa una richiesta viene bloccato

```java
//Esempio precedente con gestione delle race conditions
import java.util.concurrent.Semaphore;
public class RaceExample extends Thread {
    private Counter myCounter;
    private Semaphore theSemaphore;
    public RaceExample(Counter c, Semaphore s){
        myCounter=c;
        theSemaphore=s;
    }
    public void run() {
        for(int i=0; i<10000; i++) {
        try {
            theSemaphore.acquire();
        } catch (InterruptedException e){ }
            myCounter.add(1);
            theSemaphore.release();
        }
    }
    public static void main(String args[]) throws InterruptedException {
        Counter counter = new Counter();
        Semaphore sem = new Semaphore(1);
        RaceExample p1 = new RaceExample(counter, sem);
        RaceExample p2 = new RaceExample(counter, sem);
        p1.start(); 
        p2.start();
        p1.join();
        p2.join();
        System.out.println("Counter = " + counter.count);
    }
}
```

### Synchronized
In Java, per facilitare e sostituire l'utilizzo dei semafori, è stato introdotto un metodo alternativo per il controllo dell'accesso alle sezioni critiche: ad ogni istanza della classe Object è associato un semaforo binario, chiamato <b>intrinsic lock</b>. Per utilizzarlo si utilizza la keyword `synchronized`. <br>
Il funzionamento è il medesimo dei semafori:
1. Quando un thread entra in un metodo/area synchronized, cerca di acquisire il lock associato all’oggetto: se tale lock è già stato acquisito da un altro thread, il thread corrente si mette in attesa, altrimenti esso acquisisce il lock, e può allora iniziare a eseguire il codice di tale metodo/area.
2. Quando si esce dal metodo/area synchronized, si rilascia automaticamente il lock associato all’oggetto. Di conseguenza, un eventuale altro thread in attesa passa allo stato ready, e può acquisire il lock ed entrare.

Con questo meccanismo, non ci possono mai essere due o più thread che eseguono contemporaneamente metodi/aree synchronized dello stesso oggetto, ovvero si ha la <b>mutua
esclusione</b> su tali metodi/aree.

```java
synchronized (obj) { // Acquisizione del lock
    // Sezione critica su obj...
}   // Rilascio del lock
```

Applicandolo all'esempio precedente, sarebbe bastato l'utilizzo di `synchronized` sul metodo `add()`:
```java
public class Counter {
    private long count = 0;
    public synchronized void add(long value) {
        this.count += value;
    }
    public long getValue() {
        return count;
    }
}
```

#### Esempio `synchronized`
```java
public class AssegnatorePosto {
    private int postiDisponibili = 20;
    public boolean assegnaPosti(int postiRichiesti) {
        if (postiRichiesti <= postiDisponibili) { // Sezione
            postiDisponibili -= postiRichiesti; // critica
            return true;
        } else {
            return false;
        }
    }
    public int getPostiDisponibili() {
        return postiDisponibili;
    }
}
```
Le prime due righe del codice di assegnaPosti costituiscono una sezione critica, dato che eseguono una lettura e, successivamente, una scrittura della variabile `postiDisponibili`:
se più thread eseguissero assegnaPosti in parallelo, il numero di posti assegnati alla fine potrebbe essere maggiore di quello realmente disponibile, ovvero si potrebbe avere una _race condition_.

Per correggere questo esempio, è necessario assicurarsi che la sezione critica, contenuta nel metodo assegnaPosti, possa essere eseguita da un solo thread alla volta, contrassegnando come `synchronized`:

1. L' intero metodo
```java
public synchronized boolean assegnaPosti(int postiRichiesti) {
    if (postiRichiesti <= postiDisponibili) { // Sezione
        postiDisponibili -= postiRichiesti; // critica
        return true;
    } else {
        return false;
    }
}
```
2. Solo la porzione di codice che rappresenta la sezione critica
```java
public boolean assegnaPosti(int postiRichiesti) {
    synchronized (this) {
        if (postiRichiesti <= postiDisponibili) { // Sezione
            postiDisponibili -= postiRichiesti; // critica
            return true;
        }
    }
    return false;
}
```

### Accesso ai dati
I metodi `synchronized` hanno <b>accesso esclusivo</b>, mentre i metodi non `synchronized` non sono esclusivi e quindi permettono l'accesso concorrente.

Tali accorgimenti non interessano le variabili locali: poiché ogni thread ha il proprio stack, anche quando più thread eseguono contemporaneamente lo stesso metodo, ciascuno di essi avrà una copia separata delle variabili locali, senza pericolo di “interferenza”.

### Variabili statiche
I metodi/blocchi synchronized non assicurano l’accesso mutualmente esclusivo ai dati statici, che sono condivisi da tutti gli oggetti della stessa classe: siccome ognuno di questi oggetti ha un lock separato, thread diversi potrebbero accedere contemporaneamente a tali dati, mediante oggetti diversi.

A ogni classe Java, però, è associato un oggetto di tipo Class: per accedere in modo sincronizzato ai dati statici, si sfrutta il lock di questo oggetto. 
<br>Ci sono due modi equivalenti di farlo:
1. Dichiarando un metodo statico come `synchronized`
2. Contrassegnando un blocco come `synchronized` sull'oggetto di tipo `Class`

<b>! </b>: Il lock a livello di classe non si ottiene quando ci si sincronizza su un oggetto di tale classe, e viceversa.

```java
public class StaticSharedVariable {
    private static int shared;
    public int read() {
        synchronized (StaticSharedVariable.class) {
            return shared;
        }
    }
    public static synchronized void write(int i) {
        shared = i;
    }
}
```

### Ereditarietà
`synchronized` non fa propriamente parte della firma di un metodo.<br>
Questo permette ad una classe derivata di ridefinire un metodo `synchronized` come non `synchronized`, e viceversa.

```java
public class AssegnatoreSequenziale {
    private int postiDisponibili = 20;
    public boolean assegnaPosti(int postiRichiesti) {
        if (postiRichiesti <= postiDisponibili) {
            postiDisponibili -= postiRichiesti;
            return true;
        } else {
            return false;
        }
    }
    public int getPostiDisponibili() {
        return postiDisponibili;
    }
}

public class AssegnatoreConcorrente extends AssegnatoreSequenziale {
    public synchronized boolean assegnaPosti(int postiRichiesti) {
        return super.assegnaPosti(postiRichiesti);
    }
}
```

## Cooperazione tra thread
`synchronized` permette ad un thread di modificare in modo sicuro una risorsa. Ma spesso è necessario che un thread sappia quando tale risorsa è stata modificata. A tale scopo Java mette a disposizione i metodi `wait()`, `notify()` e `notifyAll()` della classe Object.

<b>! </b>: Un thread può chiamare questi metodi su un oggetto solo quando detiene il lock di tale oggetto, cioè all’interno di un metodo/blocco `synchronized` sull’oggetto: chiamarli in un contesto diverso genera un’eccezione.

### Metodo `wait()`
`wait()` si utilizza quando un thread, dopo essere entrato in un'area `syncronized`, non ha le condizioni necessarie per proseguire e quindi deve attendere che esse si verifichino.<br>
Un thread che esegue `wait()` viene messo in una <b>wait list</b> in attesa di essere risvegliato dai metodi `notify()` o `notifyAll()` che portano il thread in una <b>ready list</b>.

```java
synchronized (this) { // lock
    try {
        this.wait(); // unlock
        // lock
    } catch (InterruptedException e) {}
} // unlock
```

### Metodi `notify()` e `notifyAll()`
- Una chiamata `notify()` su un oggetto sposta un thread qualsiasi (scelto in modo non deterministico) dalla <b>wait list</b> alla <b>ready list</b>, rendendolo schedulabile
- Invece, invocando `notifyAll()` su un oggetto si spostano tutti i thread dalla <b>wait list</b> alla <b>ready list</b>

L'utilizzo di questi metodi dipende naturalmente dall'applicazione nel programma.

## Monitor
