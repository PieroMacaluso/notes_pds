# File-System Interface

- [File-System Interface](#file-system-interface)
  - [File](#file)
  - [Directory](#directory)
    - [Organizzazione Direttori](#organizzazione-direttori)
      - [Single-Level Directory](#single-level-directory)
      - [Two-Level Directory](#two-level-directory)
      - [Tree-Structured Directories](#tree-structured-directories)
      - [Acyclic-Graph Directories](#acyclic-graph-directories)
  - [FS Mounting System](#fs-mounting-system)
  - [File Sharing](#file-sharing)
  - [Protection](#protection)

Se non hai ancora esperienza di OS, vediamo cosa è necessario avere dai file. Cosa deve fare il file System? Cosa chiede l'utente.

## File

Un file è uno spazio di indirizzamento contiguo non in RAM, ma su disco. LOGICI. Contiene numeri, caratteri o eseguibile.

Tipicamente ha nome, identificatore, etc...
Il file è un tipo di dato astratto.
La open porta in RAM dati che posso utilizzare per manipolare il dato.

I file aperti sono contenuti all'interno di **open-file table** che ne tiene traccia.

- Shared Lock e Exclusive Lock

Possono essere anche blocchi parziali.

- Accesso sequenziale al file: si inizia a scriverlo con un cursore che procede in avanti. E' implicito.
- Accesso diretto: posiziona e leggi next.

Ci sono altri tipi di accesso più complessi come cosa accade con i database (indici). Anche se sono ordinati non è detto che siano grandi uguali. Basta creare in parallelo un file di indici che contiene i puntatori, questo permette ricerca dicotomica.

## Directory

Un direttorio su disco è una specie di file con i puntatori ai file.

Un disco può essere diviso in partizioni.

Solitamente si utilizzano FS general-purpose, ma in realtà ce ne sono di tipi diversi.

Un direttorio deve essere efficiente, garantire una nominazione comoda.

### Organizzazione Direttori

#### Single-Level Directory

Una singola cartella per tutti gli user

#### Two-Level Directory

Direttorio separato per ogni user. Path Name. Avendo due livelli è possibile andare a riutilizzare lo stesso nome.

#### Tree-Structured Directories

Direttorio organizzato ad albero che permette una ricerca più efficiente.

#### Acyclic-Graph Directories

Grafo diretto aciclico. Il ciclo è possibile solo attraverso le cartelle e non deve esserci poichè non ha senso, si perde la cognizione di contenente e contenuto. Per farlo basta andare a permettere link solo verso file e non sottodirectory.

Come si può fare il DAG?

Link simbolico è inserito in una tabella di corrispondenza di nomi.
Ma cosa succede se devo eliminare il file con i due contatori evitando il Dangling Pointer?

- Puntatori all'indietro.
- DaisyChain, strategia che mi permette di riconoscerli e di cancellare tuti archi entranti.
- Si cancella l'arco, ma non il file. Il file rimane fino a quando non sono state eliminate tutti gli archi.

Come garantisco che non siano presenti cicli?

- Permetto di evitare condividere un direttorio
- Garbage Collection, permetto cicli, ma ogni tanto viene fatta una passata e vengono rimossi.
- Ogni volta che sia ggiunge un link si va a ricercare il ciclo.

## FS Mounting System

Un file system può essere utilizzato solo se viene montato.

## File Sharing

Su un sistema abbiamo tre livelli di utente. Il proprietario, il gruppo e l'ID user.

## Protection

Read Write Execute. RWX (owner, group, public)

Append Delete List (e.g. website).
