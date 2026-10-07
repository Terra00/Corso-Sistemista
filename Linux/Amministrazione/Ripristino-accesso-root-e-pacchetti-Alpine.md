# Ripristino dell'accesso root su Alpine e gestione dei pacchetti

Questa procedura serve a reimpostare la password di un sistema Alpine installato quando se ne possiede la console. Lavorare dalla console della VM: identificare il disco o la partizione sbagliati può rendere indisponibili i dati.

## Ripristinare la password

1. Collegare l'ISO di Alpine alla macchina virtuale.
2. Avviare il firmware della VM e mettere il lettore ISO prima del disco nell'ordine di avvio. Per VMware, vedere [modificare l'ordine di avvio](../../Strumenti-e-metodologie/Virtualizzazione/VMware-ordine-di-avvio.md).
3. Avviare l'ambiente live e identificare dischi, partizioni e filesystem:

   ```sh
   lsblk -f
   fdisk -l /dev/sdX
   ```

   Se `lsblk` non è disponibile, installare temporaneamente il pacchetto `util-linux` con `apk update && apk add util-linux`.
4. Individuare la partizione che contiene la radice del sistema installato e montarla. Il tipo di filesystem e il nome del device sotto sono esempi: verificarli al passo precedente.

   ```sh
   mount -t ext4 /dev/sdaX /mnt
   ```

5. Entrare nel sistema installato:

   ```sh
   chroot /mnt
   ```

6. Reimpostare la password dell'account root:

   ```sh
   passwd
   ```

7. Se necessario, creare un utente oppure controllare gli account locali:

   ```sh
   adduser nomeutente
   cat /etc/passwd
   ls -l /etc/passwd
   ```

8. Uscire dall'ambiente `chroot`, smontare il filesystem e spegnere:

   ```sh
   exit
   umount /mnt
   poweroff
   ```

9. Scollegare l'ISO e riavviare dal disco; verificare l'accesso prima di proseguire.

> Se la radice usa LVM, cifratura o più filesystem separati, i soli passaggi `mount` e `chroot` potrebbero non essere sufficienti. Questa configurazione non è documentata qui e va verificata prima di procedere.

## Gestione dei pacchetti

Ogni distribuzione usa il proprio gestore di pacchetti: `apk` per Alpine, `apt` per Debian e derivate, `zypper` per SUSE, `dnf` per Rocky Linux e Red Hat Enterprise Linux.

- [Panoramica dei gestori di pacchetti Linux](https://linuxblog.io/linux-package-managers-apt-dnf-pacman-zypper/)

### Comandi Alpine

1. `apk update`: aggiorna l'elenco dei pacchetti disponibili.
2. `apk add nome_pacchetto` (per esempio `util-linux`): installa un pacchetto.
3. `apk del nome_pacchetto`: rimuove un pacchetto.
4. `apk --help`: mostra la guida ai comandi.

## Controlli di rete essenziali

1. `ping indirizzo_ip_o_host` (per esempio `ping 1.1.1.1`): invia richieste ICMP per verificare la raggiungibilità; l'assenza di risposta non prova da sola che l'host sia spento.
2. `ip a` (abbreviazione di `ip addr`): mostra le informazioni IP delle interfacce.
3. `ssh nome_utente@ip_o_nome_server` (per esempio `ssh manager@192.168.12.178`): apre una connessione remota SSH.
