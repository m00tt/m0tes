# Socket
I socket sono un'interfaccia per i protocolli TCP e UDP.
Grazie a essi, si ottiene l’interoperabilità dei programmi di rete:
- Due applicazioni che usano i socket (con lo stesso protocollo di trasporto) possono interagire anche se vengono eseguite su macchine con sistemi operativi diversi, e/o se sono scritte in linguaggi di programmazione diversi.

## Identificazione di una macchina
Per poter comunicare con una macchina, bisogna avere un modo di identificarla univocamente tramite un indirizzo IP.
Un indirizzo IP può essere associato ad un nome di dominio (per facilitarne la memorizzazione) e questa corrispondenza avviene tramite [DNS (Domain Name System)](https://github.com/m00tt/m0tes/blob/security/web_security/server_side/web_fundamentals.md#dns).

In Java, un indirizzo IP può essere ottenuto da un nome di dominio attraverso il metodo statico `InetAddress.getByName`, che restituisce un oggetto di tipo `InetAddress`.

```java
import java.net.InetAddress;
import java.net.UnknownHostException;

public class WhoAmI {
    public static void main(String[] args) throws UnknownHostException {
        if (args.length != 1) {
            System.err.println("Usage: WhoAmI DomainName");
            System.exit(1);
        }
        InetAddress addr = InetAddress.getByName(args[0]);
        System.out.println(addr);
    }
}
```

## Client e server in Java
Per stabilire una connessione client-server in Java è necessario che:
1. Il server deve rimanere in ascolto di richieste di connessione, usando un oggetto speciale `ServerSocket`.
2. Il client cerca di stabilire una connessione con un server, attraverso un oggetto di tipo `Socket`.
3. Una volta effettuata la connessione, vi si costruiscono sopra degli stream di I/O, che permettono di trattarla come se si stesse leggendo da e scrivendo su un file.

## Localhost
Per effettuare dei testi in locale, possiamo utilizzare l'IP di loopback, 127.0.0.1, che è associato al nome simbolico _localhost_, e riferisce, appunto, alla propria macchina.

In Java, ci sono vari modi equivalenti per ottenere un’istanza di `InetAddress` relativa a localhost, tra cui:

```java
InetAddress localhost = InetAddress.getByName(null);
InetAddress localhost = InetAddress.getByName("localhost");
InetAddress localhost = InetAddress.getByName("127.0.0.1");
```

## Identificazione di un'applicazione sulla macchina
Come detto in precedenza, in un server possono essere ospitate molteplici funzioni/applicazioni, quindi è necessario andare a specificare anche il numero di porta sulla quale è in ascolto l’applicazione con la quale si vuole comunicare.

<b>! </b>: Le porte da 0 a 1023 sono solitamente riservate dal sistema, quindi è opportuno usare numeri di porta superiori.

# Uso dei socket in Java
In Java i socket vengono usati mediante due principali classi:
- `ServerSocket`, che un server utilizza per ascoltare le connessioni in ingresso;
- `Socket`, utilizzata da un client al fine di avviare una connessione.

`ServerSocket` fornisce un metodo `accept()`, che sospende il chiamante in attesa di una richiesta di connessione, e restituisce un oggetto Socket nel momento in cui un client si connette. Da questo momento, si ha una connessione tra il socket del client e quello del server.<br>
Si possono allora usare i metodi `getInputStream()` e `getOutputStream()` di ciascun oggetto `Socket` per ottenere degli stream di I/O mediante i quali leggere/scrivere dati, senza più doversi preoccupare dei socket sottostanti.

Quando si crea un `ServerSocket`, è necessario specificare solo il numero di porta sul quale si accetteranno le connessioni (perché l’indirizzo IP è implicitamente quello della macchina su cui si esegue il server). Invece, nella creazione di un Socket (sul client), bisogna indicare sia l’indirizzo IP che il numero di porta del server al quale ci si vuole connettere.

## Stream I/O
Quello che viene trasmesso mediante i socket è una sequenza di byte. <br>
Perciò, i metodi `getInputStream()` e `getOutputStream()` restituiscono degli oggetti di tipo InputStream e OutputStream (che rappresentano, appunto, degli stream di byte).

Se si desidera trasmettere invece dati testuali, è possibile usare `InputStreamReader` e `OutputStreamWriter` per effettuare le conversioni tra byte e caratteri (ed eventualmente `BufferedReader`, `BufferedWriter`, `PrintWriter`, ecc. per una gestione più avanzata della lettura / scrittura).


## Echo example
Come primo semplice esempio di applicazione distribuita, si presenta un server che restituisce al client il testo ricevuto dal client stesso. Questo servizio è solitamente chiamato “echo” (per analogia con il fenomeno acustico dell’eco, che “ripete” ciò che viene detto).

```java
//Server
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class EchoServer {
    private static final int PORT = 8080;

    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(PORT);
        try {
            System.out.println("Started " + server);
            Socket socket = server.accept();
            try {
                System.out.println("Connection accepted: " + socket);
                serveClient(socket);
            } finally {
                System.out.println("Closing...");
                socket.close();
            }
        } finally {
            server.close();
        }
    }

    private static void serveClient(Socket socket) throws IOException {
        try (
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter( new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())), true);
            //true = autoflush
        ) {
            while (true) {
                String str = in.readLine();
                if (str.equals("END")) break;
                System.out.println("Echoing: " + str);
                out.println(str);
            }
        }
    }
}

//Client
import java.io.*;
import java.net.InetAddress;
import java.net.Socket;

public class EchoClient {
    private static final int SERVER_PORT = 8080;

    public static void main(String[] args) throws IOException {
        InetAddress serverAddr = InetAddress.getByName(null);
        System.out.println("serverAddr = " + serverAddr);
        Socket socket = new Socket(serverAddr, SERVER_PORT);
        try {
            System.out.println("socket = " + socket);
            try (
                BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())),true)
            ) {
                for (int i = 0; i < 10; i++) {
                    out.println("hello " + i);
                    String str = in.readLine();
                    System.out.println(str);
                }
                out.println("END");
            }
        } finally {
            System.out.println("Closing...");
            socket.close();
        }
    }
}
```

## Date Example
```java

//Server
import java.io.*;
import java.net.*;
import java.util.Date;

public class DateTimeServer {
    private static final int PORT = 1333;

    public static void main(String[] args) throws IOException {
        try (ServerSocket server = new ServerSocket(PORT)) {
            while (true) {
                try (
                    Socket connection = server.accept();
                    Writer out = new OutputStreamWriter(connection.getOutputStream())
                ) {
                    Date now = new Date();
                    out.write(now + "\r\n");
                    out.flush();
                }
            }
        }
    }
}


//Client
import java.io.*;
import java.net.*;

public class DateTimeClient {
    private static final int SERVER_PORT = 1333;
    
    public static void main(String[] args) throws IOException {
        InetAddress serverAddr = InetAddress.getByName(null);
        System.out.println("serverAddr = " + serverAddr);
        try (
            Socket socket = new Socket(serverAddr, SERVER_PORT);
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))
        ) {
            System.out.println("socket = " + socket);
            String str = in.readLine();
            System.out.println(str);
            System.out.println("Closing...");
        }
    }
}
```

## Caratteristiche di TCP
Gli esempi presentati finora utilizzano socket stream-based, basati sul protocollo TCP.<br>
Come già visto, quest’ultimo è progettato per la massima affidabilità, e garantisce che i dati arrivino a destinazione e nell'ordine corretto.

Il principale svantaggio è che, per garantire tale livello di controllo e affidabilità, TCP ha un notevole overhead. Allora, quando l’affidabilità non è necessaria, si possono usare invece socket basati su UDP, che non garantisce la consegna dei pacchetti né l’ordine in cui essi arriveranno, ma ha molto meno overhead.

# Socket UDP
UDP permette alle applicazioni di inviare e ricevere datagrammi, ovvero messaggi il cui arrivo, tempo di arrivo e contenuto non è garantito.

Java mette a disposizione tre classi per i socket UDP:
 - `DatagramPacket` : rappresenta un datagramma
 - `DatagramSocket` : è un socket che consente l'invio e la ricezione di `DatagramPacket`
 - `MulticastSocket` : utilizzato per inviare un messaggio a molteplici destinatari

## 

```java
//Server
import java.io.IOException;
import java.net.*;

public class UDPServer {
    private static final int SERVER_PORT = 9876;
    public static void main(String[] args) throws IOException {
        try (
            DatagramSocket serverSocket = new DatagramSocket(SERVER_PORT)
        ) {
            byte[] receiveData = new byte[1024];
            while (true) {
                DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
                serverSocket.receive(receivePacket);
                String sentence = new String( receivePacket.getData(), 0, receivePacket.getLength() );
                System.out.println("RECEIVED: " + sentence);
                String capitalizedSentence = sentence.toUpperCase();
                InetAddress clientAddr = receivePacket.getAddress();
                int clientPort = receivePacket.getPort();
                byte[] sendData = capitalizedSentence.getBytes();
                DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, clientAddr, clientPort);
                serverSocket.send(sendPacket);
            }
        }
    }
}

//Client
import java.io.*;
import java.net.*;

public class UDPClient {
    private static final int SERVER_PORT = 9876;
    public static void main(String[] args) throws IOException {
        System.out.println("Enter a string: ");
        String sentence;
        try (
            BufferedReader inFromUser = new BufferedReader(new InputStreamReader(System.in))
        ) {
            sentence = inFromUser.readLine();
        }
        try (DatagramSocket clientSocket = new DatagramSocket()) {
            InetAddress serverAddr = InetAddress.getByName(null);
            byte[] sendData = sentence.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, serverAddr, 9876);
            clientSocket.send(sendPacket);
            byte[] receiveData = new byte[1024];
            DatagramPacket receivePacket =
            new DatagramPacket(receiveData, receiveData.length);
            clientSocket.receive(receivePacket);
            String modifiedSentence = new String(receivePacket.getData(), 0, receivePacket.getLength());
            System.out.println("FROM SERVER: " + modifiedSentence);
        }
    }
}
```

## Trasmissione immagini con UDP
Dato che UDP tratta datagrammi e trasporta byte, possiamo scomporre un'immagine in tanti datagrammi da 1024 byte ciascuno.

Questa tipologia di file, però, presuppone una certa affidabilità che UDP non è grado di assicurare.<br>
Infatti, data l'inaffidabilità di UDP potrebbe essere che i pacchetti vengano persi e/o ricevuti non in ordine, non si avrebbe una ricostruzione corretta dell'immagine.

# Server: gestione multi-client
Gli esempi visti finora sono in grado di gestire un client alla volta.<br>
Per la gestione multi-client, si affida il Socket ad un thread creato appositamente per gestire le attività separatamente:
Quindi:
 - Si crea un ServerSocket
 - Si resta in attesa di una nuova connessione di un client tramite `accept()`
 - Quando si connette un client, viene creato un nuovo thread e passato il Socket ritornato da `accept()`
 - Subito dopo aver creato il thread, il server torna in ascolto per la gestione di nuovi client

A livello di terminologia, il thread principale del server (quello che esegue le `accept()`) è spesso chiamato <b>master</b>, mentre i thread creati per gestire i singoli client sono detti <b>slave</b>.

## Esempio: echo multi-client
```java
//Server
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class MultiEchoServer {
    public static final int PORT = 8080;
    public static void main(String[] args) throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Server Started");
            while (true) {
                Socket socket = serverSocket.accept();
                try {
                    new OneEchoServer(socket); //Thread creation
                } catch (Exception e) {
                    // If thread creation fails, close the socket.
                    // Otherwise, normally, the socket will be closed by
                    // the thread.
                    socket.close();
                    throw e;
                }
            }
        }
    }
}

//Thread
import java.io.*;
import java.net.Socket;

public class OneEchoServer extends Thread {
    private final Socket socket;
    
    public OneEchoServer(Socket socket) {
        this.socket = socket;
        start();
    }

    public void run() {
        try (
            BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream())
        );
        PrintWriter out = new PrintWriter(
            new BufferedWriter(
            new OutputStreamWriter(socket.getOutputStream())
            ),
            true // Auto-flush
            )
        ) {
            echo(in, out);
            System.out.println("Closing...");
        } catch (IOException e) {
            System.err.println("IO Exception");
        } finally {
            try {
                socket.close();
            } catch (IOException e) {
                System.err.println("Socket not closed");
            }
        }
    }

    private void echo(BufferedReader in, PrintWriter out) throws IOException {
        while (true) {
            String str = in.readLine();
            if (str.equals("END")) break;
            System.out.println("Echoing: " + str);
            out.println(str);
        }
    }
}

//Client
import java.io.*;
import java.net.InetAddress;
import java.net.Socket;
import java.util.concurrent.ThreadLocalRandom;

public class EchoClientThread extends Thread {
private final InetAddress serverAddr;
private final int serverPort;
private final int id;
private static int nextId = 0;
private static int threadCount = 0;
public static int threadCount() {
return threadCount;
}
public EchoClientThread(InetAddress serverAddr, int serverPort) {
        this.serverAddr = serverAddr;
        this.serverPort = serverPort;
        synchronized (EchoClientThread.class) {
            id = nextId;
            nextId++;
            threadCount++;
        }
        start();
    }

    public void run() {
        System.out.println("Making client " + id);
        try (
            Socket socket = new Socket(serverAddr, serverPort);
                BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                new BufferedWriter(
                new OutputStreamWriter(socket.getOutputStream())
                ),
                true // Auto-flush
            )
        ) {
            for (int i = 0; i < 25; i++) {
                out.println("Client " + id + ": " + i);
                Thread.sleep(ThreadLocalRandom.current().nextInt(200, 800));
                System.out.println(in.readLine());
            }
            out.println("END");
        } catch (IOException e) {
            System.err.println("IO Exception");
        } catch (InterruptedException e) {
            System.err.println("Interrupted");
        } finally {
            synchronized (EchoClientThread.class) {
                threadCount--;
            }
        }
    }
}

//Main
import java.io.IOException;
import java.net.InetAddress;

public class MultiEchoClient {
    private static final int MAX_THREADS = 40;
    public static void main(String[] args)
    throws IOException, InterruptedException {
        InetAddress serverAddr = InetAddress.getByName(null);
        while (true) {
            if (EchoClientThread.threadCount() < MAX_THREADS) {
                new EchoClientThread(serverAddr, MultiEchoServer.PORT);
            }
            Thread.sleep(100);
        }
    }
}
```

# Esempio: Domande e Risposte (Q&A)
Il server pone domande al client, il client risponde.<br>
Il server segnala al client se la risposta è corretta, nel caso in cui sia sbagliata invia anche la risposta corretta.

Il server legge da un file le domande e le risposte (qna.txt).
```txt
Domanda1
Risposta1
Domanda2
Risposta2
Domanda3
Risposta3
```

