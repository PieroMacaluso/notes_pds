# IPC

L'uso di thread permette di sfruttare le risorse computazionali presenti nell'elaboratore, ma ci sono situazioni in cui la presenza di un singolo spazio di indirizzamento non è possibile o desiderabile.

- Riuso di programmi esistenti
- Scalabilità su più computer
- Sicurezza (TAB Chrome)

E' possibile decomporre un sistema complesso in un insieme di processi collegati.

- Creandoli a partire da un processo genitore
- Cooperazione indipendente dalla loro genesi

Ad ogni processo è associato almeno un thread.

## Processi in WIndows

`CreateProcess(...)` completamente indipendenti dagli altri. Sono entità separate senza relazioni di dipendenza esplicita tra loro.

- Crea un nuovo spazio di indirizamento
- Inizializza con l'immagine di un eseguibile
- Attiva il thread primario al suo interno

Il figlio può condividere variabili d'ambiente, handle a file, semafori e pipe. Non possono condividere handle a thread, processi, librerie dinamiche e regioni di memoria.

## Processi in Linux

Per creare un processo figlio si utilizza l'operazione `fork()` che va acreare uno spazio di indirizzamento identico a quello del processo genitore. RIsparmia memoria fisica fino a quando leggiamo, nel momento in cui scriviamo allora si vanno a dividere le tabelle.

La fork viene eseguita e ritorna in due posti differenti. Nel padre ritorna l'id del processo figlio, nel figlio si vede id 0 e questo può risalire al padre.

Le funzioni della famiglia exec() spianano il contenuto andandolo a sostituire. Dalla exec in poi non si torna mai!

Cosa succede se un secondario thread chiama una fork? Il processo figlio conterrà un solo thread che potrà trovarsi una situazione di memoria corrotta.

Per questo motivo si può andare ad utilizzare `int pthread_atfork() void(*prepare)(void), void (*parent)(void), void (*child)(void)`. Sono come delle callback.

- Viene chiamata la prepare nel processo padre per fare in modo che si blocchino tutte le operazioni delicate (mutex), poi viene chiamato parent che si occupa di rilasciare i mutex, mentre child fa una cosa simile però nel processo figlio.

## IPC - InterProcess Communication

Il sistema operativo permette di avere spazi di indirizzamento separati e indipendent, ma impedisce il trasferimento di dati tra processi. Ogni sistema operativo offre meccanismi per superare tale barriera premettendo lo scambio strutturato di dati e la sincronizzazione delle attività.

Dobbiamo cambiare la rappresentazione per renderle leggibili all'altro processo.

Internamente si può usare una varietà di rappresentazione che non è adatta ad essere esportata poichè il puntatore non ha senso al di fuori del proprio spazio di indirizzamento.

La rappresentazione esterna è un formato intermedio che permette la rappresentazione di struttura dati arbitrarie sostituendo i puntatori con riferimenti indipendenti.

Due strade grossolane:

- Formati basati su testo (XML, JSON, CSV)
  - Vantaggio è che sono debuggabili
  - Svantaggio: è inefficient
- Formati binari (XDR, HDF)
  - Più compatto ed efficiente
  - Ma problema che è poco debuggabile.

Da interno a esterno serializziamo i nostri dati, ma l'algoritmo di serializzazione non deve compromettere il dato e deve essere possibile deserializzarli. (marshalling)

### Tipi di IPC

#### Code di messaggi

Code di messaggi che fanno sia comunicazione, sia sincronizzazione.

Permettono il trasferimento atomico di blocchi di byte. Comportamento FIFO.

#### Pipe

Tubo in cui abbiamo comunicazione a senso unico che non hanno un boundary (un byte alla volta). Occorre inserire dei marcatori per delimitare i singoli messaggi. Può essere anche bufferizzato

#### Memoria condivisa

Accessibile ai due processi, ma bisogna sincronizzarsi. Per funzionare deve essere sullo stesso dispositivo. Servono costrutti di sincronizzazione a basso livello. Un semaforo può essere utilizzato, ma sconsigliabile. Un semaforo a 1 è un mutex, ma il mutex conosce chi l'ha creato. NON USARE SEMAFORI AL POSTO DI MUTEX

#### Altro

File (Database), socket, segnali,...

I canali creati da SO sono creati su richiesta dei singoli processi. Vengono associati ad un identificativo univoco a livello di sistema.

### Scelta del meccanismo

- Relazione tra i processi
  - Dipendenza Esplicita
  - Nessuna dipendenza
- Tipo di comunicazione
  - Monodirezionale
  - Bidirezionale
- Numero di processi coinvolti
- Parlare nella stessa macchina o in macchine differenti?

# IPC in Windows

La piattaforma Win32 offre una ricca serie di meccanismi di sincronizzazione che si basano su oggetti kernel condivisi.
Tutte le tecniche sono più generali, ma meno efficienti e si basano sull'attesa.

`WaitForSingleObject()` o `WaitForMultipleObject()` con semantica particolare (tutto nulla).

Ogni oggetto definisce le proprie politiche di attesa e risveglio. e può trovarsi in due stati differenti: Segnalato e Non Segnalato. Gli oggetti di sincronizzazione possono alternare i due stati di segnalazione.

## Eventi

Sono oggetti kernel che modellano il verificarsi di una condizione.

Vorrebbe assomigliare alle condition variable. Aspettiamo fino a quando l'evento scatta.
Auto-reset event che tornano false appena qualcuno si risveglia, oppure manual-reset event che rimanfono a true fino a quando non viene chiamato il reset.

`CreateEvent(...)` se ne indico uno già esistente mi lego.
`OpenEvent(...)` mi collego ad uno esistente.

I processi condividono l'evento tramite il nome.

`SetEvent(...)` Da false a true
`ResetEvent(...)` tutto a false
`PulseEvent(...)` una sorta di notify che non ha ascoltatori.

## Semafori

`ReleaseSemafore(...)` incrementa semaforo
`WaitForSingleObject(...)` prova a decrementare, altrimenti si blocca e resta in attesa.

## Mutex

Oggetti kernel di WIndows. Funzionano anche tra processi differenti. COnservano ID del thread e un contatore.

`CreateMutex`, `OpenMutex`
`ReleaseMutex` se arriva a zero segnala il mutex.

Molto spesso è utile tenere una o più primitive. Si può usare un mutex per regolare l'accesso a due o più eventi per indicare il completamento delle operazioni. Questo è per evitare DeadLock.

## MailSlot

Code di messaggi asincrone dove ogni processo può depositare un messaggio. Un processo può essere mailslot client e mailslot server.

Il messaggio viene conservato finchè non viene letto

`GetMailslotInfo`

`CloseHandle()`

## Pipe

Possono essere anonime o named.

### Anonymous

Efficiente tra 2 processi parenti

### Named Pipe

Possono essere bidirezionali. I messaggi non sono segmentati, quindi si possono mandare dei blocchi.

Possiamo avere la modalità di lettura con stream di byte o messaggi.

Scegliamo anche quella che è la modalità di attesa che può essere bloccante (attesa fine processi indefinita) o non bloccante (ritorna situazioni con attesa infinita.).

## File Mapping

Nel mio processo voglio uno spazio che può essere letto o modificato anche da altri . Per poter fisicamente scrivere usiamo `MapViewOfFile` e mi scollego con  `UnmapViewOfFile`. Solitamente si utilizza con un mutex poichè ci si potrebbe scrivere in contemporanea, non c'è un controllo.

## Altri meccanismi

### Socket

Permettono comunicazioni tra macchine con sistemi operativi diversi

### RPC

# IPC Linux

Abbiamo gli stessi concetti con metodi differenti. Abbiamo in più i signal.

Gli identificativi non sono nomi, ma id interi non negativi. All'atto della creazione si fornisce una chiave e il sistema la converte nell'ID associato. Coloro che possiedono la chiave potrenno farvi riferimento a meno che la struttura IPC non venga contrassegnata come `IPC_PRIVATE(0)`.

## Message QUeues

Per comunicare si accordano sul pathname di un file esistente e su un Project ID.

Come viene costruita la chiave? A partire da `ftok(...)`. Basta un file di punto di riferimento che può essere usato per ftok e la chiave.

Strutture che permettono lo scambio di messaggi composti da tipo e messaggio.

`msgget`

`msgsnd`

`msgrcv`

`msgctl` comandi particolaro

## Pipe

Permettono IPC tra padre e figlio.

Anche **named** FIFO con `mkfifo` che permette la comunicazione tra processi generici.

## SHaredMemory

`shmget()`

`shmcnt()` ottengo informazioni sul segmento

IPC_STAT, IPC_SET, IPC_RMID

`shmat` per mapparla nel mioo spazio di indirizzamento

`shmdt` detach, unmapping.

Posso mappare una shared memory (Blocco)
Oppure una Memory Mapped File per poter mappare porzioni di file mappate in memoria. `mmap`

Per il rilascio `munmap`

## Semafori System-V (Five)

Servono per sincronizzazione di thread di processi differenti costituiti da array di contatori

`semget`

`semop` per incrementare e decrementare

`semctl` permettono di modificare limiti superiori e inferiori.

Molto spesso la miglior scelta è l'utilizzo di Socket.