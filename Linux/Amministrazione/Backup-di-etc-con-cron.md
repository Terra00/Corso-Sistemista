# Esercizio backup con crontab

a) Realizzare la procedura (semplice) “etc-backup.sh” che realizzi il backup della directory /etc creando il file /tmp/etc-backup_YYYYMMDD_HHMMSS.tar.gz
b) Automatizzare il lancio della procedura tramite crontab


## script - etc-backup.sh

```bash
#!/bin/bash

DATA_ORA="$(date +"%Y%m%d_%H%M%S")"
echo $DATA_ORA

FILE_NAME=etc-backup_${DATA_ORA}.tar.gz
echo $FILE_NAME

OUTPUT_DIR=/tmp

sudo tar czf $FILE_NAME -C $OUTPUT_DIR /etc
```


```
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
# *  *  *  *  * echo "Messaggio da crontab $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
  */30 * * * * echo "Sono passati 30 minuti e sono le $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
  */4 * * * * echo "Sono passati 4 minuti e sono le $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
  */2 * * * * echo "backup avvenuto con successo!" >> /tmp/crontab_demo.log && /home/manager/bin/etc-backup.sh >> /tmp/crontab_demo.log
```

## script migliorato - etc-backup.sh

```bash
#!/bin/bash

DATA_ORA="$(date +"%Y%m%d_%H%M%S")"
echo $DATA_ORA

FILE_NAME="etc-backup_${DATA_ORA}.tar.gz"
echo $FILE_NAME

OUTPUT_DIR="/tmp"

tar czf "$OUTPUT_DIR/$FILE_NAME" -C "/" etc

rc=$?

if [ $rc -eq 0 ]; then

        echo "backup avvenuto con successo!"

fi

if [ $rc -eq 1 ]; then

        echo "Alcuni file sono cambiati mentre tar li leggeva. L'archivio viene creato comunque."

fi

if [ $rc -eq 2 ]; then

        echo "Errore fatale!"

fi
```

ricordarsi di cambiare i privilegi di etc-backup.sh `chmod -v 750 etc-backup.sh`

## script per la password - askpass.sh (permessi 700)

Va a leggere un file chiamato .sudopass contenente la password.

```bash
#!/bin/bash

cat $HOME/bin/.sudopass
```

## crontab migliorato

```
# For details see man 4 crontabs

SUDO_ASKPASS=/home/manager/bin/askpass.sh

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
# *  *  *  *  * echo "Messaggio da crontab $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
# */30 * * *  * echo "Sono passati 30 minuti e sono le $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
# */4 * *  *  * echo "Sono passati 4 minuti e sono le $(date +'\%H-\%M-\%S')" >> /tmp/crontab_demo.log
# */2 * *  *  * sudo -p '' -S /home/manager/bin/etc-backup.sh < /home/manager/bin/.sudopass >> /tmp/backup.log
# */2 * *  *  * sudo -A /home/manager/bin/etc-backup.sh  >> /tmp/backup.log
```



## Fornire la password a sudo durante le azioni cron (soluzione non consigliata)

> Conservare la password in chiaro sul disco è una pratica da evitare. Le due soluzioni qui descritte servono solo a completare il laboratorio; in produzione si preferisce `root` con chiavi SSH oppure un gestore di segreti.

### File `.sudopass`

La password viene fornita a sudo tramite lo standard input, utilizzando il flag `-S`. Resta memorizzata in un file chiamato `~/.sudopass`:

```sh
echo "corso" > ~/.sudopass
```

La riga di crontab corrispondente è:

```cron
*/2 * * * * sudo -p '' -S /home/manager/bin/etc-backup.sh < /home/manager/.sudopass >> /tmp/backup.log
```

### Helper `askpass`

In alternativa la password viene fornita a sudo tramite un helper, `/home/manager/bin/askpass.sh`, utilizzando il flag `-A`. Il percorso dell'helper va memorizzato nella variabile `SUDO_ASKPASS`.

L'helper può essere come il seguente:

```bash
#!/bin/bash
# askpass.sh
# echo "corso"
# oppure come segue
cat $HOME/.sudopass
```

```cron
SUDO_ASKPASS=/home/manager/bin/askpass.sh
*/2 * * * * sudo -A /home/manager/bin/etc-backup.sh >> /tmp/backup.log
```

## Automazione del backup tramite creazione di un nuovo utente

### Creazione dell'utente "backup"

1. Attività creazione dell'utente che non può effettuare il login e di un gruppo con il medesimo nome.


```bash

sudo useradd --shell /sbin/nologin \
        --system \
        --user-group \
        --comment "utente per l'esecuzione di backup" \
        --home-dir /var/lib/backup --create-home \
        backup

getent passwd backup # backup:x:995:995:utente per l'esecuzione di backup:/var/lib/backup:/sbin/nologin
getent group backup # backup:x:995:
sudo id backup # uid=995(backup) gid=995(backup) groups=995(backup)
sudo passwd -l backup
sudo passwd -S backup # backup L 2026-10-01 -1 -1 -1 -1

# ulteriori controlli
cat /etc/passwd | grep backup
sudo cat /etc/shadow | grep backup # backup:!:20727:::::: ==> il ! indica il password lock
cat /etc/group | grep backup
```


2. Preparazione della struttura della directory dell'utente backup
creazione della directory bin `sudo mkdir -pv /var/lib/backup/bin`
3. Copiare la procedura del backup `sudo cp -v bin/etc-backup.sh /var/lib/backup/bin`
4. Cambio dei proprietà dello user e del gruppo `sudo chown backup:backup /var/lib/backup/bin/etc-backup.sh`
`sudo ls -l /var/lib/backup/bin/etc-backup.sh `


### Preparazione di /etc/sudoers per accogliere l' utente di backup
5. Apertura del sudo visudo `sudo visudo -f /etc/sudoers.d/backup`
6. Aggiunta delle righe di /etc/sudoers.d/backup l'unica necessaria è sotto (**):
```
# abilitazione a sudo dello user 'backup' senza richiesta password

# con questa configurazione l'utente backup può lanciare ogni comando
# come root senza password

# passo 1) VERY DANGEROUS da scartare
# utente backup
# backup  ALL=(ALL)       NOPASSWD:       ALL

# gruppo backup
# %backup  ALL=(ALL)       NOPASSWD:       ALL

# passo 2) corretto (**)
backup  ALL=(root)       NOPASSWD:       /var/lib/backup/bin/etc-backup.sh ""
```


### Attivazione del crontab di backup
7. Apertura del crontab `sudo crontab -u backup -e`
8. Aggiunta della riga  `*/2 * * * * sudo /var/lib/backup/bin/etc-backup.sh >> /tmp/backup_$USER.log`
9.  Controllo del log di crontab `sudo tail -f /var/log/cron` oppure `sudo journalctl -u crond`


## evoluzione script - etc-backup-v2.sh
```bash
#!/bin/bash
#
# procedura etc-backup-v2.sh
# versione 2 del 2026/10/01
# author:
# script di backup della directory /etc
# lanciato tramite crontab dall'utente nologin backup ed da eseguire come root
# crontab -u backup -e
#
# correlato con /etc/sudoers.d/backup
#
# esempio di run

# 2026-10-01 11:14:02 backup procedure started
# mkdir: created directory '/var/backup'
# mkdir: created directory '/var/backup/etc'
# etc-backup_20261001_111402.tar.gz
# backup avvenuto con successo!
# etc-backup_20261001_111801.tar.gz creato con successo in /var/backup/etc
# 2026-10-01 11:18:02 backup procedure ended
# fine esempio

echo "$(date +'%F %T') backup procedure started"

set -euo pipefail # si ferma al primo errore anche di pipe

# UGO => ?00
umask 077 # serve per mantenere le permission + basse possibili => permission 400

# dove metto i backup?
OUTPUT_DIR="/var/backup/etc"
mkdir -pv "$OUTPUT_DIR" # crea il path/subpath se non esiste altrimenti skip della creazione

# che profondita di backup tengo? quanti backup tengo? quanto sono i giorni di conservazione?
KEEP_DAYS=7

# timestamp
DATA_ORA=$(date +"%Y%m%d_%H%M%S")

FILE_NAME="etc-backup_${DATA_ORA}.tar.gz"
echo $FILE_NAME

rc=0
tar --selinux --xattrs --acls -czf "$OUTPUT_DIR/$FILE_NAME" -C "/" etc
rc=$?

if [ $rc -eq 0 ]; then
        echo "backup avvenuto con successo!"
fi

if [ $rc -eq 1 ]; then
        echo "Alcuni file sono cambiati mentre tar li leggeva. L'archivio viene creato comunque."
fi

if [ $rc -eq 2 ]; then
        echo "Errore fatale!"
fi

# Rotazione
find "$OUTPUT_DIR/" -name "etc-backup_*.tar.gz" -mtime +"$KEEP_DAYS" -delete

echo "$FILE_NAME creato con successo in $OUTPUT_DIR"
echo "$(date +'%F %T') backup procedure ended"

```




