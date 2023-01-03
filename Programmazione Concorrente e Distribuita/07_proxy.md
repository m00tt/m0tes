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

