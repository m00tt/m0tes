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

## Esercizio 3: ListaDir
- Se sulla linea di comando c’è un argomento lo interpreta come il nome di un file o di una directory; se sulla linea di comando non c’è alcun argomento, si assume come argomento la directory corrente.
- Se l’argomento è il nome di un file, stampa a terminale il suo path assoluto e la sua dimensione in byte;
- Se l’argomento è una directory, stampa a terminale il suo contenuto, cioè la lista dei file e delle directory che essa contiene.


```java
import java.io.File;

public class ListaDir {
	public static void main(String[] args) {
		
		File f;

		if (args.length > 0) {
			f = new File(args[0]);
		} else {
			
			f = new File(System.getProperty("user.dir"));
		}
		
		if (f.isFile()) {
			System.out.println("Absolute path: " + f.getAbsolutePath());
			System.out.println("Space: " + f.length() + " bytes");
		} else if(f.isDirectory()) {
			String[] lis = f.list();
			for (int i=0; i<lis.length; i++) {
				System.out.println(lis[i]);
			}
		} else {
			System.out.println("Path non corretto.");
		}
	}
}
```

## Esercizio 4: Analisi del Testo
Scrivere una classe Testo che modelli un testo letto da un file e che fornisca i seguenti costruttori e metodi:
- `public Testo(File file)`: Costruisce l’istanza dell’oggetto che modella il testo contenuto nel file specificato come argomento. Si assuma che l’argomento sia il riferimento ad un file di testo esistente.
- `public int numeroParole()`: Restituisce il numero di parole che compaiono nel testo modellato dall’oggetto che esegue il metodo.
- `public int numeroParoleDistinte()`: Restituisce il numero di parole distinte che compaiono nel testo modellato dall’oggetto che esegue il metodo.
- `public int contaOccorrenzeParola(String daCercare)`: Restituisce il numero di occorrenze della parola specificata come argomento nel testo che esegue il metodo.
- `public LinkedList<String> paroleDistinteInOrdineAlfabetico()`: Restituisce la lista delle parole del testo (senza ripetizioni) in ordine alfabetico.

```java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.StringTokenizer;

public class Testo {
	
	static String text_import="";
	
	public Testo(File file) throws IOException {
		if (!file.isFile()) {
			System.out.println("Inserire il path ad un file valido.");
			System.exit(0);
		}
		
		FileReader fr = new FileReader(file);
		BufferedReader br = new BufferedReader(fr);
		String tmp;
		while((tmp = br.readLine()) != null) {
			text_import += tmp;
		}
		
		System.out.print(text_import);
		
		br.close();
		fr.close();
	}

	public static void main(String[] args) throws IOException {
		
		File f = new File(args[0]);
		Testo t = new Testo(f);
		
		System.out.println("\nNumero di parole: " + t.numeroParole());
		System.out.println("\nNumero di parole distinte: " + t.numeroParoleDistinte());
		System.out.println("\nNumero di occorrenze di 'copia': " + t.contaOccorrenzeParola("copia"));
		System.out.println(paroleDistinteInOrdineAlfabetico());
		
	}
	
	public int numeroParole() {
		if (text_import == null || text_import.isEmpty()) { return 0; } 
		
		StringTokenizer tokens = new StringTokenizer(text_import); 
		return tokens.countTokens();
	}
	
	public int numeroParoleDistinte() {
		if (text_import == null || text_import.isEmpty()) { return 0; }
		
		String[] words = text_import.split(" ");
		ArrayList<String> listWithDuplicates = new ArrayList<>();
		for(int i=0; i<words.length; i++) {
			listWithDuplicates.add(words[i]);
		}
		ArrayList<String> listWithoutDuplicates = new ArrayList<>(new LinkedHashSet<>(listWithDuplicates));
        
        return listWithoutDuplicates.size();
	}
	
	public int contaOccorrenzeParola(String search) {
		if (text_import == null || text_import.isEmpty()) { return 0; }
		
		int count = 0;
		
		String[] words = text_import.split(" ");
		for (int i=0; i<words.length; i++) {
			if (search.equalsIgnoreCase(words[i])) {
				count++;
			}
		}
        return count;
	}
	
	public static LinkedList<String> paroleDistinteInOrdineAlfabetico() {
		
		String[] words = text_import.split(" ");
		LinkedList<String> listWithDuplicates = new LinkedList<>();
		for(int i=0; i<words.length; i++) {
			listWithDuplicates.add(words[i]);
		}
		LinkedList<String> listWithoutDuplicates = new LinkedList<>(new LinkedHashSet<>(listWithDuplicates));
        Collections.sort(listWithoutDuplicates);
        return listWithoutDuplicates;
	}
}
```