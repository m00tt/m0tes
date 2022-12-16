# Thread Advanced Practice

## Esercizio 1: producer-consumer with monitor

```java
//Definizione variabile condivisa tra i thread
public class Shared {

	private static final int BUFFER_SIZE = 1;
	private int numItems = 0;
	private int value;
	
	
	public synchronized int getItem() throws InterruptedException {
		while(numItems == 0) { wait(); }
		numItems--;
		System.out.println("Consumed: " + value);
		notifyAll();
		return value;
	}
	
	public synchronized void setItem(int v) throws InterruptedException {
		while(numItems == BUFFER_SIZE) { wait(); }
		value = v;
		System.out.println("Produced: " + value);
		numItems++;
		notifyAll();
	}
}

//Producer
public class Producer extends Thread {
	
	Shared shared;
	
	public Producer(Shared s) {
		shared = s;
		start();	
	}
	
	public void run() {
		for(int i=0; i<100; i++) {
			try {
				shared.setItem(i);
			} catch (InterruptedException e) {
				break;
			}
		}
	}
}

//Consumer
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
		new Producer(shared);
		new Consumer(shared);
		new Consumer(shared);
	}
}
```

## Esercizio 2: producer-consumer with semaphore

```java
//Definizione variabile condivisa tra i thread
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

//Producer
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

//Consumer
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

//Main
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
```

## Esercizio 3: producer-consumer queue with semaphore
Creazione di una coda FIFO condivisa tra i thread, che sia thread safe.<br>
La coda viene creata manualmente dal programmatore in modo che sia ciclica.

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

## Esercizio 4: producer-consumer queue with monitor
Creazione di una coda FIFO condivisa tra i thread, che sia thread safe.<br>
La coda viene creata manualmente dal programmatore in modo che sia ciclica.

```java
//Creazione classe cella condivisa tra thread
public class Shared {
	private int BUFFER_SIZE;
	private int num_items = 0;
	private int[] values;
	private int first = 0;
	private int last = 0;
	
	public Shared (int size) {
		BUFFER_SIZE = size;
		values = new int[BUFFER_SIZE];
	}

	public synchronized int getItem() throws InterruptedException {
		if(num_items == 0) { wait(); }
		num_items--;
		int tmp = values[first];
		first = (first + 1) % BUFFER_SIZE;
		System.out.println("GET: " + tmp);
		notify();
		return tmp;
	}

	public synchronized void setItem(int v) throws InterruptedException {
		if(num_items == BUFFER_SIZE) { wait(); }
		num_items++;
		values[last] = v;
		last = (last + 1) % BUFFER_SIZE;
		System.out.println("SET: " + v);
		notify();
	}
}

//Producer
public class Producer extends Thread {

	Shared shared;
	
	public Producer (Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			try {
				shared.setItem(i);
			} catch(Exception e) {
				break;
			}
		}
	}
}

//Consumer
public class Consumer extends Thread {
	
	Shared shared;
	
	public Consumer(Shared s) {
		shared = s;
		start();
	}
	
	public void run() {
		for(int i = 0; i<10; i++) {
			try {
				shared.getItem();
			} catch(Exception e) {
				break;
			}
		}
	}
}

//Main
public class Main {

	public static void main(String[] args) {
		
		Shared shared = new Shared(5);
		new Producer(shared);
		new Consumer(shared);
	}
}
```