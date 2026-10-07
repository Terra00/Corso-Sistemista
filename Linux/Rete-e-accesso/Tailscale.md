# Connessione tramite rete Tailscale

> **Sicurezza:** una chiave di autorizzazione non va salvata nella documentazione. Nei comandi seguenti sostituire `<chiave-di-autorizzazione>` con una chiave creata e gestita nel proprio pannello Tailscale. Se una chiave è stata già condivisa o pubblicata, va revocata e sostituita.

## Installazione alpine

### comandi

```bash
sudo apk add tailscale

# attivazione al boot
sudo rc-update add tailscale
sudo rc-service tailscale start
sudo rc-service tailscale status

# join al network, dove -xy sono le iniziali legate alla macchina
sudo tailscale up --authkey <chiave-di-autorizzazione> --hostname alpine-xy
```

## Installazione Rocky

### comandi

```bash
# $(rpm -E %rhel) risolve alla versione corrente di RHEL/Rocky (provare con echo)

sudo dnf install -y tailscale

# attivazione al boot
sudo systemctl enable --now tailscaled

# join al network, dove -xy sono le iniziali legate alla macchina
sudo tailscale up --authkey <chiave-di-autorizzazione> --hostname rocky-xy

```

In alternativa, seguire le istruzioni del produttore su [Tailscale per Linux](https://tailscale.com/download/linux); installazione e collegamento alla rete restano passaggi distinti.
```bash
sudo curl -fsSL https://tailscale.com/install.sh | sh
```

## Configurazione + esercizio Alpine

### settaggio in sicurezza delle macchine

- cambiare la password di root se era una di quelle condivise
- cambiare la password di manager da "corso" ad una password personale


### creazione di un utente amministrativo aggiuntivo (a cura del proprietario della macchina)

- creare un utente amministrativo (X) in modo casuale sulla propria macchina (utente=x, gruppo=x)

```bash
sudo useradd -m utente-xy
id utente-xy
sudo passwd utente-xy #corso
sudo usermod -a -G wheel utente-xy # aggiunge il gruppo wheel al'utente
id utente-xy
```


### creazione degli altri utenti (a cura dell'amministratore (X) aggiuntivo)

- l'utente X usa sulla macchina bersaglio il gruppo "users" gid=100 per creare tutte le altre utenze (??)
- tutti i colleghi ad esclusione di se stessi e del proprietario della macchina

```bash
sudo useradd -m -g 100 utente-?? # per ogni utente della classe
sudo passwd utente-?? #corso modifica della password
```
- crea tutte le utenze della classe all'infuori della propria e del proprietario sulla macchina target


- nel caso di errori e sistemazione utenze
```bash
sudo usermod -g 100 utente-?? # cambio gruppo primario utente
sudo delgroup prova wheel # toglie lo user prova dal gruppo wheel
sudo deluser --remove-home prova # elimina l'utente prova unitamente alla sua home
sudo usermod -a -G aa,bb,cc utente-?? # aggiunge all'elenco dei gruppi associati a utente-?? i gruppi aa,bb,cc
sudo usermod -G aa,bb,cc utente-?? # riscrive l'elenco dei i gruppi associati a utente-??
```

- per esaminare i gruppi associati ad un utente
```bash
cat /etc/group | grep wheel
# oppure
grep wheel /etc/group
```

### al termine

- in modo random collegarsi alle macchine bersaglio usando la propria utenza e cambiando la password

### comandi vari

`tailscale status` esaminare lo stato della rete
