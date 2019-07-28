# Mass-Storage Systems

Attualmente abbiamo hard disk drives (HDD) e nonvolatile memory (NVM).

![Disco](img/11/disco.png)

I bit spesso sono in parallelo su più dischi. Una traccia è composta da numerosi settori che contengono un tot di bit. Una volta i bit a raggio differenti erano diversi, mettendo lo stesso numero di bit, oggigiorno invece hanno la medesima dimensione. Inoltre gran parte delle operazioni viene fatta dall'HDD stesso.

Per misurare la performance abbiamo i seguenti parametri:

- Transfer Rate
- Effective Transfer rate
- Tempo di seek: movimento della testina
- Latenza di rotazione: quanto tempo ci mette per ritornare sul dato che vogliamo leggere? Espressa in giri al minuto
- La latenza media è la metà


Latenza di Accesso = Tempo medio di accesso = average seek time + average latency

Il tempo di I/O è quindi = average access time + (quantità da trasferire/transfer rate) + overhead controller.

In poche parole conviene sempre trasmettere KB per non rendere vano il tempo che si perde in latenza e seek.

I dischi non volatili sono chiamati Solid-state disks (SSDs) se hanno la forma di disco. Invludono anche le USB drives. Può essere più affidabile di un hdd, ma sono più costosi e con un tempo di vita minore. solitamente hanno minore capacità, ma una velocità molto più alta. Non ci sono parti in movimento.

Il blocco in questo caso deve essere prima cancellato e poi scritto. Il numero di cancellazionei è però limitato. La vita è misurata in **drive writes per day (DWPD)**.

Il dispositivo è diviso in pagine con una tabella che permette di mappare indirizzi e pagine. Fa in modo che in quel disco un determinato indirizzo si vada ad allocarlo. L'obiettivo è bilanciare le riscritture.