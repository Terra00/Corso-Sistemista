# Il Filesystem Hierarchy Standard (FHS)

Il **Filesystem Hierarchy Standard** (FHS, versione 3.0 del 2015, mantenuta
dalla Linux Foundation) stabilisce che cosa sta in ciascuna directory di un
sistema di tipo Unix. È un riferimento, non una legge: le distribuzioni lo
seguono nelle linee generali e se ne discostano in punti precisi, alcuni dei
quali sono segnalati in questa scheda.

**Fonte.** Il testo dello standard, e il confronto con le macchine del corso.
La fusione di `/bin`, `/sbin`, `/lib` in `/usr` è stata verificata sull'host
Fedora (`ls -ld /bin /sbin /lib /lib64`) e confermata dall'autore su Rocky
(2026-09-30). Gli altri comportamenti specifici di Rocky e di Alpine sono
dichiarati nel §6 come punti da collaudare.

**Riferimenti nel corpus.** `howto/LinuxFilesystem.md` (il filesystem in
generale), `howto/backup-etc-cron.md` (un caso d'uso: dove mettere uno script
di amministrazione, dove la home di un utente di servizio).

---

## 1. Le directory della radice `/`

| Directory | Contenuto |
|---|---|
| `/bin` | comandi di base per tutti gli utenti (`ls`, `cp`, `bash`). Su Rocky e Fedora è un link a `/usr/bin` (§2) |
| `/boot` | kernel, immagine *initramfs*, configurazione del boot loader (GRUB) |
| `/dev` | file di dispositivo: dischi (`/dev/sda`, `/dev/vda`), terminali, `/dev/null`. Popolata dal kernel e da `udev` all'avvio |
| `/etc` | configurazione del sistema e dei programmi installati. Solo file di testo di configurazione: niente programmi, niente dati che cambiano da soli |
| `/home` | le home delle **persone** (`/home/anna`) |
| `/lib`, `/lib64` | librerie condivise e moduli del kernel (`/lib/modules`). `/lib64` contiene le librerie a 64 bit sui sistemi x86_64 di famiglia Red Hat. Su Rocky e Fedora sono link a `/usr/lib` e `/usr/lib64` |
| `/media` | punti di montaggio creati automaticamente per i supporti rimovibili (chiavette, DVD) |
| `/mnt` | punto di montaggio temporaneo, a disposizione dell'amministratore |
| `/opt` | pacchetti di terze parti installati in blocco, ciascuno nella sua sottodirectory (`/opt/prodotto`) |
| `/proc` | filesystem **virtuale**: il kernel vi espone i processi (`/proc/1234`) e il proprio stato (`/proc/meminfo`, `/proc/cpuinfo`). Non occupa disco |
| `/root` | la home dell'utente root. Sta fuori da `/home` perché `/home` può essere su un'altra partizione o su un disco di rete, e root deve poter lavorare anche quando non è montata |
| `/run` | dati dei processi in esecuzione dall'avvio corrente: file PID, socket, lock. È in memoria (`tmpfs`), e si svuota a ogni riavvio |
| `/sbin` | comandi di amministrazione (`fdisk`, `mkfs`, `ip`). Su Rocky e Fedora è un link a `/usr/sbin` |
| `/srv` | dati dei servizi offerti dalla macchina (`/srv/www`, `/srv/ftp`). Poco usata: molte distribuzioni mettono questi dati in `/var` (`/var/www` su Rocky) |
| `/sys` | filesystem **virtuale**, come `/proc`: il kernel vi espone dispositivi, driver e parametri. Non fa parte del FHS 3.0, ma c'è su ogni sistema Linux attuale |
| `/tmp` | file temporanei di qualunque utente. Chi la svuota, e quando, dipende dalla distribuzione (§3) |
| `/usr` | la seconda gerarchia: programmi, librerie, documentazione. Dati **statici**: cambiano solo installando o aggiornando pacchetti (§4) |
| `/var` | dati **variabili**: cambiano durante il funzionamento normale — log, code, cache, stato dei servizi (§5) |

---

## 2. La fusione di `/bin`, `/sbin`, `/lib` in `/usr`

Lo standard descrive `/bin`, `/sbin` e `/lib` come directory distinte da
`/usr/bin`, `/usr/sbin`, `/usr/lib`. La distinzione aveva una ragione storica:
`/usr` poteva stare su un disco separato, montato più tardi durante l'avvio, e
in `/bin` e `/sbin` doveva esserci l'indispensabile per arrivare fin lì o per
riparare il sistema in modalità *single-user*.

Oggi l'avvio passa per l'*initramfs*, che monta `/usr` prima di tutto il
resto, e quella ragione è venuta meno. Molte distribuzioni hanno quindi
**fuso** le directory (*usr merge*): quelle della radice sono rimaste come
collegamenti simbolici.

```bash
ls -ld /bin /sbin /lib /lib64
```

**Che cosa si vede** su Fedora (misurato sull'host) e su Rocky (confermato
dall'autore):

```
lrwxrwxrwx. 1 root root 7 ... /bin -> usr/bin
lrwxrwxrwx. 1 root root 7 ... /lib -> usr/lib
lrwxrwxrwx. 1 root root 9 ... /lib64 -> usr/lib64
lrwxrwxrwx. 1 root root 8 ... /sbin -> usr/sbin
```

Conseguenza pratica: `/bin/ls` e `/usr/bin/ls` sono **lo stesso file**, e un
percorso vale l'altro. Nei file di configurazione (crontab, regole di
`sudoers`) si scrive comunque il percorso reale, `/usr/bin/...`, che vale sia
sui sistemi fusi sia su quelli non fusi.

Su Alpine la situazione va verificata (§6): Alpine ha a lungo mantenuto le
directory separate, con i comandi di base forniti da BusyBox in `/bin`.

---

## 3. `/tmp` e `/var/tmp`: chi le svuota

| | `/tmp` | `/var/tmp` |
|---|---|---|
| Scopo | temporanei di breve durata | temporanei che devono sopravvivere a un riavvio |
| Fedora | in memoria (`tmpfs`): si svuota a ogni riavvio | su disco; ripulita da `systemd-tmpfiles` (30 giorni) |
| Rocky | su disco, di serie; ripulita da `systemd-tmpfiles` dopo 10 giorni di inattività dei file | su disco; ripulita dopo 30 giorni |

Le regole di pulizia di `systemd-tmpfiles` si leggono in
`/usr/lib/tmpfiles.d/tmp.conf`. La conseguenza da ricordare: **su Rocky un
file lasciato in `/tmp` sopravvive al riavvio**, e sparisce dopo dieci giorni
senza che nessuno lo abbia cancellato esplicitamente.

Entrambe sono scrivibili da tutti, con il bit *sticky* (`drwxrwxrwt`): ognuno
può creare file, ma solo il proprietario può cancellare i propri.

---

## 4. Dentro `/usr`

Il nome è spesso spiegato come *Unix System Resources* o *User Resources*: è
un acronimo inventato a posteriori. In origine, nei primi Unix, `/usr`
conteneva le home degli utenti (*user*); i programmi vi furono spostati quando
il primo disco si riempì.

| Directory | Contenuto |
|---|---|
| `/usr/bin` | quasi tutti i comandi per gli utenti (`git`, `curl`, `python3`), e — dopo la fusione — anche quelli di base |
| `/usr/sbin` | i comandi di amministrazione |
| `/usr/lib`, `/usr/lib64` | le librerie dei programmi, e i file di supporto dei pacchetti (unità di `systemd` in `/usr/lib/systemd`) |
| `/usr/libexec` | programmi ausiliari che non si lanciano a mano, ma che altri programmi invocano (famiglia Red Hat; Debian li mette in `/usr/lib`) |
| `/usr/share` | dati indipendenti dall'architettura: manuali (`/usr/share/man`), documentazione, fusi orari, icone. Non configurazione: quella sta in `/etc` |
| `/usr/include` | le intestazioni per compilare in C |
| `/usr/local` | software installato **a mano** dall'amministratore, fuori dal gestore dei pacchetti. Ha la stessa struttura di `/usr`: `bin`, `sbin`, `lib`, `share` |

### `/usr/local`: dove stanno gli script dell'amministratore

`dnf` e `apk` non scrivono mai in `/usr/local`: ciò che vi si mette non viene
sovrascritto da un aggiornamento né rimosso con un pacchetto. È il posto degli
script scritti in casa:

| Directory | Per |
|---|---|
| `/usr/local/bin` | script destinati a tutti gli utenti |
| `/usr/local/sbin` | script di amministrazione, da eseguire come root (per esempio `backup-etc` in `howto/backup-etc-cron.md`) |

Entrambe appartengono a root e non sono scrivibili da altri. È una condizione
di sicurezza: uno script eseguito come root non deve poter essere modificato
da chi non è root.

---

## 5. Dentro `/var`

| Directory | Contenuto |
|---|---|
| `/var/log` | i log. Su Rocky: `messages` (generale), `secure` (autenticazioni, `sudo`), `cron`, `dnf.log`; molti servizi scrivono però solo nel journal di `systemd` (`journalctl`). Su Alpine: `messages` |
| `/var/cache` | dati ricostruibili (la cache di `dnf` in `/var/cache/dnf`): cancellarli non fa perdere nulla, costa solo tempo |
| `/var/lib` | lo **stato** dei programmi e dei servizi: dati persistenti che i programmi modificano da soli (`/var/lib/rpm`, `/var/lib/mysql`, `/var/lib/chrony`). È il posto giusto anche per la home di un utente di servizio |
| `/var/spool` | code di lavoro in attesa: posta in uscita, stampe, e le **crontab degli utenti** (`/var/spool/cron/<utente>` su Rocky) |
| `/var/tmp` | temporanei che sopravvivono al riavvio (§3) |
| `/var/www` | i siti web del server `httpd` (su Rocky, invece di `/srv/www`) |

Fra `/var/cache` e `/var/lib` la differenza è la domanda «se lo cancello,
perdo qualcosa?». Dalla cache no; dallo stato sì.

