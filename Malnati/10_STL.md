# STL

- [STL](#STL)
  - [I/O](#IO)
    - [I/O File](#IO-File)
    - [Manipolatori `<iomanip>`](#Manipolatori-iomanip)
  - [Standard Template Library](#Standard-Template-Library)
    - [Container](#Container)
      - [Sequenziali](#Sequenziali)
      - [Associativi](#Associativi)
      - [Adapter](#Adapter)
      - [Interfacce Comuni](#Interfacce-Comuni)
      - [Container Adaptor](#Container-Adaptor)
    - [Algoritmi](#Algoritmi)
        - [Algoritmi non modificanti](#Algoritmi-non-modificanti)
        - [Algoritmi non modificanti](#Algoritmi-non-modificanti-1)
        - [Algoritmi per il partizionamento](#Algoritmi-per-il-partizionamento)
        - [Algoritmi per l'ordinamento](#Algoritmi-per-lordinamento)
        - [Algoritmi per ricerca binaria](#Algoritmi-per-ricerca-binaria)
      - [Execution Policy](#Execution-Policy)

## I/O

Possono essere utilizzate le funzioni di C, ma possono trattare solo i tipi semplici. Ci viene fornita da C++ una struttura con complessa ereditarietà e polimorfismo che parte da `ios`

`ios` è la classe base virtuale che contiene gli attributi e i metodi comuni. SOlitamente contiene degli stream che contengono lo stato dello stream. Manipolazione del formato e non può essere instanziata direttamente

Da questo derivano `istream` e `ostream` che rappresentano i metodi di lettura e scrittura dello stream. Gli operatori utilizzati sono `<<` e  `>>`. L'accesso è indiretto allo stream tramite buffer. (Una volta che il buffer è pieno a sufficienza allora viene messo sulla console). Può essere chiamata la funzione `flush()` che svuoterà il buffer

Abbiamo infine `iostream` che esegue operazioni di lettura e scrittura.

Le operazioni su file avvengono tramite `ifstream` e `ofstream`. Contengono i metodi per la creazione dello stream attraverso `open` e `close`.

Ci sono delle variabili già preconfezionate che sono `std::cout|cin|cerr`.

Utilizzando la funzione `width()` per estendere l'input a seconda delle esigenze dell'utente.

Utilizzando la funzione `precision()` si può a ndare a specificare il livello di precisione dei numeri reali.

### I/O File

```c++
#include <iostream>
#include <fstream>

int main() {
    std::ifstream f;
    std::ofstream out;

    f.open("ciao.txt");
    out.open("out.txt");
    if (!f.is_open()) return -1;
    if (!out.is_open()) return -1;

    int n;
    f >> n;
    for(int i = 0; i< n; i++){
        int a;
        f >> a;
        out << a*2 << std::endl;
    }
    f.close();
    out.close();
}
```

### Manipolatori `<iomanip>`

`hex` per scrivere in esadecimale.

`cerr` non è bufferizzato. Per fare input di esegue `cin >> ch;`

Metodo più furbo `readLine()`;

`cin.get(ch);`

`cin.getline(s, n);`

Le operazioni di I/O non sollevano eccezioni, il programmatore deve andare a controllare il risultato.

Per testare gli indicatori andiamo ad utilizzare:

- `eof()`
- `fail()` irrecuperabile
- `bad()` errore che causa perdita dei dati
- `good()` true se gli altri sono false.

Tutte le operazioni successive a un fallimento falliranno, quindi bisogna ripristinarlo con `clear()`.

## Standard Template Library

Libreria di template standard.

E' costituita da container e da algoritmi (programmazione funzionale), funzioni che possono essere applicati ai container. Il collante tra i due sono gli iteratori.

Gli iteratori sono delle generalizzazione di puntatori che permettono di accedere agli elementi di una classe container.

Perchè vogliamo usate la STL? Non vogliamo inventare cose già inventate. La STL facilita e velocizza la programmazione.

Le funzioni manipolano i dati, ma sono indipendenti dai container.

L'uso della STL permette di semplificare la manutenzione del codice. Impatto minimo sui tempi di esecuzione.

### Container

#### Sequenziali

`array` (fissa), `vector` (dinamica), `deque`(testa e coda), `list` (liste linkate), `forward_list` (linkate semplicemente, solo forward)

- Gli array sono allocati in maniera costante. Diventa staticamente polimorfico che mi permette di poter utilizzare gli algoritmi. Questi non sono riallocabili. Non si hanno metodi di delete. Possono essere ritornati come parametri di una funzione.
Accesso casuale a tempo costante.

- I vector sono array allocati dinamicamente. I vettori hanno una capacità che è diverso dalla dimensione. Una `std::string` equivale di fatto a `std::vector<char>`, ma non fa parte dei contenitori della STL.

#### Associativi

- ordinati: `set`, `map`, `multiset`, `multimap`
  - La chiave deve implementare operator<, ordinabile, copiabile e movibile.
- non ordinati: `unordered_set`, `unordered_map`, `unordered_multiset`, `unordered_multimap`
  - La chiave deve essere equal comparable, copiabile o movibile,disponibile come hash.
  - Si basano sull'uso di `pair<t1, t2>` che devono essere Assignable, ovvero copiabile e riassegnabile.

Iteratori inclusi tra `.begin` e `.end()`.

Tempo di accesso logaritmico.

I secondi sono basati su hashing.

`make_pair`

#### Adapter

- `stack`, `queue`, `priority_queue` non implementano gli iteratori


#### Interfacce Comuni

Nonostante ogni container sia specializzato per uno specifico uso hanno molto in comune.

Ci sono molti costruttori che è possibile utilizzare: *slide 30*

- `cont.size()`: non usabile su `forward_list`
- `cont.empty()`: costa meno
- `cont.max_size()`: numero massimo di elementi

Iteratori:
- `begin()` - `end()`
- `cbegin()` - `cend()`
- `rbegin()` - `rend()`
- `rcbegin()` - `rcend()`

Swap

Compare, è possibile andarli a comparare. Per quanto riguarda i container associativi non ordinati abbiamo solo gli operatori di uguaglianza.

Un iteratore è una generalizzazione dei puntatori. Questo permette l'applicazione degli stessi a differenti container andando a definire le diverse incrementazioni.

Iteratori possono essere `forward iterator`, `bidirectional iterator`, `random access iterator`.

Esistono anche le versioni di iteratori scritte come iteratori.

#### Container Adaptor

Contenitori sequenziali specializzati `stack`, `queue` e `priority_queue`.

### Algoritmi

Gli algoritmi sono base, ma sono ottimizzati e con implementazione sempre uguale. Un loop for non descrive cosa sta facendo, dobbiamo andare a leggere internamente al loop. Gli algoritmi generici forniti dalla libreria sono invece self-descriptive.

Nella libreria `<numeric>` andiamo a trovare algoritmi numerici come `accumulate`, `inner_product`, `adjacent_difference`.

Nella libreria `<algorithm>` troviamo invece tutti gli algoritmi per la ricerca, la ricerca binaria, la trasformazione dei dati, partizionamento, ordinamento, merge, operazioni insiemistiche, ...

##### Algoritmi non modificanti

*see slide 56*

##### Algoritmi non modificanti

*see slide 58,59*

##### Algoritmi per il partizionamento

*see slide 60*

##### Algoritmi per l'ordinamento

*see slide 62*

##### Algoritmi per ricerca binaria

*see slide 64*

#### Execution Policy

Gli algoritmi possono essere eseguiti anche in maniera concorrente utilizzando la libreria `<execution>`

`std::execution::par`, `std::execution::seq`,`std::execution::par_unseq` (Single Instruction Multiple Data - SIMD)

E' compito del programmatore andare a verificare concorrenza e race conditions.
