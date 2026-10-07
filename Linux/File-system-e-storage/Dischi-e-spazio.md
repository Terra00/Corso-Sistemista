# Dischi e spazio

## Comandi per ispezionare i dischi

| Comando | Scopo |
|---|---|
| `lsblk` | Elenca i dispositivi a blocco, con dimensioni e punti di mount. |
| `lsblk -f` | Aggiunge informazioni su filesystem, UUID e punto di mount. |
| `df -h` | Spazio libero e occupato sui filesystem montati. |
| `df -i` | Utilizzo degli inode, cioè il numero di file già presenti. |
| `fdisk -l disk.img` | Elenca le partizioni di un immagine o di un disco. |
| `fdisk disk.img` | Avvia l'editor partizioni interattivo su un'immagine. |

> Negli esempi `disk.img` è il nome del file; per agire sul disco di sistema sostituirlo con il device effettivo, per esempio `/dev/sdb`.

## `/boot`

Contiene i file necessari all'avvio del sistema: kernel, initramfs e bootloader. Su Rocky Linux è una partizione separata da `/`.

## LVM su Rocky

LVM permette di aggiungere altri dischi e di farli apparire come un unico disco logico. Le operazioni sono descritte in [Estendere la root con LVM su Rocky](Estendere-root-con-LVM-su-Rocky.md) e in [LVM su Alpine](LVM-su-Alpine.md).

## Due cause di esaurimento dello spazio

- **Occupazione del contenuto**: file e programmi consumano lo spazio fisico disponibile.
- **Occupazione degli inode**: esauriti gli inode non è più possibile creare nuovi file, anche se lo spazio fisico è ancora libero.
