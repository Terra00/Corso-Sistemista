# MODULO 5.C - AMMINISTRAZIONE **Prestazioni e attività pianificate**

## Misurare nel tempo con i contatori, automatizzare uno script e verificarne l'esecuzione

Contatori Monitoraggio prestazioni Utilità di pianificazione Laboratorio e quiz

# Obiettivi e prerequisiti Dalla misura alla automazione verificata

## Alla fine saprete - Trovare i nomi italiani dei contatori - Misurare CPU e memoria nel tempo

- Usare Monitoraggio prestazioni e un rapporto

- Creare, registrare e verificare un'attività pianificata - Diagnosticare un'attività che non parte

## Prerequisiti - Moduli 5.A e 5.B completati

- SVR-LAB-01: PowerShell come amministratore, accesso  LAB\Administrator - Lettura degli eventi con Get-WinEvent (5.B)

| A | Monitoraggio prestazioni | 17′ |
|:--- |:--- |:--- |
| B | Utilità di pianificazione | 32′ |
| ✓ | Laboratorio «trova il guasto», restituzione e quiz | 23′ | **B** **Utilità di pianificazione** **32′**

✓ **Laboratorio «trova il guasto», restituzione e quiz** **23′**

# A

# Monitoraggio prestazioni Misurare, non indovinare **17 minuti** 4 / 31

# Oggetto, istanza, contatore Come è fatto il nome di un contatore

## Oggetto che cosa misuro: Processore

## Istanza quale: \_Total = tutti i core insieme

## Contatore quale misura: % Tempo processore \\Processore(\_Total)\% Tempo processore

**Attenzione:** su Windows in italiano i nomi dei contatori sono tradotti. Il percorso inglese \\Processor(\_Total)\% Processor Time delle guide non funziona così com'è.

**PASSO 1** **Trovare i nomi italiani** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 6 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-Counter -ListSet Process* | Select-Object CounterSetName
```

```text
CounterSetName
-------------
Processo
Processore
...
```

```powershell
PS C:> (Get-Counter -ListSet Processore).Paths
```

```text
\Processore(*)\% Tempo processore
\Processore(*)\% Tempo utente
...
```

## Suggerimento: -ListSet accetta l'asterisco. Se Process* non trova l'insieme, Get-Counter -ListSet * li elenca tutti. Poi copiate il percorso esatto da.Paths.

## Nota: (*) in.Paths significa «qualsiasi istanza»: nel comando la sostituiremo con \_Total.

**PASSO 2** **La CPU nel tempo** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 7 / 31 **Si esegue su: SVR-LAB-01**

PS C:> Get-Counter "\\Processore(\_Total)\% Tempo processore" -SampleInterval 2 -MaxSamples 5

```text
Timestamp  CounterSamples
---------  --------------
05/10/2026 11:02:01  \\svr-lab-01\processore(\_total)\% tempo processore:
| 3,12
05/10/2026 11:02:03  ...
```

| Parametro | Significato |
|:--- |:--- |
| -SampleInterval 2 | un campione ogni 2 secondi |
| -MaxSamples 5 | cinque campioni a 2 secondi l'uno dall'altro, poi si ferma |
| valore | percentuale di tempo in cui il processore ha lavorato nell'intervallo |

**Nota:** è la misura che mancava nel 5.B: il carico in un intervallo, non i secondi accumulati.

**PASSO 3** **La memoria disponibile** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 8 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-Counter "\Memoria\MByte disponibili"
```

```text
Timestamp            CounterSamples
05/10/2026 11:04:10  \\svr-lab-01\memoria\mbyte disponibili:  2359
```

**Nota:** MByte disponibili = memoria che Windows può assegnare subito ai processi. 2359 MB sono circa 2,3 GB.

**Attenzione:** il numero da solo non dice se è poco. Va confrontato con la RAM assegnata alla VM (Gestione attività › Prestazioni › Memoria) e osservato nel tempo.

**Suggerimento:** un valore basso letto una volta non prova una perdita di memoria: serve l'andamento.

**CONSOLE** **Monitoraggio prestazioni** **Console · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 9 / 31 **Si esegue su: SVR-LAB-01** **perfmon.msc**

Strumenti di monitoraggio › Monitoraggio prestazioni: grafico in tempo reale **› pulsante + (Aggiungi contatori)** **› Processore › % Tempo processore › istanza \_Total › Aggiungi** **› Memoria › MByte disponibili › Aggiungi › OK** Il grafico mostra le due linee: passate il mouse per leggere i valori

**Nota:** sono gli stessi contatori di GetCounter, con gli stessi nomi italiani.

**Suggerimento:** CPU (%) e MByte hanno scale diverse: leggete i valori nella legenda, un contatore alla volta.

**CONSOLE** **Raccolta e rapporto** **Console · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 10 / 31 **Si esegue su: SVR-LAB-01** **perfmon.msc**

**› Insiemi agenti di raccolta dati › Sistema › Prestazioni di sistema** **› tasto destro › Avvia: raccoglie dati per un periodo (durata da collaudare)** **› Rapporti › Sistema › Prestazioni di sistema** **› aprite il rapporto: riepilogo di CPU, memoria, disco e rete**

## Tempo reale Monitoraggio prestazioni

## Raccolta Insiemi agenti di raccolta dati

## Risultato Rapporti

**Nota:** avviatela ora. Il rapporto lo apriremo durante l'attesa del trigger (blocco B).

# Interpretare senza soglie fisse Quattro domande prima di dire «è lento»

**Quanto dura?** un picco di pochi secondi è normale; un valore alto per minuti va indagato

**Rispetto a che cosa?** RAM della VM, numero di core, carico abituale del server

**Chi lo causa?** processo (5.B, Gestione attività) e servizi ospitati

**Che cosa dicono gli eventi?** errori o riavvii nello stesso intervallo (5.B)

**Attenzione:** le fonti consultate non danno soglie valide per ogni server. Il confronto si fa con il comportamento abituale dello stesso server.

# B

# Utilità di pianificazione

Un'attività che scrive un log, e la prova che è partita **32 minuti** 12 / 31

# I componenti di un'attività

## Che cosa costruiremo, pezzo per pezzo

| Componente | Domanda | Cmdlet |
|:--- |:--- |:--- |
| Trigger | quando parte? | New-ScheduledTaskTrigger |
| Azione | che cosa esegue? | New-ScheduledTaskAction |
| Principal | con quale account e quali privilegi? | New-ScheduledTaskPrincipal |
| Impostazioni e condizioni | come si comporta (es. solo con alimentazione) | valori predefiniti nel laboratorio |
| Registrazione | crea l'attività nell'Utilità | Register-ScheduledTask |

**Nota:** l'account e i privilegi elevati non sono un'impostazione: appartengono al principal.

# Prima di creare: i prerequisiti Chi esegue, dove, con quali regole

**Dove si lavora** SVR-LAB-01, PowerShell come amministratore (LAB\Administrator)

**Cartella** C:\Scripts per lo script e per il file di log

**Account dell'attività** SYSTEM: gira anche senza nessuno collegato; la prova che scrive è la riga del PASSO 9

**Criterio di esecuzione** sul server il predefinito è RemoteSigned: uno script creato sul server può partire

**Attenzione:** -ExecutionPolicy Bypass vale solo per il processo avviato. Se una GPO impone un criterio (Modulo 6.c), prevale quello della GPO.

**PASSO 4** **Creare lo script e rileggerlo** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 15 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> New-Item -Path "C:\Scripts" -ItemType Directory -Force
PS C:> $Riga = 'Add-Content -Path "C:\Scripts\Log.txt" ' + `
  '-Value ("Esecuzione: " + (Get-Date -Format "yyyy-MM-dd HH:mm:ss"))'
PS C:> Set-Content -Path "C:\Scripts\ScriviLog.ps1" -Value $Riga
PS C:> Get-Content -Path "C:\Scripts\ScriviLog.ps1"
```

```text
Add-Content -Path "C:\Scripts\Log.txt" -Value ("Esecuzione: " + (Get-Date -Format "yyyy-MM-dd HH:mm:ss"))
```

| Elemento | Perché |
|:--- |:--- |
| New-Item... -Force | crea la cartella; nessun errore se esiste già |
| $Riga tra apici singoli | il testo non viene eseguito ora: Get-Date girerà a ogni esecuzione |
| Set-Content | scrive la riga nel file ScriviLog.ps1 |
| Get-Content | rilegge il file: deve essere identico alla riga voluta |

$Riga tra apici singoli il testo non viene eseguito ora: Get-Date girerà a ogni esecuzione

**Set-Content** scrive la riga nel file ScriviLog.ps1

**Get-Content** rilegge il file: deve essere identico alla riga voluta

**Nota:** la data aaaa-mm-gg evita di confondere giorno e mese.

**PASSO 5** **Eseguirlo a mano** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 16 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> & "C:\Scripts\ScriviLog.ps1"
PS C:> Get-Content -Path "C:\Scripts\Log.txt"
```

```text
Esecuzione: 2026-10-05 10:41:07
```

| Scrivere | PASSO 4: Set-Content |
|:--- |:--- |
| Rileggere | PASSO 4: Get-Content |
| Eseguire a mano | & esegue il file (operatore di chiamata) |
| Controllare | una riga nuova in Log.txt |
| Solo ora: automatizzare | PASSI 6-8 | **Rileggere** PASSO 4: Get-Content

**PASSO 6** **Azione e account** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 17 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> $Azione = New-ScheduledTaskAction -Execute "powershell.exe" `
  -Argument '-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\ScriviLog.ps1"'
PS C:> $Account = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
```

| Parte | Significato |
|:--- |:--- |
| -Execute "powershell.exe" | l'Utilità avvia un programma; lo script lo esegue PowerShell |
| -Argument '...' | su una riga sola: opzioni e percorso dello script |

**-UserId "SYSTEM" -LogonType ** **ServiceAccount** account di sistema, senza password

**-RunLevel Highest** privilegi elevati (i permessi sui file dipendono invece dalle ACL)

**Nota:** il trigger lo creiamo più tardi, subito prima di registrare (PASSO 8): così l'orario non scade mentre spieghiamo.

**PASSO 7** **Prima: abilitare la Cronologia** **Console · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 18 / 31 **Si esegue su: SVR-LAB-01** **taskschd.msc**

Utilità di pianificazione › pannello Azioni a destra **› «Abilita cronologia di tutte le attività»** **› se compare «Disabilita cronologia di tutte le attività», è già attiva: non toccatela** Lasciate la console aperta: la userete durante l'attesa

**Attenzione:** va fatto prima del trigger. Un registro spento non registra a posteriori: abilitata dopo, la Cronologia della prima esecuzione resta vuota.

**Nota:** la Cronologia della console e il registro Operational del PASSO 10 mostrano le stesse tracce dell'attività.

**PASSO 8** **Trigger e registrazione** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 19 / 31 **Si esegue su: SVR-LAB-01**

| TaskPath  TaskName | State |
|:--- |:--- |
| \ | ScriviLog  Ready |
| PS C:> Get-ScheduledTaskInfo -TaskName "ScriviLog" |  |
| LastRunTime |: (data predefinita: mai eseguita) |
| LastTaskResult |: 267011 |
| NextRunTime |: (l'orario del trigger: tra circa 5 minuti) |

| Elemento | Significato |
|:--- |:--- |
| (Get-Date).AddMinutes(5) | calcolato quando premete Invio: le due righe vanno eseguite una dopo l'altra |
| LastTaskResult 267011 | 0x00041303: «l'attività non è ancora stata eseguita». Non è un errore |
| NextRunTime | deve essere tra circa 5 minuti: se è vuoto, il trigger è già passato |

**Nota:** i nomi dei campi vanno confermati sul laboratorio: la pagina del cmdlet non li elenca.

# Durante l'attesa · 5 minuti

Il tempo del trigger si usa per leggere, non per aspettare

**Il rapporto delle prestazioni** perfmon.msc › Rapporti › Sistema › Prestazioni di sistema: la raccolta avviata nel blocco A

**La Cronologia** taskschd.msc › ScriviLog › scheda Cronologia: c'è l'evento di registrazione?

**L'orario** NextRunTime: quanto manca? Annotate l'orario per il PASSO 9

**Attenzione:** non avviate l'attività a mano adesso. Un avvio con Start-ScheduledTask non prova il trigger.

**Suggerimento:** nel rapporto guardate il riepilogo di CPU e memoria: sono gli stessi contatori del blocco A.

**PASSO 9** **Dopo il trigger** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 21 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-Content -Path "C:\Scripts\Log.txt"
```

```text
Esecuzione: 2026-10-05 10:41:07
Esecuzione: 2026-10-05 10:52:00
```

```powershell
PS C:> (Get-ScheduledTaskInfo -TaskName "ScriviLog").LastTaskResult
```

**taskschd.msc**

Libreria Utilità di pianificazione **› ScriviLog** **› scheda Generale: account (SYSTEM)** **› scheda Trigger: orario** **› scheda Cronologia: avvio e completamento**

**Risultato:** una riga nuova all'orario del trigger e LastTaskResult 0: l'azione è terminata senza errori.

**Attenzione:** la prova è la riga con l'orario del trigger, scritta come SYSTEM. La prima riga è quella della prova manuale (PASSO 5).

**PASSO 10** **Il registro Operational** **Output atteso · da collaudare**

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 22 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-WinEvent -LogName "Microsoft-Windows-TaskScheduler/Operational" -MaxEvents 10
```

```text
TimeCreated          Id LevelDisplayName Message
----------          -- ---------------- ------
05/10/2026 10:52:02  ... Informazioni     (attività \ScriviLog completata)
05/10/2026 10:52:00  ... Informazioni     (attività \ScriviLog avviata)
```

| Che cosa cercare | Che cosa prova |
|---|---|

**registrazione dell'attività** quando è stata creata o modificata

**avvio all'orario del trigger** il trigger è scattato (un avvio manuale avrebbe un altro orario)

**completamento** l'azione è finita, con il suo codice di ritorno

**Nota:** i 10 eventi più recenti possono riguardare altre attività: cercate \ScriviLog e l'orario del trigger. Gli ID non sono nelle fonti del corso: leggete il messaggio.

# Leggere LastTaskResult Un numero da cercare, non da indovinare

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 23 / 31 **Valore** **Esadecimale** **Significato**

| 0x0 | l'azione è terminata senza errori: confermatelo con la riga nel log |
|:--- |:--- |
| 0x00041301 | l'attività è in esecuzione adesso |
| 0x00041303 | l'attività non è ancora stata eseguita |
| altro valore | -- cercatelo nella tabella dei codici di Task Scheduler, poi registro Operational | 0x00041301 l'attività è in esecuzione adesso

0x00041303 l'attività non è ancora stata eseguita

**altro valore** - - cercatelo nella tabella dei codici di Task Scheduler, poi registro Operational

**Attenzione:** un valore diverso da 0 non è per forza un errore dell'azione: 267009 e 267011 sono stati dell'attività.

# Se l'attività non parte

## Ipotesi da verificare una alla volta

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 24 / 31 **Ipotesi** **Come si verifica**

**Trigger nel passato o sbagliato** NextRunTime vuoto, scheda Trigger; LastTaskResult 267011

**Codice diverso da 0** cercarlo (slide precedente); registro Operational; script provato a mano (PASSO 5)

**Account o privilegi** scheda Generale: account e «Esegui con i privilegi più elevati»

**Condizioni non soddisfatte** scheda Condizioni (es. solo con alimentazione da rete)

**Attenzione:** Start-ScheduledTask avvia e non aspetta la fine. Finché LastTaskResult vale 267009 l'attività è in esecuzione; poi controllate la riga nuova nel log.

**Nota:** un'esecuzione con Start-ScheduledTask prova l'azione e l'account, non il trigger.

**Servizi: Running = funziona** serve una prova del servizio (es. una query DNS)

**Servizi: Set-Service per l'avvio ritardato** in PS 5.1: sc.exe config... start= delayed-auto

**Eventi: filtrare sul testo inglese** filtrare per registro e provider

**Processi: CPU = carico attuale** sono secondi accumulati

**Contatori: percorsi inglesi** nomi tradotti: Get-Counter -ListSet

**Attività: codice ≠ 0 = errore** 267009 e 267011 sono stati, non errori

# Riferimento rapido

## I comandi del Modulo 5.C

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 26 / 31 **Comando** **Uso**

**Get-Counter -ListSet Process* · -ListSet *** nomi italiani dei contatori

**Get-Counter "<percorso>" -SampleInterval 2 -MaxSamples 5** misura nel tempo **perfmon.msc** Monitoraggio prestazioni

**New-ScheduledTaskAction -Execute... -Argument...** che cosa esegue

**New-ScheduledTaskPrincipal -UserId... -RunLevel...** account e privilegi

**New-ScheduledTaskTrigger -Once -At...** quando parte (subito prima di registrare)

**Register-ScheduledTask -TaskName... -Action... -Trigger...** crea l'attività

**Get-ScheduledTaskInfo -TaskName...** ultimo risultato e prossima esecuzione

**Start-ScheduledTask -TaskName...** avvio immediato (asincrono)

**Get-WinEvent -LogName "Microsoft-Windows-TaskScheduler/Operational"** registro dell'Utilità

# Laboratorio finale · Modulo 5

## Trova il guasto: un caso per gruppo, poi la restituzione comune

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 27 / 31 **Caso** **Sintomo** **Da dove partire**

l'attività ScriviLog_Lab non ha mai scritto nel log Get-ScheduledTaskInfo, scheda Trigger, Cronologia

sul client la stampa non funziona e lo Spooler non si avvia Get-Service, Get-CimInstance Win32_Service, eventi del Sistema

uno script di monitoraggio con Get-Counter va in errore messaggio di errore, Get-Counter -ListSet

$$ \to $$ **Metodo** osservare → ipotesi → comando che la prova → correzione → verifica finale

$$ \to $$ **In chat** la causa, il comando che la dimostra e la verifica finale

# Restituzione dei tre casi

## Ogni gruppo spiega il proprio caso: tutti vedono le tre diagnosi

| Caso | Causa | Comando che la prova | Verifica finale |
|:--- |:--- |:--- |:--- |
| trigger nel passato | NextRunTime vuoto, LastTaskResult 267011, scheda | Trigger | riga nuova nel log, LastTaskResult 0 |
| Spooler Disabled | Get-CimInstance: Stopped e Disabled; Start-Service in | errore | State Running e StartMode Auto |

trigger nel passato NextRunTime vuoto, LastTaskResult 267011, scheda Trigger riga nuova nel log, LastTaskResult 0

Spooler Disabled Get-CimInstance: Stopped e Disabled; Start-Service in errore State Running e StartMode Auto

percorso inglese errore di Get-Counter; Get-Counter -ListSet Memoria misura con il percorso italiano

**Nota:** in ogni caso la correzione non basta: serve la verifica finale, come in tutto il modulo (verifica → azione → verifica).

Get-Service mostra DNS Running. Che cosa sai?  A) che DNS risolve i nomi  B) che il servizio è in esecuzione  C) che ripartirà al riavvio

Con Windows PowerShell 5.1, come imposti l'avvio automatico ritardato?  A) Start-Service -Delayed  B) Set-Service -StartupType AutomaticDelayedStart  C) sc.exe config Spooler start= delayed-auto

Perché sul DC lo Spooler è Disabled?  A) raccomandazione Microsoft per i Domain Controller  B) per liberare memoria  C) perché il DC non ha stampanti

Quale filtro trova gli eventi dei servizi su un Windows in qualsiasi lingua?  A) Where-Object sul testo «Print Spooler»  B) solo -MaxEvents 10  C) FilterHashtable con LogName e ProviderName

In Get-Process la colonna CPU vale 99. Significa:  A) 99 secondi di processore dall'avvio del processo  B) 99% di CPU adesso  C) 99 thread attivi

Get-Counter "\\Processor(\_Total)\% Processor Time" fallisce sul laboratorio. Perché?  A) manca -MaxSamples  B) i nomi dei contatori sono tradotti  C) perfmon.msc è chiuso

LastTaskResult vale 267011. Significa:  A) lo script contiene un errore  B) accesso negato  C) l'attività non è ancora stata eseguita

Subito dopo Start-ScheduledTask il log non ha la riga nuova. Quale comportamento del comando può spiegarlo?  A) avvia l'attività e non aspetta che finisca  B) cambia il trigger  C) cambia l'account

# Correzione

## Risposta corretta e perché

Modulo 5.C · Prestazioni e attività pianificate · Windows Server 2022 30 / 31 **N.** **Risposta** **Perché**

| B | Running è lo stato: la funzione si prova con una query (5.A, PASSO 2) |
|:--- |:--- |
| C | Set-Service di PS 5.1 non ha l'avvio ritardato; AutomaticDelayedStart esiste solo in PS 7 |
| A | Microsoft raccomanda di disattivarlo sui DC e sui sistemi di amministrazione di AD |
| C | registro e provider non dipendono dalla lingua; il testo sì |
| A | CPU è il tempo accumulato; il carico attuale si misura su un intervallo |
| B | i nomi dei contatori sono localizzati: si trovano con -ListSet |
| C | 0x00041303 = «the task has not yet run»: non è un errore |
| A | Start-ScheduledTask è asincrono: l'attività può essere ancora in esecuzione (267009) |

C Set-Service di PS 5.1 non ha l'avvio ritardato; AutomaticDelayedStart esiste solo in PS 7

A Microsoft raccomanda di disattivarlo sui DC e sui sistemi di amministrazione di AD

C registro e provider non dipendono dalla lingua; il testo sì

A CPU è il tempo accumulato; il carico attuale si misura su un intervallo

B i nomi dei contatori sono localizzati: si trovano con -ListSet

C 0x00041303 = «the task has not yet run»: non è un errore

A Start-ScheduledTask è asincrono: l'attività può essere ancora in esecuzione (267009)

| 6.a | PowerShell | variabili, pipeline, hashtable e criteri di esecuzione visti qui di passaggio |
|:--- |:--- |:--- |
| 6.b | Batch | sc.exe query e un'attività pianificata che esegue un file.cmd |
| 6.c | GPO | la disattivazione dello Spooler sui DC e il registro Operational di Group Policy |

**Nota:** lasciate registrata l'attività ScriviLog e lo Spooler del DC su Disabled: sono lo stato di partenza del Modulo 6. ScriviLog_Lab serviva solo al laboratorio.
