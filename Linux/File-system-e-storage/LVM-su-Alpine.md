# Installazione di LVM su Alpine Linux

## pre-requisiti

Alpine **non ha LVM** nell'installazione di base.

```bash
sudo apk update
sudo apk upgrade
sudo reboot # se necessario per modifiche all'OS
sudo apk add lvm2 e2fsprogs e2fsprogs-extra
sudo rc-update add lvm boot
sudo rc-service lvm start
sudo rc-service lvm status          # status: started
```

## 2. La partizione

```bash
sudo fdisk /dev/sdb
```

creare una partizione di 5GB
```text
Command (m for help): p
Disk /dev/sdb: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 5CFEE8D7-F959-4B75-8CC8-CB5716B565F7

Device Start End Sectors Size Type
/dev/sdb1 2048 10487807 10485760 5G Linux filesystem
```


## 3. Il volume fisico — PV

```bash
sudo pvcreate /dev/sdb1
sudo pvs
```

```text
alpine-pm:~$ sudo pvs
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb1     lvm2 ---  5.00g 5.00g
```


## 4. creazione del Volume Group VG_NAME=vg_01


```bash
sudo vgcreate vg_01 /dev/sdb1
sudo vgs
```

```text
$ sudo vgs
  VG    #PV #LV #SN Attr   VSize  VFree 
  vg_01   1   0   0 wz--n- <5.00g <5.00g

$ sudo pvs
  PV         VG    Fmt  Attr PSize  PFree 
  /dev/sdb1  vg_01 lvm2 a--  <5.00g <5.00g
```


## 5. creazione del logical volume

```bash
sudo lvcreate -l +100%FREE -n lv_01 vg_01
sudo lvs
sudo vgs
``` 

```text
$ sudo lvs
  LV    VG    Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_01 vg_01 -wi-a----- <5.00g   

$ sudo vgs
  VG    #PV #LV #SN Attr   VSize  VFree
  vg_01   1   1   0 wz--n- <5.00g    0 
```


## 6. creare il file system (di basso livello) ext4

```bash
sudo mkfs.ext4 -L DATI /dev/vg_01/lv_01
sudo mkdir -v /media/dati # mountpoint
sudo mount /dev/vg_01/lv_01 /media/dati
sudo df -h /media/dati
sudo lsblk -f
sudo touch /media/dati/prova.txt
```

`sudo mkfs.ext4 -L DATI /dev/vg_01/lv_01`
```text
$ sudo mkfs.ext4 -L DATI /dev/vg_01/lv_01
mke2fs 1.47.4 (6-Mar-2025)
Creating filesystem with 1309696 4k blocks and 327680 inodes
Filesystem UUID: 3ee1fbea-f969-4e1e-81f9-7409cb249422
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```

`sudo lsblk -f`
```text
$ sudo lsblk -f
NAME            FSTYPE      FSVER LABEL                     UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                                               
├─sda1          ext4                                        b2c4282c-7f2b-4c63-a2d4-d33eb1d2c052    224.4M    10% /boot
├─sda2          swap                                        30660bc4-f739-4956-91e4-3cdb4482646a                  [SWAP]
└─sda3          ext4                                        9c2c2654-ee7f-4a7f-a687-764df1b33a06     11.7G     2% /
sdb                                                                                                               
└─sdb1          LVM2_member                                 XLTkbD-xmRM-zoEg-SDv1-v7Ol-KH1K-EQ1gny                
  └─vg_01-lv_01 ext4              DATI                      3ee1fbea-f969-4e1e-81f9-7409cb249422      4.6G     0% /media/dati
sr0             iso9660           alpine-virt 3.24.1 x86_64 2026-06-13-16-38-55-00 
```

## 7. Rendere permanente in fstab
- Fare backup prima di modificare fstab
- `sudo cp /etc/fstab /etc/fstab.orig`
- `sudo lsblk -f` -> controllare UUID del disco
- `echo "UUID=$(sudo blkid -s UUID -o value /dev/vg_01/lv_01) /media/dati ext4 defaults 0 0" | sudo tee -a /etc/fstab`
Aggiungiamo in una variabile il comando sudo blkid -s UUID -o value /dev/vg_01/lv_01 per avere il nostro UUID per il disco che ci interessa
'blkid' -> per prendere lo uuid '-s' -> solo per lo uuid '-o value' -> prende solo il valore '/dev/vg_01/lv_01'-> disco che ci interessa aggiungere
Tutto va in echo con l'aggiunta di UUID=$()  e "/media/dati" -> cartella di mount "ext4"-> formato del disco "defaults 0 0" -> impostazioni di defaults e backup
- `sudo mount -a -v` -> lanci per ricaricare il fstab e le cartelle montate
- `sudo reboot` -> controllo se tutto è andato bene e il sistema parte
- `sudo ls -l /media/dati` -> controlli che la cartella montata non ha problemi


## 8. Permessi
### Premessa dobbiamo cambiare gruppo e permessi per la cartella appena montata tramite fstab
- `sudo groupadd share-dati` -> creazione gruppo nuovo per la cartella
- `sudo chown root:share-dati /media/dati` -> cambia proprietario e gruppo della cartella dati
- `sudo usermod -aG *gruppo* *utente*`-> aggiunta dell'utente/i interessato per il gruppo
  se aggiungi l'utente che stai usando in questo momento esci e rientra nella macchina per controllare con id se il cambiamento c'è stato
- `sudo chmod 2770 /media/dati` -> cambio permessi della cartella e propagazione di questi ultimi per file e cartelle al suo interno
- `touch /media/dati/file-di-prova.txt`

`for u in $(getent passwd | cut -d: -f1); do id "$u"; done`
Comando utile per vedere utenti e gruppi



## 9, Estendere
### Estendere una partizione esistente da un nuovo disco
- `sudo pvcreate /dev/sdc1` -> creazione del disco
- `sudo vgextend vg_01 /dev/sdc1` -> estendere al disco appena creato al gruppo a cui vogliamo aggiungero
- `sudo lvextend -l +70%FREE -r /dev/vg_01/lv_01` -> espandere la partizione dopo aver creato e aggiunto il disco
  In questo caso si usa `-r` per aggiungere ed espandere il filesystem al nuovo disco invece che `-n` come per la creazione
