# Ereditarietà e Polimorfismo

La sottoclasse specializza il comportamento della superclasse. Una classe può ereditare da una o più classi (più si può fare, ma meglio di no). 

Polimorfismo, tutto dipende da come vengono implementati e dichiarati i metodi nella classe base.

Il polimorfismo viene messo solo se lo richiediamo, in slide 8 restituisce 1! Davanti al metodo che vogliamo sia polimorfico dobbiamo scrivere `virtual`. Le virtual astratte vanno dichiarate con ` = 0` Una classe astratta se contiene almeno una funzione virtuale astratta.

Esistono anche i distruttori virtuali.

## Polimorfismo

Come è possibile che uno ritorni 1 e uno ritorni 2? Tutto può essere fattibile grazie alla **V-Table**. Per i metodi virtuali si aggiunge un livello di indirezione attraverso un campo nascosto che viene chiamato V-Table. Nell'esempio tutte le v-table hanno lunghezza 1. La chiamata virtuale costa un pochetto di più poichè dobbiamo andare a dereferenziare il puntatore e accedere. I singoli oggetti diventano un pochetto più grandi.

Il polimorfismo lo abbiamo solo nei metodi virtual. La V-Table è una per la classe particolare.

Una classe derivata può avere più di un padre. Espone tutti i metodi di tutti i genitori, non deve derivare dalla stessa cosa, se proprio vogliamo creare questa cosa a diamante allora dobbiamo inserire virtual nel nella ereditarietà.

## TypeCast

C++ supporta il type-cast come C, ma l'ereditarietà o rende pericoloso!

In C++ abbiamo operatori per il type-cast più efficiente
- `static_cast<T>`: semplice, voglio trasformarlo in quello che metto tra parentesi acute. Se c'è un costruttore che lo costruisce a partire da quell'elemento. Se voglio evitare questo costruttore venga utilizzato per cast metto `explicit`. Può essere anche nella classe dell'altro con il metodo `to()`. Il controllo non viene fatto, controlla solo che sia lecita. Conversione senza overhead, ma risultato non predicibile. Solitamente lo usiamo per conversioni in verticale. Cambia  i puntatori. Verso l'alto è possibile, ma crea problemi.
- `dynamic_cast<T>`: ereditarietà multipla come static cast, ma fa il controllo runtime ci garantisce che l'operazione sia buona, ma aggiunge RTI (Runtime Type Information). Ritorna 0 se il cast non è valido. Preferibilmente verso l'alto
- `reinterpret_cast<T>`: tipo quello del C, pericoloso, ma talvolta necessario a basso livello.
- `const_cast<T>`: Una variabile const è una variabile in cui non si può modificare il contenuto. Viene usato quando vogliamo togliere il const.
