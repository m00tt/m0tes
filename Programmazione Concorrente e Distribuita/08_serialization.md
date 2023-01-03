# Serializzazione e Deserializzazione
La scrittura di uno stream di byte di un oggetto prende il nome di <b>serializzazione</b>.

Come facilmente intuibile, la <b>deserializzazione</b> è l'operazione inversa che comporta la lettura da uno stream di byte di un oggetto serializzato e la ricostruzione dell'oggetto originale.

![Serialization](/assets/programmazione_concorrente_e_distribuita/serialization.png)


# Interfacce `ObjectInput` e `ObjectOutput`
L'interfaccia <b>ObjectOutput</b> consente di serializzare tipi primitivi e oggetti.<br>
Oltre alle operazioni di `flush()` e `close()`, mette a disposizione anche:

- Metodi per scrivere byte
    - `void write(int b) throws IOException;`
    - `void write(byte[] b) throws IOException;`
    - `void write(byte[] b, int off, int len) throws IOException;`

- Metodi per scrivere tipi primitivi
    - `void writeInt(int v) throws IOException;`
    - `void writeDouble(double v) throws IOException;`
    - `etc`

- Un metodo per la scrittura degli oggetti generici
    - `void writeObject(Object obj) throws IOException;`


L'interfaccia <b>ObjectInput</b> permette la deserializzazione di un oggetto o un tipo primitivo.<br>

- Metodi per la lettura di byte
    - `int read() throws IOException;`
    - `int read(byte[] b) throws IOException;`
    - `int read(byte[] b, int off, int len) throws IOException;`

- Metodi per la lettura di tipi primitivi
    - `int readInt() throws IOException;`
    - `double readDouble() throws IOException;`
    - `etc`

- Un metodo per la lettura degli oggetti generici
    - `Object readObject() throws ClassNotFoundException, IOException;`

- Il metodo `close()`

- Il metodo `long skip(long n) throws IOException;` : per saltare _n_ byte di input

- Il metodo `int available() throws IOException;` : indica quanti byte possono essere ancora letti senza bloccarsi

<b>! </b>: Siccome il tipo restituito da `readObject` è `Object`, spetta al programmatore convertire l’oggetto deserializzato al tipo giusto.

Le interfacce `ObjectOutput` e `ObjectInput` sono implementate rispettivamente dagli stream di byte `ObjectOutputStream` e `ObjectInputStream`.


# Serializzazione di un oggetto primitivo
La serializzazione di tipi primitivi avviene semplicemente usando gli appositi metodi della classe `ObjectOutputStream`.
```java
import java.io.*;
public class PrimitiveSerialization {
    public static void main(String[] args) throws IOException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os =
            new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            int i = 11;
            os.writeInt(i);
            os.flush();
        }
    }
}
```

In modo analogo, la deserializzazione avviene attraverso la classe `ObjectInputStream`.
```java
import java.io.*;
public class PrimitiveDeserialization {
    public static void main(String[] args) throws IOException {
        String fileName = "tmp.bin";
        try (
            ObjectInput is =
            new ObjectInputStream(new FileInputStream(fileName))
        ) {
            int i = is.readInt();
            System.out.println("Read: " + i);
        }
    }
}
```

# Serializzazione di un oggetto
```java
import java.io.*;
public class ObjectSerialization {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            os.writeObject("Now");
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String s = (String) is.readObject();
            System.out.println("Read: " + s);
        }
    }
}
```

# Serializzazione di più oggetti
```java
import java.io.*;
import java.util.Date;
public class ObjectsSerialization {
    public static void main(String[] args)
    throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            os.writeObject("Now");
            os.writeObject(new Date());
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String s = (String) is.readObject();
            Date d = (Date) is.readObject();
            System.out.println(s + " " + d);
        }
    }
}
```

# Serializzazione di un array di oggetti
```java
import java.io.*;
import java.util.Arrays;
public class StringArray {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        String fileName = "tmp.bin";
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream(fileName))
        ) {
            String[] array = new String[]{"rosso", "giallo", "blu"};
            os.writeObject(array);
            os.flush();
        }
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream(fileName))
        ) {
            String[] array = (String[]) is.readObject();
            System.out.println(Arrays.toString(array));
        }
    }
}
```

# Serializzazione di oggetti definiti dal programmatore
Consideriamo la seguente classe

```java
public class Punto {
    private int x, y;
    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return "Il punto ha coordinate " + x + " e " + y;
    }
}
```

Per far si che sia <b>serializzabile</b> è necessario che:
- La classe implementi l'interfaccia `Serializable`
- È buona norma, definire una costante chiamata `serialVersionUID` che serve da numero di versione della classe. Serve in fase di deserializzazione per verificare che le classi usate da chi ha serializzato l'oggetto e chi lo ha serializzato siano compatibili: se le versioni non corrispondono verrà sollevata un'eccezione `InvalidClassException`. Se omessa, la JVM calcola un numero di versione automaticamente che potrebbe causare errori in alcuni casi.

```java
public class Punto implements Serializable {
    private static final long serialVersionUID = 1;
    
    private int x, y;
    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return "Il punto ha coordinate " + x + " e " + y;
    }
}
```

Una volta preparata la classe, possiamo procedere alle operazioni di serializzazione e deserializzazione.

```java
//Serializzazione
import java.io.*;
public class SerializzaPunto {
    public static void main(String[] args) throws IOException {
        try (
            ObjectOutput os = new ObjectOutputStream(new FileOutputStream("dati.dat"))
        ) {
            Punto p = new Punto(2, 3);
            os.writeObject(p);
            os.flush();
        }
        System.out.println("Serializzazione completata.");
    }
}

//Deserializzazione
import java.io.*;
public class DeserializzaPunto {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        try (
            ObjectInput is = new ObjectInputStream(new FileInputStream("dati.dat"))
        ) {
            Punto p = (Punto) is.readObject();
            System.out.println(p);
        }
    }
}
```

# Regole di serializzazione in Java
- Tutti i tipi primitivi sono serializzabili
- Un oggetto è serializzabile se la sua classe o la sua superclasse implementano l'interfaccia `Serializable`
- Una vlasse può implementare `Serializable` anche se la sua superclasse non è serializzabile, purché tale superclasse abbia un costruttore senza argomenti
- Se si desidera serializzare solo alcuni campi di una classe, è possibile contrassegnare quelli da non salvare con l'attributo `transient`
- I campi `static` di ua classe non vengono serializzati
- Se i campi di un oggetto serializzabile contengono un riferimento a un oggetto non serializzabile, verrà sollevata una `NotSerializableException`