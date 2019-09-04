# Main Memory

- [Main Memory](#main-memory)
  - [Introduzione](#introduzione)
    - [Protezione](#protezione)
    - [Base e Limit](#base-e-limit)
    - [Address Binding](#address-binding)
    - [Logico contro Fisico](#logico-contro-fisico)
    - [Dynamic Loading](#dynamic-loading)
    - [Dynamic Linking](#dynamic-linking)
  - [Allocazione contigua](#allocazione-contigua)
    - [Problema Allocazione](#problema-allocazione)
    - [Frammentazione](#frammentazione)
  - [Paginazione](#paginazione)
    - [Address Translation Scheme](#address-translation-scheme)
    - [Page Table implementation](#page-table-implementation)
      - [Effective Access Time](#effective-access-time)
    - [Memory Protection](#memory-protection)
    - [Pagine condivise](#pagine-condivise)
    - [Struttura Page Table](#struttura-page-table)
      - [Tabella delle Pagine Gerarchica](#tabella-delle-pagine-gerarchica)
      - [Hashed Page Tables](#hashed-page-tables)
      - [Inverted Page Table](#inverted-page-table)
  - [Swapping](#swapping)

## Introduzione

Un programma deve essere portato dal disco alla memoria per poterli essere eseguito. La memoria centrale e i registri sono le uniche unità a cui la CPU può accedere direttamente per poter agire in maniera efficiente. La **MMU** è quella parte di CPU che si interfaccia con la RAM. QUest'ultima vede sono una serie di indirizzi + richieste di lettura o indirizzi e richieste di dati e scrittura.

L'accesso ad un registro può essere fatto in un colpo di clock o ancora meno. L'accesso alla ram può essere effettuata in più cicli e può causare lo stallo.

Le memorie cache sono memorie più piccole, ma più veloci e si trovano tra la memoria centrale e i registri della CPU.

La protezione della memoria deve essere assicurata per poter assicurare operazioni corrette.

### Protezione

> Spectre e Meltdown: utilizzo speculativo di operazioni che ha creato bug.

Un processo può accedere solo a quegli indirizzi di memoria che sono nel suo address space. Nel corso del tempo sono state sperimentate diverse tecniche per garantire questa proprietà.

### Base e Limit

Per farlo nella maniera più rudimentale possibile possiamo usare i registri **base** e **limit**. E' possibile farlo andando a utilizzare due comparatori HW che lanciano una trap in caso di errore.

Le istruzioni per accedere ai registri sono privilegiate. La CPU deve sempre andare ad analizzare ad ogni accesso in memoria.

### Address Binding

Chi associa gli indirizzi ai dati? Quando i programmi vengono eseguiti, questi vengono portati in memoria, ma gli indirizzi possono essere rappresentati in maniere molto diverse all'interno della vita del programma. Nel codice sorgente abbiamo simboli, nel codice compilato andiamo a indirizzarli in indirizzi rilocabili (salta a 14 byte dopo quel punto, somma di due pezzi, chi fa la somma?). Il linker o il loader andranno a legare gli indirizzi a indirizzi assoluti.

Servono delle mappe.

In sistemi molto semplici abbiamo esecuzioni sempre negli stessi indirizzi, ma nei sistemi odierni non è così.

Il binding degli indirizzi può avvenire in:

- **Compilazione**: Codice assoluto, ma deve essere ricompilato se la locazione di inizio cambia.
- **Loading**: deve generare codice rilocabile se la memoria non è conosciuta in fase di compilazione
- **Esecuzione**: Il binding è ritardato alla fase di esecuzione se il processo può muoversi in esecuzione da un segmento ad un altro. Ha bisogno di supporto hardware per le mappe degli indirizzi.

### Logico contro Fisico

- **Indirizzo logico**: generato dalla CPU, spesso ci riferiamo a questo anche come virtuale (simili, ma sottile differenza).
- **Indirizzo fisico**: indirizzo visto dalla MMU.

Sono identici in compilazione e loading, ma sono diversi in esecuzione.

Il passaggio viene effettuato nella MMU, che mappa gli indirizzi logici su quelli fisici.

Il registro **base** diventa **relocation register**. L'utente manipola solo gli indirizzi logici, non interagisce con i fisici. Il passaggio logico/fisico è un semplice sommatore.

Se l'indirizzo logico è 100 e il relocation è 1000, l'indirizzo fisico sarà 1100. Ovviamente all'interno ci deve essere anche un controllo sul valore massimo.

### Dynamic Loading

Ma il programma è necessario che sia tutto in memoria? Si può pensare che una routine venga caricata solo quando chiamata. Questo serve per ottenere più memoria. Questo è utile per non inserire in memoria parti di codice che non utilizzeremo molto grandi.

### Dynamic Linking

Lo **static linking** è il caricamento di librerie e programma nel binario del programma.

Il linking dinamico sposta tutto in esecuzione. Spesso è molto utile per le librerie. L'aggancio dinamico avviene dopo. Se è la prima volta che verrà chiamata, si andrà a caricare la funzione legata, altrimenti si andrà a riutilizzare quella già caricata (stub).

Meglio eseguibile statico o dinamico? Dipende dalle librerie presenti sul sistema su cui andremo ad eseguire.

## Allocazione contigua

Primo metodo.

La memoria centrale è divisa in 2 partizioni. Sistema operativo residente (low memory con interrupt vector). I processi user vengono messi negli indirizzi alti. Ogni processo è conteuto in una singola sezione di memoria contigua. In questo contesto i relocation register proteggono i processi utente tra loro: il base register contiene l'indirizzo più basso e il limit register contiene il range degli indirizzi logici del processo.

Una volta continuato questo approccio ci troveremo ad avere partizioni variabili di processi con buchi nella memoria di diverse dimensioni.

### Problema Allocazione

- **First-fit**: il primo buco grande abbastanza
- **Best-fit**: le guardo tutte e prendo la più piccola
- **Worst-fit**: le guardo tutte e prendo l'intervallo più grande

Tutto dipende dal contesto, di solito sono meglio le prime in termini di velocità e utilizzo dello storage.

### Frammentazione

GLi schemi di allocazione contigua tendono a generare problemi di frammentazione. Ne esistono due tipi:

- **Frammentazione esterna**: somma totale di ciò che è disponibile, ma è fatto a pezzettini
- **Frammentazione interna**: sovrallocazione dello spazio allocato per un processo

L'utilizzo della politica first-fit rivela che dato N blocchi allocati abbiamo 0.5N blocchi persi  con la frammentazione (1/3 potrebbe essere inutilizzabile perchè troppo piccola, regola del 50%).

La frammentazione esterna può essere ridotta con **compaction**. Mescolare la memoria per piazzare tutti gli spazi liberi in un unico punto, può essere possibile solo se la relocazione è dinamica ed è fatta in esecuzione.

## Paginazione

Lo spazio di indirizzamento logico è sempre contiguo, ma quello fisico può non essere contiguo. La memoria fisica viene divisa in una serie di pezzi fissi detti **frame**. La memoria logica viene divisa in blocchi di dimensione fissa chiamati **pagine**. Tiene traccia di tutti i frame liberi. Per avviare un programma con N pagine abbiamo bisogno di N frame liberi. Abbiamo bisogno di una page table per poter mantenere corrispondenza tra le due, ma questo costa di più. Anche il backing store (pezzo di disco come salvataggio di processi in esecuzione) deve essere suddiviso in pagine. Ci può ancora essere frammentazione interna.

### Address Translation Scheme

- Page Number (p = m-n): indice nella tabella delle pagine che contiene gli indirizzi base di ogni pagina.
- Page offset (d=n): Permette di indicare l'indirizzo di memoria combinandolo con p.

Spazio 2^m e dimensione pagina 2^n.

La page table è una tabella ad accesso diretto. Ma dove sta questa tabella?

Quanto è grande?

In media la frammentazione è 1/2 frame, mentre il caso peggiore è 1 frame -1 byte (in realtà 1 frame). Conviene quindi ad avere pagine piccole? Ma per ogni pagina c'è una riga nella tabella delle pagine, quindi dobbiamo cercare un compromesso, un trade-off. (dal KB al MB)

Dobbiamo tenere anche traccia dei frame liberi in lista e se ci serve andare a prendere quel frame (estrazione in testa).

### Page Table implementation

Deve essere nella main memory

- Page-table base Register (PTBR): punta alla page table
- Page-table Length Register (PTLR): Lunghezza Page Table.

In questo schema però sono necessari due accessi in memoria, una per la page table e una per il dato. Si può risolvere questo con l'utilizzo della Translation Lookaside-Buffer (TLBs) chiamata anche memoria associativa, una specie di cache.
Comparatore accanto ad ogni dato presente nella TLB e si assume che solo uno risponda, ma questo costa. E' dentro la MMU, dentro la CPU.

Alcune TLB possono segnarsi anche a che processo si legano altrimenti flush per ogni context switch.

#### Effective Access Time

**Hit ratio** è la percentuale di volte che il page number viene trovato il TLB.

p = hit-ratio | t = tempo di accesso alla memoria

`EAT = p * t + (1-p)*t*2`

### Memory Protection

Possiamo aggiungere un bit di per indicare read-only o read-write, ma soprattutto indicare il validity bit per dire che una pagina sia nell'address space logico del processo, la pagina esiste.

Ogni violazione risulta in trap kernel.

### Pagine condivise

E' possibile che due processi abbiano due aree di memoria condivise. Una copia di codice read-only (reentrant, non ha effetti collaterali, non ha variabili globali) condivisa. Utile anche per la comunicazione tra processi.

Il privato può essere inserito arbitrariamente, per le condivise bisogna fare ulteriori considerazioni.

### Struttura Page Table

Ogni entry contiene un numero di byte e allargo un po' per i validity, ma ottengo quindi XMB. Dovrebbe avere indirizzo fisico della page table. Ma si può pensare a soluzioni per non avere la tabella delle pagine gerarchica.

#### Tabella delle Pagine Gerarchica

Spaccare tutto in tante page-table con più livelli.

#### Hashed Page Tables

Tabelle di hash, in ogni riga c'è una lista di pagine, in questo modo restringo la tabella delle pagine.

#### Inverted Page Table

Dimensionata sulla RAM. Un solo vettore per tutte le page table.

Esempio: Pagina 50, frame 87.

Scrivo p = 50 alla riga 87, quindi l'indice serve come parte iniziale. La page table sta in memoria principale ed è parallela alla RAM. Posso tenere tutti i processi insieme. Tempo di ricerca più lungo. No memory sharing.

## Swapping

E' possibile che ad un certo punto ci siano troppi processi in RAM, nonostante tutte le precauzioni utilizzate.

Si prende un processo che non è al momento utile in RAM e viene messo in **Backing Store** che è un disco più veloce largo abbastanza per poter contenere copia delle immagini per tutti gli users.

Le operazioni fatte sono **Roll In** e **Roll Out** per poter eseguire lo swapping.

Ma ovviamente il context switching costa a livello di tempo.

L'unico vero problema è che se un processo è in wait for I/O come mi comporto, posso mandarlo fuori?
Doppio buffering, copiali su un buffer di kernel e esegui.

Sui sistemi mobili non è solitamente supportato. Solitamente iOS chiede, mentre Android chiude, ma salva lo stato.

Swapping con la paginazione.
