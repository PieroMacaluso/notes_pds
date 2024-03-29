# PDS - Temi di programmazione Malnati

## Table of Contents

- [PDS - Temi di programmazione Malnati](#pds---temi-di-programmazione-malnati)
  - [Table of Contents](#table-of-contents)
  - [Sestupla 2016-01-26](#sestupla-2016-01-26)
    - [Testo](#testo)
    - [Implementazione](#implementazione)
  - [Phaser 2015-09-11](#phaser-2015-09-11)
    - [Testo](#testo-1)
    - [Codice](#codice)
  - [Classifica 2015-07-15](#classifica-2015-07-15)
    - [Testo](#testo-2)
    - [Implementazione](#implementazione-1)
  - [Exchanger 2016-07-01](#exchanger-2016-07-01)
    - [Testo](#testo-3)
    - [Implementazione](#implementazione-2)
  - [Looper](#looper)
    - [Testo](#testo-4)
    - [Implementazione](#implementazione-3)
  - [Dispatcher 2016-07-13](#dispatcher-2016-07-13)
    - [Testo](#testo-5)
    - [Implementazione](#implementazione-4)
  - [Stack 2011-07-26](#stack-2011-07-26)
    - [Testo](#testo-6)
    - [Implementazione](#implementazione-5)
  - [PubSubHub 2014-06-25](#pubsubhub-2014-06-25)
    - [Testo](#testo-7)
    - [Implementazione](#implementazione-6)
  - [Singleton 2019-01-XX](#singleton-2019-01-xx)
    - [Testo](#testo-8)
    - [Implementazione](#implementazione-7)
      - [Singleton.h](#singletonh)
      - [Singleton.cpp](#singletoncpp)
      - [main.cpp](#maincpp)
  - [Barriera Ciclica 2019-07-05](#barriera-ciclica-2019-07-05)
    - [Testo](#testo-9)
    - [Implementazione](#implementazione-8)

## Sestupla 2016-01-26

[Back to top](#Table-of-Contents)


### Testo

Implementare una classe con 7 thread: 4 produttori in ascolto che ricevono 6 parametri (latitudine di partenza, longitudine di partenza, latitudine di arrivo, longitudine di arrivo, timestamp inizio, timestamp fine) e 3 consumatori che aggregano le informazioni (luogo di partenza, luogo di arrivo, durata). Quando uno dei 4 produttori riceve una sestupla di parametri, li inserisce in una struttura condivisa (in modo safe) e chiama TUTTI i thread consumatori, i quali (in modo safe) aggregano le informazioni prendendo i parametri dalla struttura condivisa. (4 pt.)

### Implementazione

> Al posto di usare la sestupla ho messo una semplice `value`
```c++
class Sestupla{
public:
    int value;
    Sestupla(int v){
        value = v;
    }
};

class Aggregato {
    int value;
public:
    Aggregato(Sestupla s){
        value = s.value + 10;
    }
};

class Geo {
    std::array<std::thread, N_PRO> producers;
    std::array<std::thread, N_CON> consumers;
    std::queue<Sestupla> queue;
    std::vector<Aggregato> endVector;
    std::mutex m;
    std::condition_variable cvNew;
    std::condition_variable cvQueue;
    int value;
    bool wait;

public:
    Geo() {
        wait = false;
        for (int i = 0; i < N_PRO; i++) {
            producers[i] = std::thread([&](){
                    std::unique_lock lk{m};
                    cvNew.wait(lk, [&]() { return wait ; });
                    Sestupla s{value};
                    queue.push(s);
                    wait = false;
                    cvNew.notify_all();
                    cvQueue.notify_all();
            });
        }
        for (int i = 0; i < N_CON; i++) {
            consumers[i] = std::thread([&](){
                    std::unique_lock lk{m};
                    cvQueue.wait(lk, [&]() { return !queue.empty(); });
                    Sestupla s = queue.front();
                    queue.pop();
                    Aggregato a(s);
                    endVector.push_back(a);
                    cvQueue.notify_all();
            });
        }
    }

    ~Geo(){
        for (int i = 0; i < N_PRO; i++) {
            producers[i].join();
        }
        for (int i = 0; i < N_CON; i++) {
            consumers[i].join();
        }
    }

    void insertSestupla(int v){
        std::unique_lock lk {m};
        cvNew.wait(lk, [&](){return !wait;});
        value = v;
        wait = true;
        cvNew.notify_all();
    }

};


int main() {
    Geo geo{};

    geo.insertSestupla(1);
    geo.insertSestupla(3);
    geo.insertSestupla(40);
    geo.insertSestupla(50);

    return 0;
}
```

## Phaser 2015-09-11

[Back to top](#Table-of-Contents)


### Testo

Classe Phaser, con metodi void attendi(), void aggiungi(), void rimuovi(). La classe permette a N-1 thread di attendersi tra loro, quando sono N-1 si sbloccano.
I metodi aggiungi e rimuovi modificano il valore di N (specificato nella costruzione dell'oggetto) e di conseguenza sbloccano i thread se necessario.

### Codice

```c++
#include <thread>
#include <mutex>
#include <condition_variable>

class Phaser {
    std::mutex m;
    std::condition_variable cv;
    unsigned n_max;
    unsigned n_th;
    bool execution;
public:
    Phaser (unsigned N): n_max(N){
        execution = false;
    }

    void aggiungi(unsigned delta) {
        std::lock_guard lk{m};
        n_max = n_Max + delta;
    }

    void rimuovi(unsigned delta){
        std::lock_guard lk{m};
        if (n_max - delta < 0) nmax = 0;
        if (n_th >= n_max) {
            execution = true;
            cv.notify_all();
        }
    }

    void attendi(){
        std::unique_lock lk{m};
        n_th++;
        if (n_th == n_max){
            execution = true;
            cv.notify_all();
        } else {
            cv.wait(lk, [&](){
                return execution;
            });
        }
        n_th--;
        if (n_th == 0) execution = false;

    }

}

#define N 10

int main() {
    Phaser phaser{10};
    std::vector<std::thread> workers;
    for (int i = 0; i < N; i++) {
        workers.emplace_back(std::thread([&, i](){
            phaser.attendi();
        }));
    }

    for (auto &w : workers){
        w.join();
    }
    return 0;
}
```

## Classifica 2015-07-15

[Back to top](#Table-of-Contents)


### Testo

Una classe concorrente che gestisse la classifica di un gioco online: ogni concorrente è identificato da un nick e ha un punteggio associato. Un metodo ritornava una copia della classifica, un altro inseriva un nuovo punteggio relativo a un concorrente (inserendo un nuovo concorrente in classifica se non esistente o aggiornando il punteggio se maggiore di quello vecchio), un metodo si metteva in attesa di variazioni sulla classifica.

### Implementazione

> Non è chiaro il testo e quindi l'implementazione potrebbe essere errata.

```c++
#include <mutex>
#include <condition_variable>
#include <map>

class Classifica{
    std::mutex m;
    std::condition_variable cv;
    std::map<std::string, int> classifica;
    std::pair<std::string, int> nuovaVariazione;
    bool variazione;

public:
    Classifica(): variazione(false){};

    std::map<std::string, int> getClassifica(){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return !variazione;});
        return classifica;
    }

    void addPunteggio(std::string nickname, int score){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return !variazione;});
        auto it = classifica.find(nickname);
        if (it != classifica.end()){
            if (score > it->second){
                nuovaVariazione = std::make_pair(nickname, score);
                classifica.insert(nuovaVariazione);
                variazione = true;
                cv.notify_all();
            }
        } else {
             nuovaVariazione = std::make_pair(nickname, score);
            classifica.insert(nuovaVariazione);
            variazione = true;
            cv.notify_all();
        }
    }

    std::pair<std::string, int> listenVariations(){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return variazione;});
        variazione = false;
        return nuovaVariazione;
    }

}
```

## Exchanger 2016-07-01

[Back to top](#Table-of-Contents)


### Testo

La classe generica Exchanger<T> permette a due thread di scambiarsi un valore di tipo T.
Essa offre il metodo T exchange( T t) che blocca il thread corrente fino a che un altro thread non
invoca lo stesso metodo, sulla stessa istanza. Quando questo avviene, il metodo restituisce l'oggetto
passato come parametro dal thread opposto.
Si implementi la classe C++, usando la libreria standard C++11. (4pt)

### Implementazione

```c++
#include <iostream>

#include <mutex>
#include <condition_variable>
#include <vector>
#include <thread>

template <typename T>
class Exchanger {
    T first;
    T second;
    std::mutex m;
    std::condition_variable cv;
    int status;
    bool busy;
public:
    Exchanger(): status(1), busy(false){};

    T exchange(T t){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return !busy;});
        switch(status){
            case 1:
                first = t;
                cv.notify_all();
                status = 2;
                cv.wait(lk, [&](){return status == 1;});
                busy = false;
                cv.notify_all();
                return second;
            case 2:
                cv.wait(lk, [&](){return status == 2;});
                status = 1;
                second = t;
                busy = true;
                cv.notify_all();
                return first;
            default:
                break;
        }
    }
};

#define N 4

int main() {
    Exchanger<int> ex{};
    std::vector<std::thread> workers;
    std::recursive_mutex m;
    workers.reserve(N);
    for (int i = 0; i<N; i++){
        workers.emplace_back(std::thread([&, i](){
            int o = i;
            int v = ex.exchange(o);
            std::lock_guard lk{m};
            std::cout << "Thread " << i << ": " << v << std::endl;
        }));
    }

    for(auto &w : workers){
        w.join();
    }

    return 0;
}
```

## Looper

[Back to top](#Table-of-Contents)


### Testo
Sia data una classe Handler che gestisce una coda di tipo Message tramite un metodo handle.
```c++
template <class Message> Handler
{
public:
    virtual void handle(Message msg) = 0;
}
```
Sia data anche una classe Looper che gestisce il prelievo di tutti i messaggi presenti in coda
tramite un thread privato, mentre il metodo send(...) inserisce i messaggi in coda.
```c++
template <class Message> Looper
{
public:
    Looper(Handler *handle);
    ~Looper();
    void send(Message msg);
}
```
Il contenuto private della classe Looper è stato ommesso esplicitamente.
Realizzare la classe Looper facendo in modo che quando l’oggetto viene distrutto tutti i
messaggi presenti in coda vengano gestiti senza la possibilità di ulteriori inserimenti (4pt)


### Implementazione

```c++
#include <iostream>

#include <string>
#include <iostream>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <thread>

template<class Message>
class Handler {
public:
    virtual void handle(Message msg) = 0;
};

class HandleString : public Handler<std::string> {
public:
    void handle(std::string msg) override {
        std::cout << msg << std::endl;
    }
};

template<class Message>
class Looper {
    std::unique_ptr<Handler<Message>> handle;
    std::queue<Message> queue;
    std::mutex m;
    std::condition_variable cv;
    bool stop;
    std::thread wt;

    void worker_thread() {

        while (true) {
            std::unique_lock lk{m};
            cv.wait(lk, [&]() { return !queue.empty() || stop; });
            if (queue.empty() && stop) break;
            auto element = queue.front();
            queue.pop();
            handle->handle(element);
            cv.notify_one();
        }
    }

public:
    Looper(Handler<Message> *h) : stop(false) {
        this->handle = std::unique_ptr<Handler<Message>>(h);
        h = nullptr;
        std::thread th{&Looper::worker_thread, this};
        wt = std::move(th);
    }

    ~Looper() {
        std::unique_lock lk{m};
        stop = true;
        cv.wait(lk, [&]() { return queue.empty(); });
        wt.join();
    };

    void send(Message msg) {
        std::unique_lock lk{m};
        if (!stop) {
            queue.push(msg);
            cv.notify_one();
        }

    };
};


int main() {
    auto h = new HandleString();
    Looper<std::string> looper{h};
    looper.send("Ciao");
    looper.send("Come");
    looper.send("Stai");
    return 0;
}
```

## Dispatcher 2016-07-13

[Back to top](#Table-of-Contents)


### Testo

E' presente una classe Dispatcher<T> che gestisce sottoscrizioni, inserendole tramite un metodo `subscribe(void(*f)(T t))` e le cancella tramite il metodo `unsubscribe(void(*f)(T t))`. Esiste un ulteriore metodo chiamato `notify(T t)` che deve permettere al thread corrente di eseguire sottoscrizioni, tramite l'invocazione della `subscribe()` oppure rimuoverle tramite la `unsubscribe()`.
Qualora in presenza di una notifica dovesse arrivare una richiesta di "subscribe()" dal thread corrente, questa verrà accodata, mentre se la notifica arriva da un altro thread questa verrà bloccata.
Realizzare in C++11 la classe con i relativi metodi. (4pt)

### Implementazione

> Non implementata con Main poichè strana.

```c++
#include <mutex>
#include <list>

template <class T>
class Dispatcher {
    std::list<void(*)(T)> dispatcher;
    std::list<void(*)(T)> tempInsert;
    std::list<void(*)(T)> tempRemove;
    std::list<T> tempNotifica;
    std::mutex m;
    bool notifica;
    std::thread::id id;

public:
    Dispatcher(){};

    void subscribe(void(*f)(T t)){
        std::lock_guard lk{m};
        if (notifica)
            if (std::thread::get_id() == id)
                tempInsert.push_back(f);
        else
            dispatcher.push_back(f);
    }

    void unsubscribe(void(*f)(T t)){
        std::lock_guard lk{m};
        if (notifica)
            if (std::thread::get_id() == id) return;
                tempRemove.push_back(f);
        else
            dispatcher.remove(f);
    }

    void notify(T t){
        std::recursive_lock lk{m};
        id = std::this_thread::get_id();
        if (notifica){
            tempNotifica.push_back(t);
            return;
        }

        notifica = true;

        for (auto i = dispatcher.begin(); i != dispatcher.end(); i++){
            (*i)(t);
        }

        notifica = false;

        for (auto i = tempInsert.begin(); i != tempInsert.end(); i++){
            dispatcher.push_back(*i);
        }

        for (auto i = tempRemove.begin(); i != tempRemove.end(); i++){
            dispatcher.remove(*i);
        }

        for (auto i = tempNotifica.begin(); i != tempModifica.end(); i++){
            notifica(t);
        }
    }

}
```

```c++
#include <mutex>
#include <list>

template <class T>
class Dispatcher {
    std::list<void(*)(T)> dispatcher;
    std::list<void(*)(T)> tempInsert;
    std::list<void(*)(T)> tempRemove;
    std::list<T> tempNotifica;
    std::mutex m;
    bool notifica;
    std::thread::id id;
    std::condition_variable cv;

public:
    Dispatcher(){};

    void subscribe(void(*f)(T t)){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){
            return (notifica && std::this_thread::get_id() == id) || !notifica;
        });
        dispatcher.push_back(f);
    }

    void unsubscribe(void(*f)(T t)){
        std::lock_guard lk{m};
        cv.wait(lk, [&](){
            return (notifica && std::this_thread::get_id() == id) || !notifica;
        });
        dispatcher.remove(f);
    }

    void notify(T t){
        std::recursive_lock lk{m};
        cv.wait(lk, [&](){return !notifica});
        notifica = true;
        id = std::this_thread::get_id();
        for (auto i = dispatcher.begin(); i != dispatcher.end(); i++){
            (*i)(t);
        }
        notifica = false;
    }
}
```

## Stack 2011-07-26

[Back to top](#Table-of-Contents)


### Testo

Una applicazione Win32 presenta più thread che devono operare in modo concorrente su uno stack di stringhe unicode. Si implementi una classe C++ che incapsuli il comportamento di tale stack thread-safe. Deve permettere l'inserimento di nuove stringhe nello stack e l'estrazione dell'ultima stringa inserita (notificando il caso di stack vuoto). Porre particolare attenzione alla gestione della memoria e della concorrenza.

### Implementazione

```c++
#include <iostream>
#include <stack>
#include <condition_variable>
#include <vector>
#include <thread>

class Stack{
    std::stack<std::string> stack;
    std::mutex m;
    std::condition_variable cv;
    unsigned capacity;
    unsigned size;
public:
    Stack(unsigned N): capacity(N), size(0){};

    void push(std::string s){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return size < capacity;});
        size++;
        stack.push(s);
        std::cout << "PUSH " << s << std::endl;
        cv.notify_one();
    }

    std::string pop(){
        std::unique_lock lk{m};
        cv.wait(lk, [&](){return size > 0;});
        size--;
        std::string res = stack.top();
        stack.pop();
        cv.notify_one();
        std::cout << "POP " << res << std::endl;
        return res;
    }
};

#define N 10

int main() {
    Stack s{N};
    std::vector<std::thread> workers;
    workers.reserve(N);
    for (int i = 0; i< N; i++){
        workers.emplace_back([&, i](){
           if (i%2){
               s.push(std::to_string(i));
           } else{
               s.pop();
           }
        });
    }
    for (auto &w : workers){
        w.join();
    }
    return 0;
}
```

## PubSubHub 2014-06-25

[Back to top](#Table-of-Contents)


### Testo

Un'infrastruttura per la comunciazione asincrona tra thread supporta il paradigma `publish/subscribe` tramite istanze della classe `PubSubHub`. La classe offre i seguenti metodi thread-safe:

- `void publish(std::string topic, std::string data);`
- `void subscribe(std::string topic);`
- `bool getData(std::string& topic, std::string& data);`
- `void unsibscribeAll();`

Il metodo **subscribe** permette ad un thread di dichiarare il proprio interesse per i dati di un certo argomento; il metodo **publish** rende disponibile agli interessati un dato relativo ad uno specifico argomento. Entrambi i metodi sono non bloccanti.

Quando un thread chiama per la prima volta il metodo `subscribe()`, la classe `PubSubHub` alloca una coda di messaggi pendenti e un elenco di argomenti sottoscritti; successive chiamate al metodo `subscribe()` da parte dello stesso thread comportano il solo aggiornamento di tale struttura dati. Per accedere all'identità del thread chiamante può utilizzare il metodo statico `std::this_thread::get_id()`.

Quando si verifica un'operazione di tipo `publish()`, l'oggetto `PubSubHub` scorre la listra dei propri subscriber alla ricerca di quelli che si sono iscritti all'argomento indicato ed inserisce nella coda di ciascun subscriber che soddisfa tale criterio una copia del messaggio pubblicato. Se l'argomento non è di interesse di nessuno, il messaggio pubblicato viene ignorato.

Un thread che abbia sottoscritto almeno un argomento può invocare il metodo `getData()`. Se ci sono contenuti relativi agli argomenti sottoscritti questo metodo ne restituisce il più vecchio (eliminandolo dalla relativa coda) e torna true. Se non ci sono (ancora) messaggi il metodo si blocca in attesa di una pubblicazione (dopodichè restituirà `true` e il messaggio/argomento relativo). Se, invece, il thread non risulta avere fatto alcuna sottoscrizione, il metodo ritorna `false`.

Infine, il metodo `unsubscribeAll()` permette ad un thread di cancellare tutte le proprie sottoscrizioni e rilasciare le relative risorse.

Si implementi la classe C++ `PubSubHub`, usando la libreria standard C++.

### Implementazione

```c++
#include <mutex>
#include <condition_variable>
#include <string>
#include <thread>
#include <map>
#include <queue>
#include <set>

class Message {
public:
    std::string data;
    std::string topic;
    Message(std::string topic, std::string data): topic(topic), data(data){};
};

class ThreadData {
public:
    std::thread::id id;
    std::queue<Message> queue;
    std::set<std::string> subs;

    ThreadData (std::thread::id id): id(id){};

    void push(std::string topic, std::string data){
        Message m(topic, data);
        this->queue.push(m);
    }

    bool pop(std::string &topic, std::string &data){
        Message m = this->queue.front();
        this->queue.pop();
        topic = m.topic;
        data = m.data;
        return true;
    }

    bool empty(){
        return this->queue.empty();
    }
};

class PubSubHub {
    std::map<std::thread::id, ThreadData> thread_data;
    std::mutex m;
    std::condition_variable cv;
public:
    PubSubHub(){};
    void subscribe(std::string topic){
        std::unique_lock lk{m};
        std::thread::id tid = std::this_thread::get_id();
        auto it_th = thread_data.find(tid);
        if (it_th != thread_data.end()){
            it_th->second.subs.insert(topic);
        } else {
            ThreadData td(tid);
            td.subs.insert(topic);
            auto pair = std::make_pair(tid, td);
            thread_data.insert(pair);
        }
    }

    void publish(std::string topic, std::string data){
        std::unique_lock lk{m};
        std::thread::id tid = std::this_thread::get_id();
        for (auto it = thread_data.begin(); it != thread_data.end(); it++){
            auto it1 = it->second.subs.find(topic);
            if (it1 != it->second.subs.end()){
                it->second.push(topic, data);
                cv.notify_all();
            }
        }
    }

    bool getData(std::string &topic, std::string &data){
        std::unique_lock lk{m};
        std::thread::id tid = std::this_thread::get_id();
        auto it = thread_data.find(tid);
        if (it != thread_data.end()){
            cv.wait(lk, [&](){
                return !it->second.empty();
            });
            it->second.pop(topic,data);
            return true;
        }
        return false;
    }

    void unsubscribeAll(){
        std::unique_lock lk{m};
        std::thread::id tid = std::this_thread::get_id();
        auto it = thread_data.find(tid);
        if (it != thread_data.end()){
            thread_data.erase(it);
        }
    }
};

#define N 15

int main() {
    PubSubHub hub;
    hub.subscribe(std::to_string(0));
    hub.subscribe(std::to_string(1));

    std::vector<std::thread> workers;
    workers.reserve(N);
    for (int i = 0; i < N / 3; i++) {
        workers.emplace_back([i, &hub]() {
            hub.subscribe(std::to_string(i));
        });
    }

    for (int i = 0; i < N / 3; i++) {
        workers.emplace_back([i, &hub]() {
            std::string data;
            std::string topic;
            hub.getData(topic, data);
        });
    }

    for (int i = 0; i < N / 3; i++) {
        workers.emplace_back([i, &hub]() {
            hub.publish(std::to_string(i), std::to_string(i + 10));
        });
    }
    std::string topic;
    std::string data;
    hub.getData(topic, data);
    for (auto &w : workers) {
        w.join();
    }
    return 0;
}
```

## Singleton 2019-01-XX

[Back to top](#Table-of-Contents)


### Testo

> Testo dedotto dalle risoluzioni trovate in rete.

Si implementi una classe Singleton thread safe che possegga un metodo `log(std::string msg)` per poter scrivere in maniera concorrente in un file di testo che conterrà il log dell'applicazione.

### Implementazione

#### Singleton.h

```c++
#include "Singleton.h"

std::once_flag Singleton::inited;
std::shared_ptr<Singleton> Singleton::instance;

std::shared_ptr<Singleton> Singleton::getInstance() {
    std::call_once(inited, [](){
        instance = std::shared_ptr<Singleton>(new Singleton());
    });
    return instance;
}

void Singleton::log(std::string msg) {
    std::lock_guard<std::mutex> lg(mutex);
    file.open("file.txt", std::ofstream::app);
    if (!file.is_open()) throw;
    file << std::this_thread::get_id() << " " << msg << std::endl;
    file.close();
}
```

#### Singleton.cpp

```c++
#ifndef SINGLETON_SINGLETON_H
#define SINGLETON_SINGLETON_H

#include <iostream>
#include <future>
#include <fstream>
#include <mutex>

class Singleton {
    std::mutex mutex;
    static std::shared_ptr<Singleton> instance;
    static std::once_flag inited;
    std::ofstream file;

    Singleton(){};

public:
    static std::shared_ptr<Singleton> getInstance();
    void log(std::string msg);
};

#endif //SINGLETON_SINGLETON_H
```

#### main.cpp

```c++
#include <iostream>
#include <vector>
#include <thread>
#include "Singleton.h"

#define N 10

int main() {
    Singleton::getInstance();
    std::vector<std::thread> workers;
    workers.reserve(N);
    for (int i = 0; i< N; i++){
        workers.emplace_back(std::thread([i](){
            Singleton::getInstance()->log(std::to_string(i));
        }));
    }

    for (auto &w : workers){
        w.join();
    }
    return 0;
}
```

## Barriera Ciclica 2019-07-05

### Testo
Una Barriera Ciclica è un oggetto di sincronizzazione che permette a N thread (con N specificato all'atto della costruzione dell'oggetto) di attendersi. Tale oggetto dispone di un solo metodo "void attendi()" la cui invocazione blocca il thread chiamante fino a che altri N-1 thread non risultano bloccati insieme ad esso. Quando il numero di thread bloccati raggiunge N, tutti thread si sbloccano e il metodo attendi() ritorna. Ulteriori chiamate al metodo attendi() si comportano analogamente: le prime N-1 invocazioni si bloccano, all'arrivo della successiva invocazione tutti i thread si sbloccano e i metodi ritornano. Si implementi una classe C++ che implementi tale comportamento usando la libreria standard C++.

### Implementazione

```c++
#include <iostream>
#include <mutex>
#include <condition_variable>
#include <vector>
#include <thread>

class CycleBarrier {
    std::mutex m;
    std::condition_variable cv;
    unsigned n_t;
    unsigned max_t;
    bool svuotamento;
public:
    explicit CycleBarrier(unsigned N) : max_t(N), n_t(0), svuotamento(false) {};

    void attendi() {
        std::unique_lock lk{m};
        cv.wait(lk, [&]() { return !svuotamento; });
        n_t++;
        if (n_t == max_t) {
            svuotamento = true;
            std::cout << "Avvio esecuzione!" << std::endl;
            n_t--;
        } else {
            std::cout << "Attesa..." << std::endl;
            cv.wait(lk, [&]() { return svuotamento; });
            std::cout << "Esecuzione!" << std::endl;
            n_t--;
        }
        if (n_t == 0) {
            svuotamento = false;
        }
        cv.notify_all();
    }

    void aggiungi(int N) {
        std::unique_lock lk{m};
        cv.wait(lk, [&]() { return !svuotamento; });
        max_t += N;
        if (n_t >= N){
            std::cout << "Avvio esecuzione dopo incremento!" << std::endl;
            svuotamento = true;
        }
        cv.notify_all();
    }

    void rimuovi(int N) {
        std::unique_lock lk{m};
        cv.wait(lk, [&]() { return !svuotamento; });
        max_t -= N;
        if (n_t >= N){
            std::cout << "Avvio esecuzione dopo decremento!" << std::endl;
            svuotamento = true;
        }
        cv.notify_all();
    }

};

#define N 15

int main() {
    std::vector<std::thread> workers;
    workers.reserve(N);
    CycleBarrier b(5);
    for (int i = 0; i < N; i++) {
        workers.emplace_back(std::thread([&]() {
            b.attendi();
        }));
    }

    for (auto &w : workers) {
        w.join();
    }
    return 0;
}
```