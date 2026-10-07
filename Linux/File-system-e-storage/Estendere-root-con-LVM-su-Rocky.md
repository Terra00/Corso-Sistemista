# Estendere la root con LVM su Rocky

## Preparazione del nuovo disco

- Spegnere la macchina.
- Aggiungere un disco fisico tramite VMware.
- Riavviare e accedere in SSH.
- Verificare il nuovo disco con `lsblk -f`: il device risultante è **`nvme0n2`**.
- Partizionare il disco con `sudo fdisk /dev/nvme0n2`. Al termine la struttura è la seguente.
```text
$ sudo fdisk -l /dev/nvme0n2
Disk /dev/nvme0n2: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: CB4A210A-BAE5-4EBE-AE12-0BE83869B6DD

Device            Start      End  Sectors Size Type
/dev/nvme0n2p1     2048 20973567 20971520  10G Linux LVM
/dev/nvme0n2p2 20973568 41940991 20967424  10G Linux filesystem
```

```text
$ lsblk -f
NAME         FSTYPE      FSVER            LABEL                 UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sr0          iso9660     Joliet Extension Rocky-10-2-x86_64-dvd 2026-05-26-21-32-17-00                                
nvme0n1                                                                                                               
├─nvme0n1p1  vfat        FAT32                                  4436-0BA5                               589.9M     1% /boot/efi
├─nvme0n1p2  xfs                                                37aa586c-34b9-436c-8604-aefa945dd206      1.5G    21% /boot
└─nvme0n1p3  LVM2_member LVM2 001                               8bDRr5-ReTv-r8X0-k3HS-13kV-lloo-Nz7Oxc                
  ├─rlm-root xfs                                                dbe2b1f0-e418-44a3-8e8f-35ae07345399     13.7G    11% /
  └─rlm-swap swap        1                                      bc306ff0-cff7-4226-b577-cb850a1582b7                  [SWAP]
nvme0n2                                                                                                               
├─nvme0n2p1                                                                                                           
└─nvme0n2p2        
```

## LVM Physical Volume

- **PV**, volume fisico: un disco o una partizione consegnati a LVM ==> **/dev/nvme0n2p1**

**Quali sono i volumi fisici PV sotto il controllo di LVM?

`sudo pvs`
```text
$ sudo pvs
  PV             VG  Fmt  Attr PSize  PFree
  /dev/nvme0n1p3 rlm lvm2 a--  17.41g    0 
```

`sudo pvdisplay /dev/nvme0n1p3`
```text
$ sudo pvdisplay /dev/nvme0n1p3
  --- Physical volume ---
  PV Name               /dev/nvme0n1p3
  VG Name               rlm
  PV Size               17.41 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              4457
  Free PE               0
  Allocated PE          4457
  PV UUID               8bDRr5-ReTv-r8X0-k3HS-13kV-lloo-Nz7Oxc
```

### aggiungere /dev/nvme0n2p1 all'LVM (volume create)

`sudo pvcreate /dev/nvme0n2p1` # Physical volume "/dev/nvme0n2p1" successfully created.

```text
$ sudo pvs
  PV             VG  Fmt  Attr PSize  PFree 
  /dev/nvme0n1p3 rlm lvm2 a--  17.41g     0 
  /dev/nvme0n2p1     lvm2 ---  10.00g 10.00g
```

```text
$ lsblk -f
NAME         FSTYPE      FSVER            LABEL                 UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sr0          iso9660     Joliet Extension Rocky-10-2-x86_64-dvd 2026-05-26-21-32-17-00                                
nvme0n1                                                                                                               
├─nvme0n1p1  vfat        FAT32                                  4436-0BA5                               589.9M     1% /boot/efi
├─nvme0n1p2  xfs                                                37aa586c-34b9-436c-8604-aefa945dd206      1.5G    21% /boot
└─nvme0n1p3  LVM2_member LVM2 001                               8bDRr5-ReTv-r8X0-k3HS-13kV-lloo-Nz7Oxc                
  ├─rlm-root xfs                                                dbe2b1f0-e418-44a3-8e8f-35ae07345399     13.7G    11% /
  └─rlm-swap swap        1                                      bc306ff0-cff7-4226-b577-cb850a1582b7                  [SWAP]
nvme0n2                                                                                                               
├─nvme0n2p1  LVM2_member LVM2 001                               Z0v8r4-zL8l-1ftx-bJs0-BnPf-WSSw-RoMkhv                
└─nvme0n2p2  
```

## VG gruppo di volumi (volume group)

Volume group il «magazzino» che mette insieme lo spazio di uno o più PV.


`sudo vgs` elenca i volume group disponibili


```text
$ sudo pvs
  PV             VG  Fmt  Attr PSize  PFree 
  /dev/nvme0n1p3 rlm lvm2 a--  17.41g     0 
  /dev/nvme0n2p1     lvm2 ---  10.00g 10.00g


$ sudo vgs
  VG  #PV #LV #SN Attr   VSize  VFree
  rlm   1   2   0 wz--n- 17.41g    0
  ```


`sudo vgdisplay` dettagli legati ai VG

```text
$ sudo vgdisplay
  --- Volume group ---
  VG Name               rlm
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               17.41 GiB
  PE Size               4.00 MiB
  Total PE              4457
  Alloc PE / Size       4457 / 17.41 GiB
  Free  PE / Size       0 / 0   
  VG UUID               jyOxWm-LOyq-C3vn-84Ae-pB13-o1BF-JkfYOA
  ```

### estensione del vg rlm

`sudo vgextend rlm /dev/nvme0n2p1` estende il VG co quella partizione
```text
$ sudo vgextend rlm /dev/nvme0n2p1
[sudo] password for manager: 
  Volume group "rlm" successfully extended
```

## LV volume logico (il disco "vero" usabile dal SO)

LV si prende una fetta (al limite tutto) del "magazzino" VG

`lvs`
```text
$ sudo lvs
  LV   VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rlm -wi-ao---- 15.41g                                                    
  swap rlm -wi-ao----  2.00g  
```

estendiamo la root con il 100% dello spazio apportato

`sudo lvextend -r -l +100%FREE /dev/rlm/root` allarga il l'LV e lancia xfs_growfs per allargare la formattazione allo spazio appena aggiunto.

```text
$ sudo lvextend -r -l +100%FREE /dev/rlm/root
  File system xfs found on rlm/root mounted at /.
  Size of logical volume rlm/root changed from 15.41 GiB (3945 extents) to <25.41 GiB (6504 extents).
  Extending file system xfs to <25.41 GiB (27279753216 bytes) on rlm/root...
xfs_growfs /dev/rlm/root
meta-data=/dev/mapper/rlm-root   isize=512    agcount=4, agsize=1009920 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=4039680, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
data blocks changed from 4039680 to 6660096
xfs_growfs done
  Extended file system xfs on rlm/root.
  Logical volume rlm/root successfully resized.
  ```
