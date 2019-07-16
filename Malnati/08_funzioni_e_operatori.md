# Funzioni e operatori

- [Funzioni e operatori](#Funzioni-e-operatori)
  - [Puntatori a funzione](#Puntatori-a-funzione)

Operator overloading, almeno uno dei due operandi deve essere una UDT.

Operatori di conversione

Non si possono ridefinite `::`, `.`, `? :`, `sizeof`.

E' possibile sovrascrivere `operator[]`, subscript operator e `operator()` function call operator.

## Puntatori a funzione

Cosa me ne faccio di un puntatore a funzione?

In C++ esiste un tipo invocabile chiamato "funtore" o "oggetto funzionale".
E' possibile includere più definizioni di `operator()`. I funtori possono avere anche delle variabile all'interno. Non viene più chiamata funzione, ma **funzionale**.

Aumentare livello di generalizzazione dei propri programmi
 

Mescolare i due approcci

```c++
template <typename F>
void some_function(F& f){
    f(); // funzionale o puntatore a funzione
}
```
Funzioni minimali `std::for_each(v.begin(), v.end(), print)`, per questo motivo si introducono le lambda function `std::for_each(v.begin(), v.end(), [](int i) {std::cout << i << " ";})`

Se si ha una sola istruzione di return il valore si può inferire, altrimenti bisogna indicarlo con `[]() -> tipo`

Le parentesi quadre sono le variabili che vengono catturate per valore, per riferimento, per catturare tutto `[&]`.

Una lambda che cattura dei valori viene chiamata chiusura. I valori catturati non possono essere modificati a meno che non venga indicata la parola chiave "mutable".

`std::function<R(Args...)>`

```c++
std::function<int(int)> makeFunction(int k){
    return [k](int i) { return k*i; };
}
```

Il meccanismo di `std::function` si basa sul polimorfismo, per questo motivo richiede più complessità, poichè bisogna passare dalla V-Table.
