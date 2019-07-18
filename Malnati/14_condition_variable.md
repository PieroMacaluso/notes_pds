# Condition Variable C++11

Spesso un thread deve aspettare uno o più risultati intermedi prodotti da altri thread.

La coppia promise/future permettono di offrire una soluzione limitata al problema: valida solo per singola cosa, non per sequenze di dati.

La tecnica del polling ha limiti:
Consumo calcolo e batteria, ma introduce una latenza che non basta.

La condition variable è una primitiva di sincronizzazione che permette l'attesa di uno o più thread fino a quando non si verifica una notifica, un time out o una **notifica spuria**.
RIchiede l'utilizzo di un `std::unique_lock<Lockable>`

## `std::condition_variable`

Offre il metodo `wait(unique_lock)` per bloccare l'esecuzione del thread fino a quando non giunge una notifica (`notify_one()` e `notify_all()`) senza consumare cicli di CPU.

E' importante andarsi a proteggere dalle notifiche spurie.

Internamente ha una lista di thread in attesa della condizione. Quando un thread esegue il metodo `wait()` viene sospeso e aggiunto alla lista.

Quando sono eseguiti i metodi `notify_one()` o `notify_all()` uno o tutti i thread vengono risvegliati.

La wait ha una doppia porta e solo quando entrambe vengono superate il metodo ritorna:
- Attesa segnalazione
- Riacquisizione del Mutex. 

**Slide 15-17**

Per proteggerci dalle notifiche spurie possiamo andare a usare una lambda exception per verificare che ci siano le condizioni per uscire. Deve ritornare un boolean

**Slide 21**

## Operazioni atomiche

La classe `std::atomic<T>` permette di accedere in modo atomico all'oggetto di tipo T attraverso funzioni tipo `load()` e `store()`, ma anche operazioni di incremento.