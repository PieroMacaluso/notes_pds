# Intefacce Grafiche

La User experience è molto importante. Ogni prodotto è frutto di come il gruppo di programmatori ha interagito e ha deciso di fare il prodotto. Il fatto che questi prodotti abbiano successo o meno è molto influenzata dall'utilizzabilità e la versabilità.

Anche il software è un prodotto, ma la natura astratta delle azioni che supporta rende spesso l'interazione con lo stesso uno degli aspetti fondamentali. Occorre perciò acquisire le capacità di progettare tale interazione in modo che risulti al tempo stesso efficace (le giuste cose), efficiente (le cose giuste) e facile da imparare, ma soprattutto piacevole.

Tradizionalmente l'interazione è legata a **WIMP** (Windows, Icons, Menus, Pointer).

Il programmatore deve calarsi nei panni dell'utilizzatore imparando a guardare le cose dal loro punti di vista. Il progettista non è l'utente tipico.

Le interfacce grafiche sono il primo impatto con il cliente che può portare a un pregiudizio.

Le interfacce devono essere comprensibili. Deve presentare delle affordance, ovvero uno strumento naturale su come possa essere possibile agire con esso. L'utente deve rendersi conto che qualcosa può essere cliccato, scritto, inviato.

Serve **coerenza** tra i suggerimenti visuali e le funzionalità reali, altrimenti pulìò generare frustrazione o attrito nel cliente.

La progettazione visiva deve fare in modo che l'utente sia interessato all'interfaccia e quindi a quello che vi è dentro. Deve essere **Plesurable**, **Meaningful** e **Usable**.

I componenti grafici sono i **widget** che ci permettono di renderizzare le interfacce grafiche elementari. I widget vivono molto bene in simbiosi con le classi.

Di base esistono tre gerarchie:

- Composite: modellata ad alberi (rettangoli dentro rettangoli)
- Template: Passi intermedi
- Strategy: Ci sono delle parti che possono essere complesse e variabili, facciamolo fare a un layout manager all'esterno.

Il main inizializza tutto e infine riceve messaggi per poter operare (programmazione reattiva).

## Pattern Composite

Strutture ricorsive ad albero per realizzare la relazione "parte-tutto". Il component è la classe più astratta che può avere implementazioni foglia o ulteriori zone (pannelli della nostra interfaccia).

## Pattern Template

Definisce la struttura generale di un algoritmo demandando alcune operazioni alle sotto-classi. Queste possono adattare il proprio comportamento senza compromettere la struttura dell'algoritmo. Le classi che ereditano possono fare Override.

## Pattern Strategy

I widget demandano ad un oggetto esterno il compito di gestire funzionalità specifiche come la disposizione.

La struttura di un programma basato su interfaccia grafica è radicalmente diversa rispetto a quella di un programma a caratteri. Il programma si appoggia ad una coda di eventi. Solitamente è l'OS che inserisce in coda. L'applicazione estrae e agisce.

Le callback vengono però usate in modo sincrono. Ogni componente deve andare a manifestare l'interessa agli eventi andando a implementare la callback.

**Architettura applicativa slide 27**

- Inizializza FW
- Crea albero widget
- Registra callback
- Esegue ciclo eventi

## Gestire eventi

La relazione tra evento e callback è dinamica. Ogni framework ha un suo modo per inserire nuovi eventi nella coda. Gli algoritmi di posizionamento e visualizzazione sono inizializzati in maniera lazy poichè sono onerosi. 

La maggior parte dei messaggi sono già elaborati dai widget.

Per fare in modo che la procedura dell'evento sia troppo lunga possiamo attivare un thread (no visual tree) oppure in piccoli pezzettini.

**SOLO IL THREAD PRINCIPALE TOCCA IL VISUAL TREE**

E' prassi comune mantenere una struttura dati condivisa a cui collettivamente accedono le diverse call-back all'interno della quale sono memorizzate le informazioni raccolte dell'utente o calcolate dallo strato applicativo => **MODELLO**.

Conserva lo stato dell'applicazione.

L'albero di widget che rispecchia la situazione del modello prende il nome di **vista**. L'insieme delle callback agisce da mediatore tra vista e modello prendendo il nome di **Controllore**.

Questi tre componenti compone il paradigma MVC.

- Modello
  - Gestisce stato e dati
  - Non conosce la vista
  - Si appogge a strato di persistenza
  - Può essere riutilizzato per interfacce differenti
- Vista
  - Presentazione grafica modello
  - offre meccanismi all'utente per modellare i dati
  - Si aggiorna al cambiare del modello
  - Non memorizza dati
- Controllore
  - Intermediario
  - Traduce interazioni in richieste al modello
  - Determina quale vista deve essere presentata

Per facilitare questo compito si usa il **pattern Observer**.

Ciò che può cambiare è il **Subject** che incorpora oggetti interessati che implementano Subject che possono dire "dato aggiornato". Le viste si iscrivono ai subject.

L'iscrizione viene fatta ad un adapter che permette di filtrare solo le viste utili.

# Libreria QT

Framework multipiattaforma generale.

Scriviamo in linguaggio C++ con alcune macro poichè il meta-object compiler va a creare altro codice che permette di far funzionare il tutto.
Binding con molti linguaggi.

E' suddiviso in moduli:
- Essential: componenti base.
- Add-Ons

Estende c++ aggiungendo un sacco di funzionalità.
- Meccanismo di comunicazione con segnali e slot
- Meccanismo di proprietà
- Eventi e filtri
- Internazionalizzazione stringhe
- Timer a intervalli
- Organizzazione oggetti ad albero (no Composite).
- Puntori gestiti (QPointer): puntatore automaticamente messo a null quando liberato
- Dynamic Cast

## `QObject`

E' il cuore del modello a oggetti QT.
Da essi derivano la maggior parte delle classi del framework.
Ha tre responsabilità fondamentali:
- Gestione Memoria
- Introspezione (identificazione del tipo a runtime)
- Gestione degli eventi

Non usa Garbage collector, ma li gestisce in modo intelligente.

Ogni QObject può avere all'interno una serie di figli. Possono essere passati come parametri del costruttore il puntatore al genitore oppure tramite il `set_parent`.

Tutti vivono su heap, non ci dobbiamo preoccupare se non su quello radice. Sanno in quale thread è stato chiamato. L'albero deve essere usato solo da un thread. Si possono anche spostare `moveToThread()`.

La classe `QWidget` estende la semantica della relazione genitore figlio facendo in modo che possa essere contenuto nella sua view.

Per identificare i figli è utile andare a utilizzare un nome.

`QPointer` è il puntatore di QT che, una volta eliminato lo rende NULL. (Internamente weak::pointer).
`QSharedPointer`
`QVariant`

Ci sono le collezioni di dati che sono state implementate e che sono il duale di quelli presenti in C++. Possono essere manipolati con iteratori. Un tipo è simile a quello standard, ma c'è anche quello in stile JAVA ovvero con `it.hasNext()`.

Con quello C++ non possiamo modificare direttamente, in Java sì.

## `QWidget`

Ogni QWidget può stare sullo schermo e occupa un'area rettangolare. Possono anche essere nella stessa zona (X-Order).


Prima di poter usare i widget noi dobbiamo creare una `QApplication`.

Contiene il metodo `exec()` che non ritorna finchè non si conclude il programma. QUesto deve essere invocato per poter avviare il disegno.