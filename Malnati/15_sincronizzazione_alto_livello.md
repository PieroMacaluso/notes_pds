# Sincronizzazione di alto livello

Il basso livello ci permette di operare in condizioni che non permettono l'utlizzo di astrazioni di alto livello. Le cose che dobbiamo fare devono essere poco interconnesse.

Due sotto-compiti sono indipendenti tra loro se la computazione di uno non dipende da quella di un altro e non fanno accesso in scrittura a dati condivisi.

## `std::async()`

Prende come parametro un oggetto chiamabile e poi i parametri. Restituisce un oggetto `std::future` che si potrà utilizzare per capire quando l'operazione sarà eseguita. I metodi esposti sono `wait()`(saprò solo che posso controllare l'esito) e `get()`. Gestisco anche le eccezioni in questo modo.

La funzione chiamabile potrebbe anche essere preceduto dalla politica di lancio.`std::launch::async` invece fa partire subito il thread secondario, `std::launch::deferred` permette di valutare il risultato solo quando ne abbiamo bisogno. Se la politica viene omessa, la gestione viene fatta dal sistema.

Ma creare un thread secondario ha un costo significativo, se le operazioni sono poche conviene eseguirle direttamente.

## `std::future<T>`

E' un contenitore che ci permette di fare accesso al dato ritornato da un'operazione asincrona. Mantiene internamente uno stato condiviso con il codice responsabile della produzione del risultato.

E' possibile utilizzare i metodi `wait()` per poter triggerare il completamento o attenderlo nel tempo. `wait_for` e `wait_until` non forzano l'avvio se il task è lanciato con `deferred`.

Il future può essere distrutto solo se viene completato. L'attività inoltre non è cancellabile a meno che non venga rilasciato un sistema per andarla a terminare esplicitamente.

## `std::shared_future`

Evita data race nell'accesso ad un singolo oggetto da thread multipli. Si può costruire uno shared_future con il movimento. Lo `shared_future` è anche copiabile.

## Differenze tra  `std::thread` e `std::async`

Con thread non abbiamo scelta sulla politica di attivazione. Non abbiamo un sistema di gestione delle eccezioni, ma in async non possiamo avere dati intermedi.

## `std::promise<T>`

Il ponte tra async e future è il `std::promise<T>`. E' un impegno a fornire il dato. Il thread potrà condividerlo con gli altri. E' possibile andare a inserire una `set_value` oppure una `set_exception`.

Nei thread si deve passare la future e muovere la promise.

## Thread distaccati

Se il thread distaccato fa accesso a variabili nel main, potrebbe trovarle distrutte.
Si può usare `std::quick_exit()` per far terminare un programma senza invocare i distruttori delle variabili globali (Può essere un rimedio peggiore del male).

Se il compito dei detached è quello di creare risultati per il thread principale ci possono essere dei problemi. Se il thread fa la get ottiene il risultato e conclude potrebbero esserci ulteriori operazioni che il thread deve svolgere. Per questo motivo vengono fornite delle operazioni come `set_value_at_thread_exit(T val)`  e `set_exception_at_thread_exit(std::exception_ptr p)`

## `std::packaged_task<T(Args...)>`

Astrazione di un'attività in grado di produrre un T a partire da Args. QUando un package task viene eseguito, viene invocata la funzione incapsulata e il risultato prodotto viene utilizzato per valorizzare l'oggetto `std::future` associato.

Si presta per la realizzazione di un **Thread Pool**. Si può stimare il numero di core disponibili attraverso la funzione: `std::thread::hardware_concurrency()`. Se ci sono funzioni di I/O è meglio andare a generare un sacco di thread rispetto al numero di core.

## Lazy Evaluation

Ci sono varie situazioni in cui è utile rimandare la creazione e inizializzazione delle strutture complesse fino a quando non c'è la certezza del loro utilizzo. Un esempio è un Singleton in cui si va ad inizializzare solo nel momento in cui viene chiamato il primo getInstance(). Ma in un contesto concorrente ci sono molti problemi, poichè entrambi potrebbero creare differenti istanze che si diffondono creando problemi. Servirebbe un mutex che dovrebbe essere globale, quindi problema. Per questo motivo C++ ci propone due strutture che sono la classe `std::once_flag` e la funzione`std::call_once()` che registra se qualcuno la sta già chiamando o meno.

### `std::call_once(flag f, ...)`

Esegue f una sola volta, se dal flag non risultano chiamate procede con l'invocazione, altrimenti blocca l'esecuzione in attesa del risultato.

**Esempio slide 36**

Prima di fare compilare tutto dobbiamo farlo passare attraverso il metaobject compiler.

Solitamente si inizia con `QMainWindow` che utilizza 5 aree per poter mettere il menù, la status bar, le toolbar.

Ogni WIdget può avere dei figli che vivono dentro al rettangolo di Widget. I figli vengono passati a `QLayout` che si occupa di andare a porre i figli al posto giusto

- `HBoxLayout`: orizzontale
- `VBoxLayout`: verticale
- `FormLayout`: primo terzo verticale e due terzi 
- `GridLayout`: Righe e colonne
- `StackedLayout`: Uno sopra l'altro.

## Dialog

Focus sulla finestra, vengono filtrati solo i messaggi per il dialog.

## Meta-Object System

Qt mantiene dentro gli oggetti un sistema di meta-informazioni che permettono di farle dialogare.
Basato su tre componenti:
- Classe base `QObject`
- Macro `QOBJECT` che serve per inserire tutti i supporti del MOC
- MOC

Il Moc è un generatore di codice da parte di Qt che permette l'estensione dei concetti di `signal`, `slot` e `properties` abilitando i meccanismi di:
- `type introspection`: Esaminare il tipo, i metodi e le proprietà di un oggetto in fase di esecuzione.
- `reflection`: Va un passo oltre la introspection e consente ad un oggetto di modificare durante l'esecuzione la propria struttura e il proprio comportamento.

`public slot:` possono essere target di messaggi
`signals:` sono metodi la cui implementazione la fa il MOC.

## Gestire l'interazione

Signal e Slot, COnnect permette di connettere Signal e Slot 

## Eventi

Ogni volta che viene chiamata una signal parte la chiamata ad ogni slot in ascolto e, dipende da che chiamata deve fare, il processo potrebbe essere lungo. Se però si trova in un thread diverso? Viene chiamato in maniera asincrona. Possiamo anche farlo se hanno la stessa affinita.

`QEvent` ci permette di modellare gli eventi in maniera asincrona.

**Slide 100** dove ci sono tutte inheritance Eventi.

### Consegna Eventi

GLi eventi sono generalmente emessi dal sistema operativo (`QEvent::spontaneous()==true`), ma possono essere emessi manualmente con `QCoreApplication::sendEvent(QObject receiver, QEvent e)` `QCoreApplication::postEvent(QObject receiver, QEvent e)` (usata di solito).

Tutti i QObject hanno la funzione `event()` che ci permette di analizzare l'evento ricevuto.

`QCoreApplication` gestisce un loop di eventi per le applicazioni Qt senza una Gui (`QGuiApplication` con GUI). Preleva eventi da una coda, li traduce in appropriate sottoclassi di `QEvent` e lo consegna all'oggetto chiamando il metodo `event()`.

In alternativa a event è possibile andare a installare un EventFIlter che applichi una restrizione specifica.

### Eventi ri-disegno

GLi oggetti di tipo `QPaintEvent` vengono consegnati a un widget ogni volta che il sistema ritiene che l'area abbia bisogno di un aggiornamento, questo accade quando il widget è mostrato la prima volta, quando viene spostato, quando un altro contenuto sovrapposto si va a togliere o quando viene richiamato programmaticamente.

Il metodo `paintEvent(..)` ha il compito di gestire qeusto evento creando un oggetto di tipo `QPainter()` 

### Creare un proprio Widget

Bisogna andare a fare l'overloading di molte funzioni ed è possibile anche andare a customizzare `paintEvent(QPaintEvent *event)`.

### Eventi periodici

Si può andare a creare il timeout `startTimer()` fino a `killTimer()`

`QTimer` è migliore.

## Qt Build System

- MOC: Meta Object Compiler
- UIC: User Interface Compiler
- RCC: Resource Compiler

`#include "ui_main.h"` prende il file xml creato per avere tutto.

I file sorgente vengono compilati in oggetto
In header possiamo includere i file uic e i moc. Anche le risorse vengono compilate.

Background con QThread e QtConcurrent (alto livello).