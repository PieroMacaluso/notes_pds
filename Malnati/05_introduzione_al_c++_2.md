# Introduzione al C++

- [Introduzione al C++](#Introduzione-al-C)
  - [Passaggio dei parametri](#Passaggio-dei-parametri)
  - [Copia e Assegnazione](#Copia-e-Assegnazione)
  - [Regola dei tre](#Regola-dei-tre)
  - [Assegnazione per movimento](#Assegnazione-per-movimento)
  - [Paradigma Copy&Swap](#Paradigma-CopySwap)

## Passaggio dei parametri

- per valore, i parametri vengono duplicati appoggiandosi al costruttore di copia
- per indirizzo, viene passata una copia dell'indirizzo del dato.
- per riferimento, sintatticamente è un passaggio per valore, ma in semanticamente è un passaggio per indirizzo. Non può essere NULL.
- per riferimento costante come sopra, ma in sola lettura
- per riferimento rvalue

Si passano per valore tutte quelle cose che hanno un costo basso, per riferimento tutti gli altri.

Trasferimento di proprietà: `std::move(data)` come parametro.
Per restituire più parametri in uscita possiamo usare le tuple. `std::tuple<T, A>`, per farla `std::make_tuple()`

## Copia e Assegnazione

***See Slide 39***

In assegna di ulteriori comunicazioni l'assegnazione si limita a fare la copia bruta . Questo può creare leakage e puntano entrambi alla stessa porzione di memoria.
***See Slide 43***

Si può andare a ridefinire l'assegnazione `CBuffer& operator=(const CBuffer& source);` Distrugge l'oggetto su cui andiamo a copiare

```c++
CBuffer& operator=(const CBuffer& source) {
    if (this != &source) { // Se vado a fare A = A grandi problemi
        delete[] this->ptr;
        this->ptr = null; // In eccezione, distruttore rilascia 2 volte la memoria
        this->size = source.size;
        this->ptr = new char[size];
        memcpy(this->ptr, source.ptr, size);
    }
    return *this; // Ritorna per reference ritorna oggetto assegnato
    // Motivazione A = B = C = 0;
}
```

IL COSTRUTTORE DI COPIA E DI ASSEGNAZIONE DEVONO ESSERE SEMANTICAMENTE EQUIVALENTI.

## Regola dei tre

Se si implementa uno di questi tre sarà necessario fare gli altri due!

- Costruttore di Copia
- Operatore di assegnazione
- Distruttore

## Assegnazione per movimento

```c++
CBuffer& operator=(const CBuffer&& source) {
    if (this != &source) { // Se vado a fare A = A grandi problemi
        delete[] this->ptr;
        this->size = source.size;
        this->ptr = source.ptr;
        source.ptr = nullptr; // Importante altrimenti libero lo stesso
    }
    return *this; // Ritorna per reference ritorna oggetto assegnato
    // Motivazione A = B = C = 0;
}
```

## Paradigma Copy&Swap

Possiamo ridurre tutte le prblematiche andando a definire una funzione **`friend`**, può accedere alle mie variabili private.

```c++
friend void swap (intArray& a, intArray& b) {
    std::swap(a.msize, b.mSize);
    std::swap(a.mArray, b.mArray);
}

intArray& operator=(intArray that) {
    swap(*this, that);
    return *this;
}


intArray(intArray&& that): mSize(0), mArray(nullptr) {
    swap(*this, that);
}
```

`std::move(...)` è una sorta di cast a RValue.