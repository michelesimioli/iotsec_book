# Target Analysis

## Modi Operativi

**Drill**

* Compromissione macchine deboli non bersaglio
  * Acquisizione piattaforme intermedie (Stepping Stones)
  * Acquisizione Agenti per DDoS
  * Esperienza operativa e Training
  * Confondere le acque
* Esecuzione lenta e flessibile

**Earnest**

* Attacco a bersaglio finale
  * Attraverso Stepping Stones
  * Contromisure al Backtracking
  * Utilizzando Agenti
* Esecuzione veloce e mirata, forse automatica

### Scenario di Attacco

* Acquisizione di informazioni sul target
* Scansioni di servizi di rete disponibili - **Scanners**
* Penetrazione di macchine deboli della rete - Clients
* Captazione di password di accesso - **Sniffers**, **Key Loggers**
* Scansioni di vulnerabilità
* Penetrazione di macchine bersaglio - **Exploits**
* Installazione di ‘Backdoor’ - **Troiani**, **Root Kits**
* Operazioni Illegali desiderate
* Cancellazione di Log di Sistema – **Bashers**

### Acquisizione di Informazioni

![AcqInfo](../gitbook/images/acqinfo.png)

Reconnaissance o Intelligence Gathering.

Passiva: **Open Source Intelligence** (_OSINT_):

* Dati pubblici economici o legali
* Motori di ricerca
* Siti web
* Conferenze e ambienti accademici
* Blogs e Social Networking
* Metadati da foto e documenti

### Footprinting

![Footprinting](../gitbook/images/footprinting.png)

Enumerazione non intrusiva della rete target

* Server DNS
* Range di indirizzi IP
* Banner
* Sistemi Operativi
* Tipi di IDS/IPS in uso
* Tecnologie usate
* Dispositivi di rete

Obiettivo: acquisire una mappa completa della rete.

### DNS - nslookup

![DNS](../gitbook/images/dns.png)

In alcuni Linux è pesantemente tarpato

Uso standard:

```bash
nslookup example.com
```

Sessione con comandi interattivi:

```bash
$ nslookup
> server 156.154.70.22
> set type=ns
> example.com
> set type=mx
> example.com
> ls example.com
> exit
$
```

