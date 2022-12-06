# Intro
<b>Programma</b>: insieme di istruzioni di alto livello o istruzioni in linguaggio macchina.<br>
<b>Processo</b>: istanza di un programma in esecuzione.

### Differenze tra processo e thread
Quando i processi hanno il proprio spazio degli indirizzi, allora sono chiamati processi.
 - I processi possono comunicare attraverso meccanismi messi a disposizione dall'OS
 - Tutte le comunicazioni passano dall'OS

Qiando i processi condividono lo stesso spazioni degli indirizzi, allora sono chiamati thread.
 - I thread comunicano senza passare dall'OS, ma spesso attraverso spazi di memoria condivisa

<br>

In Java, ogni programma in esecuzione è un thread. <br>
Il metodo `main()` è associato al main thread. <br>
Per accedere alle proprietà del main thread è necessario ottenere un riferimento tramite il metodo `currentThread()`.

```java
public class ThreadMain {
    public static void main(String args []) {
        Thread t = Thread.currentThread( );
        System.out.println(”Thread corrente: ” + t );
        t.setName(”Mio Thread”);
        System.out.println(”Dopo cambio nome: ” + t );
    }
}

//OUTPUT
// Thread corrente: Thread[main,5,main]
// Dopo cambio nome: Thread[Mio Thread,5,main]

// Thread[Nome thread, priorità, gruppo di appartenenza]
```

### Programmazione concorrente
per programmazione concorrente si intende l'implementazione di programmi che contengono più flussi di esecuzione. È una buona pratica tenere leggero il main thread e delegare l'esecuzione a più thread concorrenti.

<br>

# Thread in Java

## `java.lang.Thread`

Per eseguire un thread in Java:
1. Si estende la classe `java.lang.Thread`, che contiene il metodo `run()` vuoto
2. Si effettua l'ovverride del metodo `run()`
3. Il codice inserito nel metodo, sarà utilizzato dal thread
4. Si crea un'istanza della sottoclasse
5. Si richiama il metodo `start()` in modo da far partire il thread

```java
public class ThreadExample extends Thread {
    public void run() {
        System.out.println("run");
    }
    public static void main(String arg[]){
        ThreadExample t1=new ThreadExample();
        t1.start();
    }
}
```

Inserendo il metodo `start()` nel costruttore della classe, si può far partire il thread in modo 'automatico'.

```java
public class ThreadExample extends Thread {
    ThreadExample(){
        this.start();
    }
    public void run() {
        System.out.println("run");
    }
    public static void main(String arg[]){
        ThreadExample t1=new ThreadExample();
    }
}
```

## `java.lang.Runnable`
Per la creazione di un nuovo thread si puo' utilizzare anche l'interfaccia `java.lang.Runnable`:
1. Si definisce una classe che implementa `Runnable` dotata di un metodo `run()`
2. Si crea l'istanza di questa classe
3. Si istanzia un nuovo `Thread` passando al costruttore l'istanza della classe che implementa `Runnable`
4. Si chiama il metodo `start()` sull'istanza di `Thread`

```java
public class RunnableExample implements Runnable{
    public void run() {
        System.out.println("run");
    }
    public static void main(String arg[]){
        RunnableExample re = new RunnableExample();
        Thread t1=new Thread(re);
        t1.start();
    }
}
```

### Il metodo `run()`
Il metodo `run()` costituisce l'entry point di un thread. <br>
Un thread è considerato <b>alive</b> finché il metodo `run()` non termina. <br>
Quando `run()` termina, il thread va in stato <b>dead</b> <br>
Una volta che il thread è in stato <b>dead</b> NON puo' essere rieseguito (altrimenti verrà generata un'eccezione), ma dovrà essere creata una nuova istanza per rieseguire quel thread.

### `start()`
Una chiamata `t.start()` rende il thread `t` pronto all'esecuzione. Successivamente, in base alle politiche di scheduling, lo scheduler invocherà il metodo `run()` del thread `t`.

<b>!</b> : L'ordine con cui ogni thread eseguirà le sue istruzioni è noto, ma l'ordine di esecuzione delle istruzioni tra i vari thread è indeterminato o non deterministico.


### Stati di un thread
![Thread States](/assets/programmazione_concorrente_e_distribuita/thread-states-in-java.png)

<br>

# Programma Concorrente

```java
public class ThreadExample extends Thread{
    public void run() {
        System.out.println("I'm " + Thread.currentThread());
    }
    public static void main(String args []) {
        ThreadExample t1=new ThreadExample();
        ThreadExample t2=new ThreadExample();
        ThreadExample t3=new ThreadExample();
        t1.start();
        t2.start();
        t3.start();
        System.out.println("Main: completed");
    }
}
```

L'esecuzione di questo codice non è deterministico in quanto ogni linea di codice viene schedulata in base alle risorse della macchine ed alle politiche di scheduling.
Quindi, i risultati di questo programma puo' variare ad ogni esecuzione.

## Flusso di controllo
Un programma termina quando tutti i suo thread terminano. <br>
L'unica eccezione sono i <b>deamon thread</b>.

Quindi un programma termina quando tutti i suo thread NON deamon termiano.

### Deamon thread
Questa tipologia di thread forniscono un servizio generale e non essenziale in background mentre il programma esegue altre operazioni.

I <b>deamon thread</b> sono particolari tipi di Thread che hanno le seguenti caratteristiche:
- Bassa priorità
- Normalmetne utilizzati come fornitori di servizi per i thread normali
- Non impediscono l'arresto del programma

Tipicamente si creano inserendo l'istruzione `setDeamon(true)` nel costruttore di un thread.

```java
public class DaemonExample extends Thread {
    public DaemonExample(boolean isDaemon) {
        setDaemon (isDaemon);
    }
    public void run() {
        int count = 0;
        while(true) {
            System.out.println("run " + count++);
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) { }
        }
    }
    public static void main(String args []) {
        DaemonExample t1 = new DaemonExample(true);
        t1.start();
        try {
            Thread.sleep(7000);
        } catch (InterruptedException e) { }
        System.out.println("Main thread terminates.");
    }
}
```

# Politiche di Scheduling
La JVM schedula l'esecuzione dei thread utilizzando un algoritmo di scheduling <b>preemptive e priority based</b>.
- Tutti i thread hanno una priorità e il thread con la priorità più alta tra quelli ready viene schedulato per essere eseguito.
- Con il diritto di preemption lo scheduler puo' sottrarre la CPU al processo che la sta usando per assegnarla ad un altro processo

Le politiche di scheduling della JVM dipendono dall'OS.

## Esempio di programma concorrente

```java
public class NondeterminismExample extends Thread{
    final static int numIterations = 10;
    public void run() {
        for(int i=0; i<numIterations; i++) {
            System.out.println("new thread");
        }
    }
    public static void main(String args []) {
        NondeterminismExample t = new NondeterminismExample();
        t.start();
        for(int i=0; i<numIterations; i++) {
            System.out.println("Main");
        }
    }
}

//Questo esempio potrebbe generare diversi output in base allo scheduling.
```
Un output potrebbe essere

![OutputExample](/assets/programmazione_concorrente_e_distribuita/concurrent-eg.png)

## Sleep
Se si vuole essere certi che dopo `t.start()` inizi ad eseguire il task `t`, un modo semplice consiste nel madare in sleep il `main`.

```java
public class NondeterminismExample extends Thread{
    final static int numIterations = 5;
    public void run() {
        for(int i=0; i<numIterations; i++) {
            System.out.println("new thread");
        }
    }
    public static void main(String args []) {
        NondeterminismExample t = new NondeterminismExample();
        t.start();
        try { Thread.sleep(1);
        } catch (InterruptedException e) { }
            for(int i=0; i<numIterations; i++) {
            System.out.println("Main");
        }
    }
}
```

Ma, nulla vieta che succeda questo

![OutputExample](/assets/programmazione_concorrente_e_distribuita/output_example.png)

### Descrizione del metodo `sleep()`

```java
public static native void sleep (long msToSleep) throws InterruptedException
```

Il metodo `sleep()` non utilizza cicli del processore e quindi non occupa inutilmente la CPU (come in un busy loop). Si tratta di un metodo statico che mette in pausa il thread corrente.

<b>!</b> : scaduto il tempo di sleep, il thread torna Ready, ma non è detto che vada subito in esecuzione. Quindi, `sleep(100)` sospende il thread per <b>almeno</b> 100ms.

## Il metodo `isAlive()`
Questo metodo può essere utilizzato per vedere se un thread è vivo.
Un thread è considerato vivo dopo che è stato chiamato `start()`, e resta Alive fino a quando termina l'esecuzione del metodo `run()`.

Potrebbe essere utilizzato per attendere la terminazione di un thread
```java
while(t1.isAlive()) {
    Thread.sleep(100);
}
```
Ma non è efficiente, infatti si preferisce l'utilizzo del metodo `join()`.

## Il metodo `join()`
- Tale metodo attende la terminazione del thread sul quale è stato chiamato.
- Il thread che esegue `join()` rimane bloccato in attesa della terminazione dell'altro thread
- Può lanciare una `InterruptedException`

```java
public class JoinWaitExit {
    public static void main(String args[]) {
        System.err.println("I thread stanno per partire");
        Thread t1 = new ExampleWithSleeps("t1"); t1.start();
        Thread t2 = new ExampleWithSleeps("t2"); t2.start();
        Thread t3 = new ExampleWithSleeps("t3"); t3.start();
        System.err.println("I thread sono partiti\n");

        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) { }

        System.err.println("chiude l'applicazione");
        System.exit(0); //chiude l'applicazione in ogni caso
    }
}
```

Il metodo `join()` può essere chiamato con diversi parametri.

| Type  | Method                       | Description                                                                      |
| ----- | ---------------------------- | -------------------------------------------------------------------------------- |
| void  | join()                       | Waits for this thread to die.                                                    |
| void  | join(long millis)            | Waits at most millis milliseconds for this thread to die.                        |
| void  | join(long millis, int nanos) | Waits at most millis milliseconds plus nanos nanoseconds for this thread to die. |


# Terminare un thread
Il metodo `interrupt()` setta un flag di interruzione nel thread di destinazione.
Il thread che riceve l'interrupt, non viene effettivamente interrotto, ma è compito del programmatore andare l'`InterruptedException` e far eventualmetne terminare il thread.

```java
public class ExampleWithSleeps extends Thread {
    final static int numIterations = 10;
    public ExampleWithSleeps(String s) {
        super(s);
    }
    public void run() {
        for(int i=0; i<numIterations; i++) {
            System.out.println("thread "+getName()+" in esec.");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                System.err.println(getName() + ": interrotto");
                break;
            }
        }
        System.out.println("thread "+getName()+": finito.");
    }
}
```

Il metodo `interrupt()` non genera eccezioni se il thread 'interotto' non esegue mai metodi di attesa come: `sleep()`, `join()`, ...

I thread che si aspettano degli interrupt devono cooperare e gestire tali stati.
- `isInterrupted()` controlla il flag di interruzione senza resettarlo
- `Thread.interrupted()` controlla il flag di interruzione corrente e, se settato, lo resetta

```java
//Thread che controlla la ricezione di un interrupt
public void run(){
    boolean interrupted=false;
    while(condizione){
        // esegue un'operazione complessa
        if (Thread.interrupted()){
            interrupted=true;
            break;
        }
    }
    if(interrupted) {
        // gestisce l'interruzione
    }
}
```

# Il metodo `yield()`
la chiamata al metodo statico `Thread.yield()` da parte di un thread sta ad indicare che tale thread ha terminato le funzioni principali e che lo scheduler può dedicare CPU ad un altro thread.

In pratica, con `yield()` si cede il controllo allo scheduler, che sceglierà un altro thread (se esiste) da mandare in esecuzione al posto di quello che ha fatto `yield()`.

```java
public class Decollo implements Runnable {
    protected int countDown = 10;
    private static int taskCount = 0;
    private final int id = taskCount++;
    public String status () {
        return " "+id+"("+
        (countDown > 0 ? countDown: "Decollo !")+")";
    }
    public void run() {
        while(countDown-- > 0) {
            System.out.println(status());
            Thread.yield();
        }
    }
}

public class YieldExample {
    public static void main(String[] args) {
        for (int i=0; i<5; i++) {
            new Thread (new Decollo()).start();
        }
        System.out.println("In attesa del decollo");
    }
}
```