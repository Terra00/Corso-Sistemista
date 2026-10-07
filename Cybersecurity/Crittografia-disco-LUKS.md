# Crittografia del disco con LUKS

La cifratura del disco trasforma tutti i dati memorizzati su un hard disk o un SSD in codice illeggibile per chiunque non disponga della password o della chiave di decrittografia corretta.

- **Protezione dei dati**: file, applicazioni e sistema operativo vengono rimescolati tramite un algoritmo matematico.
- **Accesso trasparente**: all'accensione, inserendo la password giusta, il sistema sblocca i dati e li mostra normalmente.
- **Sicurezza fisica**: se il computer o l'hard disk viene rubato o perso, i dati restano inaccessibili anche collegando il disco a un altro sistema.

## Prerequisiti

Equipaggiamento hardware necessario, a scelta tra:

- chiavetta USB;
- hard disk esterno o interno;
- immagine disco generata con `dd`.

Pacchetti software da installare:

- `sudo dnf install usbutils`, se il device da cifrare è collegato all'USB;
- `sudo dnf install cryptsetup`;
- `sudo dnf install ntfs-ng`, se il device cifrato deve essere formattato con NTFS per compatibilità con Windows.

## Collegamento USB

In ambiente virtualizzato il device USB va sganciato dall'host e assegnato alla macchina virtuale. In VMware:

dal menù principale della VM, **Removable Devices**, selezionare i device necessari.

## Preparazione del device USB

Lanciare il comando `sudo journalctl -f` ed inserire il device USB catturando il nome del device collegato: es. **sdb**

`sudo fdisk -l /dev/sdb` per verificare la presenza di eventuali partizioni, farne l'eventuale backup preventivo.

`sudo fdisk /dev/sdb` per il partizionamento del device.

Siamo pronti per procedere con la criptazione, il device da criptare sarà **/dev/sdb1**

## Preparazione disco da file immagine

`mkdir -v dischi`

`cd dischi`

`dd if=/dev/urandom of=disco-criptato.img bs=100M count=10` per ottenere un disco da 1GB.

`fdisk disco-criptato.img` creiamo la partizione necessaria.

`sudo losetup -f -P --show disco-criptato.img`

A questo punto abbiamo ottenuto il device da criptare che è **/dev/loop0p1**

## Cifratura del device

Nel seguito il device da cifrare Ã¨ identificato da `/dev/disco`:

- nel caso di una chiavetta USB, `/dev/sdb1`;
- nel caso di un'immagine disco, `/dev/loop0p1`.

Inizializzare la configurazione della cifratura sul device:

```sh
sudo cryptsetup luksFormat /dev/disco
```

Digitare la password e conservarla in un luogo sicuro: senza di essa i dati non sono recuperabili.

Aprire il device usando la password. Il nome `vault` Ã¨ il *mapping name* e dÃ  nome al device decifrato:

```sh
sudo cryptsetup open /dev/disco vault
```

Creare un filesystem ext4 sul volume appena sbloccato:

```sh
sudo mkfs.ext4 /dev/mapper/vault
```

Creare il punto di montaggio dedicato all'utente corrente:

```sh
sudo mkdir $HOME/crypt
```

Montare la partizione cifrata:

```sh
sudo mount -v /dev/mapper/vault $HOME/crypt
```

Assegnare la directory di montaggio all'utente, in modo che ci possa accedere in lettura e scrittura:

```sh
sudo chown $USER $HOME/crypt
```

Creare un file di prova per verificare i permessi:

```sh
touch $HOME/crypt/file.txt
```

## Utilizzo normale del device cifrato

Ogni volta che serve accedere ai dati, aprire il device con la password e montarlo in lettura e scrittura:

```sh
sudo cryptsetup open /dev/disco vault
sudo mount -v /dev/mapper/vault $HOME/crypt
```

## Smontaggio e chiusura del device

Al termine, smontare il filesystem e chiudere il mapping. `cryptsetup close` riceve il *mapping name*, non il percorso del device:

```sh
sudo umount $HOME/crypt
sudo cryptsetup close vault
```

> Il percorso `/dev/mapper/vault` si usa con i comandi che operano su un device, come `mkfs` e `mount`; il nome `vault` da solo si usa con `cryptsetup close`.
