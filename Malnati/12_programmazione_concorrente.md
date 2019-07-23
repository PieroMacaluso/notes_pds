# Programmazione Concorrente

- [Programmazione Concorrente](#Programmazione-Concorrente)
  - [Generalità sulla programmazione concorrente](#Generalit%C3%A0-sulla-programmazione-concorrente)
    - [Vantaggi](#Vantaggi)
    - [Svantaggi](#Svantaggi)
  - [Complessità](#Complessit%C3%A0)
  - [Modello di memoria del C++](#Modello-di-memoria-del-C)
  - [Problemi aperti](#Problemi-aperti)
    - [Errori](#Errori)
  - [Interferenza](#Interferenza)

> Thread nativi Windows e Linux non saranno trattati poichè ci concentreremo sull'implementazione fornita da C++. Ci nasconde i dettagli esterni proponendoci un'interfaccia astratta.

## Generalità sulla programmazione concorrente

Un programma concorrente è un programma che ha due o più flussi di esecuzione paralleli contemporanei nello stesso spazio di indirizzamento per perseguire un obiettivo comune.

All'atto di creazione di un processo, questo ha un solo flusso di elaborazione (**thread principale**), da questo thread è possibile creare altri thread: ciscun thread potrà continuare a produrne fino a quanto consente la nostra applicazione.

Si possono gestire operazioni con tempistiche molto più accettabili. Il sistema operativo e le librerie di supporto andranno ad allocare le risorse necessarie. Lo **scheduler** è il componente che ripartisce l'utilizzo dei core disponibili tra i diversi thread in modo **non deterministico**. Ogni thread riceverà per un certo periodo per poter eeguire l'applicazione. Il sistema cerca in tutti i modi di gestire il tempo per tutti, ma non è obbligatorio che impieghi lo stesso tempo per tutti

### Vantaggi

I vantaggi della concorrenza sono:

- Sovrapposizione tra computazione e operazioni di I/O;
- Riduzione sovraccarico dovuto alla comunicazione dei processi;
  - Esempio: i puntatori non potevano essere passati da un processo all'altro poichè possedevano spazi di indirizzamento diversi. Questo necessita di una modalità che fosse indipendente dallo spazio di memoria andando a creare un grosso overhead. Due thread che condividono lo stesso processo possono passarsi in maniera più facile i dati.
- Utilizzo di CPU multicore permette di fare un vero parallelismo riducendo il tempo di elaborazione.

### Svantaggi

- Aumento significativo della complessità del programma
  - Nuove fonti e tipologie di errore
  - Non determinismo dell'esecuzione

## Complessità

La memoria non può essere pensata come un deposito *statico* nel momento in cui esistono numerosi thread che possono andare a scrivere nella stessa variabile di memoria.
Per far sì che questo non capiti è importante andare a coordinare l'**accesso alla memoria** tramite opportuni costrutti di sincronizzaione. La presenza della cache dei core introduce ulteriore non determinismo nell'ordinamento e nella visibilità delle singole azioni sulla memoria. La cache non è più ignorabile.

Scrivere un programma concorrente ha un approccio molto più vicino alla dimostrazione di un teorema che la scrittura di un semplice programma. E' molto più complicato poichè è difficilmente debuggabile.

## Modello di memoria del C++

Quando un thread legge il contenuto di una locazione di memoria potrebbe trovare:

- Il valore iniziale contenuto nell'eseguibile mappato in memoria
- Il valore che lo stesso thread ha depositato all'interno della locazione precedentemente.
- Valore lasciato da un altro thread.

Come già detto la presenza di una **cache hardware** e il possibile riordinamento delle istruzioni contribuiscono a rendere il terzo caso problematico. Dobbiamo andare a inserire degli operatori che ci possano garantire l'ordine.

## Problemi aperti

- Atomicità: quali istruzioni devono avere effetti indivisibili. Il problema principale è da localizzare nella variabili globali e su quelle istanza.
- Visibilità: sotto quali condizioni le scritture compiute in un thread possono essere visibile in un altro thread?
- Ordinamento: Sotto quali condizioni gli effetti di più operazioni possono apparire in ordine diverso su altri thread?

### Errori

L'uso superficiale dei costrutti di sincronizzazioni porta a blocchi passivi o attivi di un programma o possono portare a malfunzionamenti casuali. Questi ultimi in particolare sono più importanti poichè il comportamento non deterministico lo rendono difficilmente ripetibile.

Ogni thread dispone di :

- Proprio stack (Exception Pointer, Stack Pointer, Instruction Pointer)
- Proprio contatore all'ultimo contesto per eccezioni
- Stato del proprio processore virtuale

I thread condividono:

- Variabili globali
- Area del codice
- Area delle costanti
- Heap

> SLIDE 11 DISEGNO

Se più thread sono in esecuzione non è possibile fare assunzioni sulle velocità relative di avanzamento. L'esecuzione ripetuta può portare a risultati diversi.

## Interferenza

L'interferenza è un fenomeno che si manifesta con malfunzionamenti completamente casuali dovuti ad accessi simultanei di thread al medesimo dato senza alcun tipo di ptoezione.

Libreria C++ `#include <thread>`.

Come mai nell'esempio in C++ abbiamo anche differenze negative?

```c++
#include <iostream>
#include <thread>

int a = 0;

void f() {
    while (a >= 0) {
        int before = a;
        a++;
        int after = a;
        if (after-before!=1) {
            std::string s;
            s = std::to_string(before) + " -> " + std::to_string(after) + "(" + std::to_string(after-before) + ")";
            std::cout << s << std::endl;
        }
    }
}

int main() {
    std::thread t1(f);
    std::thread t2(f);
    t1.join();
    t2.join();
    return 0;
}
```
Quando più processi fanno read/modify/write che comporta malfunzionamenti casuali molto difficili da identificare. Il problema è `a++;` che è una operazione che contiene due operazioni in cascata.

E' necessario andare a permettere l'accesso a più thread nelle zone **critiche**.
