# Funzioni e operatori

- [Funzioni e operatori](#Funzioni-e-operatori)
  - [Overloading operatori](#Overloading-operatori)
  - [Operatori di conversione](#Operatori-di-conversione)
  - [Puntatori a funzione](#Puntatori-a-funzione)
  - [Template](#Template)
  - [Funzioni minimali](#Funzioni-minimali)
  - [`std::function`](#stdfunction)
  - [Spunto di riflessione](#Spunto-di-riflessione)

## Overloading operatori

Operator overloading, almeno uno dei due operandi deve essere una UDT.

Operatori di conversione

Non si possono ridefinite `::`, `.`, `? :`, `sizeof`.

E' possibile sovrascrivere `operator[]`, subscript operator e `operator()` function call operator.

Non è possibile andare a cambiare quelle che sono le precedenze e le arità.

## Operatori di conversione

Gli operatori di conversione possono essere definiti nel seguente modo, andando a inserire il nome del tipo verso cui si vuole convertire:

```c++
class Frazione{
public:
    operator float(){
        return num * 1.f / den;
    }
private:
    int num, den;
}
```

## Puntatori a funzione

In C++ come in C sarà possibile utilizzare una variabile di tipo puntatore a funzione con la sintassi
`<tipo_ritornato> (*var)(<argomenti>)` come ad esempio `int (*f)(int, double)`.

In C++ esiste un tipo invocabile chiamato "funtore" o "oggetto funzionale", un'istanza di qualsiasi classe che abbia ridefinito l'operatore `operator()`.
E' possibile includere più definizioni di `operator()`. I funtori possono avere anche delle variabile all'interno. Non viene più chiamata funzione, ma **funzionale**.

## Template

L'utilizzo di template e delle programmazione generica in C++ permette di aumentare livello di generalizzazione dei propri programmi mescolando gli approcci.

```c++
template <typename F>
void some_function(F& f){
    f(); // funzionale o puntatore a funzione
}
```

## Funzioni minimali

Funzioni minimali `std::for_each(v.begin(), v.end(), print)`, per questo motivo si introducono le lambda function `std::for_each(v.begin(), v.end(), [](int i) {std::cout << i << " ";})`

Se si ha una sola istruzione di return il valore si può inferire, altrimenti bisogna indicarlo con `[]() -> tipo`

Le parentesi quadre sono le variabili che vengono catturate per valore, per riferimento, per catturare tutto `[&]`.

Una lambda che cattura dei valori viene chiamata chiusura. I valori catturati non possono essere modificati a meno che non venga indicata la parola chiave `mutable`.

`std::function<R(Args...)>`

```c++
std::function<int(int)> makeFunction(int k){
    return [k](int i) { return k*i; };
}
```

## `std::function`

Il meccanismo di `std::function<R(Args...)` permette di modellare una funzione che riceva come parametri Args e restituisca un valore di tipo R. Occorre includere il file `<functional>`

Si basa sul polimorfismo, per questo motivo richiede più complessità, poichè bisogna passare dalla V-Table.

## Spunto di riflessione

```c++
std::function <bool(float)> interval(float low, float high){
    std::function <bool(float)> fun = [low, high](float value) {
        return
                value >= low && value < high;
    };

    return fun;
}

int main() {
    auto between_2_10 = interval(2, 10);

    std::cout << between_2_10(15) << std::endl;
}
```