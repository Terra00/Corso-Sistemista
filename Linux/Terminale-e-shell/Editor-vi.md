# vi — cheat sheet (BusyBox / Alpine) - vim (vi improved)

**Persi? `Esc` (anche due volte) → modalità comando.**

## Entrare e uscire

| Tasti | Effetto |
|---|---|
| `vi file` | apre (o crea) il file |
| `i` | inserimento prima del cursore |
| `a` | inserimento dopo il cursore |
| `o` | inserimento su riga nuova sotto |
| `shift+O` | inserimento su riga nuova sopra |
| `Esc` | torna in modalità comando |
| `:w` | salva |
| `:q` | esce |
| `:wq` | salva ed esce |
| `ZZ` | salva ed esce dalla modalità comando (scorciatoia, senza `:`) |
| `:q!` | esce SENZA salvare |

## Muoversi (in modalità comando)

| Tasti | Movimento |
|---|---|
| `h j k l` / frecce | ← ↓ ↑ → |
| `0` / `$` | inizio / fine riga |
| `gg` / `G` | prima / ultima riga |
| `:12` | vai alla riga 12 |

## Modificare (in modalità comando)

| Tasti | Effetto |
|---|---|
| `x` | cancella carattere sulla riga |
| `nx` | cancella n caratteri |
| `cw` | change word modifica la parola dove è posizionato il cursore |
| `dw` | delete word elimina la parola dove è posizionato il cursore |
| `dd` | taglia riga |
| `nnndd` | taglia nnn righe |
| `yy` | copia riga |
| `nyy` | copia n righe |
| `P` | incolla sopra |
| `p` | incolla sotto |
| `nP` | incolla sopra ripetendo n volte |
| `np` | incolla sotto ripetendo n volte |
| `u` | annulla (solo ultimo passo!) |
| `3dd`, `5x`, `2yy` | il numero moltiplica |

## Cercare e sostituire

| Comando | Effetto |
|---|---|
| `/testo` | cerca in avanti |
| `n` | occorrenza successiva |
| `N` | occorrenza precedente |
| `:s/vecchio/nuovo/` | sostituisce, prima occorrenza nella riga |
| `:s/vecchio/nuovo/g` | tutte le occorrenze nella riga |
| `:%s/vecchio/nuovo/` | la prima occorrenza nella riga in tutto il file |
| `:%s/vecchio/nuovo/g` | tutte le occorrenze nelle righe in tutto il file |

## altri comandi

| Comando | Effetto |
|---|---|
| `:set number` | attiva la numerazione di linea |
| `:set nonumber` | disattiva la numerazione di linea |

## Limiti del vi BusyBox

- `u` annulla un solo passo → salvare spesso con `:w`
- niente selezione visuale → si lavora per righe (`dd`, `yy`)
- niente colorazione, niente plugin

*Le stesse nozioni valgono su vim (che aggiunge undo multiplo, visuale, colori) e su qualunque unix.*
