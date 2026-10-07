# Journalctl

## Chi può leggere il journal

Un utente normale vede solo i messaggi dei **propri** processi. Il journal
completo lo vedono root e i membri dei gruppi `wheel` e `adm`; altrimenti si
premette `sudo`. Se `journalctl` risponde con un avviso del tipo
`Hint: You are currently not seeing messages from other users and the system`,
si tratta di questo.


## Lettura di base

| Comando | Effetto |
|---|---|
| `journalctl` | tutto, dal più vecchio, dentro `less` (`G` va in fondo, `/` cerca, `q` esce) |
| `journalctl -e` | come sopra, ma si apre **in fondo** |
| `journalctl -n 50` | le ultime 50 righe |
| `journalctl -f` | segue i nuovi messaggi in tempo reale (`Ctrl+C` per uscire); come `tail -f` |
| `journalctl -r` | ordine inverso: dal più recente |
| `journalctl --no-pager` | senza `less`, per redirigere o incollare |
| `journalctl -n 50 -f` | visualizza le ultime 50 righe e segue i nuovi messaggi in tempo reale |


## Filtri

**Per servizio (unità systemd)**

```sh
journalctl -u sshd                  # il servizio sshd
journalctl -u crond -u chronyd      # più servizi insieme
journalctl -u sshd -f               # seguire sshd in tempo reale
```

**Per avvio**

```sh
journalctl --list-boots             # elenco degli avvii registrati
journalctl -b                       # solo l'avvio corrente
journalctl -b -1                    # l'avvio precedente (utile dopo un blocco o un riavvio inatteso)
```

**Per priorità**

```sh
journalctl -p err                   # err e più gravi
journalctl -p warning -b            # avvisi e più gravi, dall'ultimo avvio
journalctl -p 3                     # equivalente di -p err
```

| N. | Nome | Significato |
|---|---|---|
| 0 | `emerg` | sistema inutilizzabile |
| 1 | `alert` | intervento immediato |
| 2 | `crit` | condizione critica |
| 3 | `err` | errore |
| 4 | `warning` | avviso |
| 5 | `notice` | normale ma significativo |
| 6 | `info` | informativo |
| 7 | `debug` | diagnostica |

`-p` indica una **soglia**: mostra quel livello e tutti quelli più gravi. Nel
terminale gli errori compaiono in rosso, gli avvisi in giallo.


**Per processo, programma, utente**

```sh
journalctl _PID=1234                # un processo
journalctl _COMM=sudo               # un programma (nome del comando)
journalctl _UID=1000                # un utente, per UID (id -u manager)
journalctl /usr/sbin/sshd           # un eseguibile, per percorso
```

## I file di testo in `/var/log/`

`rsyslog` riceve i messaggi da journald e li smista per argomento in file di
testo. Su Rocky i principali sono:

| File | Contenuto |
|---|---|
| `/var/log/messages` | il registro generale: quasi tutto, **tranne** autenticazione, posta e cron |
| `/var/log/secure` | autenticazione: login, `ssh`, `sudo`, `su`, cambi di password |
| `/var/log/cron` | esecuzioni di cron e anacron |
| `/var/log/maillog` | posta |
| `/var/log/boot.log` | messaggi dei servizi all'avvio |
| `/var/log/dnf.log` · `dnf.rpm.log` | installazioni, aggiornamenti e rimozioni di pacchetti |
| `/var/log/audit/audit.log` | eventi di sicurezza di `auditd`, compresi i rifiuti di **SELinux** |
| `/var/log/lastlog` · `wtmp` · `btmp` | binari: accessi riusciti e falliti, si leggono con `lastlog`, `last`, `sudo lastb` |

Quasi tutti sono leggibili solo da root.

```sh
sudo tail -n 50 /var/log/messages          # ultime 50 righe
sudo tail -f /var/log/secure               # in tempo reale (Ctrl+C per uscire)
sudo less +F /var/log/messages             # come tail -f, ma Ctrl+C torna a less per scorrere e cercare
sudo grep -i "failed password" /var/log/secure
sudo grep CMD /var/log/cron                # che cosa ha lanciato cron
```

**Rotazione.** I file non crescono all'infinito: `logrotate` li ruota ogni
settimana (configurazione in `/etc/logrotate.conf` e `/etc/logrotate.d/`). I
file vecchi prendono la data nel nome: `messages-20260927`. Per cercare anche
lì: `sudo grep "testo" /var/log/messages*`.
