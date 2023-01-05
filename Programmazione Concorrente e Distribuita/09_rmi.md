# Remote Method Invocation
RMI è una tecnologia del mondo Java che permette a processi distribuiti di comunicare attraverso la rete.<br>
Un processo client può effettuare chaiamate di un metodo ad un oggetto remoto (server) come se l'oggetto fosse sulla stessa macchina.

Tipicamente, quando si realizza un'architettura client-server con RMI:
- il server crea gli oggetti remoti, li rende visibili e aspetta che i client invochino metodi su di essi
- il client ottiene dei riferimenti agli oggetti remoti ed invoca metodi su di essi

RMI fornisce il meccanismo attraverso il quale server e client comunicano per costruire un'applicazione distribuita.

# Registry
Affinché un server possa esporre un oggetto remoto attraverso RMI, questo oggetto deve essere pubblicato su un <b>registry</b>.
- Il server crea l'oggetto e lo pubblica sul registry
- Il client cerca presso il registry il servizio di cui ha bisogno, e gli viene restituito un riferimento all'oggetto remoto sul server
- Il client usa il riferimento per poter ottenere il servizio

# Message passing
Con RMI, l'interazione tra client e server è quella tipica dei sistemi object-oriented: <b>il message passing</b>

L'invocazione di un metodo di un oggetto remoto avviene mediante l'invio di un messaggio che contiene il nome del metodo e gli eventuali argomenti.
L'oggetto remoto, a sua volta, invia un messaggio al chiamante con il risultato del metodo.

La costruzione, spedizione, ricezione e ricostruzione dei messaggi sono gesite da RMI, e sono trasparenti al programmatore.

## Passaggio di parametri
Consideriamo la chiamata:

```java
String result = server.sayHello(obj);
```
`server` è un oggetto remoto.
In questo caso si ha un comportamento diverso a seconda della natura dell'argomento `obj`:
- Se è un riferimento ad un oggetto remoto, al metodo remoto viene passato tale riferimento
- Se è un oggetto locale, viene inviata al metodo remoto una copia dell'oggetto mediante serializzazione. In particolare quello che avviene è una _deep copy_: se l'oggetto contiene riferimenti ad altri oggetti, anche questi vengono serializzati e passati al server
- Se è di tipo primitivo, viene passato per valore

Potrebbe essere che l'oggetto remoto non conosca la classe di `obj`. In questo caso, è possibile passare al server anche il codice di tale classe (<b>dynamic class loading</b>).


# Implementazione di RMI
Un'invocazione di un metodo di un oggetto remoto è fornita localmente da uno <b>stub</b>: esso si occupa della clareazione del messaggio contenente il nome e gli argomenti del metodo (parameter marshalling), e l'invio verso il server.

Lato server, il messaggio è ricevuto da uno <b>skeleton</b>: esso ricostruisce i parametri (unmarshalling) e chiama il metodo sull'oggetto vero e proprio. Successivamente prepara il risultato e lo invia allo stub.

## Layer

![RMI Layers](/assets//programmazione_concorrente_e_distribuita/rmi_layer.png)

- Transport Layer: TCP
- Remote Reference Layer (RRL): crea e gestisce i riferimenti remoti
- stub e skeleton comunicano tra di loro utilizzando i riferimenti remoti e fanno da intermediari a client e server

Tutta questa struttura è praticamente trasparente al programmatore.

# Code
## Oggetti remoti
Un server RMI è un oggetto remoto.<br>
Un oggetto remoto è descritto da un'<b>interfaccia remote</b>:
- che estende `java.rmi.Remote`
- i cui metodi devono dichiarare l'eccezione `java.rmi.RemoteException`

```java
import java.math.BigInteger;
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface PowerService extends Remote {
    BigInteger square(int n) throws RemoteException;
    BigInteger power(int n1, int n2) throws RemoteException;
}
```

## Passaggio di parametri
- Se si passa un oggetto locale o un valore di tipo primitivo, RMI fa automaticamente marshalling e unmarshalling dei parametri, purché i tipi dei parametri siano _serializzabili_. Quindi, gli oggetti passati o restituiti da un oggetto remoto devono implementare l'interfaccia `Serializable`.

- Se si passa un riferimento ad un oggetto remoto, viene effettivamente trasmesso solamente il riferimento, senza bisogno di serializzare l'oggetto.

## Fasi di sviluppo di un'app
1. Definizione dell'interfaccia remota
2. Definizione del codice dell'oggetto remoto, che deve:
    - Implementare l'interfaccia remota
    - Estendere la classe `java.rmi.server.UnicastRemoteObject` e chiamare uno dei suoi costruttori, o, in alternativa, non estendere tale classe ma chiamare uno dei suoi metodi statici `exportObject`
    - Pubblicare il servizio sul registry
3. Definire il codice del client
    - Richiedere al registry un riferimento all'oggetto remoto
    - Assegnare il riferimento ad una variabile che ha come tipo l'interfaccia remota

### Esempio
Come primo esempio, si realizza un’applicazione client–server nella quale il client invoca un metodo sayHello del server, che non ha parametri e restituisce una stringa.

1. Creazione dell'interfaccia remota
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Hello extends Remote {
    String sayHello() throws RemoteException;
}
```

2. Implementazione del server
```java
import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class HelloImpl extends UnicastRemoteObject implements Hello {

    public HelloImpl() throws RemoteException {
        super(); //rende ogni istanza di HelloImpl un oggetto remoto
    }

    public String sayHello() {
        return "Hello, world!";
    }
    public static void main(String[] args) throws Exception {
        if (System.getSecurityManager() == null) {
            System.setSecurityManager(new SecurityManager());
        }
        HelloImpl obj = new HelloImpl(); //crea l'oggetto da esporre
        Naming.rebind("HelloServer", obj); //lo pubblica sul registry
        System.out.println("Server ready");
    }
}
```

3. Implementazione del client
```java
import java.rmi.Naming;

public class HelloClient {
    public static void main(String[] args) throws Exception {
        System.setSecurityManager(new SecurityManager());

        Hello stub = (Hello) Naming.lookup("//" + args[0] + "/HelloServer"); //crea il riferimento all'oggetto tramite il registry
        String response = stub.sayHello(); //invoca il metodo dell'oggetto remoto
        System.out.println("response: " + response);
    }
}
```

## Sicurezza in RMI
Con RMI, è possibile caricare delle classi da remoto. Queste sono necessariamente considerate “non affidabili” (in quanto potrebbero contenere codice malevolo), quindi, di
default, il caricamento delle classi remote è disabilitato (ogni tentativo di farlo genera un errore).

Per consentirlo, è necessario usare un opportuno <b>security manager</b>, che è un’istanza della classe `java.lang.SecurityManager` (o di una sua sottoclasse).<br>
L’istruzione per usare un security manager è
```java
System.setSecurityManager(new SecurityManager());
```
e deve essere eseguita prima che RMI abbia bisogno di caricare classi da remoto (dunque, tipicamente, come prima operazione).

Se si eseguono client e server sulla stessa macchina, è facile fare in modo che abbiano entrambi accesso a tutte le classi necessarie, quindi un SecurityManager è superfluo.

<b>ATTENZIONE</b>: Dal 2021, l'utilizzo del SecurityManager è deprecato. Non vi è un sostituto o l'intenzione di un sostituto lato Java in quanto è stata delegata l'attività alla JVM.

### Policy
Un security manager richiede un <b>file di policy</b>, in cui si specifica quali tipi di operazioni possono eseguire le classi caricate dai vari codebase sulla rete.<br>
Tale file può essere collocato in una qualsiasi directory e può avere un nome qualsiasi: la sua posizione viene specificata attraverso la proprietà
`java.security.policy`, che si imposta nel comando usato per avviare la JVM:
```java
java -Djava.security.policy=<nomefile> <programma> <argomenti...>
```

Il file di policy più semplice è quello che definisce una “global permission”, cioè permette a tutte le classi caricate di eseguire tutti i tipi di operazioni.
```
grant {
    	permission java.security.AllPermission;
};
```
Ovviamente, non è opportuno usare una policy di questo tipo in un ambiente di produzione.

## Registry
Il registry è un processo che solitamente gira sul server (ma non obbligatoriamente) e di default è in ascolto sulla porta 1099.

Il server, per poter esporre un suo oggetto/servizio, deve registrarlo presso il registry associandogli una label nota ai client.
```java
Naming.rebind("HelloServer", obj) //Assegno l'etichetta "HelloServer" all'oggetto obj
```

### Metodi dell'interfaccia Registry

| Metodo                                                                                               | Descrizione                                                         |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `void bind(String name, Remote obj) throws RemoteException, AlreadyBoundException, AccessException;` | Associa un riferimento remoto a un nome                             |
| `void rebind(String name, Remote obj) throws RemoteException, AccessException;`                      | Come `bind`, ma sovrascrive un’eventuale associazione già esistente |
| `void unbind(String name) throws RemoteException, NotBoundException, AccessException;`               | Elimina l’associazione di un riferimento remoto a un nome           |
| `String[] list() throws RemoteException, AccessException;`                                           | Fornisce l’elenco dei servizi di cui il registry è a conoscenza     | 
| `Remote lookup(String name) throws RemoteException, NotBoundException, AccessException;`             | Restituisce il riferimento remoto associato a un nome               |

<b>bind, rebind e unbind</b> sono utilizzati dal server, mentre <b>list e lookup</b> sono utilizzati dal client.


## Esecuzione dell'applicazione
Poniamo di prendere come esempio il codice [qui](#esempio), per eseguirlo bisogna seguire i seguenti passaggi:
1. Si avvia il registry
    - `rmiregistry <port>` se si vuole cambiare la porta, altrimenti verrà usata quella di default (1099)
    - Si esegue il comando:
        - Unix > `rmiregistry &`
        - Windows > `start rmiregistry`

2. Si avvia il server indicando il percorso del file di policy. Ad esempio, se quest’ultimo è chiamato policy e si trova nella stessa cartella di HelloImpl.class:
```java
java -Djava.security.policy=policy HelloImpl
```

3. Si avvia il client indicando il percorso del file di policy, e, in questo caso si passa come argomento il nome host del server
```java
java -Djava.security.policy=policy HelloClient localhost
```

## Deployment

![RMI Deploy](/assets/programmazione_concorrente_e_distribuita/rmi_deploy.png)

Questo tipo di deployment è detto _statico_, perché si posizionano manualmente sulle varie macchine i file di cui esse hanno bisogno, quindi non è necessario il _dynamic class loading_.

<b>! </b>: Se si dovessero usare directory diverse, il comando `rmiregistry` deve essere eseguito nella directory dove si sono messi i file del server, altrimenti potrebbero verificarsi degli errori.

## Esempio 2: passaggio di un argomento
In questo secondo esempio il metodo dell’oggetto remoto che il client invoca ha un argomento di tipo non primitivo, che deve quindi essere _serializzabile_.

```java
//Definizione della classe serializzabile
import java.io.Serializable;

public class Person implements Serializable {
    private static final long serialVersionUID = 1;
    private String name;
    public Person(String name) { this.name = name; }
    public String getName() { return name; }
}


//Adattamento dell'interfaccia remota in modo che preveda anche il metodo "sayHello(Person p)"
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface HelloPerson extends Remote {
    String sayHello() throws RemoteException;
    String sayHello(Person p) throws RemoteException;
}

//Server
import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class HelloPersonImpl extends UnicastRemoteObject implements HelloPerson {
    public HelloPersonImpl() throws RemoteException {
        super();
    }

    public String sayHello() {
        return "Hello, world!";
    }

    public String sayHello(Person p) { //Aggiunge la definizione del metodo sayHello(Person p)
        return "Hello, " + p.getName();
    }

    public static void main(String[] args) throws Exception {
        if (System.getSecurityManager() == null) {
            System.setSecurityManager(new SecurityManager());
        }
        HelloPersonImpl obj = new HelloPersonImpl();
        Naming.rebind("HelloPersonServer", obj);
    }
}

//Client
import java.rmi.Naming;

public class HelloPersonClient {
    public static void main(String[] args) throws Exception {
        System.setSecurityManager(new SecurityManager());
        
        HelloPerson stub = (HelloPerson) Naming.lookup("//" + args[0] + "/HelloPersonServer");
        
        String response = stub.sayHello();
        System.out.println("response: " + response);
        
        Person someone = new Person("Caio Sempronio"); //Creazione dell'oggetto Person e invio verso lo skeleton
        response = stub.sayHello(someone);
        System.out.println("response: " + response);
    }
}
```

### Deployment

![RMI Deploy](/assets/programmazione_concorrente_e_distribuita/rmi_deploy2.png)


## Gestione alternativa del Registry (migliore)
Invece della classe `Naming`, per gestire il registry si può usare la classe `LocateRegistry`, che consente di ottenere un oggetto di tipo `Registry`.

I metodi messi a disposizione dalla classe sono:
- Diverse varianti di `getRegistry()`
    - `public static Registry getRegistry() throws RemoteException;`
    - `public static Registry getRegistry(int port) throws RemoteException;`
    - `public static Registry getRegistry(String host) throws RemoteException;`
    - `public static Registry getRegistry(String host, int port) throws RemoteException;`

Che permettono di ottenere un riferimento ad un registry esistente.

- `public static Registry createRegistry(int port) throws RemoteException;`

Che permette di creare un registry sull'host corrente e sulla porta specificata, eliminando la necessità di avviare un registry separatamente.

### Esempio con `createRegistry`
```java
//Server
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class HelloPersonImpl extends UnicastRemoteObject implements HelloPerson {
    public HelloPersonImpl() throws RemoteException {
        super();
    }

    public String sayHello() {
        return "Hello, world!";
    }

    public String sayHello(Person p) { //Aggiunge la definizione del metodo sayHello(Person p)
        return "Hello, " + p.getName();
    }

    public static void main(String[] args) throws RemoteException {
        if (System.getSecurityManager() == null) {
            System.setSecurityManager(new SecurityManager());
        }
        HelloPersonImpl obj = new HelloPersonImpl();
        Registry registry = LocateRegistry.createRegistry(1099); //Creazione del registry sul server sulla porta 1099
        registry.rebind("HelloPersonServer", obj);
    }
}

//Client
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class HelloPersonClient {
    public static void main(String[] args) throws Exception {
        System.setSecurityManager(new SecurityManager());
        
        Registry registry = LocateRegistry.getRegistry(args[0], 1099); //getRegistry per accedere al registry creato dal server
        HelloPerson stub = (HelloPerson) registry.lookup("HelloPersonServer");
        
        String response = stub.sayHello();
        System.out.println("response: " + response);
        
        Person someone = new Person("Caio Sempronio");
        response = stub.sayHello(someone);
        System.out.println("response: " + response);
    }
}
```

## Deploy in ambiente distribuito
Quando si esegue il deployment di un’applicazione RMI in un ambiente distribuito, come già detto si hanno tipicamente server e registry su una macchina, e il client su un’altra.

Potrebbe capitare che il server invii al client degli oggetti di classi che quest’ultimo non conosce. Per permettere al client di procurarsi dinamicamente il codice di tali classi, si possono installare i file `.class` su un web server.

Quando si lancia il server RMI, bisogna indicare, con la proprietà java.rmi.server.codebase, l’URL dove si trovano le classi:
```java
java -Djava.security.policy=policy
     -Djava.rmi.server.codebase=<URL> ServerMain
```

# RMI Callback
Nel paradigma distribuito object-oriented, i ruoli di client e server non sono rigidamente fissati: infatti anche il client può comportarsi come server.

Se il client è a sua volta un oggetto remoto, può inviare il suo riferimento al server, permettendo a quest'ultimo di chiamare metodi remoti del client.<br>
Da qui, il termine <b>callback</b>.

# Esempio: sistema di chat
Al fine di introdurre la tecnica del _callback_, si svilupperà un sistema di chat, dove:
- i partecipanti (client) entrano in una stanza virtuale gestita dal server
- ogni volta che un partecipante invia un messaggio sulla chat, tutti i partecipanti alla stanza ricevono il messaggio
- i client hanno la possibilità di abbandonare la stanza

## Interfaccia remota del server
Il server definisce, nella propria interfaccia remota, i metodi che i client invocheranno per inviare i dati.
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface TtyChat extends Remote {
    void enterRoom(TtyChatClient client) throws RemoteException;
    void exitRoom(TtyChatClient client) throws RemoteException;
    void saySomething(String something, TtyChatClient speaker) throws RemoteException;
}
```
- `TtyChatClient` è l'interfaccia remota del client
- Ogni client che vuole entrare nella stanza, invocherà il metodo `enterRoom`
- Quando un client vuole inviare un messaggio, invoca il metodo `saySomething`

## Implementazione del server
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.*;

public class TtyChatImpl extends UnicastRemoteObject implements TtyChat { //Le istanze di questa classe saranno remote dato che estende UnicastRemoteObject
    private final List<TtyChatClient> occupants = new ArrayList<>(); //lista di partecipanti
    
    public TtyChatImpl() throws RemoteException {}

    public synchronized void enterRoom(TtyChatClient client) {
        occupants.add(client); //viene aggiunto un nuovo partecipante alla stanza
    }

    public synchronized void exitRoom(TtyChatClient client) {
        occupants.remove(client); //viene rimosso un partecipante dalla stanza
    }

    public synchronized void saySomething(String something, TtyChatClient speaker) throws RemoteException {
        String message = speaker.name() + ": " + something; //viene effettuata una chiamata al client per recuperare il nome
        System.out.println(Thread.currentThread() + ":Server: received '" + message + "'");

        Iterator<TtyChatClient> iter = occupants.iterator();
        while (iter.hasNext()) {
            TtyChatClient client = iter.next();
            try {
                client.somethingSaid(message); //viene invocato il metodo del client per fargli stampare il messaggio
            } catch (RemoteException e) {
                System.out.println("Someone left");
                iter.remove(); //se viene generato un errore, probabilmente il client si è disconnesso, quindi viene tolto dalla lista
            }
        }
    }
}


//Main
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;

public class TtyChatServer {
    public static void main(String[] args) throws RemoteException {
        TtyChatImpl obj = new TtyChatImpl(); //creazione dell'oggetto remoto
        Registry registry = LocateRegistry.createRegistry(Registry.REGISTRY_PORT); //creazione del registry
        registry.rebind("TtyChat", obj);  //bind dell'oggetto sul registry
        System.out.println("TtyChat Server bound in registry");
    }
}
```

## Implementazione del client
```java
import java.io.*;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class TtyChatClientImpl extends UnicastRemoteObject implements TtyChatClient {
    private final String myName;

    public TtyChatClientImpl(String myName) throws RemoteException {
        this.myName = myName;
    }

    public String name() {
        return myName;
    }

    public void somethingSaid(String something) {
        System.out.println(something);
    }

    public static void main(String[] args) throws Exception {
        try (BufferedReader input = new BufferedReader(new InputStreamReader(System.in))) {
            System.out.println("What is your name?");
            TtyChatClientImpl me = new TtyChatClientImpl(input.readLine()); //Creazione dell'oggetto remoto con il nome letto in input

            Registry registry = LocateRegistry.getRegistry(); 
            TtyChat server = (TtyChat) registry.lookup("TtyChat"); //riferimento all'oggetto remoto server

            server.enterRoom(me); //invocazione del metodo remoto lato server per entrare nella stanza
            System.out.println("You can now chat in the room");

            while (true) {
                String s = input.readLine(); //lettura dei messaggi da tastiera
                if (s.equals("<quit>")) {
                    break;
                }
                server.saySomething(s, me); //invoca il metodo remoto lato server per inviare il messaggio a tutti nella stanza
            }

            server.exitRoom(me); //invoca il metodo remoto lato server per uscire dalla stanza
            UnicastRemoteObject.unexportObject(me, false); //viene rimosso l'oggetto remoto
        }
    }
}
```