# AUTOMAZIONE

**File batch: automatizzare con il Prompt dei ** **comandi**

## Variabili, parametri, condizioni, codici di uscita e file di log: script semplici e verificabili su SVR-LAB-01

Primo script Variabili e condizioni Errorlevel e log Laboratorio

## 6a · PowerShell oggetti, pipeline, script.ps1

## 6b · Batch file.cmd eseguiti da cmd.exe

# 6c · GPO

criteri di gruppo, anche per distribuire script

## Alla fine saprete - Scrivere ed eseguire un file batch - Usare variabili, parametri e condizioni

- Controllare l'esito di un comando con errorlevel

- Scrivere un file di log e uscire con un codice

## Prerequisiti - SVR-LAB-01, accesso come LAB\Administrator

- C:\Scripts (Modulo 5C), con permesso di scrittura

- Condivisione \\SVR-LAB-01\Documenti (Modulo 4) - Servizio Server, LanmanServer (Modulo 5A) - Un editor di testo: Blocco note

# Che cos'è un file batch

Un file di testo con un comando per riga, eseguito da cmd.exe dall'alto verso il basso

Un file batch è un normale file di testo con estensione.cmd (o.bat). Quando lo si esegue, l'interprete dei comandi cmd.exe legge le righe una dopo l'altra e le esegue come se le digitassimo noi al prompt.

## Perché esiste ancora

Script di accesso degli utenti, attività pianificate, piccoli controlli: funziona su ogni Windows senza preparazione.

##.cmd o.bat

Due estensioni per lo stesso tipo di file. Nel corso usiamo.cmd, l'estensione pensata per cmd.exe.

## Il limite

Lavora solo con testo: per dati complessi (utenti AD, oggetti) PowerShell è più adatto.

# Batch e PowerShell a confronto

Due strumenti, due filosofie: quando usare l'uno o l'altro

| Caratteristica | Batch (cmd.exe) | PowerShell |
|:--- |:--- |:--- |
| Che cosa passa tra i comandi | testo | oggetti con proprietà |
| Estensione degli script |.cmd /.bat |.ps1 |
| Criteri di esecuzione | non soggetto all'Execution Policy | Execution Policy (RemoteSigned sul server) |
| Adatto per | controlli semplici, script di accesso | amministrazione completa, AD, report |

**Regola pratica:** compiti brevi e compatibili ovunque → batch; dati strutturati, Active Directory, automazione seria → PowerShell.

**PASSO 1** **Il primo script: Ciao.cmd** **Output atteso · da collaudare**

```bat
@echo off
rem Primo script batch del corso
echo Ciao %USERNAME%
echo Lo script e' terminato
```

```text
C:\Scripts>notepad Ciao.cmd
C:\Scripts>dir /b Ciao.*
  Ciao.cmd
C:\Scripts>Ciao.cmd
Ciao Administrator
Lo script e' terminato
```

| Riga | Significato |
|:--- |:--- |
| @echo off | non mostra i comandi mentre li esegue; la @ nasconde anche questa riga |
| rem | commento: una nota per chi legge, ignorata da cmd.exe |
| echo | scrive un messaggio sullo schermo |

**PASSO 2** **Variabili: set e %nome%** **Output atteso · da collaudare**

Modulo 6.b · Batch · Windows Server 2022 6 / 19 **File: C:\Scripts\Reparto.cmd**

```bat
@echo off
set REPARTO=IT
echo Utente: %USERNAME%
echo Reparto: %REPARTO%
```

```text
C:\Scripts>Reparto.cmd
Utente: Administrator
Reparto: IT
```

## Creare una variabile

set NOME=valore Senza spazi intorno all'uguale: con gli spazi il nome e il valore li conterrebbero.

## Usare una variabile

%NOME% viene sostituito dal valore prima di eseguire la riga. Le variabili d'ambiente (come %USERNAME%) esistono già.

**PASSO 3** **Parametri: %0, %1 e %~dp0** **Output atteso · da collaudare**

Modulo 6.b · Batch · Windows Server 2022 7 / 19 **File: C:\Scripts\Saluta.cmd**

```bat
@echo off
echo Script: %0
echo Primo parametro: %1
echo Cartella dello script: %~dp0
```

```text
C:\Scripts>Saluta.cmd Mario
Script: Saluta.cmd
Primo parametro: Mario
Cartella dello script: C:\Scripts\
```

| Parametro | Significato |
|:--- |:--- |
| %0 | il nome dello script, così come è stato digitato |
| %1... %9 | i valori scritti dopo il nome dello script, nell'ordine |
| %~dp0 | unità e percorso della cartella in cui si trova lo script |

**PASSO 4** **Condizioni: if, if exist, else** **Output atteso · da collaudare**

Un comando riuscito restituisce il codice di uscita 0, oppure nessun codice. Molti programmi, come find, usano codici diversi da 0 per segnalare un problema o un risultato negativo. cmd.exe conserva l'ultimo codice nella variabile %ERRORLEVEL%.

| Codice di find | Significato |
|---|---|
| 0 | la stringa cercata è stata trovata |
| 1 | la stringa cercata non è stata trovata |
| 2 | file non trovato o opzione non valida |

**Attenzione:** «if errorlevel 1» è vero se il codice è 1 O MAGGIORE. Per questo si controlla dal valore più alto al più basso.

**PASSO 5** **Errorlevel in pratica: il servizio Server** **Output atteso · da collaudare**

```text
C:\Scripts>sc.exe query LanmanServer
SERVICE_NAME: LanmanServer
STATE: 4  RUNNING
C:\Scripts>sc.exe query LanmanServer | find "RUNNING" >nul
C:\Scripts>echo %ERRORLEVEL%
0
C:\Scripts>sc.exe query ServizioInesistente >nul
C:\Scripts>echo %ERRORLEVEL%
1060
```

| Esito | Che cosa si può concludere |
|---|---|
| sc.exe restituisce un codice ≠ 0 | la query non è riuscita: lo stato non è stato letto |
| find restituisce 1 | la parola RUNNING non c'è: leggere l'output completo |
| find restituisce 0 | RUNNING compare: il servizio è in esecuzione |

**Attenzione:** find dice se ha trovato una parola, non perché manca. «Non trovato» non significa «servizio fermo».

**PASSO 5 bis** **&& e \|\|: decidere su una riga** **Output atteso · da collaudare**

```text
C:\Scripts>sc.exe query LanmanServer | find "RUNNING" >nul && echo RUNNING rilevato
RUNNING rilevato
C:\Scripts>sc.exe query ServizioInesistente >nul || echo Query non riuscita
Query non riuscita
```

| Operatore | Il secondo comando parte... |
|---|---|
| && | solo se il primo riesce (codice 0) |
| \|\| | solo se il primo non riesce |

**&** in ogni caso: due comandi sulla stessa riga

**Nota:** && e \|\| sono scorciatoie per i casi semplici. Nel laboratorio usiamo if errorlevel, che si legge meglio quando gli esiti sono tre.

**PASSO 6** **Redirezione: scrivere in un file di log** **Output atteso · da collaudare**

```text
C:\Scripts>echo Prima riga> Prova.log
C:\Scripts>echo Seconda riga>> Prova.log
C:\Scripts>type Prova.log
Prima riga
Seconda riga
C:\Scripts>sc.exe query LanmanServer 1> Stato.log 2>&1
C:\Scripts>
```

| Simbolo | Effetto |
|:--- |:--- |
| > | crea o sovrascrive il file |
| >> | aggiunge in fondo al file |
| 2>&1 | manda gli errori dove va l'output |
| >nul | scarta l'output normale, non gli errori |

## Attenzione: > cancella il contenuto precedente del file. Per un log che cresce nel tempo si usa sempre >>.

**Suggerimento:** niente spazio prima di > (echo Prima riga> Prova.log), altrimenti lo spazio finisce nel file.

**PASSO 7** **Ripetere: il ciclo for** **Output atteso · da collaudare**

```text
C:\Scripts>for %F in (*.cmd) do @echo Trovato: %F
Trovato: Ciao.cmd
Trovato: Controlla.cmd
Trovato: Reparto.cmd
Trovato: Saluta.cmd
```

**File: C:\Scripts\Elenca.cmd**

```bat
@echo off

for %%F in (*.cmd) do echo Trovato: %%F
```

**Attenzione:** al prompt si scrive %F, dentro uno script %%F. È l'errore più comune con il ciclo for.

# Dove si usano davvero i file batch

Gli script che abbiamo scritto possono partire da soli

**%F invece di %%F nello script** dentro uno script la variabile del for si scrive con due %

**set NOME = valore** niente spazi intorno all'uguale

**percorsi con spazi senza virgolette** scrivere "C:\Cartella con spazi\file.txt"

> al posto di >> > cancella il log a ogni esecuzione

**exit al posto di exit /b** exit chiude anche la finestra di cmd.exe

**if errorlevel 0 per "riuscito"** con codici non negativi (come quelli di find) è sempre vero: usare if errorlevel 1

**find non trova RUNNING = servizio fermo** significa solo «parola non trovata»: leggere l'output

# Riferimento rapido

## Da consultare durante il laboratorio

Modulo 6.b · Batch · Windows Server 2022 16 / 19 **Comando o simbolo** **Uso**

**@echo off · rem** non mostrare i comandi · commento

**set NOME=valore · %NOME%** creare e usare una variabile

**%1... %9 · %~dp0** parametri · cartella dello script **if... else · if exist** condizioni

**sc.exe query X \| find "RUNNING" >nul** cercare un testo e usare il codice di uscita

**if errorlevel 1** vero se il codice è 1 o maggiore

**a && b · a \|\| b** b solo se a riesce · solo se a non riesce

> · >> · 2>&1 · >nul redirezione (>nul: solo output normale)

**for %F in (...) do... · %%F nello script** ripetere un comando **type file** leggere un file di testo

**exit /b N** uscire dallo script con il codice N

# Laboratorio finale · 8 minuti

Scrivete C:\Scripts\Verifica.cmd e controllatene il risultato

| Controllo | Come si verifica |
| --- | --- |
| Servizio Server | sc.exe query riuscita? RUNNING rilevato? Tre esiti, tre messaggi |
| Condivisione | \\SVR-LAB-01\Documenti è accessibile per l'utente corrente? (if exist) |
| Log | scrivere ogni esito in C:\Scripts\Verifica.log con >> |
| Codice di uscita | exit /b 0 se entrambi i controlli riescono, altrimenti exit /b 1 |
| Verifica | eseguire, poi echo %ERRORLEVEL% e type Verifica.log |

**Suggerimento:** una variabile ESITO vale 0 e diventa 1 al primo controllo non riuscito. Il codice riguarda i controlli: il log si verifica con type.

# Quiz di verifica

Rispondete in chat con il numero della domanda e la lettera

Un file.cmd viene eseguito da: A) cmd.exe  B) powershell.exe  C) explorer.exe

Per aggiungere una riga in fondo a un file di log si usa: A) >  B) >>  C) \|

Dentro uno script, la variabile del ciclo for si scrive: A) %F  B) $F  C) %%F

find "RUNNING" non trova la parola. %ERRORLEVEL% vale: A) 0  B) 1  C) 2

Per uscire dallo script restituendo 1 senza chiudere la finestra: A) exit /b 1  B) exit 1  C) quit 1

# Correzione del quiz Risposta corretta e perché

Modulo 6.b · Batch · Windows Server 2022 19 / 19 **N.** **Risposta** **Perché**

| A | cmd.exe è l'interprete che esegue i file batch |
|:--- |:--- |
| B | >> aggiunge in fondo; > sovrascriverebbe il file |
| C | nello script si raddoppia il %; al prompt basta %F |
| B | find restituisce 1 quando non trova la stringa |
| A | exit /b esce solo dallo script e imposta il codice di uscita |

B >> aggiunge in fondo; > sovrascriverebbe il file

C nello script si raddoppia il %; al prompt basta %F

B find restituisce 1 quando non trova la stringa

A exit /b esce solo dallo script e imposta il codice di uscita

**Prossima unità · 6c GPO:** gli script come questi possono essere distribuiti agli utenti con i criteri di gruppo.
