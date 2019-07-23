# Librerie

- [Librerie](#Librerie)
  - [Librerie a collegamento statico](#Librerie-a-collegamento-statico)
    - [Librerie a collegamento dinamico](#Librerie-a-collegamento-dinamico)

Quando realizziamo un programma questo è un insieme di sorgenti ognuno compilato in maniera separata. I file header permettono di dichiarare simboli definiti esternamente.

Il compilatore produce moduli oggetto. In una compilation unit possono rimanere dei buchi che possono essere riempiti durante la fase di linking.

Il linker va ad unire più moduli in un eseguibile e va a riempire tutte le lacune con le funzioni di altre unità di compilazione o di libreria richieste.

Le librerie ci permettono di avere moduli oggetto archiviati in un file per evitare di scrivere funzioni base. Solitamente vengono utilizzate per radunare funzioni e strutture dati pertinenti ad uno stesso dominio.

Le librerie contengono le variabili e le funzioni esportate (globali), quelle private che sono accessibili solo all'interno della libreria, ma anche costanti e altre risorse.

L'uso di una libreria richiede due fasi:

- Identificazione dei moduli necessari e il loro caricamento in memoria
- Aggiornamento degli indirizzi per puntare correttamente ai moduli caricati.

Le due operazioni possono essere fatte in due modi diversi:

- Durante il collegamento (linker), ovvero collegamento statico.
- Possiamo rimandare la fase del caricamento dei moduli al loader, ovvero al caricamento del programma. Guarda nella import table e vede cosa manca. Però potrebbe succedere che non ci sia, se non la trova non funziona più. (Windows `.dll`, Linux `.so`).
- Può anche avvenire durante l'esecuzione e il programma stesso potrà decidere lui quando avrà la necessità. (esempio Plug-in)

Quindi esistono le librerie
- Statiche
- Dinamiche o condivise
  - Collegate dinamicamente
  - Caricate dinamicamente

## Librerie a collegamento statico

Contengono funzionalità collegate staticamente al codice binario cliente in fase di compilazione. La libreria statica è un file archivio. Contiene un insieme di file object creati dal compilatore a partire dal codice sorgente e l'impacchettamento viene effettuato dall'archiver.

Il linker identifica in quali moduli della libreria si trovano le funzioni richiamate dal programma, carica solo quelle necessarie. Alla fine del processo i moduli sono annegati nel codice sorgente.

**Vantaggi**:

- Codice delle librerie nell'eseguibile
- Non ci sono dubbi sulla versione della libreria utilizzata

**Svantaggi**:

- Gli stessi contenuti sono presenti in processi differenti, bassa efficienza
- L'uso delle librerie statiche comporta una riduzione della modularità del codice. Ogni applicazione che ne fa uso porta a una ricompilazione.

In Linux come archiver si usa il tool ar e avranno `.a` come estensione. Per quanto riguarda Windows hanno estensione `.lib` e sono create dal tool lib.

### Librerie a collegamento dinamico

Il file eseguibile non contiene i moduli della libreria. Vengono caricati successivamente nello spazio del processo attraverso il loader (Linux `ld.so` libreria ELF condivisa). Quando l'applicativo viene mappato in memoria viene caricato tutto ciò che serve. In Windows esiste il dynamic linker che fa parte del kernel Windows stesso.

E' possibile controllare in maniera esplicita il caricamento delle librerie condivise, può farne richiesta solo quando è necessario

Linux:
- `dlopen()` rende un object accessibile al programma
- `dlsym()` recupera l'indirizzo di un simbolo di un file object aperto
- `dlerror()` se una delle operazioni fallisce possiamo ottenere l'errore
- `dlclose()` non mi interessa più usare la dll specifica, non viene più mappata e viene liberata se non serve a nessun'altro.

Le `dll` in windows hanno funzioni di caricamento in ingresso e in uscita (`DllMain`). Per caricare le librerie utilizziamo `LoadLibrary(..)` ci restituisce una handle che è un numero che riflette dove si trova nello spazio di indirizzamento.

I parametri di `DllMain` sono:

- HINSTANCE h, handle, l'indirizzo a partire dal quale è stata mappata la DLL.
- r indica il tipo di evento (`DLL_PROCESS_ATTACH`, `DLL_PROCESS_DETACH`).

Le reason non si considera per problematiche di concerrenza in windows.

I dati nelle DLL non venono condivise anche se le dll vengono mappate nello stesso spazio di indirizzamento.

Se vogliamo utilizzare le dll dobbiamo fare riferimento alle funzioni che contengono, ma usare delle direttive chiave `dllimport`, `dllexport` se usiamo visual studio.
Altrimenti dobbiamo creare un file `.def` per poter costruire la export table.

*Esempio implementazione slide 28, 29*

Con import significa che il linker non deve pensarci, altrimenti andiamo a esportare la funzione.

Se abbiamo cpp dobbiamo stare attenti, per permettere che sia linkabile il nome effettivo è un misto del nome che gli abbiamo dato e i parametri di ritorno, il nome viene sporcato. Dobbiamo disabilitare il nostro name Mangling inserento extern "C".