# Account e gruppi locali

## `/etc/passwd` — chi esiste

Leggibile da chiunque. Una riga per utente, sette campi separati da `:`.

```sh
cat /etc/passwd
getent passwd nome               # una riga sola, per nome o per UID
```

```
manager:x:1000:1000:Manager:/home/manager:/bin/bash
   1    2  3    4      5         6            7
```

| # | Campo | Significato |
|---|---|---|
| 1 | nome | l'etichetta, unica sulla macchina |
| 2 | password | **`x`**: l'hash sta in `/etc/shadow`. Un campo vuoto qui significa *nessuna password*, ed è un guasto grave |
| 3 | UID | il numero che conta |
| 4 | GID | il gruppo **primario**, uno solo |
| 5 | GECOS | campo descrittivo, libero; storicamente nome esteso, ufficio, telefono |
| 6 | home | la directory iniziale; **non è detto che esista** |
| 7 | shell | il programma avviato al login; `/sbin/nologin` o `/bin/false` negano la sessione interattiva |

**Il settimo campo non è un permesso.** Mettere `/sbin/nologin` impedisce la
shell di login, non impedisce all'utente di esistere, di possedere file, né di
autenticarsi presso servizi che non aprono una shell.



## `/etc/shadow` — le password

Leggibile **solo da root**: contiene gli hash. Nove campi.

```sh
cat /etc/shadow                  # solo come root
getent shadow nome               # idem
```

```
manager:$6$xxxx$yyyy:20352:0:99999:7:::
   1          2        3   4   5   6 789
```

| # | Campo | Significato |
|---|---|---|
| 1 | nome | corrisponde a `/etc/passwd` |
| 2 | hash | sale e digest; i prefissi dicono l'algoritmo (v. sotto) |
| 3 | **ultimo cambio** | giorni dal 1970-01-01. **`0` significa: va cambiata al prossimo accesso** |
| 4 | minimo | giorni prima che sia lecito ricambiarla |
| 5 | massimo | giorni dopo i quali scade |
| 6 | preavviso | giorni di avviso prima della scadenza |
| 7 | inattività | giorni di tolleranza dopo la scadenza |
| 8 | **scadenza account** | giorni dal 1970: oltre quella data l'account non entra più |
| 9 | riservato | |

Il campo 2 non contiene la password ma il risultato di una funzione a senso
unico, preceduto dall'identificativo dell'algoritmo:

| Prefisso | Algoritmo |
|---|---|
| `$6$` | SHA-512 |
| `$y$` | yescrypt |
| `!` o `!!` in testa | **account bloccato**: il login interattivo è negato |
| `*` | nessuna password utilizzabile; tipico degli account di sistema |
| campo **vuoto** | nessuna password richiesta: guasto grave |

**Il terzo campo è il più utile in pratica**, e non per la sicurezza: è
l'unico punto della macchina che registra, senza alcuna configurazione, *se e
quando* un utente ha cambiato la propria password.

## `/etc/group` — chi sta con chi

Leggibile da chiunque. Quattro campi.

```sh
cat /etc/group
getent group wheel
```

```
wheel:x:10:manager,anna
  1   2  3       4
```

| # | Campo | Significato |
|---|---|---|
| 1 | nome del gruppo | |
| 2 | password | quasi sempre `x`; l'hash, se esiste, sta in `/etc/gshadow` |
| 3 | GID | |
| 4 | **elenco** | i membri **supplementari**, separati da virgola |

⚠️ **Il quarto campo non elenca tutti i membri.** Chi ha quel gruppo come
gruppo **primario** (campo 4 di `/etc/passwd`) è membro a pieno titolo **e non
compare qui**. È l'errore di lettura più frequente su questo file: si cerca
l'utente nell'elenco, non lo si trova, e si conclude che non appartiene al
gruppo.

```sh
id nome                          # la risposta giusta: legge entrambe le vie
groups nome
```


## Aggiungere un gruppo su Rocky e Alpine

```sh
sudo groupadd -g 2000 guest
```

## Creare e verificare un account locale

```sh
sudo useradd -m utente-sg         # crea anche la home directory
sudo passwd utente-sg             # imposta o modifica la password
id utente-sg                      # mostra UID, GID e gruppi effettivi
sudo usermod -aG wheel utente-sg  # aggiunge al gruppo supplementare wheel
```

> In alcune distribuzioni il gruppo amministrativo può essere `sudo` anziché `wheel`. Verificare la configurazione in `/etc/sudoers` prima di assegnare privilegi amministrativi.
