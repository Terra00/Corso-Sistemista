# Comandi di base per file e directory

Riferimento breve per le operazioni più frequenti. Per esempi ragionati su copie, collegamenti e permessi, vedere [Gestione di file e collegamenti](../File-system-e-storage/Gestione-file-e-collegamenti.md).

| Comando | Scopo | Opzioni utili |
|---|---|---|
| `ls` | Elenca il contenuto di una directory. | `-l` dettagli, `-a` inclusi i nascosti, `-h` dimensioni leggibili. |
| `tree` | Mostra una gerarchia ad albero. | `-L 2` limita la profondità, `-d` mostra solo directory. Può richiedere l'installazione del pacchetto `tree`. |
| `mkdir` | Crea directory. | `-p` crea anche i livelli mancanti, `-v` mostra le azioni. |
| `touch` | Crea un file vuoto oppure aggiorna i timestamp. | `-a` solo accesso, `-m` solo modifica. |
| `cp` | Copia file e directory. | `-r` ricorsivo, `-i` chiede prima di sovrascrivere, `-v` descrive l'azione. |
| `mv` | Sposta o rinomina. | `-i` chiede prima di sovrascrivere, `-n` non sovrascrive, `-v` descrive l'azione. |
| `rm` | Elimina file o directory. | `-i` chiede conferma; `-r` è necessario per directory. Usare con estrema cautela. |
| `file` | Riconosce il tipo di un file dal contenuto. | `-b` omette il nome, `-i` mostra il tipo MIME. |
| `cat` | Visualizza o concatena file di testo. | `-n` numera tutte le righe, `-b` solo quelle non vuote. |

> Prima di usare `rm -r`, verificare il percorso con `pwd` e `ls`. L'opzione `-f` elimina molte protezioni ed è normalmente inappropriata negli esercizi e nelle operazioni manuali.
