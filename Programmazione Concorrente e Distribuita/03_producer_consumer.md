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

