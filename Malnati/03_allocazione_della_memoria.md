# Allocazione della memoria

Dangling Pointer: Puntatore che punta ad uno spazio di memoria che viene liberato. Non si può predire a che punto di memoria andrà a puntare. Conviene reinizializzarlo con NULL.

Memory Leakage = Memoria dinamica allocata attraverso malloc o new e non liberata alla fine del processo.

Wild Pointer = Puntatore dichiarato, ma non inizializzato.