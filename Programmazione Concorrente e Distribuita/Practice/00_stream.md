# Stream Practice

## Esercizio 1: TestIOTerminale
- Leggere un numero intero e visualizzarlo a terminale, utilizzando le classi di input/output di java.io.
- Fare in modo che se l’utente non inserisce un numero, il programma segnali l'errore e richieda nuovamente l'inserimento del numero. 
- Fare in modo che il programma richieda e accetti l'inserimento di numeri interi fino a quando l'utente non inserisce «Basta».

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class TestIOTerminale {

	public static void main(String[] args) throws IOException {
		BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
		
		String n = "";
		while(!n.equalsIgnoreCase("basta")) {
			System.out.print("Inserisci un numero: ");
			n = in.readLine();
			try {
				System.out.println(Integer.parseInt(n));
			} catch (Exception e) {
				if(!n.equalsIgnoreCase("basta")) {
					System.out.println("Inserire un numero intero.");
				}
				continue;
			}
		}
		in.close();
	}
}
```

## Esercizio 2: CopiaFileArgs
- Interpreta il primo argomento sulla linea di comando come nome di un file di testo origine e il secondo argomento sulla linea di comando come nome di un file di testo destinazione.
- Copia il contenuto del file origine nel file destinazione, leggendo e scrivendo un solo carattere alla volta.
- Riportare esito e diagnostici.
- Riportare numero di caratteri copiati.

```java
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class CopiaFileArgs {

	public static void main(String[] args) throws IOException {
		File source = new File(args[0]);
		File dest = new File(args[1]);
		FileReader fr = new FileReader(source);
		FileWriter fw = new FileWriter(dest);
		
		int count = 0;
		int r;
		while((r = fr.read()) != -1) {
			fw.write((char)r);
			count++;
		}
		
		fw.close();
		fr.close();
		System.out.println("Numero di caratteri copiati: " + count);
	}
}
```