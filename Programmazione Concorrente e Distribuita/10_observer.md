# Events Management - Observer
## Pattern observer
Il pattern <b>observer</b> riassume e formalizza il meccanismo di gestione degli eventi.<br>
Esso è un pattern comportamentale che gestisce la comunicazione tra oggett, definendo il modo in cui un certo numero di classi possono ricevere _notifiche di eventi_ a cui sono interessate.

L'observer devinisce una dipendenza _uno a molti_ tra oggetti: per ogni oggetto osservato ci possono essere tanti osservatori, che ricevono tutti le notifiche per ogni evento.

### Motivi
La motivazione principale del pattern observer è quella di evitare il _polling_, in quanto:
- poco efficiente
- poco efficace, poiché l'evento potrebbe verificarsi subito dopo che si è fatta l'interrogazione

### Partecipanti e comportamento
I partecipanti del pattern observer sono:
- il soggetto osservabile, dove accadono gli eventi
- gli osservatori, che ricevono le notifiche

![ClassDiagram](/assets/programmazione_concorrente_e_distribuita/observer_classdiagram.png)

- Sono definiti i ruoli astratti di soggetto osservabile: `Subject` (a volte chiamata `Observable`) e di `Observer`. Poi si creano le classi concrete che ricoprono questi ruoli implementando la logica specifica per l'applicazione.
- La classe `Observer` è dotata di un metodo `inform` (a volte chiamato `update`), che viene invocato per notificare l'osservatore di un evento.
- `Subject` prevede i metodi:
    - `attach` e `detach`, per aggiungere e togliere osservatori
    - `notify`, per notificare gli osservatori (solitamente invocato per segnalare cambiamenti di stato del `ConcreteSubject`)


# Observer in Java
In Java, non è necessario implementare da zero il pattern observer, poiché il package `java.util` mette già a disposizione un'interfaccia `Observer` e una classe `Observable`.

- L'interfaccia `Observer` prevede un unico metodo:
    - `void update(Observable o, Object arg);`
    viene chiamato dall'oggetto `Observable` per notificare un'evento all'Observer.<br>
    Argomenti:
    - `Observable o`: riferimento dell'oggetto dal quale proviene l'evento.
    - `Object arg`: oggetto di qualsiasi tipo (per rendere generale il processo di notifica)
<br>

- I metodi della classe `Observable` sono:
    - `public void addObserver(Observer o);` : aggiunge un'osservatore al soggetto osservabile
    - `public void notifyObservers();` o `public void notifyObservers(Object arg);` :  notifica tutti gli osservatori, chiamando il metodo update di ciascuno di essi
    - `protected void setChanged();` : indica come cambiato il soggetto
    - `protected void setChanged();` : indica come non cambiato il soggetto (Esso può essere chiamato manualmente, se necessario, ma altrimenti viene chiamato automaticamente da notifyObservers dopo l’invio delle notifiche: così, un osservatore non riceverà mai due notifiche relative allo stesso cambiamento)
    - `public boolean hasChanged();` : ritorna lo stato del soggetto

## Esempio
- il soggetto osservato gestisce l’input da tastiera;
- l’inserimento di ogni riga è considerato un evento, che deve essere notificato agli osservatori.

Il programma verrà realizzato usando l’implementazione dell’observer fornita dalla libreria standard: a ogni stringa letta da tastiera, si chiamerà `notifyObserver` per informare tutti gli osservatori dell’evento di lettura.

### Osservatore concreto
L’osservatore concreto è la classe in grado di gestire l’input: esso viene ricevuto come argomento del metodo update, convertito in una stringa, e semplicemente stampato:

```java
import java.util.Observable;
import java.util.Observer;

public class InputHandler implements Observer {
    public void update(Observable o, Object arg) {
        String info = (String) arg;
        System.out.println(info + " communicated");
    }
}
```

## Soggetto concreto
Il soggetto osservabile è un thread che, per ogni stringa letta da tastiera (in un ciclo infinito), chiama setChanged e notifyObserver in modo da inviare la stringa a tutti gli osservatori:

```java
import java.io.*;
import java.util.Observable;
public class EventSource extends Observable implements Runnable {
    public void run() {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in), true)) {
            while (true) {
                System.out.print("Enter text > ");
                System.out.flush();
                String str = br.readLine();
                setChanged();
                notifyObservers(str);
            }
        } catch (IOException e) {}
    }
}
```

## Main
Nel main bisogna semplicemente istanziare il soggetto e l’osservatore, registrare l’osservatore presso il soggetto, e avviare un thread per il soggetto:
```java
public class MyApp {
    public static void main(String[] args) {
        EventSource source = new EventSource();
        InputHandler handler = new InputHandler();
        source.addObserver(handler);
        Thread sourceThread = new Thread(source);
        sourceThread.start();
        // Adesso MyApp può proseguire la sua elaborazione senza
        // preoccuparsi di gestire l'input.
        // ...
    }
}
```

# Pattern Observer con RMI
Per mostrare come il pattern observer può essere utilizzato in un’applicazione distribuita, realizzata con RMI, verrà creato un server che, ogni 5 secondi, invia l’ora esatta a tutti i client (che ne hanno fatto richiesta). <br>Dunque, il server è un Observable, e i client sono Observer.

Essendo necessario l’uso di callback, client e server sono entrambi oggetti remoti, come al solito.

## Interfaccia remota client
L’interfaccia remota `RemoteObserver` assomiglia molto all’interfaccia `Observer`
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface RemoteObserver extends Remote {
    void remoteUpdate(Object observable, Object updateMsg) throws RemoteException;
}
```

## Client
Il client implementa l’interfaccia RemoteObserver, semplicemente stampando i messaggi ricevuti. Anche il main non è particolarmente interessante: dopo la solita parte di preparazione dei riferimenti remoti, il deve solo client registrarsi come osservatore presso il server.
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;

public class RemoteObserverImpl implements RemoteObserver {
    public void remoteUpdate(Object observable, Object updateMsg) {
        System.out.println("Got message: " + updateMsg);
    }

    public static void main(String[] args) throws Exception {
        Registry registry = LocateRegistry.getRegistry();
        RMIObservableService remoteService = (RMIObservableService) registry.lookup("Observable");
        
        RemoteObserver client = new RemoteObserverImpl();
        RemoteObserver clientStub = (RemoteObserver)
        UnicastRemoteObject.exportObject(client, 3949);
        remoteService.addObserver(clientStub);
    }
}
```

## Interfaccia remota server
L’interfaccia remota `RMIObservableService` prevede solo il metodo addObserver (analogo all’omonimo metodo della classe `java.util.Observable`), che permette la registrazione di un osservatore remoto:
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface RMIObservableService extends Remote {
    void addObserver(RemoteObserver o) throws RemoteException;
}
```

## Server
Il server è un `Observable` che implementa `RMIObservableService`
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.Date;
import java.util.Observable;

public class RMIObservableServiceImpl extends Observable implements RMIObservableService {
    public void addObserver(RemoteObserver o) {
        WrappedObserver wo = new WrappedObserver(o);
        addObserver(wo);
        System.out.println("Added observer: " + wo);
    }

    private void run() throws InterruptedException {
        while (true) {
            Thread.sleep(5000);
            setChanged();
            notifyObservers(new Date());
        }
    }

    public static void main(String[] args) throws RemoteException, InterruptedException {
        RMIObservableServiceImpl obj = new RMIObservableServiceImpl();
        RMIObservableService stub = (RMIObservableService) UnicastRemoteObject.exportObject(obj, 3939);
        
        Registry registry = LocateRegistry.createRegistry(1099);
        registry.rebind("Observable", stub);
        System.err.println("Server ready");
        obj.run();
    }
}
```

- l'implementazione di `RMIObservableService.addObserver` crea un wrapper locale per l’observer remoto e lo passa a `Observable.addObserver`.
- l metodo `run` (che, a differenza di quanto si potrebbe pensare dal nome, qui nondefinisce un thread) contiene un ciclo infinito in cui, ogni 5 secondi, si usano i metodi `setChanged` e `notifyObservers` di `Observable` per inviare la data e ora correnti ai `WrappedObserver`, che la inoltreranno agli osservatori remoti.

## Classe `WrappedObserver`
La classe WrappedObserver, che implementa Observer, funge da proxy locale al server per gli osservatori remoti:
```java
import java.rmi.RemoteException;
import java.util.Observable;
import java.util.Observer;

public class WrappedObserver implements Observer {
    private final RemoteObserver remoteClient;

    public WrappedObserver(RemoteObserver ro) {
        remoteClient = ro;
    }
    public void update(Observable o, Object arg) {
        try {
            remoteClient.remoteUpdate(o.toString(), arg);
        } catch (RemoteException e) {
            System.err.println("Remote exception; removing observer: " + this);
            o.deleteObserver(this);
        }
    }
}
```
Ogni volta che viene inviata una notifica a questo osservatore, tramite una chiamata locale del metodo update, esso prova a inoltrarla al client remoto, invocando il metodo `remoteUpdate` di quest’ultimo, e inoltrando anche gli argomenti della chiamata.

Se l’invocazione remota fallisce, si presume che l’osservatore remoto si sia disconnesso, perciò si indica all’`Observable` di rimuovere anche il proxy locale.
