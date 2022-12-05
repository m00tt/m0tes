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

