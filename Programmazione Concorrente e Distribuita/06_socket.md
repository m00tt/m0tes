# Socket
I socket sono un'interfaccia per i protocolli TCP e UDP.
Grazie a essi, si ottiene l’interoperabilità dei programmi di rete:
- Due applicazioni che usano i socket (con lo stesso protocollo di trasporto) possono interagire anche se vengono eseguite su macchine con sistemi operativi diversi, e/o se sono scritte in linguaggi di programmazione diversi.

## Identificazione di una macchina
Per poter comunicare con una macchina, bisogna avere un modo di identificarla univocamente tramite un indirizzo IP.
Un indirizzo IP può essere associato ad un nome di dominio (per facilitarne la memorizzazione) e questa corrispondenza avviene tramite [DNS (Domain Name System)](https://github.com/m00tt/m0tes/blob/security/web_security/server_side/web_fundamentals.md#dns).

In Java, un indirizzo IP può essere ottenuto da un nome di dominio attraverso il metodo statico `InetAddress.getByName`, che restituisce un oggetto di tipo `InetAddress`.

```java
import java.net.InetAddress;
import java.net.UnknownHostException;

public class WhoAmI {
    public static void main(String[] args) throws UnknownHostException {
        if (args.length != 1) {
            System.err.println("Usage: WhoAmI DomainName");
            System.exit(1);
        }
        InetAddress addr = InetAddress.getByName(args[0]);
        System.out.println(addr);
    }
}
```

## Client e server in Java
Per stabilire una connessione client-server in Java è necessario che:
1. Il server deve rimanere in ascolto di richieste di connessione, usando un oggetto speciale `ServerSocket`.
2. Il client cerca di stabilire una connessione con un server, attraverso un oggetto di tipo `Socket`.
3. Una volta effettuata la connessione, vi si costruiscono sopra degli stream di I/O, che permettono di trattarla come se si stesse leggendo da e scrivendo su un file.

## Localhost
Per effettuare dei testi in locale, possiamo utilizzare l'IP di loopback, 127.0.0.1, che è associato al nome simbolico _localhost_, e riferisce, appunto, alla propria macchina.

In Java, ci sono vari modi equivalenti per ottenere un’istanza di `InetAddress` relativa a localhost, tra cui:

```java
InetAddress localhost = InetAddress.getByName(null);
InetAddress localhost = InetAddress.getByName("localhost");
InetAddress localhost = InetAddress.getByName("127.0.0.1");
```

## Identificazione di un'applicazione sulla macchina
Come detto in precedenza, in un server possono essere ospitate molteplici funzioni/applicazioni, quindi è necessario andare a specificare anche il numero di porta sulla quale è in ascolto l’applicazione con la quale si vuole comunicare.

<b>! </b>: Le porte da 0 a 1023 sono solitamente riservate dal sistema, quindi è opportuno usare numeri di porta superiori.

# Uso dei socket in Java
In Java i socket vengono usati mediante due principali classi:
- `ServerSocket`, che un server utilizza per ascoltare le connessioni in ingresso;
- `Socket`, utilizzata da un client al fine di avviare una connessione.

`ServerSocket` fornisce un metodo `accept()`, che sospende il chiamante in attesa di una richiesta di connessione, e restituisce un oggetto Socket nel momento in cui un client si connette. Da questo momento, si ha una connessione tra il socket del client e quello del server.<br>
Si possono allora usare i metodi `getInputStream()` e `getOutputStream()` di ciascun oggetto `Socket` per ottenere degli stream di I/O mediante i quali leggere/scrivere dati, senza più doversi preoccupare dei socket sottostanti.

Quando si crea un `ServerSocket`, è necessario specificare solo il numero di porta sul quale si accetteranno le connessioni (perché l’indirizzo IP è implicitamente quello della macchina su cui si esegue il server). Invece, nella creazione di un Socket (sul client), bisogna indicare sia l’indirizzo IP che il numero di porta del server al quale ci si vuole connettere.

## Stream I/O
Quello che viene trasmesso mediante i socket è una sequenza di byte. <br>
Perciò, i metodi `getInputStream()` e `getOutputStream()` restituiscono degli oggetti di tipo InputStream e OutputStream (che rappresentano, appunto, degli stream di byte).

Se si desidera trasmettere invece dati testuali, è possibile usare `InputStreamReader` e `OutputStreamWriter` per effettuare le conversioni tra byte e caratteri (ed eventualmente `BufferedReader`, `BufferedWriter`, `PrintWriter`, ecc. per una gestione più avanzata della lettura / scrittura).
