# Piattaforme di esecuzione

- [Piattaforme di esecuzione](#Piattaforme-di-esecuzione)
  - [API - Application Programming Interface](#API---Application-Programming-Interface)
  - [ABI - Application Binary Interface](#ABI---Application-Binary-Interface)
  - [Gestione degli Errori](#Gestione-degli-Errori)
    - [Windows](#Windows)
    - [Linux](#Linux)
  - [Uso delle stringhe](#Uso-delle-stringhe)
    - [Windows](#Windows-1)
    - [Linux](#Linux-1)
  - [Standard POSIX](#Standard-POSIX)

L'esecuzione di una applicazione avviene in un contesto di Sistema Operativo, offrendo servizi, funzionalità e convenzioni che permettono il funzionamento.
Tali servizi devono perciò conformarsi alle specifiche dell'OS.

## API - Application Programming Interface

Insieme di funzioni e strutture che vengono proposte così come sono al programmatore. Le API offerte dai sistemi operativi (System Call) sono prevalentemente annegate in funzioni di libreria.

## ABI - Application Binary Interface

Definisce quale formato debba avere il software per poter essere compatibile con l'OS. Comprende ad esempio come si passano i parametri, come vengono utilizzati i registri del processore, innalzamento del privilegio, collegamento tra moduli.

E' supportata da Compilatore, Linker, Debugger, Profiler e Inspector.

Per poter sviluppare un programma è necessario andare a capire come interfacciarsi con il sistema operativo. Per accedere alle PAI è importante dichiarare le funzioni e definire i tipi di parametri o collegando librerie specifiche.

Al proprio interno l'OS mantiene un insieme di strutture dati che descrivono lo stato del sistema che **NON** sono esposte al programmatore, ma le API permettono di accedervi in modo indiretto detti **HANDLE** in Windows e **FileDescriptor** in Linux.

## Gestione degli Errori

Tutte le invocazioni possono avere successo o fallire ed è importante verificarne sempre l'esito.

### Windows

Ci si basa sul tipo di ritorno. Bisogna andarlo subito a controllare attraverso la funzione `DWORD GetLastError()` poichè ulteriori chiamate andrebbero a nascondere il tipo precedente. Il valore che viene ritornato è un numero a 32 bit.

### Linux

Si può accedere al valore effettivo della variabile con `#define errno(*__errno_location())`. Perchè nasce in ambito senza thread, ma poi con l'evoluzione si è dovuto trovare una soluzione.

***See slides at 15***

## Uso delle stringhe

8 bit è inadatto per rappresentare tutti i caratteri dell'alfabeto.

Il consorzio Unicode ha poi definito degli standard che richiede almeno 21 bit. Possono avere ordinamenti differenti (Big endian, Little endian).

### Windows

- Due tipi base
  - char: 8bit, codifica ASCII estesa (API suffisso -A)
  - wchar_t: 16 bit UNICODE (API suffisso -W)
- Tipo Generico:
  - TCHAR: definito in tchar.h
- Tipi derivati

### Linux

GCC interpreta direttamente i non-ASCII con UTF-8. `strlen()` non andrà più a descrivere il numero di caratteri effettivamente presenti, ma solo il numero di byte nulli.

Il tipo `wchar_t` esiste, ma lunghezza 4 byte. Inserire `L` prima della costante stringa.

## Standard POSIX

POSIX (Portable Operating System Interface) è un modello concettuale di API a basso livello per rendere i programmi portabili. Linux e MAC sono conformi, Android in parte WIndows no!

Le assunzioni riguardano:

- Struttura File System
- Uso di Handle
- Uso sottoinsiemi di caratteri portabili
- Gestione errori
- Aritmetica e rappresentazione dei numeri
- Thread e processi

C e C++ Libraries sono state definite sulla base di questi standard.