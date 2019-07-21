# Ereditarietà e Polimorfismo

La sottoclasse specializza il comportamento della superclasse. Una classe può ereditare da una o più classi (più si può fare, ma meglio di no). 

Polimorfismo, tutto dipende da come vengono implementati e dichiarati i metodi nella classe base.

Il polimorfismo viene messo solo se lo richiediamo, in slide 8 restituisce 1! Davanti al metodo che vogliamo sia polimorfico dobbiamo scrivere `virtual`. Le virtual astratte vanno dichiarate con ` = 0` Una classe astratta se contiene almeno una funzione virtuale astratta.

In caso di polimorfismo anche il distruttore deve essere dichiarato virtuale per fare in modo che venga chiamato quello corretto.

## Polimorfismo

Il Polimorfismo è possibile grazie all'utilizzo delle **V-Table**. Per i metodi virtuali si aggiunge un livello di indirezione attraverso un campo nascosto che viene chiamato V-Table. Le V-Table contengono tante righe quanti sono i metodi virtuali dell'oggetto creato. Ognuno di questi punterà all'effettiva implementazione del metodo virtuale. La chiamata virtuale costa un pochetto di più poichè dobbiamo andare a dereferenziare il puntatore e accedere. I singoli oggetti diventano un po' più grandi.

Il polimorfismo lo abbiamo solo nei metodi virtual. La V-Table è una per la classe particolare.

Una classe derivata può avere più di un padre. Espone tutti i metodi di tutti i genitori, non deve derivare dalla stessa cosa, se proprio vogliamo creare questa cosa a diamante allora dobbiamo inserire virtual nella ereditarietà.

## Cast

### TypeCast

C++ supporta il type-cast come C, ma l'ereditarietà o rende pericoloso!

In C++ abbiamo operatori per il type-cast più efficiente

### `static_cast<T>`

Metodo più semplice per convertire un oggetto in T. Questo avviene se c'è un **costruttore** che lo costruisce a partire da quell'elemento oppure il metodo `to()` nella classe T. Se voglio evitare questo costruttore venga utilizzato per cast metto `explicit`.

Non viene fatto alcun controllo, controlla solo che sia lecita, per questo motivo è una conversione senza overhead e quindi più efficiente. Si limita ad utilizzare la conversione così come è conosciuta dal compilatore portando a risultati non predicibili. Solitamente lo usiamo per conversioni da figlio a padre (upcast). Cambia i puntatori. 

Da padre verso figlio (downcast) è possibile, ma crea problemi e risultati impredicibili: il compilatore penserà di avere tutti i campi della classe figlia, ma in realtà vedrà solo quelli della classe padre.

Una classe derivata castata ad un suo parente potrà accedere solo ai metodi della classe radice, ma per noi potrebbe accedere a tutti. (?)

### `dynamic_cast<T>`

Ereditarietà multipla come static cast, ma fa il controllo runtime ci garantisce che l'operazione sia buona, ma aggiunge RTTI (Run-Time Type Information) il quale verifica se l'oggetto di partenza non è compatibile con quello di arrivo, in tale caso il dynamic_cast restituisce `null`/0 evitando così di accedere alla memoria altrui. Può essere utilizzato per downcast e upcast, ma sempre preferibilmente verso l'alto.

In caso di downcast, il compilatore andrà a verificare che le V-Table per verificare che questo downcast possa essere effettuato.

Se viene applicato ad un riferimento non valido genererà eccezione.

### `reinterpret_cast<T>`

Ricorda lo stati cast C, pericoloso, ma talvolta necessario a basso livello. Viene infatti utilizzato prevalentemente per dati ritornati dal sistema operativo.

### `const_cast<T>`

Una variabile const è una variabile in cui non si può modificare il contenuto. Viene usato quando vogliamo togliere il const, ma può generare problemi se usato su variabili globali poste in sola lettura dal compilatore.
