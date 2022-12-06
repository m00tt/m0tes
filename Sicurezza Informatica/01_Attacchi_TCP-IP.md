# Introduzione

## Classi di indirizzo
![Classi di indirizzo](assets/sicurezza_informatica/classi-indirizzo.png)<br>
Gli indirizzi IP vengono assegnati a classi di indirizzi IP.
| Class | Netowrk Address | Host Address | Example Address |
| ----- | --------------- | ------------ | --------------- | 
|<b>Classe A</b> <br>Address Range = 1.0.0.1 to 126.255.255.254 | I primi 8 bit definiscono il network address. L'indirizzo binario del primo ottetto inzia sempre con 0. L'indirizzo decimale ha un range da 1 a 126 (127 netowrk) | I rimanenti 24 bit definiscono l'host address | 110.160.212.156 <br> Network = 110<br> Host = 162.212.156 |
|<b>Classe B</b> <br>Address Range = 128.1.0.1 to 191.255.255.254 | I primi 16 bit definiscono il network address. L'indirizzo binario del primo ottetto inizia sempre con 10. L'indirizzo decimale ha un range che va da 128 a 191 (16000 network). 127 è riservato per il loopback testing su localhost | I 16 bit rimanenti definiscono l'host address (65000 host) | 168.110.226.155<br> Network = 168.110 <br> Host = 226.155 |
|<b>Classe C</b> <br>Address Range = 192.0.1.1 to 223.255.254.254 | I primi 24 bit definiscono il network address. L'indirizzo binario del primo ottetto inizia sempre con 110. L'indirizzo decimale ha un range che va da 192 a 223 (2 milioni di network)| Gli 8 bit rimanenti definiscono l'host address (254 host) | 200.168.198.156<br> Network = 200.168.198 <br> Host = 156 |
|<b>Classe D</b> <br>Address Range = 224.0.0.0 to 239.255.255.255 | L'indirizzo binario del primo ottetto inizia sempre con 1110. L'indirizzo decimale ha un range che va da 224 a 239 | Riservati per il multicasting | |