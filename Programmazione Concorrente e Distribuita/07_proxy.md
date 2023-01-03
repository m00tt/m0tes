# Proxy
Al fine di separare la parte di comunicazione server-client dalla parte "operativa" si vanno ad implementare dei proxy lato client e lato server (il proxy lato server è spesso chiamato <b>skeleton</b>).

In questo modo, così come il client interagisce con un “server locale” (il proxy), senza doversi occupare della comunicazione, anche il server riceve le richieste da un “client locale” (lo skeleton).

![Proxy](/assets/programmazione_concorrente_e_distribuita/proxy.png)


## Implementazione proxy lato client
Si presenta come un'implementazione del servizio remoto, locale al client ("localizza" le operazioni server) fornendo un'interfaccia uguale a quella del server reale.
Tutti i meccanismi per la comunicazione verso il client sono gestite direttamente dal proxy, che si occuperà di prendere le richieste dal client e inoltrarle verso il server (e viceversa).

## Implementazione skeleton
Lo skeleton ha lo stesso scopo del proxy lato client, ovvero isolare la parte di comunicazione da quella operativa.<br>
Ha il compito di:
- ricevere le richieste di servizio
- gestire la comunicazione verso il client (ovvero il proxy client)
- fare chiamate al server reale per eseguire i comandi richiesti
- ricevere dal server i risultati e inviarli al client

Lo skeleton, può essere implementato in 2 modi:
- _Delega_: la classe <b>Skeleton</b> presenta al suo interno un riferimento alla classe server.
- _Ereditarietà_: la classe <b>Skeleton</b> eredita dalla classe server, aggiungendo gli schemi di comunicazione


# Contatore con proxy e skeleton (senza risorsa condivisa)
Implementazione di un contatore che gestisca client multipli.<br>
Ogni client ha un contatore dedicato, quindi non si ha una risorsa condivisa che rischia race conditions.<br>
Viene vista l'implementazione dello Skeleton tramite delega.


```java
//Interfaccia per definire le funzioni permesse lato server
import java.io.IOException;

public interface ServerInterface extends AutoCloseable {
	public static final int PORT = 9090;
	
	String reset() throws IOException;
	String increment() throws IOException;
	String sum(int s) throws IOException;
	
}


//CounterServerSlave: classe che implementa l'interfaccia, crea il contatore e gestisce le operazioni lato server
import java.io.IOException;
import java.net.Socket;

public class CounterServerSlave implements ServerInterface {

	private int counter = 0;
	private final Socket socket;
	
	public CounterServerSlave(Socket socket) {
		this.socket = socket;
	}

	@Override
	public String reset() throws IOException {
		counter = 0;
		return String.valueOf(counter);
	}

	@Override
	public String increment() throws IOException {
		counter++;
		return String.valueOf(counter);
	}

	@Override
	public String sum(int s) throws IOException {
		counter += s;
		return String.valueOf(counter);
	}
	
	public String counterState() throws IOException {
		return String.valueOf(counter);
	}

	@Override
	public void close() throws Exception {
		socket.close();
	}
}


//ServerSkeleton: crea un CounterServerSlave tramite delega, crea gli oggetti per la comunicazione con server e il client
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;

public class ServerSkeleton extends Thread {
	
	private Socket socket;
	private final CounterServerSlave server;
	
	public ServerSkeleton(Socket socket) {
		this.socket = socket;
		server = new CounterServerSlave(socket);
		start();
	}
	
	public void run() {
		try(
			BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			PrintWriter out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()), true);
			){
			
				serveClient(in, out);
			
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				socket.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	
	private void serveClient(BufferedReader in, PrintWriter out) throws IOException {
		String operation;
		while(true) {
			
			operation = in.readLine();
			if(operation.equalsIgnoreCase("exit")) break;
						
			if(operation.equalsIgnoreCase("<reset>")) {
				out.println("Counter resetted to: " + server.reset());
			} else if (operation.equalsIgnoreCase("<increment>")) {
				out.println("Counter incremented: " + server.increment());
			} else if (operation.startsWith("<sum>")) {
				String[] split = operation.split(" ");
				int num = Integer.parseInt(split[1]);
				out.println("Sum: " + server.counterState() + " + " + num + " = " + server.sum(num));
			} else {
				out.println("Invalid operation.");
			}
		}
		
		System.out.println("Close connection to client.");
	}
}


//Server: si occupa solamente di accettare socket dai client e crea un nuovo Skeleton
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
	
	public static void main(String[] args) throws IOException {
		
		try(ServerSocket serverSocket = new ServerSocket(ServerInterface.PORT)){
			System.out.println("Server up and running...");
			while(true) {
				Socket clientConnection = serverSocket.accept();
				System.out.println("New client connected.");
				new ServerSkeleton(clientConnection);			
			}
		}
	}
}


//ProxyClient: implementa l'interfaccia ServerInterface e mette localmente a disposizione del client le operazioni offerte dal server.
//Crea una socket con il server per gestire la comunicazione proxy-skeleton
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.Socket;

public class ProxyClient implements ServerInterface {
	
	private Socket socket;
	private BufferedReader in;
	private PrintWriter out;
	
	public ProxyClient() throws IOException {
		try {
			
			InetAddress serverAddress = InetAddress.getByName(null);
			socket = new Socket(serverAddress, ServerInterface.PORT);
			in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()), true);
			
		} catch(Exception e) {
			if(in!=null) in.close();
			if(out!=null) out.close();
			socket.close();
			throw e;
		}
	}

	@Override
	public String reset() throws IOException {
		out.println("<reset>");
		return in.readLine();
	}

	@Override
	public String increment() throws IOException {
		out.println("<increment>");
		return in.readLine();
	}

	@Override
	public String sum(int s) throws IOException {
		out.println("<sum> " + s);
		return in.readLine();
	}
	
	public void close() throws IOException
	{
		out.println("exit");
		in.close();
		out.close();
		socket.close();
	}
}


//ClientThread: thread che creano un oggetto ProxyClient ed inviano le richieste al proxy
public class ClientThread extends Thread {
	
	private int id;
	
	public ClientThread(int id) {
		this.id = id;
		start();
	}
	
	public void run() {
		try(ServerInterface localServer = new ProxyClient()){
			log(localServer.reset());
			long startTime = System.currentTimeMillis();
			for(int i=0; i<10; i++) {
				log(localServer.increment());
			}
			log(localServer.sum(100));
			localServer.close();
			long elapsedTime = System.currentTimeMillis() - startTime;
			log("Elapsed time = " + String.valueOf(elapsedTime) + "ms");
			
		} catch(Exception e) {
			
		}
	}
	
	private void log(String m) {
		System.out.println("Client " + id + ": " + m);
	}
}


//Client: si occupa di creare i ClientThread
public class Client {
	
	private static final int NUM_CLIENT = 3;
	
	public static void main(String[] args) throws InterruptedException {
		for (int i=0; i<NUM_CLIENT; i++) {
			new ClientThread(i);
			Thread.sleep(1000);
		}
	}
}
```

# Contatore con proxy e skeleton (con risorsa condivisa)
Implementiamo lo stesso esempio fatto in precedenza ma introduciamo una risorsa condivisa che quindi richiede opportuni accorgimenti in modo che non si verifichino race conditions.

```java
//ServerInterface: interfaccia che dichiara le operazioni effettuabili dal server
import java.io.IOException;

public interface ServerInterface {
	
	public static final int SERVER_PORT = 9090;
	
	String reset() throws IOException;
	String increment() throws IOException;
	String sum(int s) throws IOException;
}


//Counter: classe lato server che implementa l'interfaccia e ne definisce i metodi
import java.io.IOException;

public class Counter implements ServerInterface {

	private int counter = 0;

	@Override
	public synchronized String reset() throws IOException {
		counter = 0;
		return String.valueOf(counter);
	}

	@Override
	public synchronized String increment() throws IOException {
		counter++;
		return String.valueOf(counter);
	}

	@Override
	public synchronized String sum(int s) throws IOException {
		counter += s;
		return String.valueOf(counter);
	}
	
	public String getCounter() {
		return String.valueOf(counter);
	}
}


//Skeleton: proxy lato server che gestisce le comunicazioni verso il ClientProxy
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;

public class Skeleton extends Thread {
	
	private final Socket socket;
	private final Counter counter;
	
	public Skeleton(Socket socket, Counter counter) {
		this.socket = socket;
		this.counter = counter;
		start();
	}
	
	public void run() {
		try {
			
			try(
					BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
					PrintWriter out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()), true);
					
				){
				
				serveClient(in, out);
				
			} finally {
				socket.close();
			}
			
		} catch(Exception e) {}
	}
	
	private void serveClient(BufferedReader in, PrintWriter out) throws IOException {
		String operation;
		while(true) {
			operation = in.readLine();
			if(operation.equalsIgnoreCase("exit")) break;
			if(operation.equalsIgnoreCase("<reset>")) {
				out.println("Counter resetted to " + counter.reset());
			} else if (operation.equalsIgnoreCase("<increment>")) {
				out.println("Counter incremented: " + counter.increment());
			} else if (operation.startsWith("<sum>")) {
				String[] split = operation.split(" ");
				int n = Integer.parseInt(split[1]);
				out.println("Sum: " + counter.getCounter() + " + " + n + " = " + counter.sum(n));
			} else {
				out.println("Invalid input");
			}
		}
		System.out.println("Client disconnected");
	}
}


//Server: si occupa di accettare le richieste client e creare uno skeleton dedicato ad ogni richiesta
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {

	public static void main(String[] args) throws IOException {
		
		Counter counter = new Counter();
		
		try(ServerSocket serverSocket = new ServerSocket(ServerInterface.SERVER_PORT)){
			System.out.println("Server up and running...");
			while(true) {
				Socket socket = serverSocket.accept();
				System.out.println("New client connected");
				new Skeleton(socket, counter);
			}
			
		} catch(Exception e) {
			throw e;
		}
	}
}


//ProxyClient: implementa l'interfaccia ServerInterface e mette a disposizione localmente le operazioni effettuabili al client
//Instaura una connessione con il server in modo da inoltrare le richieste del client e ricevere risposte
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.Socket;

public class ProxyClient implements ServerInterface, AutoCloseable {

	private Socket socket;
	private BufferedReader in;
	private PrintWriter out;
	
	public ProxyClient() throws IOException {
		try {
			InetAddress serverAddress = InetAddress.getByName(null);
			socket = new Socket(serverAddress, ServerInterface.SERVER_PORT);
			in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()), true);
		} catch(Exception e) {
			if(in!=null) in.close();
			if(out!=null) out.close();
			socket.close();
			throw e;
		}
	}
	
	@Override
	public String reset() throws IOException {
		out.println("<reset>");
		return in.readLine();
	}

	@Override
	public String increment() throws IOException {
		out.println("<increment>");
		return in.readLine();
	}

	@Override
	public String sum(int s) throws IOException {
		out.println("<sum> " + s);
		return in.readLine();
	}

	@Override
	public void close() throws Exception {
		out.println("exit");
		in.close();
		out.close();
		socket.close();
	}
}


//ClientThread: Thread lato client per l'esecuzione di operazioni lato server
public class ClientCounter extends Thread {
	
	private int id;

	public ClientCounter(int id) {
		this.id = id;
		start();
	}
	
	public void run() {
		try(ProxyClient server = new ProxyClient()){
			log(server.reset());
			Thread.sleep(3000);
			
			for(int i=0; i<20; i++) {
				log(server.increment());
			}
			
			log(server.sum(17));
			server.close();
			
		} catch (Exception e) {}
	}
	
	private void log(String message) {
		System.out.println("Client " + id + ": " + message);
	}
}


//Client: si occupa della creazione dei ClientThread
public class Client {
	
	private static final int NUM_CLIENT = 4;

	public static void main(String[] args) throws InterruptedException {
		
		System.out.println("Client started");
		
		ClientCounter[] threads = new ClientCounter[NUM_CLIENT];
		for(int i=0; i<NUM_CLIENT; i++) {
			threads[i] = new ClientCounter(i);
		}
		
		for(ClientCounter client : threads) {
			client.join();
		}
	}
}
```