# Problema del produttore-consumatore con `synchronized`

Il problema del produttore-consumatore, già introdotto nella lezione precedente, è caratterizzato da due task che comunicano attraverso un buffer con capacità limitata:
- il produttore genera dati e li deposita nel buffer;
- il consumatore utilizza i dati prodotti, rimuovendoli di volta in volta dal buffer;
- i due task producono e consumano continuamente, ma con una frequenza variabile
e non nota a priori.

Bisogna quindi assicurare che il produttore non crei dati nel caso in cui il buffer sia pieno, ed il consumatore non prelevi dati quando il buffer risulta vuoto.

```java
// Classe per risorsa condivisa
public class Shared {

	private static final int BUFFER_SIZE = 1;
	private int numItems = 0;
	private int value;
	
	
	public synchronized int getItem() throws InterruptedException {
		if(numItems == 0) { wait(); } //Se il buffer è vuoto attendo
		numItems--;
		System.out.println("Consumed: " + value);
		notify();
		return value;
	}
	
	public synchronized void setItem(int v) throws InterruptedException {
		if(numItems == BUFFER_SIZE) { wait(); } //Se il buffer è pieno attendo
		value = v;
		System.out.println("Produced: " + value);
		numItems++;
		notify();
	}
	
}

//Classe thread producer
public class Producer extends Thread {
	
	Shared shared;
	
	public Producer(Shared s) {
		shared = s;
		start();	
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				shared.setItem(i);
			} catch (InterruptedException e) {
				break;
			}
		}
	}
}

//Classe thread consumer
public class Consumer extends Thread {
	
	Shared shared;
	
	public Consumer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				shared.getItem();
			} catch (InterruptedException e) {
				break;
			}
		}
	}
}

//Main
public class Main {

	public static void main(String[] args) {
		Shared shared = new Shared();
		new Producer(shared);
		new Consumer(shared);
	}
}
```

## Posizione delle chiamate `wait()` e `notify()`
- chiamare `wait()` prima di aver effettuato qualsiasi operazione sull’oggetto (all’inizio di una sezione critica), perché, mentre si è in attesa, viene rilasciato il lock, e quindi altri thread possono accedere all’oggetto;
- chiamare `notify()` dopo aver completato le operazioni sull’oggetto

Un uso improprio di queste chiamate porterebbe ad un [deadlock](/Programmazione%20Concorrente%20e%20Distribuita/02_problemi_programmazione_concorrente.md#deadlock).


## Buffer di dimensioni maggiori a 1
Andando ad inrementare il `BUFFER_SIZE` presente nell'esempio, si potranno effettuare più inserimenti e letture in base alla dimensione del buffer.

Impostando quindi
```java
private static final int BUFFER_SIZE = 10;
```
potranno essere effettuati fino a 10 `setItem()` consecutivi e fino a 10 `getItem()` consecutivi.

## Gestione di più thread produttori e consumatori
Per adattare l'esempio precedente alla presenza di più produttori e più consumatori si effettuano alcune modifiche alla classe `Shared` e `Main`.

```java
public class Shared {

	private static final int BUFFER_SIZE = 10;
	private int numItems = 0;
	private int value;
	
	
	public synchronized int getItem() throws InterruptedException {
		while(numItems == 0) { wait(); } //prima if, ora while
		numItems--;
		System.out.println("Consumed: " + value);
		notifyAll(); //prima notify(), ora notifyAll()
		return value;
	}
	
	public synchronized void setItem(int v) throws InterruptedException {
		while(numItems == BUFFER_SIZE) { wait(); }  //prima if, ora while
		value = v;
		System.out.println("Produced: " + value);
		numItems++;
		notifyAll();  //prima notify(), ora notifyAll()
	}
}

public class Main {

	public static void main(String[] args) {
		
		Shared shared = new Shared();
		new Producer(shared);  //Creati più thread
		new Producer(shared);
		new Consumer(shared);
		new Consumer(shared);
	}
}
```

Dato che ora entrano in gioco più thread, è stato necessario:
1. sostituire gli if usati per l’attesa con dei while: così, se la condizione
di risveglio viene “annullata” da un altro thread mentre il thread che è stato svegliato aspetta di essere schedulato, quest’ultimo ricontrollerà la condizione e tornerà subito in attesa.
2. le chiamate `notify()` vengono sostituite con `notifyAll()`, per garantire di svegliarli sempre tutti i thread ogni volta che cambia lo stato del buffer.

# Problema del produttore-consumatore con `semaphore`

In questo caso vengono creati degli appositi Semafori per la gestione della race condition.
1. `mutex` per bloccare l'accesso alla risorsa condivisa
2. `full` per mettere in wait il produttore quando il buffer è pieno
3. `empty` per mettere in wait il consumatore quando il buffer è vuoto

```java
public class Main {

	public static Semaphore mutex = new Semaphore(1);
	public static Semaphore full = new Semaphore(0);
	public static Semaphore empty = new Semaphore(1);
	
	public static void main(String[] args) {
		
		Shared shared = new Shared();
		new Producer(shared);
		new Consumer(shared);
		
	}
}

public class Shared {
	
	private int value;

	public int getItems() throws InterruptedException {
		Main.mutex.acquire();
		int tmp = value;
		Main.mutex.release();
		
		return tmp;
	}

	public void setItems(int v) throws InterruptedException {
		Main.mutex.acquire();
		value = v;
		Main.mutex.release();
	}
}

public class Producer extends Thread {
	
	Shared shared;
	
	public Producer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		
		for(int i=0; i<10; i++) {
			try {
				Main.empty.acquire();
				shared.setItems(i);
				System.out.println("Set: " + i);
			} catch (InterruptedException e) {
				break;
			}
			Main.full.release();
		}
	}
}

public class Consumer extends Thread {
	
	Shared shared;
	
	public Consumer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				Main.full.acquire();
				int v = shared.getItems();
				System.out.println("Get: " + v);
			} catch (InterruptedException e) {
				break;
			}
			Main.empty.release();
		}
	}
}
```

# Buffer FIFO
Solitamente, il buffer usato dai produttori e consumatori ha dimensioni maggiori di uno, e gli elementi vengono consumati nello stesso ordine in cui sono prodotti. Per memorizzare gli elementi, serve allora una coda (cioè un buffer FIFO, First In First Out).

```java
public class Main {

	public static final int BUFFER_SIZE = 5;
	public static Semaphore mutex = new Semaphore(1);
	public static Semaphore empty = new Semaphore(BUFFER_SIZE);
	public static Semaphore full = new Semaphore(0);
	
	public static void main(String[] args) {
		
		Shared s = new Shared(BUFFER_SIZE);
		new Producer(s);
		new Producer(s);
		new Consumer(s);
		new Consumer(s);	
	}	
}

public class Shared {
	
	private final int BUFFER_SIZE;
	private int numItems = 0;
	private int[] values;
	private int first = 0;
	private int last = 0;
	
	public Shared(int buffer) {
		BUFFER_SIZE = buffer;
		values = new int[BUFFER_SIZE];
	}

	public int getItems() throws InterruptedException {
		Main.mutex.acquire();
		if (numItems == 0) {
			System.err.println("Buffer vuoto");
			System.exit(0);
		}
		numItems--;
		int tmp = values[first];
		first = (first+1) % BUFFER_SIZE;
		System.out.println("Get: " + tmp);
		Main.mutex.release();
		return tmp;
	}

	public void setItems(int v) throws InterruptedException {
		Main.mutex.acquire();
		if(numItems == BUFFER_SIZE) {
			System.err.println("Buffer pieno");
			System.exit(1);
		}
		numItems++;
		values[last] = v;
		last = (last+1) % BUFFER_SIZE;
		System.out.println("Set: " + v);
		Main.mutex.release();
	}
}

public class Producer extends Thread {
	
	Shared shared;
	
	public Producer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				Main.empty.acquire();
				shared.setItems(i);
			} catch (InterruptedException e) {
				break;
			}
			Main.full.release();
		}
	}
}

public class Consumer extends Thread {

	Shared shared;
	
	public Consumer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				Main.full.acquire();
				shared.getItems();
			} catch (InterruptedException e) {
				break;
			}
			Main.empty.release();
		}
	}
}
```

## BlockingQueue
Esiste un'apposita struttura dati chiamata corrispondente all'interfaccia `java.util.concurrent.BlockingQueue`. In particolare, si tratta di una struttura dati FIFO (appunto, una coda) thread-safe, che è in grado di mettere in attesa (bloccare) un produttore che cerca di inserire quando la coda è piena, o un consumatore che cerca di prelevare un valore quando questa è vuota, e può anche essere usata con produttori/consumatori multipli.

BlockingQueue è un’interfaccia, per usarla bisogna scegliere una delle sue
implementazioni. Alcune delle principali sono:
1. `ArrayBlockingQueue`: memorizza gli elementi in un array, e ha una capacità
limitata;
2. `LinkedBlockingQueue`: memorizza gli elementi in una linked list, con capacità
illimitata o, opzionalmente, limitata.

### Metodi
Esistono varie tipologie di metodi divisi per la loro funzione e per il valore di ritorno, qui sotto è presente uno schema:

| Funzione  | Eccezione | Valore speciale           | Blocca | Timeout              |
| --------- | --------- | ------------------------- | ------ | -------------------- |
| Inserisci | add(e)    | offer() - `false`         | put()  | offer(e, time, unit) |
| Rimuovi   | remove()  | poll() - `null`           | take() | poll(time, unit)     |
| Esamina   | element() | peek() - `null`           |        |                      |

Nel caso del problema del produttore e consumatore, fanno al caso i metodi

```java
public void put(E e) throws InterruptedException;
public E take() throws InterruptedException;
```

### Applicazione nel problema dei produttori-consumatori
Si vuole simulare lo scambio di messaggi tra produttore e consumatore.

```java
public class Message {
	private final String msg;
	
	public Message(String msg) {
		this.msg = msg;
	}

	public String getMsg() {
		return msg;
	}
}

public class Main {

	public static void main(String[] args) {
		BlockingQueue<Message> queue = new ArrayBlockingQueue<Message>(10);
		new Producer(queue);
		new Consumer(queue);
	}
}

public class Producer extends Thread {
	BlockingQueue<Message> queue;
	
	public Producer(BlockingQueue<Message> q) {
		queue =  q;
		start();
	}
	
	public void run() {
		for(int i=0; i<100; i++) {
			try {
				Message msg = new Message(String.valueOf(i));
				queue.put(msg);
				System.out.println("Set: " + i);
			} catch (Exception e) {
				break;
			}
		}
		
		try {
			Message msg = new Message("stop");
			queue.put(msg);
		} catch (InterruptedException e) { }
		System.out.println("Producer stops");
	}
}

public class Consumer extends Thread {
	
	BlockingQueue<Message> queue;
	
	public Consumer(BlockingQueue<Message> q) {
		queue = q;
		start();
	}
	
	public void run() {
		for(int i=0; i<100; i++) {
			try {
				Message msg = queue.take();
				if (msg.getMsg().equalsIgnoreCase("stop")) {
					break;
				}
				System.out.println("Get: " + msg.getMsg());
			} catch (Exception e) {
				break;
			}
		}
		System.out.println("Consumer stops");
	}
}
```
