# Introduzione al C++

- [Introduzione al C++](#Introduzione-al-C)
  - [Tipi base e derivati](#Tipi-base-e-derivati)
  - [Classi](#Classi)
  - [Costruttore di copia](#Costruttore-di-copia)
  - [Costruttore di Movimento](#Costruttore-di-Movimento)
  - [Distruttore](#Distruttore)

Linguaggio vasto e articolaro che è compatibile con il C, ma ha estensioni esoteriche e uniche, per garantire espressività senza penalizzare prestazioni.

Obiettivi:

- Astrazione a costo nullo
- Espressività (UDT - User Defined Type) devono possedere la stessa espressività offerto dai tipi base del linguaggio
- Sostituzione: usare un UDT ovunque sia utilizzabile un tipo base.

Offre incapsulamento, Composizione, Ereditarietà e Polimorfismo. Programmazione Generica, Strutturata e Funzionale.

## Tipi base e derivati

- Riferimenti: puntatore inizializzato e automaticamente dereferenziato

```c++
int i = 0;
int& r = i; // r è alias di i, si può accedere all'originale
```
Il riferimento non può essere messo a NULL.

## Classi
Struttura dati con supporto per incapsulamento.

Definirla con `#ifndef`

**Non si usa il new!!**
`CBuffer globalBuffer2 { 32 };`

## Costruttore di copia

Costruttore che riceve come parametro un riferimento all'oggetto da copiare. e.g. `CBuffer(const CBuffer& source);`
In assenza di specifiche da parte del programmatore questo viene implementato in maniera automatica dal compilatore. Per questo motivo se fosse necessario evitare che possa essere fatta una copia dell'oggetto si deve andare a impostare il metodo in questo modo: `CBuffer::CBuffer(const CBuffer& source) = delete;`

## Costruttore di Movimento

Costruttore che riceve come parametro un riferimento RValue indicato con `&&`. Il costruttore di occuperà di costruire il nuovo oggetto modificando la sorgente. I puntatori vengono semplicemente copiati avendo cura di settare quelli dalle `source` a null, onde evitare che vengano liberate le porzioni di memoria appena mosse.

 `CBuffer(CBuffer&& source);`

Questa volta non è const poichè lo modifico. Non viene fatto di base dal compilatore, ma deve essere implementato.

Per il movimento ci si appoggia alla funzione `std::move(...)` che possiamo considerare come una sorta di cast a RValue Reference.

## Distruttore

Non ha mai parametri e non deve mai essere chiamato, solo il compilatore lo chiama. E' il testamento dell'oggetto che racchiude la liberazione di tutte le strutture di memoria dinamiche.
