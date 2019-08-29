# File System Implementation

- [File System Implementation](#file-system-implementation)
  - [Struttura File System](#struttura-file-system)
    - [Device Driver](#device-driver)
    - [Basic File System](#basic-file-system)
    - [File Organization Module](#file-organization-module)
    - [Logical File System](#logical-file-system)
  - [Operazioni FS](#operazioni-fs)
    - [Boot Control Block](#boot-control-block)
    - [VOlume Control Block](#volume-control-block)
  - [FS Implementation](#fs-implementation)
  - [Directory Implementation](#directory-implementation)
  - [Metodi allocazione](#metodi-allocazione)
    - [Allocazione contigua](#allocazione-contigua)
    - [ExtentBased System](#extentbased-system)
    - [Linked Allocation](#linked-allocation)
    - [FAT File Allocation Table](#fat-file-allocation-table)
    - [Indexed Allocation](#indexed-allocation)
    - [INODE](#inode)
  - [Performance](#performance)
  - [Free-Space Gestione](#free-space-gestione)
  - [Efficiency and Performance](#efficiency-and-performance)
  - [Page Cache](#page-cache)
  - [Log Structured File Systems](#log-structured-file-systems)

## Struttura File System

Strutturazione logica del disco fisso e una collezione di informazioni relative.

Si lavora in memoria secondaria.

FCB (File Control Block) struttura che contiene informazioni sul dato per poterci arrivare.

Chi accede ai dati? Noi sappiamo che il device driver permette di controllare e accedere al settore.

Per potervi accedere andiamo ad utilizzare una serie di strati.

### Device Driver

SOno i dispositivi che gesticono I/O e leggono il disco, cilindro, tracciato e settore per poter portare il dato in memoria. Basso livello

### Basic File System

Ci serve il blocco X? Questo dispositivo trasforma nel dato che serve al Device Driver. Qua si potrebbe andare a gestire problematiche di caching.

### File Organization Module

COnosce la struttura del file e permette di passare da indirizzi logici a fisici.

### Logical File System

Passaggio da nomi a dato vero e proprio, gestione direttori e file. COnverte i nomi dei file in id del file, file handle, la positione  dal control block.

## Operazioni FS

Abbiamo anche altre strutture dati

### Boot Control Block

Pezzo che permette di bootstrappare il FS e il sistema operativo di quel volume

### VOlume Control Block

Contiene tutti i dettagli del volume

## FS Implementation

Per ogni file è presente un **FCB (File COntrol Block)** che contiene varie informazioni sul file.

- Permessi
- Date varie (creazione, accesso, scrittura)
- Owner
- Dimensione
- Data block che puntano ai dati.

Deve stare tutto su disco vero, ma la modifica degli stessi avviene in RAM quasi sempre.

Ci saranno strutture dati in-memory che sono una sorta di ridondanza.

- Mount table
- System-wide open-file table
- Per-process open-file table

Le informazioni che ci servono sul secondary storage sono FCB e i direttori (open) o  data blocks (Read).

In open noi cerchiamo nome nei direttori, mentre in read partiamo dal file descriptor che ci permette di arrivare a un FCB che possiamo usare per manipolare il file.

Perchè ho bisogno di due tabelle? Nella system-wide open-table abbiamo l'FCB che è un pezzo comune, mentre ne abbiamo un altro che deve essere scorporato e personale del processo. L'altra motivazione è anche necessaria per gestire le differenti manipolazioni da parte dei diversi processi. Basti pensare ad esempio dai permessi che sono dati a processi diversi.

Nella system wide la struttura scomparirà appena non ci saranno più open relative a quel file. Il FCB non viene replicato e basta, ma vengono aggiunti dati in più utili al sistema.

## Directory Implementation

- Linear List: con puntatori a data blocks. Semplice, ma time-consuming
- Hash-Table: linear list with hashed data structure. Diminusce il tempo di ricerca. Problema delle collisioni

## Metodi allocazione

### Allocazione contigua

numero blocco iniziale e quanti sono i blocchi. Ma frammentazione esterna e critica. SOlitamente si fanno schemi di compattazione online o offline.

### ExtentBased System

Un po' di allocazione contigua in un extent, se non basta si procede con un altro.

### Linked Allocation

Prima vera alternativa alla allocazione contigua.
Non è contigua e non ha frammentazione esterna, ma paghiamo nel momento in cui dobbiamo fare accesso diretto. Questo perchè quasi sicuramente un file verrò utilizzato in maniera sequenziale.

Vantaggi:

- No ext fragm
- Puntatore a blocco successivo
- Non è necessario compattazione
- Basta lista blocchi liberi

Svantaggi:

- Problemi accesso diretto, se so dove voglio andare devo analizzarli tutti, ovvero portarli da disco in ram.
- Affidabilità problematica, se mi perdo un puntatore, mi perdo tutto il resto in avanti.
- Il puntatore porta via spazio ai dati.

### FAT File Allocation Table

L'indirizzo è la posizione vettore che contiene i successori. Questo serve per evitare di portare in memoria i blocchi su disco. Scansione sequenziale più rapida, utile nel momento in cui ci serve accesso diretto.

### Indexed Allocation

Tabella di indici, ogni file ha una tabella con indici di blocchi non contigui. Un blocco con l'elenco di tutti gli indici. Ma se abbiamo tanti blocchi satura.

### INODE

- Blocchi diretti
- Blocchi indiretto singolo
- Blocco indiretto doppio
- Blocco  indiretto triplo

Una tipologia di blocco viene utilizzato solo nel momento in cui il tipo di indirezione successivo non basta.

## Performance

Il metodo dipende con il tipo di accesso che vogliamo fare.
ALlocazione contigua va bene per accesso sequanziale o random.
La linked è buona per sequenziale, ma non per random.

L'utilizzo di indici è più complesso,

## Free-Space Gestione

- Free-SPace List. In FAT può essere fatta direttamente dentro.
- Vector di bit o bit map. Il problema è la ricerca in bitmap, ci sono algoritmi logaritmici appositi. La bitmap richiede ulteriore spazio "rubati" allo spazio reale.

Una linked list di free space è come un file in più nel direttorio?

## Efficiency and Performance

- COme si allocano le cose du disco e algoritmi delle directory.

Possono essere influenzati da dove si trovano i meta dati.

- Buffer Cache: virtualizzare parti di disco in RAM, cache per il disco.

- Sincrono o Asincrono

## Page Cache

Utilizzo di pagine al posto di blocchi che permettono di sfruttare tecniche della memoria virtuale.

Il memory mapping io prende un file e lo nasconde in una sorta di malloc che non è heap. Iesima casella che ha dietro un pezzo di file. L'OS gestirà una page cache, ma si rischiano 2 livelli di caching molto simili.

Potevano essere riutilizzate -> Unified Buffer Cache.

## Log Structured File Systems

Journaling salva ogni metadata di un record come una transazione. Se qualcosa va male posso rifar girare la transazione per recuperare tutto.
