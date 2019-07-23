# Thread

I thread sono stati introdotti con C++11 diventandone parte integrante. Nelle versioni successive si sono andati a inserire un nuovo tipo di mutex.

La classe introduce un livello di astrazione per facilitare il programmatore.

Due approcci:

- Alto livello con `std::async` e `std::future`
- Basso livello che chiede l'uso esplicito di thread e costrutti di sincronizzazione.

## Classe `std::thread`

Modella un oggetto che rappresenta un singolo thread di esecuzione del sistema operativo.

L'esecuzione avviene immediatamente dopo la costruzione dell'oggetto thread associato tranne per eventuali ritardi con lo scheduling.

`t.join()` il flusso si blocca fino a che il thread termina.

Il costruttore di `thread` riceve un oggetto chiamabile e opzionalmente una serie di parametri. I parametri sono inoltrati attraverso la funzione `std::forward` che mette a disposizione del destinatario un riferimento al dato originale.

I parametri passati per riferimento dobbiamo andare a garantire che il parametro abbia un ciclo di vita più grande di quello del thread. Prima di poter andare ad analizzare il contenuto devo aspettare la fine del thread. e.g. `std::ref(res1)`

Spesso vengono utilizzate le funzioni lambda

Se creiamo il thread possiamo:

- `join()` aspettiamo la conclusione
- `detach()` rinunciamo ad essere interessati a lui
- traferire per movimento

Se non avviene nessuna di queste azioni, viene evocata la funzione `std::terminate()`. Tutti i thread principali e secondari termineranno improvvisamente senza possibilità di effettuare nessuna forma di salvataggio.

Non bisogna dimenticarsi mai di andare a richiamare `join()` o `detach()` anche in caso di eccezione (RAII).

### Problemi dei thread

#### Restituire un risultato

La soluzione richiede l'utilizzo di una variabile condivisa, ma coma fanno a sapere gli altri thread che il contenuto della variabile è stato aggiornato? La prima soluzione potrebbe essere quella di un flag, ma anche in questo caso è una variabile condivisa.

#### Gestire i thread

Se durante la computazione del thread si verifica un'eccezione non recuperata da un catch, l'intero programma termina. Ma comunque gli altri thread non possono intercettare il comportamento

#### Accedere a dati condivisi

In assenza di sincronizzazione si potrebbe avere interferenza. Bisogna fare in modo che questi dati vengano racchiusi in un costrutto di sincronizzazione.

## Costrutti di sincronizzazione

### `std::mutex`

Gli oggetti in questa classe permettono l'accesso controllato a porzioni di codice un solo thread alla volta. Si farà riferimento allo stesso mutex.

Metodi: `lock()` e `unlock()`.

Il lock si blocca fino a quando non si libera il mutex, se ci sono più oggetti in attesa si andrà a sceglierne uno a caso.

Il mutex non è ricorsivo.

`std::recursive_mutex` può essere acquisito più volte consecutivamente dallo stesso thread.

`std::timed_mutex` aggiunge i metodi `try_lock_for()` e `try_lock_until()`

Anche classe `std::recursive_timed_mutex`.

Esiste una classe chiamata `std::lock_guard<Lockable>` che sfrutta il paradigma RAII ed ha solo costruttore, sia distruttore. In ogni caso viene liberata la memoria.

E' possibile utilizzare il blocco condizionale con `try_lock()`. Il costrutto `std::lock_guard` è adottabile, aggiungendo il flag `std::adopt_lock` permette di riutilizzare il lock_guard.

Molto spesso bisogna mantenere uno o più lock, possiamo utilizzare `std::scoped_lock`. In situazioni del genere è possibile andare a prendere i lock in ordine lessicografico (ordine di indirizzo).

`std::unique_lock<Lockable>` estende il lock_guard aggiungendo `lock` e `unlock` (lo devo comunque riprenderlo per farlo liberare al distruttore.

`std::shared_mutex` (C++17) posso usare shared_lock, 
due forme di accesso:

- Lock for Read (Condiviso)
- Lock for Write (Esclusivo)

> **Importante slide 42**

## `std::atomic<T>`

Le normali operazioni di accesso in lettura e scrittura non hanno dessuna garanzia sulla visibilità delle operazioni che sono eseguite in parallelo da più thread. I processori hanno alcune istruzioni specializzati per permettere l'accesso atomico al singolo valore.

Ci viene offerta la classe `atd::atomic<T>`, si garantisce che questo venga letto, scritto e modificato in modo osservabile nell'ordine in cui avvengono. Non possono essere riordinate le operazioni dal processore (spin-lock). Abbiamo due funzioni `load` e `store`. Crea una barriera che permette di poter far persistere le modifiche. Sul boolean possiamo andare anche ad utilizzare la `exchange(T t)`, ma anche `fetch_add(val)` `fetch_sub(val)` o `operator++` anche `operator--`sugli int.

## `std::this_thread`

Permette di accedere ai dati dell'ID.

`sleep_for(duration)` sospende l'esecuzione, per almeno il thread corrente il thread non viene eseguito.

`yield()` offre al listema la possibilità di rischedulare l'esecuzione del thread.