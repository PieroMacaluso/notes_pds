# File-System Interface

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