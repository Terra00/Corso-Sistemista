# Alpine: installare GNU tar

Su Alpine il client `tar` di default è quello di BusyBox, che supporta un sottoinsieme ridotto delle opzioni. Il pacchetto `tar` fornisce la versione GNU, con tutte le funzioni previste.

## Aggiornare i repository APK

```sh
doas apk update
```

```text
doas (manager@alpine-pm) password:
fetch http://dl-cdn.alpinelinux.org/alpine/v3.24/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.24/community/x86_64/APKINDEX.tar.gz
http://dl-cdn.alpinelinux.org/alpine/v3.24/main
http://dl-cdn.alpinelinux.org/alpine/v3.24/community
OK: 28645 distinct packages available
```

## Installare GNU tar

```sh
doas apk add tar
```

```text
(1/2) Installing acl-libs (2.3.2-r1)
80K(2/2) Installing tar (1.35-r5)
Executing busybox-1.37.0-r31.trigger
syncing disks...
78.0 MiB in 94 packages
```

## Verificare la versione installata

```sh
tar --version
```

## Elencare il contenuto di un archivio

```sh
tar tf banco.tar.gz
```

```text
banco/
banco/.dati_0182.bin
banco/archivio_0001.bin
....
```

> `t` elenca il contenuto e `f` indica il nome del file: con GNU tar i due argomenti possono anche essere separati, come in `tar -t -f banco.tar.gz`.
