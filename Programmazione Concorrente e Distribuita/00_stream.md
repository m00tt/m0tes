# Intro
Gli stream sono sequenze ordinate di dati che hanno una sorgente (input stream) o una destinazione (output stream).
<br>
Le classi di `java.io` nascondono i dettagli del sistema operativo sosttostante e dei dispositivi di ingresso e uscita coinvolte nelle operazioni.

- <b>Input Stream</b>: per ricevere le informazioni, un'app apre uno stream collegato ad una sorgente dalla quale legge sequenzialmente le informazioni.
- <b>Output Stream</b>: per scrivere informazioni, un'app apre uno stream verso una destinazione e vi scrive sequenzialmente le informazioni.

## Java.io Package
Il package `java.io` è composto principalmente da 2 sezioni:
 - Byte stream
    - L'unità di informazione gestita è il byte
    - I/O binario
    - Le classi che realizzani i byte stream sono: `input stream` e `output stream`

 - Character Stream
    - L'unità di informazione gestita sono i caratteri Unicode
    - I/O testuale
    - Le classi che realizzano i character stream sono: `reader` e `writer`

### Errori
Naturalmente durante l'utilizzo degli stream, possono essere generati degli errori. Essi vengono segnalati principalmente in 2 modi:
 - Cambiando lo stato dello stream
 - Lanciando una `IOException`


## Esempi
### Lettura di file binari (input stream)
Per aprire un input stream verso un file si utilizza il costruttore
```java
FileInputStream in = new FileInputStream(<filename>);
```
e successivamente si utilizza il metodo `read()` che restituisce il byte letto.
<br>

 - All'EOF il metodo `read()` restituisce `-1`.
 - Al termine delle operazioni bisogna chiudere lo stream tramite il metodo `close()`


### Scrittura di file binari (output stream)
Per aprire un output stream verso un file si utilizza il costruttore
```java
FileOutputStream in = new FileOutputStream(<filename>);
```
e successivamente si utilizza il metodo `write(int c)` per scrivere 
<br>

 - Al termine delle operazioni bisogna chiudere lo stream tramite il metodo `close()`

### Esempio: clone file
```java
import java.io.*;
public class CloneBin {
    public static void main (String[] args) throws IOExcepion {
        int c = 0;
        FileInputStream in = new FileInputStream(<file_location>);
        FileOutputStream out = new FileOutputStream(<file_location>);
        while((c = in.read()) != -1){
            out.write(c);
        }
        out.close();
        in.close();   
    }
}
```
<br>
<br>

### Lettura di un file di testo (reader)
L'utilizzo è molto simile a FileInputStream. <br>
Si utilizza il costruttore 
```java
FileReader fr = new FileReader(<filename>);
```
Esistono però vari metodi, tra cui:
 - `public int read() throws IOException`: legge un singolo carattere dallo stream e se si è raggiunta la fine del file restituisce `-1`.
 - `public int read(char[] buf) throws IOException`: legge dallo stream che esegue il metodo una sequenza di caratteri 
della stessa lunghezza dell’array specificato come argomento e li 
memorizza nelle posizioni successive dell’array.
 - `public void close() throws IOException`: deve essere invocato per rilasciare la risorsa.

### Esempio: input di un file di caratteri
```java
import java.io.*;
public class CharFile {
    public static void main (String[] args) throws IOExcepion {
        FileReader fr = new FileReader(<file_location>);
        int i = 0;
        while((i = fr.read()) != -1){
            System.out.print((char)i);
        }
        fr.close();
    }
}
```

### Esempio: scrittura di un file di caratteri
```java
import java.io.*;
public class WriteFile {
    public static void main (String[] args) throws IOExcepion {
        FileWriter fw = null;
        try {
            fw = new FileWriter(<file_location>);
            fw.write('H');
            fw.write('e');
            fw.write('l');
            fw.write('o');
        } catch(Exception e){
            e.printStackTrace();
        } finally {
            try {
                if(fw != null) {
                    fw.flush();
                    fw.close();
                }
            } catch(IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Lettura di una riga di testo (BufferedReader)
Tramite il costruttore 
```java
public BufferedReader(Reader in)
```
ed il suo metodo `public String readLine() throws IOException` è possibile leggere direttamente un'intera riga di testo.
 - Se al momento dell'invocazione è stata raggiunta la fine del file, ritornerà `null`

### Esempio: visualizzazione di un file di caratteri
```java
import java.io.*;
public class CharFileReadLine {
    public static void main (String[] args) throws IOExcepion {
        FileReader fr = new FileReader(<file_location>);
        BufferedReader br = new BufferedReader(fr);
        String str;
        while ((str = br.readLine()) != null){
            System.out.println(str);
        }
        br.close();
        fr.close();
    }
}
```


## Scrittura di una riga di testo (BufferedReader)
Tramite il costruttore 
```java
public PrintWriter(Writer out)
```
è possibile l'utilizzo dei metodi:
 - `print(arg)`
 - `println(arg)`: uguale a print ma aggiunge un EOL
Questi metodi convertono l'argomento in una stringa e la trasferiscono in uscita.

### Esempio: copia di un file di caratteri
```java
import java.io.*;
public class CopyFile {
    public static void main (String[] args) throws IOExcepion {
        String str = "";
        FileReader fr = new FileReader(<file_location>);
        BufferedReader br = new BufferedReader(fr);
        FileWriter fw = new FileWriter(<file_location>);
        PrintWriter pw = new PrintWriter(fw);

        while((str = br.readLine()) != null){
            pw.println(str);
        }

        pw.close();
        fw.close();
        br.close();
        fr.close();
    }
}
```

## Conversione in stream di byte
La classe `PrintStream` consente di convertire i dati primitivi in sequenze di byte. <br>
Tramite il costruttore
```java
public PrintStream(OutputStream out)
```
è possibile accedere ai seguenti metodi
```java
void print(boolean b)
void print(int i) 
void print(long l)
void print(char c)
void print(char[] s)
void print(String s)
void print(float f) 
void print(double d)
void print(Object obj)
```
*Per ciascuno esiste anche il metodo `println`

### Esempio: scrittura di dati primitivi
```java
File file = new File(<file_location>);
FileOutputStream out = new FileOutputStream(file);
PrintStream ps = new PrintStream(out);

ps.println("Provo a scrivere vari primitivi");
ps.println(123);
ps.println(3/4.0);
ps.println(true && false);

```