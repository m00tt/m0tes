# Thread and Multithreading Practice

## Esercizio 1
Scrivere un programma che crea un thread come estensione della classe thread e lo lancia.
- Il thread scrive tre volte sullo standard output «thread».
- Il main scrive tre volte sullo standard output «main».

```java
public class Es1 extends Thread{

	public static void main(String[] args) {
		Thread t = new Es1();
		t.start();
		
		for(int i=0; i<3; i++) {
			System.out.println("main");
		}
	}
	
	public void run() {
		for(int i=0; i<3; i++) {
			System.out.println("thread");
		}
	}

}
```

## Esercizio 2
Scrivere un programma che crea un thread passando al costruttore di thread una apposita istanza di runnable.
- Il thread scrive tre volte sullo standard output «thread».
- Il main scrive tre volte sullo standard output «main».

```java
public class Es2 implements Runnable {

	public static void main(String[] args) {
		
		Es2 cl = new Es2();
		Thread t = new Thread(cl);
		t.start();
		
		for(int i=0; i<3; i++) {
			System.out.println("main");
		}
	}

	@Override
	public void run() {
		for(int i=0; i<3; i++) {
			System.out.println("thread");
		}
	}

}
```

## Esercizio 3
Scrivere un programma che:
- Definisce un nuovo thread come estensione della classe thread
- Ne crea un’istanza passandogli (come stringa) un nome.
- Il thread scrive tre volte sullo standard output il proprio nome (che è uguale alla stringa passatagli all’atto della creazione).
- Il main scrive tre volte sullo standard output «main».

```java
public class Es3 extends Thread {

	public static void main(String[] args) {
		
		Thread t = new Es3("Java");
		
		for(int i=0; i<3; i++) {
			System.out.println("main");
		}
	}
	
	public Es3(String name) {
		this.setName(name);
		start();
	}
	
	public void run() {
		for(int i=0; i<3; i++) {
			System.out.println("Il mio nome è: " + this.getName());
		}
	}

}
```

## Esercizio 4
Scrivere un programma che:
- Definisce un nuovo thread passandogli un nome (come stringa) e un’apposita istanza di runnable
- Il thread scrive tre volte sullo standard output il proprio nome (che è 
uguale alla stringa passatagli all’atto della creazione)
- Il main scrive tre volte sullo standard output «main»

```java
public class Es4 implements Runnable {
	
	private String name;
	
	public static void main(String[] args) {
		
		Thread t = new Thread(new Es4("Java"));
		t.start();
		
		for(int i=0; i<3; i++) {
			System.out.println("main");
		}
	}
	
	public Es4(String name) {
		this.name = name;
	}

	@Override
	public void run() {
		Thread.currentThread().setName(name);
		for(int i=0; i<3; i++) {
			System.out.println("Il mio nome è: " + Thread.currentThread().getName());
		}
	}

}
```

## Esercizio 5
Scrivere un programma che crea un thread come estensione della 
classe thread.
- Il thread scrive tre volte sullo standard output «thread».
- Il main scrive tre volte sullo standard output «main».
- Il thread deve iniziare ad eseguire appena creato, senza che il main
debba invocare start

```java
public class Es5 extends Thread{

	public static void main(String[] args) throws InterruptedException {
		Thread t = new Es5();
		Thread.sleep(1);
		
		for(int i=0; i<3; i++) {
			System.out.println("main");
		}
	}
	
	public Es5() {
		start();
	}
	
	public void run() {
		for(int i=0; i<3; i++) {
			System.out.println("thread");
		}
	}

}
```

## Esercizio 6
Scrivere un programma che definisce un thread che scrive 10 volte sullo standard output il suo numero (ricevuto all’atto della creazione). <br>
Il main:
- Legge dalla riga di comando un numero N compreso tra 1 e 5 (estremi compresi) 
- Crea N istanze del thread
- Attende la terminazione di tutte le istanze di thread create

```java
public class Es6 extends Thread {

	public static void main(String[] args) throws IOException {
		
		BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
		int n=0;
		while(n < 1 || n > 5) {
			System.out.println("Inserire un valore compreso tra 1 e 5");
			String user_input = in.readLine();
			try {
				n = Integer.parseInt(user_input);
			} catch(Exception e) {
				System.out.println("Inserimento non valido.");
				continue;
			}
		}
		for(int i=0; i<n; i++) {
			Thread t = new Es6(String.valueOf(i));
		}
	}
	
	public Es6(String name) {
		this.setName(name);
		start();
	}
	
	public void run() {
		for(int i=0; i<10; i++) {
			System.out.println("Il mio numero: " + this.getName());
		}
	}

}
```

## Esercizio 7
Scrivere un programma in cui il main crea un thread che indefinitamente scrive “helo” sullo standard output, ogni mezzo secondo (approssimativamente).
- Il main legge iterativamente da standard input. Quando legge «stop» 
manda un segnale al thread
- Il thread reagisce al segnale scrivendo «termino» e termina
- Dopo la terminazione del thread, il main termina a sua volta
```java
public class Es7 extends Thread {

	public static void main(String[] args) throws IOException, InterruptedException {
		
		Thread t = new Es7();
		t.start();
		
		String input="";
		while(!input.equalsIgnoreCase("stop")) {
			BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
			System.out.print("Write something (stop to finish): ");
			input = in.readLine();
		}
		
		t.interrupt();
		t.join();
		
		System.out.println("main closed");
	}
	
	public void run() {
		while(true) {
			try {
				if(this.isInterrupted()) {
					break;
				}
				sleep(500);
				System.out.println("helo");
			} catch (InterruptedException e) {
				break;
			}
		}	
		System.out.println("thread closed");
	}
}
```