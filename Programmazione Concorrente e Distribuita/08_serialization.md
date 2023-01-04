# Serializzazione e Deserializzazione
La scrittura di uno stream di byte di un oggetto prende il nome di <b>serializzazione</b>.

Come facilmente intuibile, la <b>deserializzazione</b> è l'operazione inversa che comporta la lettura da uno stream di byte di un oggetto serializzato e la ricostruzione dell'oggetto originale.

![Serialization](/assets/programmazione_concorrente_e_distribuita/serialization.png)


# Interfacce `ObjectInput` e `ObjectOutput`
L'interfaccia <b>ObjectOutput</b> consente di serializzare tipi primitivi e oggetti.<br>
Oltre alle operazioni di `flush()` e `close()`, mette a disposizione anche:

- Metodi per scrivere byte
    - `void write(int b) throws IOException;`
    - `void write(byte[] b) throws IOException;`
    - `void write(byte[] b, int off, int len) throws IOException;`

- Metodi per scrivere tipi primitivi
    - `void writeInt(int v) throws IOException;`
    - `void writeDouble(double v) throws IOException;`
    - `etc`

- Un metodo per la scrittura degli oggetti generici
    - `void writeObject(Object obj) throws IOException;`


L'interfaccia <b>ObjectInput</b> permette la deserializzazione di un oggetto o un tipo primitivo.<br>

- Metodi per la lettura di byte
    - `int read() throws IOException;`
    - `int read(byte[] b) throws IOException;`
    - `int read(byte[] b, int off, int len) throws IOException;`

- Metodi per la lettura di tipi primitivi
    - `int readInt() throws IOException;`
    - `double readDouble() throws IOException;`
    - `etc`

- Un metodo per la lettura degli oggetti generici
    - `Object readObject() throws ClassNotFoundException, IOException;`

- Il metodo `close()`

- Il metodo `long skip(long n) throws IOException;` : per saltare _n_ byte di input

- Il metodo `int available() throws IOException;` : indica quanti byte possono essere ancora letti senza bloccarsi

<b>! </b>: Siccome il tipo restituito da `readObject` è `Object`, spetta al programmatore convertire l’oggetto deserializzato al tipo giusto.

Le interfacce `ObjectOutput` e `ObjectInput` sono implementate rispettivamente dagli stream di byte `ObjectOutputStream` e `ObjectInputStream`.


# Serializzazione di un oggetto primitivo
La serializzazione di tipi primitivi avviene semplicemente usando gli appositi metodi della classe `ObjectOutputStream`.
```java
import java.io.*;
public class PrimitiveSerialization {
    public static void main(String[] args) throws IOException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os =
            new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            int i = 11;
            os.writeInt(i);
            os.flush();
        }
    }
}
```

In modo analogo, la deserializzazione avviene attraverso la classe `ObjectInputStream`.
```java
import java.io.*;
public class PrimitiveDeserialization {
    public static void main(String[] args) throws IOException {
        String fileName = "tmp.bin";
        try (
            ObjectInput is =
            new ObjectInputStream(new FileInputStream(fileName))
        ) {
            int i = is.readInt();
            System.out.println("Read: " + i);
        }
    }
}
```

# Serializzazione di un oggetto
```java
import java.io.*;
public class ObjectSerialization {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            os.writeObject("Now");
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String s = (String) is.readObject();
            System.out.println("Read: " + s);
        }
    }
}
```

# Serializzazione di più oggetti
```java
import java.io.*;
import java.util.Date;
public class ObjectsSerialization {
    public static void main(String[] args)
    throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            os.writeObject("Now");
            os.writeObject(new Date());
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String s = (String) is.readObject();
            Date d = (Date) is.readObject();
            System.out.println(s + " " + d);
        }
    }
}
```

# Serializzazione di un array di oggetti
```java
import java.io.*;
import java.util.Arrays;
public class StringArray {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            String[] array = new String[]{"rosso", "giallo", "blu"};
            os.writeObject(array);
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String[] array = (String[]) is.readObject();
            System.out.println(Arrays.toString(array));
        }
    }
}
```

# Serializzazione di oggetti definiti dal programmatore
Consideriamo la seguente classe

```java
public class Punto {
    private int x, y;
    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return "Il punto ha coordinate " + x + " e " + y;
    }
}
```

Per far si che sia <b>serializzabile</b> è necessario che:
- La classe implementi l'interfaccia `Serializable`
- È buona norma, definire una costante chiamata `serialVersionUID` che serve da numero di versione della classe. Serve in fase di deserializzazione per verificare che le classi usate da chi ha serializzato l'oggetto e chi lo ha serializzato siano compatibili: se le versioni non corrispondono verrà sollevata un'eccezione `InvalidClassException`. Se omessa, la JVM calcola un numero di versione automaticamente che potrebbe causare errori in alcuni casi.

```java
public class Punto implements Serializable {
    private static final long serialVersionUID = 1;
    
    private int x, y;
    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return "Il punto ha coordinate " + x + " e " + y;
    }
}
```

Una volta preparata la classe, possiamo procedere alle operazioni di serializzazione e deserializzazione.

```java
//Serializzazione
import java.io.*;
public class SerializzaPunto {
    public static void main(String[] args) throws IOException {
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream("dati.dat"))
        ) {
            Punto p = new Punto(2, 3);
            os.writeObject(p);
            os.flush();
        }
        System.out.println("Serializzazione completata.");
    }
}

//Deserializzazione
import java.io.*;
public class DeserializzaPunto {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream("dati.dat"))
        ) {
            Punto p = (Punto) is.readObject();
            System.out.println(p);
        }
    }
}
```

# Regole di serializzazione in Java
- Tutti i tipi primitivi sono serializzabili
- Un oggetto è serializzabile se la sua classe o la sua superclasse implementano l'interfaccia `Serializable`
- Una vlasse può implementare `Serializable` anche se la sua superclasse non è serializzabile, purché tale superclasse abbia un costruttore senza argomenti
- Se si desidera serializzare solo alcuni campi di una classe, è possibile contrassegnare quelli da non salvare con l'attributo `transient`
- I campi `static` di ua classe non vengono serializzati
- Se i campi di un oggetto serializzabile contengono un riferimento a un oggetto non serializzabile, verrà sollevata una `NotSerializableException`


# Stream di oggetti tramite socket
Per inviare tramite socket un oggetto Java da un client ad un server (o viceversa) sarà necessario creare degli stream di `ObjectInputStream` e `ObjectOutputStream` sul socket.

Il programma non varia affatto da quelli visti in precedenza, ma semplicemente saranno presenti `ObjectInputStream` e `ObjectOutputStream` al posto dei classici `BufferedReader` e `PrintWriter`.

## Esempio: invio di una stringa
```java
//Server
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
	
	public static final int SERVER_PORT = 9090;

	public static void main(String[] args) {
		
		try {
			ServerSocket serverSocket = new ServerSocket(SERVER_PORT);
			Socket socket = serverSocket.accept();
			ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
			
			String toSend = new String("Sent");
			out.writeObject(toSend);
			socket.close();
			serverSocket.close();
		} catch(Exception e) { }
	}
}

//Client
import java.io.ObjectInputStream;
import java.net.InetAddress;
import java.net.Socket;
import java.net.UnknownHostException;

public class Client {

	public static void main(String[] args) throws UnknownHostException {
		
		InetAddress serverAddress = InetAddress.getByName(null);
		try(Socket socket = new Socket(serverAddress, Server.SERVER_PORT)){
			
			ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
			String received = (String) in.readObject();
			
			System.out.println("Received: " + received);
			socket.close();
		}catch(Exception e) {}
	}
}
```

## Esempio: invio di un array
```java
//Server
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
	
	public static final int SERVER_PORT = 9090;

	public static void main(String[] args) {
		
		try {
			ServerSocket serverSocket = new ServerSocket(SERVER_PORT);
			Socket socket = serverSocket.accept();
			ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
			
			String[] toSend = new String[]{
				"sent1",
				"sent2",
				"sent3"
			};
			out.writeObject(toSend);
			socket.close();
			serverSocket.close();
		} catch(Exception e) { }
	}
}

//Client
import java.io.ObjectInputStream;
import java.net.InetAddress;
import java.net.Socket;
import java.net.UnknownHostException;

public class Client {

	public static void main(String[] args) throws UnknownHostException {
		
		InetAddress serverAddress = InetAddress.getByName(null);
		try(Socket socket = new Socket(serverAddress, Server.SERVER_PORT)){
			
			ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
			String[] received = (String[]) in.readObject();
			
			for(String item : received) {
				System.out.println(item);
			}
			
			socket.close();
		}catch(Exception e) {}
	}
}
```

# ClassNotFoundException
Quando si trasferisce un oggetto serializzato, dato che il client deve ricostruire l'oggetto, deve anche sapere di che oggetto si tratta.

Se l'oggetto serializzato è primitivo, la JVM sa sempre come interpretare i byte serializzati. Altrimenti, è necessario conoscere il codice della classe di cui l'oggetto è istanza.
Se tale codice non è disponibile, verrà sollevata una `ClassNotFoundException`.

## Esempio
Supponiamo di avere costruito la classe `Punto` e che sia pronta per essere serializzata:

```java
import java.io.Serializable;
public class Punto implements Serializable {
    private static final long serialVersionUID = 1;
    private int x, y;
    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return "Il punto ha coordinate " + x + " e " + y;
    }
}
```

Una volta che la classe viene passata al client, anch'esso deve essere a conoscenza della classe `Punto` altrimenti non risucirà a ricostruire correttamente l'oggetto e sarà sollevata una `ClassNotFoundException` (anche cercando di dichiarare un `Object` generico si otterrebbe comunque l'eccezione).

Soluzione: fare in modo che anche il client conosca la classe `Punto.class`, che deve essere la stessa usata dal server.

# Serializzaione personalizzata
Ci sono alcuni casi limite dove la serializzazione di default non è sufficiente. Risulta, quindi, necessario definire manualmente alcune operazioni di serializzazione e/o deserializzazione.

## Il problema
La classe che si vuole serializzare è `PersistentClock`: quando viene istanziata crea un thread che stampa la data e l'ora correnti a intervalli regolari:

```java
import java.util.Date;
public class PersistentClock implements Runnable {
    private Thread animator;
    private long animationInterval;

    public PersistentClock(int animationInterval) {
        this.animationInterval = animationInterval;
        animator = new Thread(this);
        animator.start();
    }
    public void run() {
        while (true) {
            System.out.println(new Date());
            try {
                Thread.sleep(animationInterval);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        new PersistentClock(1000);
    }
}
```

Ora si rende la classe serializzabile:
- Si implementa l'interfaccia `Serializable`
- Il campo `animator` va dichiarato `transient` in quanto contiene un riferimento a `Thread` che non è serializzabile
- L'oggetto viene serializzato su un file (ma potrebbe essere inviato tramite socket)


<b>! <b>: dichiarando una variabile come `transient` non si esclude la serializzazione completa di quella variabile ma verrà serializzata ad un valore di default (null, 0, etc)


```java
import java.io.*;
import java.util.Date;
public class PersistentClock implements Serializable, Runnable {
    private static final long serialVersionUID = 1;
    private transient Thread animator;
    private long animationInterval;

    public PersistentClock(int animationInterval) {
        this.animationInterval = animationInterval;
        animator = new Thread(this);
        animator.start();
    }

    public void run() {
        while (true) {
            System.out.println(new Date());
            try {
                Thread.sleep(animationInterval);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws IOException {
        PersistentClock p = new PersistentClock(1000);
        try (ObjectOutput os = new ObjectOutputStream(new FileOutputStream("tmp.ser"))
        ) {
            os.writeObject(p);
            os.flush();
        }
    }
}
```

Quando si andrà a deserializzare l'oggetto, esso verrà ricostruito senza il thread <b>animator</b>: si ha una ricostruzione parziale.

## Soluzione: serializzazione manuale
Java consente di effettuare un override dei metodi di serializzazione e deserializzazione:

- `private void writeObject(ObjectOutputStream out) throws IOException;`
- `private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException;`

che verranno richiamati al posto dei metodi di default.

All’interno di questi metodi, si può richiamare il comportamento di default, invocando rispettivamente `out.defaultWriteObject()` e `in.defaultReadObject()`. Ciò è utile, ad esempio, se si vogliono solo aggiungere delle operazioni in più, senza invece cambiare come vengono scritti i campi (non static e non transient) dell’oggetto.

### Codice
Vogliamo che una volta ricostruito l'oggetto riparta subito la sua attività di scrittura della data e ora ad intervalli.

```java
import java.io.*;
import java.util.Date;
public class PersistentClock implements Serializable, Runnable {
    private static final long serialVersionUID = 1;
    private transient Thread animator;
    private long animationInterval;

    public PersistentClock(int animationInterval) {
        this.animationInterval = animationInterval;
        startAnimation();
    }

    private void startAnimation() {
        animator = new Thread(this);
        animator.start();
    }

    public void run() {
        while (true) {
            System.out.println(new Date());
            try {
                Thread.sleep(animationInterval);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    //Si definisce manualmente la riscostruzione di default di tutti i campi e si richiama la creazione del thread in modo che venga subito avviato
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        startAnimation();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream("tmp.ser"))
        ) {
            PersistentClock p = (PersistentClock) is.readObject();
        }
    }
}
```