# MODULO 5.B - AMMINISTRAZIONE **Eventi e processi**

## Ritrovare le tracce nel registro eventi e capire chi sta usando le risorse

Visualizzatore eventi Get-WinEvent Gestione attività Dal PID al servizio

# Obiettivi e prerequisiti

Dalla traccia di un'operazione alla diagnosi di un processo

## Alla fine saprete

- Distinguere i registri e i livelli degli eventi

- Filtrare gli eventi nella console e con Get-WinEvent - Ritrovare avvii, arresti e riavvii - Leggere Gestione attività e Get-Process

- Risalire da un processo ai servizi che ospita

## Prerequisiti - Modulo 5.A completato

- SVR-LAB-01: PowerShell come amministratore, accesso  LAB\Administrator

- Stato e tipo di avvio dei servizi (5.A, blocco C)

**A** **Il Visualizzatore eventi: registri, livelli, filtri** **26′**

**B** **Gestione attività e processi** **22′**

**Chiusura:** errori comuni, riferimento rapido, ponte verso il 5.C (circa 8 minuti).

# A

# Il Visualizzatore eventi Dove Windows registra quello che succede **26 minuti** 4 / 21

# Registri di Windows e registri di applicazioni

## Dove cercare

| Registro | Che cosa contiene | Nel corso |
|:--- |:--- |:--- |
| Sistema | eventi dei componenti di Windows, compresi i servizi | avvii e arresti dello Spooler (5.A) |
| Applicazione | eventi delle applicazioni installate | consultazione |
| Sicurezza | controllo degli accessi (audit) | accessi riusciti e falliti |
| Registri applicazioni e servizi | un registro per componente, es. Operational | TaskScheduler (5.C), GroupPolicy (6.c) |

**Nota:** i registri di applicazioni e servizi hanno categorie (Admin, Operational, Analytic, Debug). Operational raccoglie il funzionamento normale di un componente: è quello che useremo.

# I livelli degli eventi

## La gravità, e il numero che la rappresenta nei filtri

**Livello** **Valore** **In console (IT)** **Significato**

| Critical | Critico | problema grave (es. riavvio imprevisto) |
|:--- |:--- |:--- |
| Error | Errore | un'operazione non è riuscita |
| Warning | Avviso | possibile problema futuro |
| Informational | Informazioni | un'operazione è avvenuta |
| Verbose | Dettagliato | diagnostica dettagliata | **Error** Errore un'operazione non è riuscita **Warning** Avviso possibile problema futuro

**Attenzione:** Informazioni non significa «tutto bene». Dice che un evento è avvenuto: per capire se è un problema si legge il messaggio, e si guardano solo gli eventi che riguardano l'operazione cercata.

**CONSOLE** **Il Visualizzatore eventi** **Console · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 7 / 21 **Si esegue su: SVR-LAB-01** **eventvwr.msc**

Registri di Windows › Sistema: gli eventi più recenti in alto **› Filtra registro corrente... (pannello Azioni)** **› Origini eventi: Service Control Manager** **› Livello e ID evento, se servono, poi OK** **› Crea visualizzazione personalizzata...: salva lo stesso filtro con un nome** Doppio clic su un evento: scheda Generale con il messaggio, scheda Dettagli

**Nota:** la vista personalizzata si salva, si esporta e si importa su un altro server.

**Suggerimento:** l'origine (provider) dice chi ha scritto l'evento. Per i servizi è il Service Control Manager del 5.A.

**PASSO 1** **Gli ultimi eventi del Sistema** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 8 / 21 **Si esegue su: SVR-LAB-01**

PS C:> Get-WinEvent -LogName System -MaxEvents 10 ProviderName: Service Control Manager

```text
TimeCreated  Id LevelDisplayName Message
-----------  -- ---------------- -------
05/10/2026 10:21:14... Informazioni  (testo del messaggio in italiano)
```

| Campo | Significato |
|:--- |:--- |
| TimeCreated | quando è avvenuto l'evento |
| Id · LevelDisplayName | codice dell'evento · livello |
| ProviderName · Message | chi lo ha scritto · che cosa dice | **TimeCreated** quando è avvenuto l'evento

**Nota:** -MaxEvents 10 restituisce gli ultimi dieci eventi del registro, di qualsiasi provider.

**PASSO 2** **Solo gli eventi dei servizi** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 9 / 21 **Si esegue su: SVR-LAB-01**

PS C:> $Filtro = @{LogName='System'; ProviderName='Service Control Manager'} PS C:> Get-WinEvent -FilterHashtable $Filtro -MaxEvents 20

TimeCreated          Id LevelDisplayName Message -----------          -- ---------------- ------05/10/2026 10:21:14... Informazioni     (Spooler di stampa: tipo di avvio cambiato) 05/10/2026 10:20:51... Informazioni     (Spooler di stampa: arrestato)...

| Elemento | Perché |
|:--- |:--- |
| $Filtro = @{... } | registro e provider: il filtro lavora nel registro, non dopo |
| ProviderName | non dipende dalla lingua: vale anche su Windows in italiano |

$Filtro = @{... } registro e provider: il filtro lavora nel registro, non dopo

**ProviderName** non dipende dalla lingua: vale anche su Windows in italiano

**Attenzione:** lo stesso ID compare per molti servizi diversi. Quale servizio e quale cambio di stato lo dice il messaggio, non l'ID.

**PASSO 3** **Ritrovare lo Spooler del DC** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 10 / 21 **Si esegue su: SVR-LAB-01**

PS C:> Get-WinEvent -FilterHashtable $Filtro -MaxEvents 50 | >> Where-Object { $_.Message -like "*Spooler di stampa*" }

| TimeCreated | Id LevelDisplayName Message |
|:--- |:--- |
| 05/10/2026 10:21:14... Informazioni | (Spooler di stampa: tipo di avvio cambiato) |
| 05/10/2026 10:20:51... Informazioni | (Spooler di stampa: arrestato) |

**Prima il filtro nel registro** $Filtro riduce gli eventi letti (PASSO 2)

**Poi il filtro sul testo** Where-Object tiene solo i messaggi che citano il servizio

**Il testo va copiato** «Spooler di stampa» è il nome italiano letto al PASSO 2

**Attenzione:** il filtro sul testo guarda solo i 50 eventi letti e dipende dalla lingua («*Print Spooler*» qui non trova nulla). Nessun risultato non prova che l'evento manchi nel registro.

**PASSO 4** **Quando si è riavviato il server?** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 11 / 21 **Si esegue su: SVR-LAB-01**

$$ PS C:> $Avvii = @{LogName='System'; Id=6005,6008,1074,41} $$ PS C:> Get-WinEvent -FilterHashtable $Avvii -MaxEvents 10 | Select-Object TimeCreated, Id, ProviderName

| TimeCreated | Id ProviderName |
|:--- |:--- |
| ----------- | -- ------------ | **ID** **Che cosa indica**

avvio del servizio Registro eventi: indizio di un avvio di Windows l'arresto precedente è stato imprevisto

riavvio o arresto avviato da un processo per conto di un utente (chi e perché)

riavvio senza arresto regolare: non ne indica la causa

# Event ID da tenere a portata di mano

## Sempre con registro e provider, e con il livello di certezza

**ID** **Registro** **Provider (ProviderName)** **Significato e fonte**

**6005 · 6008** Sistema EventLog avvio del Registro eventi · arresto imprevisto - certo

Sistema User32 riavvio o arresto avviato da un processo - certo

Sistema Microsoft-Windows-Kernel-Power riavvio senza arresto regolare - certo (client)

**4624 · 4625** Sicurezza Microsoft-Windows-Security-Auditing accesso riuscito · non riuscito, se l'auditing è attivo - certo

**7036 · 7031** Sistema Service Control Manager nessuna fonte ufficiale: leggere il messaggio

## Suggerimento: non serve memorizzare gli ID. Serve sapere in quale registro cercare, con quale provider, e leggere il messaggio.

# B

# Gestione attività e processi Chi sta usando le risorse, adesso **22 minuti** 13 / 21

**CONSOLE** **Gestione attività** **Console · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 14 / 21 **Si esegue su: SVR-LAB-01**

**Ctrl+Maiusc+Esc (oppure taskmgr) › Più dettagli**

**› Processi: percentuale di CPU e memoria usate adesso** **› Prestazioni: grafici di CPU, memoria, disco e rete** **› Utenti: chi è collegato al server** **› Dettagli: ogni processo con il suo PID (numero identificativo)** **› Servizi: nome, PID e stato dei servizi**

**Nota:** la scheda Processi mostra il carico attuale, aggiornato ogni secondo circa.

**Attenzione:** non terminate processi sul DC. Qui si guarda soltanto.

**PASSO 5** **Get-Process e il tempo di CPU** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 15 / 21 **Si esegue su: SVR-LAB-01**

PS C:> Get-Process \| Sort-Object CPU -Descending \| >> Select-Object -First 5 Name, Id, CPU, WorkingSet

```text
Name  Id  CPU WorkingSet
----  --  --- ----------
MsMpEng  3408  99,03 263725056
System  4  57,73  147456
lsass  720  9,11 68259840
```

| Colonna | Significato |
|:--- |:--- |
| CPU | secondi di processore usati dall'avvio del processo (somma di tutti i core) |
| WorkingSet | memoria fisica occupata, in byte |
| Attenzione: CPU è un totale accumulato, non il carico di adesso. Un processo acceso da giorni ha molti secondi anche se ora è fermo: per il carico | attuale usate la scheda Processi o i contatori (5.C). |

**Attenzione:** CPU è un totale accumulato, non il carico di adesso. Un processo acceso da giorni ha molti secondi anche se ora è fermo: per il carico attuale usate la scheda Processi o i contatori (5.C).

# Processo, servizio e svchost Perché un PID non basta

## svchost.exe un processo «contenitore» con un proprio PID **Servizio A** **Servizio B** **Servizio C**

**Nota:** molti servizi di Windows non hanno un processo proprio: girano dentro un svchost.exe, che può ospitarne più di uno. Vedere svchost con molta CPU non dice ancora quale servizio la usa.

**Suggerimento:** è il legame annunciato nel 5.A: un servizio vive sempre dentro un processo.

**PASSO 6** **Dal PID ai servizi ospitati** **Output atteso · da collaudare**

Modulo 5.B · Eventi e processi · Windows Server 2022 17 / 21 **Si esegue su: SVR-LAB-01**

(Gestione attività › Dettagli: svchost.exe con CPU alta → PID, per esempio 1234) PS C:> Get-CimInstance Win32_Service -Filter "ProcessId=1234" \| >> Select-Object Name, DisplayName, State

| Name     DisplayName                       State | Winmgmt  Strumentazione gestione Windows   Running |
|:--- |:--- |
| Passaggio | Perché |
| PID dalla scheda Dettagli | il PID cambia: va letto adesso, non copiato da una slide |
| -Filter "ProcessId=..." | restituisce tutti i servizi ospitati da quel processo | **Passaggio** **Perché**

## Attenzione: questo è l'inizio della diagnosi, non la conclusione. Se il processo ospita più servizi, non sapete ancora quale consuma CPU.

**Nota:** un'alternativa dal prompt dei comandi è tasklist /svc, che mostra i servizi per PID.

# Come si prosegue la diagnosi Dal sospetto alla prova

**Servizi ospitati** PASSO 6: quali servizi girano nel processo

**Eventi del servizio** PASSI 2-3: errori o arresti recenti di quei servizi

**Consumo nel tempo** 5.C: contatori per alcuni minuti, non una sola lettura

**Solo dopo, l'azione** riavvio di un servizio non critico, o intervento pianificato

**Nota:** domanda tipica da colloquio. «Uno svchost usa il 70% di CPU: che cosa fai?» Risposta: PID, servizi ospitati, eventi, misura nel tempo; il riavvio arriva per ultimo.

# Errori comuni

Da riconoscere prima che diventino diagnosi sbagliate

**Informazioni = tutto bene** dice solo che un evento è avvenuto: leggete il messaggio

**Filtrare sul testo inglese** i messaggi sono tradotti: filtrate per registro e provider

**Memorizzare gli ID senza provider** lo stesso ID ha senso solo con registro e provider

**CPU di Get-Process = carico attuale** sono secondi accumulati dall'avvio

**Copiare il PID da una slide** il PID cambia: leggetelo ogni volta

**svchost con CPU alta = colpevole trovato** prima i servizi ospitati, poi gli eventi

# Riferimento rapido

## I comandi del Modulo 5.B, con la sintassi minima

Modulo 5.B · Eventi e processi · Windows Server 2022 20 / 21 **Comando** **Uso**

**Get-WinEvent -LogName System -MaxEvents 10** ultimi eventi di un registro

$Filtro = @{LogName='System'; ProviderName='...'} prepara un filtro

Get-WinEvent -FilterHashtable $Filtro -MaxEvents 20 filtra nel registro... | Where-Object { $_.Message -like "*testo*" } filtra sul testo (lingua!)

$$ @{LogName='System'; Id=6005,6008,1074,41} $$ avvii e riavvii

**Get-Process \| Sort-Object CPU -Descending** processi per tempo di CPU

**Get-CimInstance Win32_Service -Filter "ProcessId=..."** servizi ospitati da un PID **eventvwr.msc · taskmgr** le due console

**Che cosa abbiamo visto** il registro dice che cosa è successo; Gestione attività che cosa succede ora

**Che cosa manca** misurare CPU e memoria per un periodo: le misure dicono quanto e quando, non chi

**Nel 5.C** Monitoraggio prestazioni, poi un'attività pianificata da verificare nel registro Operational

**Nota:** il registro Operational dell'Utilità di pianificazione è un registro di applicazioni e servizi, come quelli visti nel blocco A.
