# Redirezioni e pipe

Riferimento rapido sugli stream standard e sul loro reindirizzamento. Per l'elenco dei comandi di base, vedere [Comandi di base](Comandi-di-base.md).

## I tre stream standard

Ogni comando scrive su almeno uno di questi flussi: `0` (standard input, `stdin`), `1` (standard output, `stdout`) e `2` (standard error, `stderr`).

## Lettura di un file

```bash
cat /etc/passwd

# root:x:0:0:root:/root:/bin/sh
# bin:x:1:1:bin:/bin:/sbin/nologin
# daemon:x:2:2:daemon:/sbin:/sbin/nologin
# lp:x:4:7:lp:/var/spool/lpd:/sbin:/sbin/nologin
# ....
```

## Annullamento dell'output

La redirezione verso `/dev/null` scarta ciò che il comando produce. `>` sovrascrive il file, `>>` vi accoda.

```bash
cat /etc/passwd > /dev/null
```

## Redirezione in un file

```bash
cat /etc/passwd > /tmp/a.txt
cat /tmp/a.txt

# root:x:0:0:root:/root:/bin/sh
# bin:x:1:1:bin:/bin:/sbin/nologin
# daemon:x:2:2:daemon:/sbin:/sbin/nologin
# lp:x:4:7:lp:/var/spool/lpd:/sbin:/sbin/nologin
# ....
```

## Pipe e redirezione

La pipe `|` collega lo standard output di un comando allo standard input del successivo, senza file intermedi.

```bash
cat /etc/passwd | sort | grep h > /tmp/a.txt
cat /tmp/a.txt
# halt:x:7:0:halt:/sbin:/sbin/halt
# manager:x:1000:1000:corso:/home/manager:/bin/sh
# root:x:0:0:root:/root:/bin/sh
# shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
# sshd:x:22:22:sshd:/dev/null:/sbin/nologin
```

## Errori di battitura e file non trovato

Se il nome del file è sbagliato, `cat` scrive il messaggio su `stderr`: la pipe non lo trasmette, quindi il file di destinazione resta vuoto.

```bash
cat /etc/passwdr | sort | grep h > /tmp/a.txt
# cat: can't open '/etc/passwdr': No such file or directory
cat /tmp/a.txt
```

## Fusione di stdout e stderr

`2>&1` indica allo shell di duplicare il descrittore dello standard error sullo standard output. Deve precedere la pipe o la redirezione finale: se viene dopo, la shell non può più collegarli.

```bash
cat /etc/passwdr 2>&1 | sort | grep h > /tmp/a.txt
cat /tmp/a.txt
# cat: can't open '/etc/passwdr': No such file or directory
```
