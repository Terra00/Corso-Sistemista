# ITIL e problemi del disco

## ITIL

ITIL (*Information Technology Infrastructure Library*) è un insieme di pratiche e pubblicazioni per la gestione dei servizi IT. Non è una certificazione che attesta l'idoneità di una macchina: le certificazioni, quando previste, riguardano le persone e i percorsi formativi. Per una panoramica introduttiva: [Wikipedia](https://it.wikipedia.org/wiki/Information_Technology_Infrastructure_Library).

## Problemi col disco

1. Fare una prova non distruttiva quando lo strumento la supporta.
2. Salvare prima i file importanti.
3. Eseguire l'intervento effettivo solo dopo avere verificato il dispositivo interessato.
4. Considerare la reinstallazione soltanto se la corruzione non è recuperabile.

Su Windows, `chkdsk C: /scan` esegue una scansione online del volume; non sostituisce un backup né equivale necessariamente a ogni tipo di «dry run».
