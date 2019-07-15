## Introduzione al C++

Linguaggio vasto e articolaro che è compatibile con il C, ma ha estensioni esoteriche e uniche, per garantire espressività senza penalizzare prestazioni.

Obiettivi:
- Astrazione a costo nullo
- Espressività (UDT - User Defined Type) devono possedere la stessa espressività offerto dai tipi base del linguaggio
- Sostituzione: usare un UDT ovunque sia utilizzabile un tipo base.

Offre incapsulamento, Composizione, Ereditarietà e Polimorfismo. Programmazione Generica, Strutturata e Funzionale.

## Tipi base e derivati
- Riferimenti

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
 Mi dai un costruttore in cui mi fai partire da un oggetto che esiste già e mi indichi in un solo punto come copiarlo.

 `CBuffer(const CBuffer& source);`

 Per far sì che vanga vietata la copia di un oggetto possiamo andare a generarlo privato oppure  `CBuffer::CBuffer(const CBuffer& source) = delete;`

 ## Movimento
 `CBuffer(CBuffer&& source);`
&& RValue reference
Questa volta non è const poichè lo modifico. Non viene fatto di base dal compilatore, ma deve essere implementato.

## Distruttore

Non ha mai parametri e non deve mai essere chiamato, solo il compilatore lo chiama. E' il testamento dell'oggetto.

Allocazioni dinamiche con new
