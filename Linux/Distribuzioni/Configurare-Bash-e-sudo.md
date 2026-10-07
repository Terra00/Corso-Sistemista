# Alpine: installare Bash e configurare sudo

Riferimento rapido per installare Bash su Alpine, impostarla come shell predefinita e sostituire `doas` con `sudo`. I comandi sono eseguiti come utente non privilegiato con il prefisso `doas`; le righe marcate come root non richiedono il prefisso.

## Installare i pacchetti necessari

```sh
doas apk add bash sudo shadow
```

## Passare a una shell di root

```sh
doas -s
```

## Controllare la shell configurata

```sh
echo $SHELL
```

## Cambiare la shell predefinita

```sh
chsh -s "$(which bash)" manager
chsh -s "$(which bash)" root
```

Verificare di nuovo la shell configurata:

```sh
echo $SHELL
```

## Impostare `/bin/bash` per i nuovi utenti

```sh
mkdir -pv /etc/default
useradd -D -s /bin/bash
```

Controllare il valore di default impostato:

```sh
useradd -D
```

## Abilitare sudo

Controllare che `visudo` sia disponibile e che il file sia sintatticamente valido:

```sh
doas visudo -c
```

Aprire il file e rimuovere il commento alla riga che abilita il gruppo `wheel`:

```sh
doas visudo
```

Verificare di appartenere al gruppo `wheel`:

```sh
id
```

A quel punto `sudo` è utilizzabile:

```sh
sudo cat /etc/sudoers
```

Rimuovere infine `doas`, ormai sostituito:

```sh
sudo apk del doas
```

## Aggiungere un utente con gruppo dedicato

```sh
sudo groupadd -g 2000 guest
sudo useradd -m -d /home/guest -u 2000 -g 2000 -s /bin/bash -c "utente ospite" guest
```
