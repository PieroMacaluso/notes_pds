# Eccezioni

- [Eccezioni](#Eccezioni)
  - [Errori Attesi](#Errori-Attesi)
  - [Errori non attesi](#Errori-non-attesi)
  - [Le eccezioni](#Le-eccezioni)
  - [Strategie di Gestione](#Strategie-di-Gestione)
  - [RAII](#RAII)

Non tutte le operazioni che facciamo hanno successo, molto spesso molte cose sono ascrivibili a programmatore, cliente o sistema.

Possibili fallimenti possono essere: saturazione della memoria, disco pieno, file assente, accesso negato, malfunzionamento hardware, rete inaccessibile.

## Errori Attesi

Gli errori attesi bisogna verificare che siano presenti o meno. Principalmente ce lo dimentichiamo, ma spesso l'errore viene rilevato in una funzione che non sa come trattarlo e occorre propagarlo all'indietro.

## Errori non attesi

Gli errori non attesi sono molto più difficili da gestire ed è molto improbabile che si possano gestire nel posto in cui sono verificati.

Spesso vengono usati come ritorno dei valori convenzionali, ma spesso non basta!

## Le eccezioni

Per questo motivo ci vengono in aiuto le eccezioni. Meccanismo che permette di trasferire il flusso di esecuzione del codice dove si trova l'errore a dove può essere gestito, solitamente passato al chiamante finchè serve e finchè non si trova chi può gestirlo. Lo stack viene contratto. Solitamente si prende un oggetto qualsiasi (meglio `std::exception`) che ci permette di esprimere il disastro. Si usa l'istruzione `throw` per poter lanciare l'eccezione. Non deve andare oltre, ma deve procedere con la ricerca di una contromisura. Se sopra c'è un blocco try allora posso gestire, altrimenti continuo la contrazione. Dopodichè guardo i vari catch. Il problema non è risolto, ci permette di avere una segnalazione, di gestirla, ma poi dobbiamo ragionarci noi.

```c++
try{
    // ...
}
catch(exception& e) {
    // ...
}
```

Di base nella classe exception c'è solo una stringa e posso capire cosa è contenuto con `what()`, non serve per farci degli if, ma sono per avere l'opportunità di fare una catch.

RAII (Resouce Acquisition is Initialization)

La sintassi `...` nel catch indica una qualsiasi eccezione.

L'istruzione `throw` al termine del blocco catch rilancia l'eccezione catturata.

Le eccezioni lanciate costano, il costo è molto elevato.

## Strategie di Gestione

- Terminare in modo ordinato il programma
- Ritentare esecuzione.
- Registrare messaggio in log e rilanciare eccezione.

## RAII

Resource Acquisition is Initialization.  Paradigma garantisce che le risorse siano acquisite alla costruzione e rilasciate dai distruttori.