# `tar` — scheda di sopravvivenza

## 1. Che cosa è `tar`, in una riga

`tar` **non è uno zip**. È un *tape archiver*: mette in fila molti file in un
unico flusso, conservandone nome, percorso, permessi e date.

La compressione è un **secondo stadio**, fatto da un altro programma (`gzip`, `bzip2`, `xz`), che
`tar` si limita a chiamare per conto di chi lo usa.

Da questo discendono due conseguenze pratiche:

- `dati.tar` è un archivio **non compresso** e legittimo: non è un errore.
- `dati.tar.gz` (o `.tgz`) è quel medesimo archivio passato per `gzip`. Il
  doppio suffisso descrive i due stadi, nell'ordine in cui sono avvenuti.

Un archivio non compresso non è una stranezza da laboratorio: quando i dati sono
già compressi — immagini, video, pacchetti — la compressione non guadagna nulla
e costa tempo di CPU.

---

## 2. Le tre lettere che decidono

Ogni invocazione di `tar` sceglie **una sola** fra tre azioni. È l'unica cosa
che va imparata a memoria.

| Lettera | Azione | Scrive sul disco? |
|---|---|---|
| `c` | *create* — crea un archivio | sì, l'archivio |
| `t` | *list* — elenca il contenuto | **no, nulla** |
| `x` | *extract* — estrae | sì, i file estratti |

Le altre lettere non scelgono l'azione: la qualificano.

| Lettera | Significato |
|---|---|
| `f <file>` | *file*: su quale archivio si opera. **Il nome deve seguire immediatamente la `f`** |
| `z` | compressione che passa per `gzip` (`.gz`, `.tgz`) |
| `j` | compressione che passa per `bzip2` (`.bz2`) |
| `J` | compressione che passa per `xz` (`.xz`) |
| `v` | *verbose*: elenca i file mentre li tratta |
| `-C <dir>` | entra in `<dir>` **prima** di lavorare |

**La trappola della `f`.** Nella forma raggruppata `czf`, la `f` deve stare
**per ultima**, perché è la sola che consuma l'argomento successivo.


```bash
tar cfz arch.tar.gz dati      # funziona: forma storica senza trattino,
                              # gli argomenti si distribuiscono alle opzioni che ne chiedono

tar -cfz arch.tar.gz dati     # NON funziona: -f prende "z" come nome dell'archivio,
                              # e arch.tar.gz diventa materiale da archiviare
```

La seconda riga crea nella directory corrente un file di nome `z`, non compresso.

---

## 3. Le tre righe da sapere a memoria

```bash
tar czf /dati/archivio.tar.gz -C /percorso dati      # crea
tar tzf /dati/archivio.tar.gz                        # ELENCA (non estrae)
tar xzf /dati/archivio.tar.gz -C /dove/voglio        # estrae
```

Si leggono nell'ordine in cui vanno usate, ed è l'ordine in cui compaiono.

---

## 4. La regola: `t` prima di `x`, sempre

**Prima di estrarre si guarda che cosa c'è dentro.** Costa un secondo.

```bash
tar tzf archivio.tar.gz | head
tar tvzf archivio.tar.gz | head     # con v: permessi, proprietario, data, dimensione
```

Le ragioni sono tre, e sono tutte verificabili:

1. **Non si sa dove finiranno i file.** Un archivio può contenere un albero con
   una radice (`dati/...`, ordinato) oppure decine di file sciolti alla radice
   dell'archivio, che si spargono nella directory corrente.
2. **`x` sovrascrive senza chiedere.** Non esiste conferma, non esiste cestino,
   non esiste recupero.
3. **I percorsi possono essere ostili.** GNU `tar` toglie la '/' iniziale in
   creazione (`Removing leading '/' from member names`) e in estrazione rifiuta
   le voci contenenti `..`; ma questa difesa la offre GNU `tar`, si disattiva
   con `-P`, e non è garantita da ogni implementazione. L'elenco la rende
   superflua: si vede, e si decide.

È la stessa regola di `ls` prima di `rm`: **si guarda prima di agire.**

---

## 5. `-C`, che vale quanto la regola precedente

`-C <dir>` sposta `tar` nella directory indicata prima di lavorare.

**In creazione** decide che cosa finisce dentro l'archivio come percorso:

```bash
tar czf arch.tar.gz -C /home/manager dati       # dentro: dati/a.txt, dati/sotto/b.txt
tar czf arch.tar.gz /home/manager/dati          # dentro: home/manager/dati/a.txt  ← sgradevole
```

La prima forma produce un archivio **ripristinabile ovunque**; la seconda si
porta dietro il percorso della macchina in cui è nato.

**In estrazione** decide dove finiscono i file. Senza `-C`, finiscono nella
directory corrente — che è il modo più comune di sovrascrivere qualcosa per
sbaglio, perché la directory corrente raramente è quella che si crede.

> **Abitudine da prendere**: in estrazione, `-C` verso una directory **vuota e
> creata apposta**. Ciò che è tornato si sposta dopo averlo guardato.

---

## 6. Ciò che l'archivio porta con sé — e ciò che perde

`tar` conserva per default nome, percorso relativo, permessi, date e proprietario
(come numero e come nome). **Non conserva per default** le ACL (Access Control List), gli attributi
estesi e i contesti SELinux.

```bash
tar --acls --xattrs -czf /dati/arch.tar.gz -C /tmp dati
```

Su Rocky, dove SELinux è attivo, è la forma da usare. **Il guasto non si vede al
backup**: si vede al ripristino, quando un servizio rifiuta di partire perché il
contesto del suo file di configurazione non è più quello giusto.

Due regole sulla proprietà dei file estratti:

- **Estrazione da `root`**: proprietario e permessi dell'archivio vengono
  ripristinati tali e quali.
- **Estrazione da utente ordinario**: i file diventano dell'utente che estrae, e
  la `umask` si applica. Non è un guasto di `tar`: un utente non può regalare
  file ad altri.

Per un ripristino su una macchina diversa, dove gli stessi nomi utente possono
corrispondere a UID diversi, si aggiunge `--numeric-owner`, che ignora i nomi e
usa i numeri.

---

## 7. Estrarre una parte sola

Il nome da indicare è quello **esattamente come lo stampa `t`** — percorso
incluso, senza barra iniziale.

```bash
tar tzf arch.tar.gz | grep nota                       # prima si trova il nome
tar xzf arch.tar.gz -C /tmp/ripristino dati/nota_0003.txt
```

Con un modello invece che con un nome (GNU `tar`):

```bash
tar xzf arch.tar.gz -C /tmp/ripristino --wildcards 'dati/*.conf'
```

Altre due opzioni che risolvono situazioni frequenti:

| Opzione | Effetto |
|---|---|
| `--strip-components=1` | scarta il primo livello di percorso in estrazione |
| `-k` | non sovrascrive i file già presenti |

---

## 8. Quando `tar` sbaglia, lo dice — e va letto

`tar` restituisce un codice di uscita che va guardato:

| Codice | Significato |
|---|---|
| `0` | tutto riuscito |
| `1` | qualche file è cambiato mentre veniva letto, o differisce (`--compare`) |
| `2` | errore fatale: almeno un file non è stato archiviato |

**Il punto critico:** con codice `2` — tipicamente un `Permission denied` su
qualche file — `tar` **produce comunque un archivio**, e quell'archivio è
**incompleto**. Chi non guarda il codice di uscita si accorge del buco il giorno
del ripristino.

Imparare a contare i file ed i byte da archiviare!

```bash
tar czf /dati/arch.tar.gz -C /tmp dati
echo $?                                   # ← si guarda. Sempre.
```

Il rimedio per i file illeggibili è archiviare con privilegi sufficienti
(`sudo`), non ignorare gli errori.

---

## 9. GNU `tar` e BusyBox `tar` non sono lo stesso programma

Su Alpine `tar` è un *applet* di BusyBox: una riscrittura minima, non GNU `tar`.
L'uso quotidiano funziona — le tre azioni, la compressione, `-f` e `-C` —; le
opzioni proprie di GNU no.

| | GNU `tar` (Rocky, Debian) | BusyBox `tar` (Alpine) |
|---|---|---|
| `c` `t` `x` `z` `f` `-C` | sì | sì |
| `--acls` `--xattrs` | sì | no |
| voci *sparse* del formato GNU | sì | **no** |


```bash
apk update
apk add tar        # installa GNU tar
hash -r            # la shell dimentica il percorso memorizzato dell'applet
tar --version      # deve dire "GNU tar"
```

**Da ricordare:** un archivio che si apre su Rocky può non aprirsi su Alpine, e
questo **non è un archivio corrotto**. Prima di accusare l'archivio si verifica
quale `tar` si sta usando.

---

## 10. Il ripristino è la sola cosa che conta

Un backup che non è mai stato ripristinato **non è un backup**: è una speranza
con una dimensione in gigabyte. La verifica non è `ls` sull'archivio.

```bash
mkdir -p /tmp/ripristino
tar xzf /dati/arch.tar.gz -C /tmp/ripristino
diff -r /tmp/ripristino/dati /percorso/originale/dati
echo $?            # 0 = identici. Qualunque altro numero è un guasto.
```

Due verifiche più deboli, ma immediate, utili quando l'originale non c'è più:

```bash
tar tzf /dati/arch.tar.gz > /dev/null ; echo $?   # l'archivio si legge per intero
sha256sum /dati/arch.tar.gz                        # che sia arrivato integro
```

---

## 11. Guasti tipici, e la loro causa

| Sintomo | Causa | Rimedio |
|---|---|---|
| Esiste un file di nome `z` | `-cfz` invece di `-czf` | la `f` sta per ultima |
| L'archivio contiene `home/manager/dati/...` | percorso assoluto in creazione | `-C <dir>` e nome relativo |
| L'estrazione ha sparso file ovunque | archivio senza radice, estratto senza `-C` | `t` prima, `-C` sempre |
| L'estrazione ha sovrascritto lavoro in corso | `x` nella directory corrente | `-C` in directory vuota |
| I permessi ripristinati sono sbagliati | estratto da utente ordinario | estrarre da `root` |
| Un servizio non riparte dopo il ripristino | ACL e contesti SELinux persi | `--acls --xattrs` in creazione |
| I proprietari sono sbagliati su un'altra macchina | UID diversi a parità di nome | `--numeric-owner` |
| `unknown typeflag: 0x53` | archivio GNU letto da BusyBox | `apk add tar; hash -r` |
| L'archivio è più piccolo del previsto | `tar` è uscito con `2` e nessuno ha guardato | `echo $?`, e archiviare con `sudo` |
| `tar: Removing leading '/'` | percorso assoluto in creazione | è un avviso, non un errore: `-C` lo evita |

---

## 12. Prova da cinque minuti

Da fare una volta, per intero, sulla propria macchina.

```bash
mkdir -p /tmp/prova/sotto
echo primo   > /tmp/prova/a.txt
echo secondo > /tmp/prova/sotto/b.txt

tar czf /tmp/prova.tar.gz -C /tmp prova     # crea
echo $?

tar tvzf /tmp/prova.tar.gz                  # ELENCA: si guarda la radice

rm -rf /tmp/prova                           # si simula la perdita

mkdir -p /tmp/ripristino
tar xzf /tmp/prova.tar.gz -C /tmp/ripristino
ls -R /tmp/ripristino                       # l'albero è tornato
```

Chi arriva in fondo senza aver mai estratto nella directory sbagliata ha
imparato `tar`.

---

## Scheda tascabile

```
tar czf <archivio.tar.gz> -C <dir> <cosa>   crea
tar tzf <archivio.tar.gz>                   ELENCA  ← sempre prima di estrarre
tar xzf <archivio.tar.gz> -C <dir>          estrae, in una directory dichiarata

c crea    t elenca    x estrae        ← una sola delle tre, per volta
z gzip    j bzip2     J xz
f <file>  l'archivio — il nome la segue subito, e la f sta per ultima
v verboso    -C <dir> spostati lì prima di lavorare

--acls --xattrs     su Rocky (SELinux), in CREAZIONE
--numeric-owner     per ripristinare su un'altra macchina
--strip-components=1   scarta il primo livello di percorso
echo $?             0 va bene; 2 significa archivio INCOMPLETO

diff -r <ripristinato> <originale> ; echo $?    la verifica: 0 = identici
apk add tar ; hash -r                 su Alpine, quando BusyBox non basta
```

> **Nota sul flag `-C <directory>`**: il `-C` sposta il contesto del tar nella directory indicata, ma il suo comportamento dipende dal comando specificato.
>
> - Con **`x`** (extract) viene eseguito un `cd` nella directory e l'archivio viene estratto lì.
> - Con **`c`** (create) viene eseguito un `cd` nella directory e l'archivio viene creato prelevando i file da lì; la destinazione resta però quella indicata nel percorso passato al flag **`-f`**.
>
> In altre parole il `-C` segue il comando del tar: se crea l'archivio stabilisce da dove prendere i dati, se lo estrae stabilisce dove devono finire i file.

**Le due frasi da portarsi via:**

- `t` prima di `x`, sempre.
- Un backup che non è mai stato ripristinato non è un backup.
