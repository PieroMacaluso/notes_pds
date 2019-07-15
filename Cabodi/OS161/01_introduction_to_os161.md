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

E' il modo con cui un processo utente può chiedere qualcosa al processo utente. I programmi, per chiamare un pezzo di kernel devono utilizzare le Application Programming Interface (API). Viene fatta una chiamata alla funzione di più basso livello.

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