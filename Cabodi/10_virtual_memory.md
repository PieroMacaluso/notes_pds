# Virtual Memory

Il codice deve essere e seguito, ma serve tutto il programma in memoria? No, non serve tutto.

E' anche plausibile che certe parti servono insieme. Vogliamo misurare quale è la capacità di utilizzare un programma parzialmente memorizzato in RAM.

Cosa significa Virtual Memory? Lo spazio virtuale c'è tutto, ma il fisico è solo di alcune parti. Per questo viene chiamato virtuale e non logico.

Lo spazio di memoria virtuale è la visione logica di come il processo è indirizzato in memoria. QUesto è concettualemente contiguo, ma in MMU vnegono trasformati in indirizzi fisici. MMU deve effettuare il mapping. Molto spesso è paginazione a richiesta.