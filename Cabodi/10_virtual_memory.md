# Virtual Memory

Il codice deve essere e seguito, ma serve tutto il programma in memoria? No, non serve tutto.

E' anche plausibile che certe parti servono insieme. Vogliamo misurare quale è la capacità di utilizzare un programma parzialmente memorizzato in RAM.

Cosa significa Virtual Memory? Lo spazio virtuale c'è tutto, ma il fisico è solo di alcune parti. Per questo viene chiamato virtuale e non logico.

Lo spazio di memoria virtuale è la visione logica di come il processo è indirizzato in memoria. QUesto è concettualemente contiguo, ma in MMU vnegono trasformati in indirizzi fisici. MMU deve effettuare il mapping. Molto spesso è paginazione a richiesta.

Solitamente si vanno a inserire stack negli spazi alti e code, data e heap in basso.

## Paginazione a Richiesta

Carichiamo solo una memoria quando viene richiesta:

- Meno I/O
- Meno memoria necessaria
- Risposta più veloce
- Più utenti

Il processo è simile a quello dello swapping.

**Lazy Swapper**: una pagina viene swappata in memoria solo quando necessaria (swapper per pagine = pager)

C'è bisogno di inserire nuove funzionalità nella MMU.

### Valid-Invalid bit

Valida: c'è come frame in RAM
Invalida: non c'è come frame in RAM.

Se viene trovato i, viene scatenato un page-fault.

Procedimento page-fault:

1. Guarda un'altra tabella e decide se è un riferimento non valiso o se non è semplicemente in memoria.
2. Trovare Frame liberi
3. Swapping pagine in frame con un'operazione schedulata su disco
4. Resettare la tabella per indicare il riferimento come valido
5. Riavviare l'istruzione che ha causato il page fault.

Il caso estremo è quando viene avviato un processo senza pagine in memoria. Il sistema operativo mette il puntatore alla prima istruzione e parte il page fault. Questo è demand paging puro.

In generale potrebbe essere utile inserire una serie di pagine in memoria: Locality of reference.
