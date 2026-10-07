# Accesso SSH con chiavi

## Creare una coppia di chiavi

```sh
ssh-keygen -t ed25519 -C "nome.cognome"
```

Il comando genera una coppia di chiavi, pubblica e privata, da usare per le connessioni SSH.

Le chiavi si trovano in `~/.ssh`, cioè in `/home/<nome_utente>/.ssh` su Linux e in `C:\Users<nome_utente>\.ssh` su Windows.

## Copiare la chiave pubblica

Su Windows aprire Git Bash; su Linux il comando è sufficiente:

```sh
ssh-copy-id -i ~/.ssh/id_ed25519.pub <nome_utente>@<ip_o_host>
```

## Configurare l'accesso sulla macchina di destinazione

Aprire con un editor di testo il file `~/.ssh/config`. Sono adatti Notepad, nano, Kate o Vim.

Aggiungere una voce per host:

```ssh-config
Host alpine
    HostName 192.168.159.128
    User manager
    IdentityFile ~/.ssh/id_ed25519
```

I nomi di host dichiarati in `Host` sono quelli da usare in `ssh`; le chiavi sono rientrate di quattro spazi, come richiesto dal formato del file.
