# Modello di Esecuzione

- [Modello di Esecuzione](#Modello-di-Esecuzione)
  - [Livello di astrazione](#Livello-di-astrazione)
  - [Librerie di esecuzione](#Librerie-di-esecuzione)
  - [Modello di esecuzione di C e C++](#Modello-di-esecuzione-di-C-e-C)
  - [Concorrenza](#Concorrenza)
  - [Processi](#Processi)
    - [Modalità di esecuzione](#Modalit%C3%A0-di-esecuzione)
  - [Esecuzione di un programma](#Esecuzione-di-un-programma)
  - [Caricamento del codice eseguibile](#Caricamento-del-codice-eseguibile)
  - [Funzione di avvio](#Funzione-di-avvio)

Ogni linguaggio di programmazione propone un modello di esecuzione, ma tale modello solitamente non corrisponde ad un sistema reale.
E' compito del compilatore di introdurre uno strato di adattamento che implementi il modello (runtime support library).

## Livello di astrazione

Il programma originale viene trasformato in un nuovo programma con un modello di esecuzione più semplice. Deve farlo il compilatore.

## Librerie di esecuzione

Offrono meccanismi di base attraverso funzioni:

- Visibili e con funzionalità standard
- Invisibili al programmatore che sono inserite in fase di compilazione per supportare l'esecuzione.

## Modello di esecuzione di C e C++

I programmi sono pensati come se fossero unici (isolamento). Assume di poter accedere a qualsiasi indirizzo di memoria.

L'esecuzione è principalmente sequenziali e senza limitazioni.
Possiedono un flusso di esecuzione e una struttura a pila per gestire le funzioni annidate tra le funzioni.

## Concorrenza

A partire dal 2011 circa un programma può attivare ulteriori flussi di esecuzione.

## Processi

La libreria di esecuzione non può da sola implementare tutte le astrazioni del modello. Occorre che il sistema operativo crei un sistema di isolamento semi impermeabile.

Per sopperire a questo vengono usati i **processi**. Un processo è il contesto di esecuzione di un programma. Il sistema sovrintende la creazione dei processi.

Quando un processo viene selezionato per essere eseguito, l'OS configura il processore con le relative risorse, ma serve un supporto.

### Modalità di esecuzione

L'OS richiede un supporto HW per distinguere le due modalità di esecuzione:

- Modalità utente: sottoinsieme
- Modalità supervisore: illimitate funzioni basate su memoria e risorse, periferiche, FS e rete.

***See Slides 15***

La System Call è un punto di accesso alla modalità supervisore.

Occorre, in maniera controllata, poter innalzare il privilegio in maniera opportuna per offrire ad un processo le operazioni di cui necessita.

Di base questo avviene attraverso un **interrupt software (trap)** oppure SYSENTER/SYSEXIT che è più efficiente e che richiedono processori più recenti. E' comunque una procedura molto costosa.

## Esecuzione di un programma

Viene creato un nuovo processo, vengono acquisite le risorse necessarie e si inizializzeraà lo stato in maniera opportuna.

Fasi:

- Creazione spazio indirizzamento
- Caricamento eseguibile in memoria
- Caricamento librerie
- Avvio esecuzione

Lo Spazio di indirizzamento permette a due processi di avere una memoria contigua che appare identica ai due processi, ma sono mappate in posti diversi. (grazie MMU).

***See slides 21, 22***

Paginazione e File Mapping dal disco

## Caricamento del codice eseguibile

Il loader si occupa di inizializzare lo spazio di indirizzamento con il contenuto del programma da eseguire. Il programma è suddiviso in sezioni omogenee. Si allocano spazio codice e dati. Vengono create anche tutte le librerie dinamiche necessarie.

Una volta concluso il caricamento si avvia.

## Funzione di avvio

- Configurazione Piattaforma di esecuzione
- Invocazione costruttori globali
- Invocazione Main
- Invocazione distruttori globali
- Terminazione del processo
