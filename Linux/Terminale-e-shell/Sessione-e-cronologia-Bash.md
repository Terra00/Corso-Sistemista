# Sessione e cronologia Bash

## Registrare le attività con `script`

`script` avvia il registratore della sessione. Le attività vengono scritte sul file `typescript` nella directory corrente; se il file esiste, viene sovrascritto.

- `script` registra sul file predefinito `typescript`.
- `script nome_file` registra su `nome_file`.
- `script -a nome_file` accoda al registro precedente, creandolo se non esiste.

## Elenco dei comandi lanciati

- `history` elenca i comandi lanciati nella sessione corrente.
- `history | grep nome_comando` filtra l'elenco per nome.

Nell'output il comando più recente è in basso.

## Richiamare un comando dalla cronologia

- `!numero` esegue la voce `numero` della cronologia.
- `!stringa` esegue l'ultimo comando che inizia per `stringa`.
- `!?stringa` esegue l'ultimo comando che contiene `stringa`.

## scorciatoie di linea

- `Ctrl+A` sposta il cursore a inizio riga.
- `Ctrl+E` sposta il cursore a fine riga.
- `Ctrl+C` interrompe il comando in esecuzione.
- `Ctrl+D` invia end of file (EOF) e chiude la sessione.
