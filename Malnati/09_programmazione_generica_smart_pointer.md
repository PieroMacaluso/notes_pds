# Programmazione Generica

- [Programmazione Generica](#Programmazione-Generica)
  - [Template](#Template)
    - [Classe Generica](#Classe-Generica)
  - [Smart Pointer](#Smart-Pointer)
    - [Ownership-Handling](#Ownership-Handling)
    - [`std::unique_ptr<BaseType>`](#stduniqueptrBaseType)
    - [`std::shared_ptr<BaseType>`](#stdsharedptrBaseType)
    - [Polimorfismo](#Polimorfismo)

## Template

Molto spesso un tipo di programmazione stringente permette di avere un sistema robusto. Spesso però, per non andare a violare il sistema dei tipi si va a replicare una grande quantità di codice. Il C++ supporta il concetto di **Template**.

Un **template** è un frammento parametrico che viene espanso in fase di compilazione in base al relativo utilizzo. Mantiene perciò la robustezza di un sistema *strong-typed*.

Il template è la chiave di volta per la programmazione generica. Permette di creare polimorfismo statico, cioè eseguito in fase di compilazione senza andare a usare metodi virtuali e V-table.

La parte principale della libreria standard è basata su Template.

```c++
template <typename T>
const T& max(const T& t1, const T& t2) {
    return (t1 < t2 ? t2 : t1);
}
```

L'operazione funziona se il tipo generico T supporta l'operatore `<` e supporti il costruttore di copia.

### Classe Generica

Lo stesso principio può essere applicato ad una classe.

```c++
template <typename T>
class Accum {
    T total;
public:
    Accum(T start): total(start) {}
    T operator+=(const T& t) {
        return total = total + t;
    }
    T value() {return total;}

}

Accum<std::string> sa("");
Accum<int> ia(0);
```

Quando il compilatore incontra il template, la compilazione avviene in due fasi:

1. Creazione di un albero sintattico astratto (AST)
2. Nel momento in cui incontra i frammenti di codice in cui è specificato come viene usato: a quel punto va a produrre le funzioni relative se non sono già state compilate in precedenza, specializzando l'albero sintattico.

I parametri del template possono essere di tipo di dato o valori costanti.

Se si crea una classe generica è necessario che la sua definizione appaia nella stessa unità di traduzione (no separazione `.cpp` e `.h`). Quasi sempre si implementa tutto nell'header.

Alcuni tipi potrebbero non essere utilizzabili per assunzioni non corrette. La prima soluzione è scegliere una strada diversa modificando la classe oppure si può andare a modificare e specializzare il template. Esempio slide 15.

**Punti di forze**:

- Aumentano le possibilità espressive
- Non pregiudica tempi esecuzione
- Risparmiamo tempo
- Non mettono a rischio i tipi
- Flessibilità per futuri miglioramenti

**Punti di attenzione**:

- Chi usa un template deve verificarne le assunzioni sui tipi da utilizzare.
- Le violazioni portano ad errori di compilazione.
- Spesso non occorre scrivere nuovi template

## Smart Pointer

La ridefinizione degli operatori ci permette di dare una semantica alle operazioni `*` e `->`.
E' possibile costruire oggetti che sembrano puntatori, ma hanno altre caratteristiche:

- Garanzia di inizializzazione e rilascio
- Conteggio riferimenti
- Accesso controllato

Possono anche essere ridefinite le operazioni.

Esempio di implementazione di un wrapper per puntatore.

> Non è una buona classe perchè possiamo inserire un int i su heap con casini.

```c++
class int_ptr {
    int* ptr;
    int_ptr(const int_ptr&);
    int_ptr& operator=(const int_ptr&);
public:
    explicit int_ptr(int* p) : ptr(p) {}
    ~int_ptr() {delete ptr;}
    int& operator*() {return *ptr;}
};
```

Codici a confronto in slide 23. Nel primo codice se ci potrà essere una eccezione sulla funzione `esegui()`, quindi avremo la contrazione dello stack, ma un leakage. Nell'esempio 2 non dobbiamo ricordarci di eliminarlo, ma soprattutto viene fatto con la contrazione dello stack. Usare blocchi try catch, oltre ad essere costoso genera comunque un codice molto verboso.

Ma a chi appartiene la memoria di uno smart_pointer? Non abbiamo implementato il costruttore di copia apposta. E' possibile andare a:

- Implementare un passaggio di proprietà (`unique_ptr`)
- Posso fare una copia completa
- Condivisione con conteggio dei riferimenti (`shared_ptr`), il puntatore viene liberato quando muoiono tutti i riferimenti.
- Condivisione in lettura e duplicazione in scrittura

### Ownership-Handling

Unico utilizzatore o eliminato dall'ultimo utilizzatore.

La libreria che usiamo è `#include <memory>`

- `std::unique_ptr<BaseType>` impedisce la copia, ma permette il trasferimento.
- `std::shared_ptr<BaseType>` condivisibile attraverso un meccanismo di conteggio dei riferimenti.
- `std::weak_ptr<BaseType>` legata allo shared ma non partecipa al conteggio

### `std::unique_ptr<BaseType>`

Può essere solo trasferito esplicitamente ad un altro `unique_ptr` attraverso la funzione `std::move()`
Attenzione agli array (`unique_ptr<T[]>`).
`.reset()` ci permette di rilasciare la memoria.

Nel C++98 si aveva `auto_ptr` che però è stato deprecato a favore dello `unique_ptr` che implementa correttamente la copia che trasferisce la proprietà. In `auto_ptr` l'errore veniva rilevato a run-time, con `unique_ptr` a compile time.

### `std::shared_ptr<BaseType>`

Alloca più memoria, ma permette di condividere un puntatore. Il puntatore punta a un blocco condiviso che punta al dato. Nel pezzo condiviso avremo il contatore dei puntatori. Possiamo andare a settare una procedura di resetting.

`get()` ci permette di accedere al blocco di memoria.
`reset()`

La funzione `make_shared<BaseType>(params...)` ci permette di creare un'istanza e restituire il puntatore shared. Garantisce la contiguità dei blocchi.

Il blocco di controllo mantiene distruttore, allocatore, contatore shared_ptr e contatore weak_ptr.

**SLIDE 38**

Ogni `shared_ptr` contiene al proprio interno un puntatore al dato e un puntatore al blocco di controllo. La dimensione dello smart pointer raddoppia.

### Polimorfismo

Data una classe D che eredita in modo pubblico da B, un puntatore ad una classe derivata D è anche un puntatore di B.

- `std::sharedptr<T> static_pointer_cast<T>(shared_ptr<U>)`
- `std::sharedptr<T> dynamic_pointer_cast<T>(shared_ptr<U>)`
- ...

Molto spesso ci ritroviamo nella situazione di avere una dipendenza ciclica che manderebbe in crisi il conteggio dei puntatori, per poter risolvere questa faccenda possiamo andare ad utilizzare `std::weak_ptr<BaseType>`. Quest'ultimo mantiene un riferimento dbole al blocco custodito da shared_ptr.

Se io vado a fare una `.lock()` su un `weak_ptr` che modella il possesso temporaneo ad un blocco di memoria. Si andrà a restituire un `shared_ptr` (dobbiamo controllare che lo shared sia vivo con `.expired()`).
