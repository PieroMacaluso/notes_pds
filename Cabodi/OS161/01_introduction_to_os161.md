# Introduzione a OS161

Sistema operativo didattico ispirato ad un UNIX che può girare in un MIPS. Poco performante e mancano molti pezzi. Includono un macrokernel.

Il framework possiede le sorgenti del sistema operativo e una toolchain.

Lo stack è in una macchina Linux con SYS161, poi OS161 (con development tools) e infine User programs.

Il supporto di cose che devono essere implementate sono i Locks, le system calls, la memoria virtuale e il file system.

```c
#if OPT_HELLO
    hello()
#endif
```

Meglio andare a mettere l'`hello.h` inserito in kern/include `hello.h`. wrappato in if OPT_HELLO e con l'`ifndef`.

La filosofia è includerlo sempre, ma quando non serve lo vede vuoto.

In `HELLO` andiamo a indicare `options hello`.  

## Thread

Un thread è un flusso di esecuzione di un programma in esecuzione. Questo è composto da un contesto, uno stato. Non è composto da variabili locali o codice, ma dalle variabili private del thread.

Un processo è un programma in esecuzione composto da codice, dall'attività corrente (PC), uno stack, la sezione dei dati e lo heap per l'allocazione dinamica.

Il programma è passivo (file eseguibile), mentre il processo è attivo, l'esecuzione del programma.

In memoria il processo è rappresentato con text, data e heap nella parte bassa, stack nella parte alta.

Le parti replicate nei thread sono i registri, lo stack e il PC. Le parti comuni sono il codice e le parti globali.

Le librerie per i thread possono essere implementate a livello user o a livello kernel.

Dove è immagazzinato il contesto del thread, dove lo si salva? Se è in esecuzione si trova nei registri.

Il thread punta al contesto con un puntatore allo stack e il registro del contesto `switchframe`.

Guarda slide con tutte le funzioni per thread.

Impostazione `.gdbinit`

`thread X` per passare al thread X.

`switchframe_switch`:

Alloco memoria nello stack e vado a salvare i registri (switchframe)

processo

Il processo nella configurazione attuale non ha la lista dei thread, ma ogni thread ha il puntatore al processo.

La enter_new_process prepara tutto nella trapframe.

## System Calls

E' il modo con cui un processo utente può chiedere qualcosa al processo kernel. I programmi, per chiamare un pezzo di kernel devono utilizzare le Application Programming Interface (API). Viene fatta una chiamata alla funzione di più basso livello.

In OS161, è presente una switch che permette di richiamare le varie funzioni di basso livello a seconda della trapframe richiamata. Una sorta di tabella di interrupt.

Per passare i parametri vengono passati in OS161 attraverso una trapframe mettendoli in un blocco, il metodo più semplice è quello con i registri, ma è possibile utilizzare anche lo stack.

In MIPS c'è un solo handler che gestisce system calls, eccezioni e interrupt.

Il trap frame rimane nello stack di kernel.

Ogni eccezione è identificata con un id.

## Memory Management

Il Kernel ha i suoi indirizzi logici sì o no? Due possibilità:

- Utilizzo diretto di indirizzi fisici disabilitando la traduzione degli indirizzi bypassando la MMU.
- Mantieni viva la conversione, ma bisogna andare a definirla.

In Linux si vuole fare in modo che il kernel possa mettere mano alle sezioni di codice del processo utente SOlitamente si vanno a dividere a metà in modo da proteggere la parte kernel, ma permettere al kernel di operare.

L'address apce in DUMB_VM vengono gestiti due segmenti e uno stack.

Per poter avviare un programma vengono utilizzati i file ELF che sono costituiti da header (dimensione dati e codice).

## MEMORIA

> SLIDE annotations memory

## Tool di sincronizzazione

Concorrente (parallelismo basato sul tempo).

Problemi:

- Race Conditions
- Deadlocks
- Resource starvation

Le soluzioni sono:

- Locks
- Barriere
- Semafori

La sezione critica è il pezzo di codice e di dati a cui bisogna fare attenzione, che non può essere eseguita da un processo/thread nello stesso momento.

Si risolve con:

- Mutua Esclusione
- Progress
- Bounded Wait

Ci possono essere due approcci per realizzare le primitive:

- Preemptive: Si può essere interrotti
- Non-preemptive: Non si può essere interrotti (meno problemi di race conditions).

## Algoritmo di Peterson

Assumendo load e store atomiche. Abbiamo una variabile `turn` e un vettore di boolean `flag[2]`.

```c
while(true){
    flag[i] = true;
    turn = j;
    while(flag[j] && turn ==j);
    /* Critical */
    flag[i] = false;
    /* Remainder Section */
}
```

Si può dimostrare che questo funzioni.

Ma in realtà il processore può andare a riordinare le operazioni e questo potrebbe portare le assegnazioni a fare un riordino speculativo.

La soluzione è di inventarci un LOCK, non bastano la scrittura di variabili.

Ma cosa succede se non ho il lock?

- Continuo a provare con spin o busy wait
- Concedo il processore, ma qualcuno dovrà svegliarmi.

L'implementazione ha bisogno di un supporto hardware.

Nel modello a singolo processore si possono rimuovere gli interrupt, ma le soluzioni più serie sono operazioni istruzioni HW, Barriere di memoria o variabili atomiche.

### Memory Barriers

Se le operazioni di scrittura e lettura vengono fatte non vengono ordinate. Deve essere propagato a tutti i processori prima che a questa vi si possa accedere.

### Istruzioni atomiche

#### Test-and-Set

```c
boolean test_and_set (boolean *target){
    boolean rv = *target
    *target = true;
    return rv;
}
```

Deve essere necessariamente atomica, se abbiamo questo allora:

```c
lock = false;

do {
    while(test_and_set(&lock));
    /* Critical */
    lock = false;

    /* Remainder */
} while (true)
```

#### Compare-and-Swap

```c
int compare_and_swap(int *value, int expected, int new_value)  {
    int temp = *value;

    if (*value == expected)
        *value = new_value;
    return temp;
}
```

```c
lock = 0;

while (true) {
    while(compare_and_swap(&lock, 0, 1));
    /* Critical */
    lock = 0;

    /* Remainder */
}
```

### Bounded Waiting

Finchè sono in waiting e la chiave è uguale a 1. Nella sezione critica vado a calcolare il primo waiting dopo di me.

### Atomic Variables

L'incremento o in generale l'aggiornamento deve essere atomico.

## Mutex Locks

E' il tipo di dato più semplice che ci garantisce una protezione della sezione critica., ma la soluzione che richiede busy waiting è lo SPIN LOCK, semplice, ma bisogna limitarne l'uso perchè lo spin lock continua a girare.

TestAndTestAndSet è migliore perchè riduce leggermente il busy waiting.

## Mutex Exclusion using a Semaphore

```c
struct semaphore *s;
s = sem_create("MySem1", 1);

P(s);

/* Critical Section */

V(s);
```

E' uno strumento di sincronizzazione più complesso del mutex. E' una variabile intera a cui viene garantita l'atomicità.

### Versione semplice

####  `wait()`

```c
wait(S) {
    while(S <= 0) ; busy wait

    S--
}
```

#### `signal(S)`

```c
signal(S){
    S++;
}
```

### Versione complessa

Dobbiamo associare al semaforo una coda di attesa. Invece di continuare a fare il test mi metto in attesa, complicherò la vita a chi fa la signal.

Esistono due modelli:

- La wait si ferma con zero
- La wait si ferma ma decrementa

```c
typedef struct {
    int value;
    struct process *list;
} semaphore;
```

Disabilitare l'interrupt può essere utilizzato per fare le operazioni in modo atomico in single core. Nella versione 1.9x tutto funzionava con questa metodologia.

Mettere priorità alta significa significa disabilitare l'interrupt, bassa abilitarlo. Se non dovessi acquisire il semaforo metto il thread in sleep: gli altri non possono farlo, ma io sì.

Se non sono sicuro che sia a zero si può realizzare con:
```c
spl = splhigh(); /* Spl alto */
splx(spl); /* Resetto spl */
```

### Monitor

Oggetto della mutua esclusione. Un'astrazione di alto livello che permette una metodologia conveniente per gestire il processo di sincronizzazione.

### Wait on Condition

Lock per proteggere e Semaforo per segnalare. Ma in questa condizione se vado in attesa sono comunque owner del lock, altri thread non potranno modificare i dati -> Deadlock.

Per risolvere il problema dobbiamo:

- Rilasciare il lock quando si aspetta (NO DEADLOCK) e riprenderlo quando ci si risveglia.
- Bisogna ritestare nuovamente la condizione

### Condition Variables

Fa l'operazione del semaforo (wait e signal) e mi permette di agganciare un lock che posso rilasciare e riprendere nella wait.

Il while è presente perchè ricontrollare la situazione se lo scheduler non garantisce immediatezza.

### OS161

In os161 abbiamo spinlock per fare mutua esclusione veloce. Abbiamo anche i wait channels che implementano le condition variable a livelo del kernel utilizzando lo spin lock.

In OS161 2.0 multicore andiamo ad utilizzare i semafori. 

# Running a User Process

Quando il kernel crea un processo lo fa per avviare un eseguibile. Crea address space e carica il programma e i dati in esso.

Vengono creati con `int common_prog(int nargs, char **args)`.

La fork permette di duplicare l'address space `pid fork(void)`.

La funzione `int execv(const char *program, char**args)` invece va a sostituire l'eseguibile corrente.

## `common_prog`

crea il processo e poi thread fork.

`curthread` e `curproc` sono le strutture dati utilizzate.

`runprogram` crea l'address space e inserisce il programma.

`load_elf`

## System Calls

`tf->tf_v0` è il callno

`tf->tf_aX` è argomento Xesimo

`kgets` ha `\0`

## FS

## File Open/Close

SFT ogni apertura è una riga lì.
