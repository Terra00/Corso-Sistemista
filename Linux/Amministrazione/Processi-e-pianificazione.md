# gestione dei processi

## comandi


Aggiungere a `head` `-n 6` per i primi 5 processi, il 6 è per escludere la linea di intestazione

`ps ax --sort=-%mem | head`  mostra i primi 9 processi per occupazione di memoria.

`ps ax --sort=-%cpu | head`  mostra i primi 9 processi per utilizzo di CPU

esempio di output:

```text
 ps -aux --sort=-%cpu | head
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
manager     3556  250  0.2   9760  4400 pts/1    R+   09:11   0:00 ps -aux --sort=-%cpu
root        3528  1.5  0.5  17828  9164 ?        Ss   09:11   0:00 /usr/lib/systemd/systemd-hostnamed
manager     3523  1.4  0.4  15556  7996 ?        S    09:11   0:00 sshd-session: manager@pts/1
root        3520  0.6  0.6  15460 10420 ?        Ss   09:11   0:00 sshd-session: manager [priv]
manager     3524  0.6  0.3   8568  5388 pts/1    Ss   09:11   0:00 -bash
root           1  0.4  2.1  44864 36788 ?        Ss   08:39   0:09 /usr/lib/systemd/systemd --switched-root --system --deserialize=43 rhgb
root        3141  0.3  5.1 1355176 88940 ?       Ssl  08:49   0:04 /usr/sbin/tailscaled --state=/var/lib/tailscale/tailscaled.state --socket=/run/tailscale/tailscaled.sock --port=41641
root        1181  0.2  0.4 241828  8456 ?        Ssl  08:39   0:05 /usr/bin/vmtoolsd
root        1172  0.1  2.5 130928 42816 ?        Ssl  08:39   0:02 /usr/bin/python3 -sP /usr/sbin/firewalld --nofork --nopid
```

`ps -o pid,ppid,pgid,stat,cmd` il flag **-o** seleziona le colonne da visualizzare


`pstree -p` mostra l'albero dei processi unitamente al PID (Process ID)

```text
$ pstree -p
systemd(1)─┬─NetworkManager(1310)─┬─{NetworkManager}(1312)
           │                      ├─{NetworkManager}(1313)
           │                      └─{NetworkManager}(1314)
           ├─VGAuthService(1179)
           ├─agetty(1372)
           ├─anacron(3447)
           ├─atd(1351)
           ├─auditd(1106)─┬─sedispatch(1108)
           │              ├─{auditd}(1107)
           │              └─{auditd}(1109)
           ├─chronyd(1167)
           ├─crond(1352)
```

`top` monitoraggio dei processi in tempo reale (usare il tasto `h` help per le opzioni)

```text
top - 09:17:44 up 38 min,  2 users,  load average: 0.01, 0.01, 0.00
Tasks: 269 total,   1 running, 268 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.4 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.2 hi,  0.2 si,  0.0 st
MiB Mem :   1672.4 total,    879.1 free,    564.1 used,    383.6 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   1108.3 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   3523 manager   20   0   15556   7996   5728 S   1.0   0.5   0:00.64 sshd-session
   1340 root      20   0  256620  27572  12956 S   0.3   1.6   0:01.57 tuned
   3141 root      20   0 1355176  88940  26516 S   0.3   5.2   0:05.03 tailscaled
   3363 root      20   0       0      0      0 I   0.3   0.0   0:00.38 kworker/0:0-events
   3571 root      20   0       0      0      0 I   0.3   0.0   0:00.15 kworker/1:0-events
   3619 manager   20   0   10208   5564   3376 R   0.3   0.3   0:00.07 top
      1 root      20   0   44864  36788  10564 S   0.0   2.1   0:10.07 systemd
```

## possibili significati della colonna STAT


| Stato | Significato | Cosa comporta |
|---|---|---|
| `R` | in esecuzione o pronto | normale |
| `S` | in attesa interrompibile | **lo stato della grande maggioranza dei processi** |
| `D` | in attesa **non** interrompibile (tipicamente I/O su disco) | non risponde ai segnali, nemmeno a `SIGKILL` |
| `T` | fermato | tipicamente dopo `Ctrl+Z` |
| `Z` | zombie | terminato, ma il padre non ha ancora raccolto l'esito |

## Foreground e Background

`&` al termine di un comando lancia il comando in background lasciando libero il terminale.
`fg` porta un job in forreground
`bg` porta un job in background
`jobs` elenca i task in esecuzione

```sh
sleep 300 &                 # avvio in background: la shell restituisce subito il prompt
jobs                        # elenco dei job DI QUESTA shell
fg %1                       # riporta in primo piano il job 1
# Ctrl+Z                    # sospende: il processo passa in stato T
bg %1                       # lo fa riprendere, ma in background
```


- semplice procedura da lanciare per le prove>
-
```bash
#!/bin/bash

echo "Ore: `date '+%H%M%S'`"
sleep 500
echo "Ore: `date '+%H%M%S'`"
```

## Terminare i processi — `kill`, `pkill`, `killall`

Riferimento: `kill` di bash e di **procps-ng**, `pkill`/`pgrep` di procps-ng,
`killall` di **psmisc** (Fedora, RHEL, Debian, Ubuntu).

### Il principio: non si «uccide», si manda un segnale

Nessun comando di questa famiglia termina direttamente un processo. Tutti
**consegnano un segnale** al processo, tramite il kernel; è poi il processo a
decidere che cosa farne, entro tre possibilità:

- eseguire l'azione predefinita del segnale (quasi sempre: terminare);
- **catturarlo** con un proprio gestore (per esempio: chiudere i file, salvare lo
  stato, poi uscire, oppure rileggere la configurazione e continuare);
- **ignorarlo**.

Fanno eccezione due segnali, `SIGKILL` e `SIGSTOP`, che il processo **non può**
né catturare né ignorare: li applica il kernel. Il nome `kill` è storico e
fuorviante: `kill` è il comando che «manda segnali», e terminare è solo il suo
uso più frequente.

### I segnali che servono

| N. | Nome | Tasto / origine | Azione predefinita | Catturabile | Uso tipico |
|---|---|---|---|---|---|
| 1 | `SIGHUP` | chiusura del terminale | termina | sì | per i demoni, per convenzione: **rileggere la configurazione** |
| 2 | `SIGINT` | `Ctrl+C` | termina | sì | interruzione da tastiera |
| 3 | `SIGQUIT` | `Ctrl+\` | termina con *core dump* | sì | interruzione con immagine di memoria, per il debug |
| 9 | `SIGKILL` | — | termina **subito** | **no** | ultima risorsa |
| 15 | `SIGTERM` | — | termina | sì | **richiesta ordinata di chiusura**; è il default di `kill` |
| 18 | `SIGCONT` | `fg`, `bg` | riprende | sì | far ripartire un processo fermato |
| 19 | `SIGSTOP` | — | ferma (stato `T`) | **no** | sospensione forzata |
| 20 | `SIGTSTP` | `Ctrl+Z` | ferma (stato `T`) | sì | sospensione da tastiera |
| 10 · 12 | `SIGUSR1` · `SIGUSR2` | — | termina | sì | significato definito dal programma (es. `dd` stampa l'avanzamento con `SIGUSR1`) |

I numeri valgono per x86 e ARM; su altre architetture alcuni cambiano. Il
**nome** è sempre portabile, il numero no: negli script si scrive `-TERM`, non
`-15`. L'elenco completo della macchina si ottiene con `kill -l`.

Il segnale si può indicare in tre forme equivalenti:

```sh
kill -TERM 1234
kill -SIGTERM 1234
kill -15 1234
kill -s TERM 1234      # forma POSIX
```

### sequenza corretta di kill e verifica

```sh
kill 1234              # per default SIGTERM: il processo chiude file e connessioni ed esce
sleep 5
kill -0 1234 && echo "ancora vivo"
kill -KILL 1234        # solo se SIGTERM non è bastato kill -9 1234
```

`SIGKILL` subito è un errore di metodo: il processo non ha modo di salvare lo
stato, di cancellare i file temporanei, di rilasciare i lock. Conseguenze tipiche:
file di lock orfani (un servizio che poi «non riparte»), file scritti a metà,
figli che restano vivi senza padre.

`kill -0` non manda alcun segnale: verifica soltanto che il processo esista e
che l'utente abbia il permesso di segnalarlo. Codice d'uscita `0` = esiste.

### altri utilizzi di kill

```sh
kill 1234                  # SIGTERM a un processo
kill 1234 1240 1301        # a più processi
kill -HUP 812              # rilettura della configurazione di un demone
kill -STOP 1234            # sospende
kill -CONT 1234            # riprende
kill %1                    # al job 1 della shell corrente (vedi `jobs`)
kill -- -1234              # a tutto il GRUPPO di processi 1234 (PID negativo)
```

### `pgrep` e `pkill` — per nome o per attributo

`pgrep` e `pkill` hanno le **stesse opzioni** di selezione: `pgrep` elenca,
`pkill` segnala. La regola pratica è **provare sempre la selezione con `pgrep`
prima di lanciare `pkill`**.

```sh
pgrep sshd                 # PID dei processi il cui nome contiene "sshd"
pgrep -l sshd              # PID e nome
pgrep -a sshd              # PID e riga di comando completa (procps-ng)
pkill sshd                 # SIGTERM agli stessi processi
```

**Opzioni di selezione**

| Opzione | Significato |
|---|---|
| (nessuna) | il modello è un'**espressione regolare** confrontata con il **nome** del processo (max 15 caratteri) |
| `-x` | corrispondenza **esatta** del nome, non parziale |
| `-f` | confronta con la **riga di comando intera**, non con il solo nome |
| `-u utente` | solo i processi di quell'utente (effettivo) |
| `-U utente` | solo i processi di quell'utente (reale) |
| `-t pts/1` | solo i processi legati a quel terminale |
| `-P PID` | solo i figli di quel processo |
| `-n` · `-o` | solo il più recente · solo il più vecchio |
| `-v` | inverte la selezione |
| `-c` | (`pgrep`) conta invece di elencare |
| `-SEGNALE` | (`pkill`) segnale da mandare, es. `pkill -HUP nginx` |

#### Esempi

```sh
pkill -x top               # solo "top", non "topgrade" né "htop"
pgrep -af 'python3 server' # prova della selezione per riga di comando
pkill -f 'python3 server'  # poi il segnale
pkill -u studente1         # tutti i processi di un utente (logout forzato)
pkill -KILL -t pts/2       # tutto ciò che gira sul terminale pts/2
pkill -HUP -o sshd         # solo il sshd più vecchio: il demone padre
```

***trappole di `pkill`***

1. **Corrispondenza parziale.** `pkill ssh` colpisce `sshd`, `ssh-agent`,
   `sshd-session`: su una macchina raggiunta via SSH, chiude anche la propria
   sessione. Si usa `-x`, oppure un'ancora: `pkill '^ssh$'`.
2. **Nome troncato a 15 caratteri.** Il nome del processo che il kernel conserva
   è lungo al massimo 15 caratteri: `pgrep -x NetworkManager-dispatcher` non
   trova nulla. Si usa `-f`.
3. **`-f` trova sé stesso.** In uno script, `pkill -f 'backup.sh'` colpisce
   anche la shell che contiene quella stringa nella propria riga di comando.
   `pgrep` e `pkill` escludono sé stessi, ma non il processo padre.


### `killall` — per nome esatto

```sh
killall nginx              # SIGTERM a tutti i processi di nome esatto "nginx"
killall -HUP nginx
killall -u studente1       # tutti i processi dell'utente
killall -i nginx           # chiede conferma per ciascuno
killall -w nginx           # attende che siano davvero terminati
```

## crontab e schedulazione dei processi

- `crontab -e` porta il crontab file in `vi` per l'editing
- `crontab -l` elenca i crontab attivi

Formato per l'utente:

```
┌──────── minuto (0-59)
│ ┌────── ora (0-23)
│ │ ┌──── giorno del mese (1-31)
│ │ │ ┌── mese (1-12)
│ │ │ │ ┌ giorno della settimana (0-7; 0 e 7 sono entrambi domenica)
│ │ │ │ │
* * * * *  comando
```

Formato per root (di sistema):

```
┌──────── minuto (0-59)
│ ┌────── ora (0-23)
│ │ ┌──── giorno del mese (1-31)
│ │ │ ┌── mese (1-12)
│ │ │ │ ┌ giorno della settimana (0-7; 0 e 7 sono entrambi domenica)
│ │ │ │ │
* * * * *  utente comando
```

Esempi:
```text
#  *  *  *  *  *    command_to_execute
#  m  h  d  m  *    command_to_execute
* * * * * echo "Messaggio da crontab $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
*/5 * * * * echo "ogni minuto multiplo di 5" >> /tmp/crontab_demo.log
0 2 * * * echo "Ogni notte alle 02:00" >> /tmp/crontab_demo.log
30 8 * * 1-5 echo "Ogni giorno da lunedì a venerdì alle 08:30" >> /tmp/crontab_demo.log
0 8 1 * * echo "Il primo giorno di ogni mese alle  08:00" >> /tmp/crontab_demo.log

```

### Esempio reale

```text
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * command to be executed
*/8 * * * * echo "Messaggio da crontab */8 $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
```

con output tramite `cat /tmp/crontab_demo.log`

```text
Messaggio da crontab */8 14-56-01
Messaggio da crontab */8 15-00-01
Messaggio da crontab */8 15-08-01
Messaggio da crontab */8 15-16-01
Messaggio da crontab */8 15-24-01
Messaggio da crontab */8 15-32-02
Messaggio da crontab */8 15-40-01
Messaggio da crontab */8 15-48-01
...
attesi
Messaggio da crontab */8 15-56-01
Messaggio da crontab */8 16-00-01
Messaggio da crontab */8 16-08-01
```
