# Processi

Un processo è un programma (eseguibile) in esecuzione: il codice caricato in memoria più lo stato che il kernel gli associa (PID, memoria propria, file aperti, utente, stato). Un processo possiede una quota di CPU, di memoria e di risorse.
Nasce sempre da un altro processo: il padre (PPID), e termina restituendo un codice d'uscita o retirn code (RC). ($? in bash)
Lo stesso programma lanciato due volte dà due processi distinti, ciascuno con il proprio PID.

`ps ax` stampa l'elenco dei processi attivi

`ps` elenca i processi dell'utente corrente


## come controllo i processi `top` (di procps-ng)

### cosa evidenzia

`top` è una fotografia dei processi, ripetuta a intervalli regolari (3 secondi
per default). Lo schermo ha due parti: in alto il **riepilogo** del sistema, sotto
la **tabella dei processi**, ordinata per `%CPU` decrescente.

```text
top - 09:17:44 up 38 min,  2 users,  load average: 0.01, 0.01, 0.00
Tasks: 269 total,   1 running, 268 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.4 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.2 hi,  0.2 si,  0.0 st
MiB Mem :   1672.4 total,    879.1 free,    564.1 used,    383.6 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   1108.3 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   3523 manager   20   0   15556   7996   5728 S   1.0   0.5   0:00.64 sshd-session
```

| Riga | Campo | Significato |
|---|---|---|
| 1 | `09:17:44` | ora corrente |
| | `up 38 min` | tempo dall'avvio (lo stesso di `uptime`) |
| | `2 users` | sessioni aperte |
| | `load average` | media dei processi in coda (in esecuzione o pronti, più quelli in stato `D`) su **1, 5 e  e 15 minuti** |
| 2 | `Tasks` | processi totali, suddivisi per stato: `running`, `sleeping`, `stopped`, `zombie` |
| 3 | `us` | tempo di CPU speso in codice utente |
| | `sy` | tempo speso nel kernel |
| | `ni` | codice utente di processi con *nice* positivo |
| | `id` | CPU inattiva |
| | `wa` | CPU inattiva **in attesa di I/O** (disco lento, NFS bloccato) |
| | `hi` · `si` | interrupt hardware · interrupt software |
| | `st` | *steal*: tempo sottratto dall'hypervisor (solo in macchine virtuali) |
| 4 | `Mem` | RAM: `total`, `free`, `used`, `buff/cache` |
| 5 | `Swap` | swap: `total`, `free`, `used`; in coda **`avail Mem`**, la memoria realmente disponibile per nuovi programmi |

**Come si legge il load average.** Il valore va confrontato con il numero di CPU
(`nproc`): con 2 CPU, un load di `2.00` significa CPU piene; `4.00` significa che
metà dei processi pronti aspetta il turno. La terna mostra la tendenza: `3.0, 1.5,
0.5` è un carico in salita, `0.5, 1.5, 3.0` un carico in discesa.

**Come si legge la memoria.** `free` basso **non** è un allarme: Linux usa la RAM
libera come cache del disco (`buff/cache`) e la restituisce quando serve. Il
numero da guardare è `avail Mem`. L'allarme vero è lo swap `used` che cresce.

**Come si legge la CPU.** `us` alto: un programma sta calcolando. `sy` alto: molte
chiamate al kernel. `wa` alto con `id` alto: la CPU è ferma perché aspetta il
disco, e il collo di bottiglia non è la CPU.


### Le colonne dei processi

| Colonna | Significato |
|---|---|
| `PID` | identificativo del processo |
| `USER` | utente effettivo proprietario |
| `PR` | priorità vista dal kernel (`20` = normale; `rt` = tempo reale) |
| `NI` | valore *nice*, da `-20` (più priorità) a `19` (meno priorità) |
| `VIRT` | memoria virtuale totale, in KiB: include librerie, file mappati, memoria riservata e mai usata |
| `RES` | memoria **residente**, cioè effettivamente in RAM: è il numero che conta |
| `SHR` | parte di `RES` condivisa con altri processi (librerie) |
| `S` | stato: `R` · `S` · `D` · `T` · `Z` · `I` (vedi la tabella in calce; `I` è un thread del kernel inattivo) |
| `%CPU` | quota di **una** CPU usata dall'ultimo aggiornamento; un processo multi-thread può superare `100` |
| `%MEM` | `RES` in percentuale della RAM totale |
| `TIME+` | tempo di CPU consumato dall'avvio del processo, al centesimo di secondo |
| `COMMAND` | nome del comando (riga di comando intera con il tasto `c`) |


#### Stato dei processi

| Stato | Significato | Cosa comporta |
|---|---|---|
| `R` | in esecuzione o pronto | normale |
| `S` | in attesa interrompibile | **lo stato della grande maggioranza dei processi** |
| `D` | in attesa **non** interrompibile (tipicamente I/O su disco) | non risponde ai segnali, nemmeno a `SIGKILL` |
| `T` | fermato | tipicamente dopo `Ctrl+Z` |
| `Z` | zombie | terminato, ma il padre non ha ancora raccolto l'esito |

### Tasti interattivi

`top` si comanda con un tasto alla volta, senza `Invio`. Le lettere maiuscole e
minuscole sono comandi **diversi**.

#### Essenziali

| Tasto | Azione |
|---|---|
| `h` oppure `?` | aiuto |
| `q` | esce (quit) |
| `Spazio` | aggiorna subito |
| `d` oppure `s` | cambia l'intervallo di aggiornamento (chiede i secondi) |
| `kk` | invia un segnale: chiede il PID, poi il segnale (default `15`, SIGTERM) |
| `r` | *renice*: chiede il PID, poi il nuovo valore nice |


####  Ordinamento

| Tasto | Ordina per |
|---|---|
| `P` | `%CPU` (default) |
| `M` | `%MEM` |
| `T` | `TIME+` |
| `N` | `PID` |
| `R` | inverte l'ordine corrente |
| `<` · `>` | sposta l'ordinamento sulla colonna a sinistra · a destra |


#### Filtri

| Tasto | Azione |
|---|---|
| `u` | mostra i processi di un solo utente (chiede il nome; `Invio` vuoto li rimostra tutti) |
| `o` | filtro su una colonna, es. `COMMAND=sshd`, `%CPU>1.0`, `!USER=root` |
| `=` | toglie tutti i filtri |
| `i` | nasconde i processi inattivi |
| `n` oppure `#` | limita il numero di righe mostrate |
| `L` · `&` | cerca una stringa · occorrenza successiva |


#### Viste

| Tasto | Azione |
|---|---|
| `c` | nome del comando ↔ riga di comando completa |
| `V` | vista ad albero (padre e figli, come `pstree`) |
| `H` | mostra i singoli thread invece dei processi |
| `1` | una riga `%Cpu` per ciascuna CPU |
| `t` · `m` | cambia la resa delle righe CPU · memoria (testo, barre, nascoste) |
| `l` | mostra o nasconde la riga del load average |
| `E` · `e` | cambia l'unità della memoria nel riepilogo · nella tabella (KiB, MiB, GiB, …) |
| `f` | gestione delle colonne: `Spazio` o `d` le mostra o nasconde, `s` sceglie quella di ordinamento, `q` esce |
| `x` · `y` | evidenzia la colonna di ordina mento · i processi in esecuzione |
| `z` | colori sì/no |
| `I` | modo *Irix*: `%CPU` per singola CPU (default) ↔ diviso per il numero di CPU |
| `W` | salva la configurazione corrente in `~/.config/procps/toprc` |


## Il `top` di busybox (Alpine)

Il `top` di busybox è una versione ridotta, in cui molte funzioni sopra elencate
mancano:

- il riepilogo ha una forma diversa: righe `Mem:` e `CPU:` e il load average, ma
  niente riga `Tasks` né `avail Mem`;
- le colonne sono meno (`PID PPID USER STAT VSZ %VSZ %CPU COMMAND`): non ci sono
  `RES`, `PR`, `NI`;
- la tastiera si limita all'ordinamento e a poche commutazioni; **mancano `k`,
  `r`, `u`, `o` e `f`**, quindi segnali e *renice* si danno da shell con `kill`
  e `renice`;
- da riga di comando restano `-b`, `-n` e `-d`.

L'elenco esatto delle opzioni va controllato sulla compilazione di busybox per la distro in esame.

