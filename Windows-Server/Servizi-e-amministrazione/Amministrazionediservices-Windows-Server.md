# MODULO 5.A - AMMINISTRAZIONE **I servizi di Windows Server**

## Leggere, controllare e configurare i servizi di SVR-LAB-01 e CLI-W11-01

Concetti Leggere i servizi Stato e avvio Lo Spooler

# Obiettivi e prerequisiti

Dal concetto di servizio alla sua configurazione

## Alla fine saprete

- Spiegare che cos'è un servizio e chi lo gestisce - Leggere stato, tipo di avvio e dipendenze - Usare la console Servizi e PowerShell

- Arrestare, avviare e disattivare un servizio, verificando ogni  passo

## Prerequisiti

- Moduli 3 e 4 completati: SVR-LAB-01 è il DC di lab.local, CLIW11-01 è nel dominio - Accesso come LAB\Administrator - PowerShell aperto come amministratore

- Di PowerShell basta la pipeline: \| passa il risultato al comando  successivo

| A | Concetti: processo, servizio, Service Control Manager | 8′ |
|:--- |:--- |:--- |
| B | Leggere i servizi: PowerShell e console | 17′ |
| C | Stato e tipo di avvio | 13′ |
| D | Lo Spooler: sul DC e sul client | 12′ |

**B** **Leggere i servizi: PowerShell e console** **17′** **C** **Stato e tipo di avvio** **13′**

**D** **Lo Spooler: sul DC e sul client** **12′**

**Poi:** 5.B eventi e processi, 5.C prestazioni e attività pianificate, con laboratorio e quiz di tutto il modulo.

# A

# Concetti Che cosa stiamo gestendo, e chi lo gestisce **8 minuti** 4 / 26

# Processo e servizio Due cose diverse, spesso confuse

## Processo - Un programma in esecuzione in memoria

- Di solito avviato da un utente, spesso con una finestra

- Finisce quando l'utente lo chiude o si disconnette - Esempio: notepad.exe

## Servizio

- Un componente che lavora in background (sullo sfondo, senza  finestra)

## Voi

PowerShell (Get-Service, Stop-Service...) o la console Servizi

## Service Control Manager

componente di Windows che riceve le richieste e gestisce i servizi

## Servizio

es. Spooler, DNS: si avvia, si arresta, cambia tipo di avvio

**Nota:** PowerShell e la console non avviano il servizio da soli: chiedono al Service Control Manager (SCM), che esegue. Per questo i due strumenti mostrano gli stessi dati.

**Suggerimento:** il nome SCM tornerà nel 5.B, come autore (provider) degli eventi sui servizi.

# Nome interno e nome visualizzato Ogni servizio ha due nomi

| Nome interno (Name) | Nome visualizzato (DisplayName) | Dove |
|:--- |:--- |:--- |
| Spooler | Spooler di stampa | server e client |
| DNS | Server DNS | solo sul DC |
| Netlogon | Accesso rete | DC e membri | **Spooler** Spooler di stampa server e client **DNS** Server DNS solo sul DC **Netlogon** Accesso rete DC e membri

**Attenzione:** con -Name si usa il nome interno, che non cambia con la lingua. Il nome visualizzato (-DisplayName) è tradotto: su Windows in italiano non è quello delle guide in inglese.

**Nota:** i nomi visualizzati in italiano vanno letti sul laboratorio (colonna DisplayName).

# B

# Leggere i servizi Elenco, servizi del DC, dipendenze, console **17 minuti** 8 / 26

**PASSO 1** **Tutti i servizi del server** **Output atteso · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 9 / 26 **Si esegue su: SVR-LAB-01**

PS C:> Get-Service \| Sort-Object Status, DisplayName

Status   Name        DisplayName ------   ----        ----------Stopped  AppMgmt... Running  ADWS        Servizi Web Active Directory Running  DNS         Server DNS...

| Colonna | Significato |
|:--- |:--- |
| Status | stato adesso: Running (in esecuzione), Stopped (arrestato), Paused |
| Name | nome interno, quello da usare nei comandi |
| DisplayName | nome visualizzato, tradotto in italiano |

**Attenzione:** Running significa «in esecuzione», non «funziona bene». Per sapere se lavora serve una prova (PASSO 2).

# I servizi da conoscere su questo DC

## Che cosa succede, nel nostro laboratorio, se si fermano

| Servizio | A che cosa serve | Se si ferma, su SVR-LAB-01 |
|:--- |:--- |:--- |
| NTDS | database di Active Directory | accessi e modifiche del dominio falliscono |
| DNS | risoluzione dei nomi del dominio | il client non trova il DC (record SRV) |
| Netlogon | ricerca del DC, registrazione dei record DNS | accessi di dominio a rischio |
| Kdc | autenticazione Kerberos | nuovi accessi e ticket non vengono emessi |
| ADWS | servizio usato dai cmdlet di AD | i cmdlet di AD falliscono; il dominio funziona |

**Nota:** il laboratorio ha un solo DC, che è anche l'unico DNS. In produzione, con più DC, un altro server può rispondere: l'effetto dipende dall'architettura.

**PASSO 2** **Un servizio, e la prova che lavora** **Output atteso · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 11 / 26 **Si esegue su: SVR-LAB-01** PS C:> Get-Service -Name DNS

```text
Status  Name  DisplayName
------  ----  -----------
Running  DNS  Server DNS
```

| Name | Type TTL  Section IPAddress |
|:--- |:--- |
| lab.local  A | 600  Answer  192.168.134.130 |

| Comando | Che cosa prova |
|:--- |:--- |
| Get-Service -Name DNS | che il servizio è in esecuzione |
| Resolve-DnsName... -Server 127.0.0.1 | che il DNS di questo server risponde a una query (non prova il client) |

## Suggerimento: davanti a «il DNS non funziona» si controllano entrambe le cose: stato e funzione.

**PASSO 3** **Le dipendenze** **Output atteso · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 12 / 26 **Si esegue su: SVR-LAB-01**

PS C:> Get-Service -Name DNS -RequiredServices Status   Name   DisplayName Running...    (servizi senza i quali DNS non parte)

| PS C:> Get-Service -Name DNS -DependentServices | (se non compare nulla, nessun servizio dipende da DNS) |
|:--- |:--- |
| Parametro | Domanda a cui risponde |
| -RequiredServices | di quali servizi ha bisogno DNS per partire? |
| -DependentServices | quali servizi si fermano se arresto DNS? |
| Nota: sono le dipendenze dichiarate al Service Control Manager. Un elenco vuoto non vuol dire che nulla dipenda dal DNS: il dominio ne ha bisogno | (slide 10). | **Parametro** **Domanda a cui risponde**

**Nota:** sono le dipendenze dichiarate al Service Control Manager. Un elenco vuoto non vuol dire che nulla dipenda dal DNS: il dominio ne ha bisogno (slide 10).

**Attenzione:** prima di arrestare un servizio si guarda chi dipende da lui. Stop-Service -Force arresta anche i servizi dipendenti.

**CONSOLE** **La console Servizi** **Console · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 13 / 26 **Si esegue su: SVR-LAB-01**

**services.msc › doppio clic su «Server DNS»**

**› Generale: nome del servizio, stato, Tipo di avvio (Automatico, Automatico (avvio ** **ritardato), Manuale, Disabilitato)** **› Connessione: l'account con cui il servizio gira (es. Sistema locale)** **› Ripristino: che cosa fare se il servizio si arresta in modo anomalo** **› Relazioni di dipendenza: le stesse informazioni del PASSO 3**

**Nota:** la console mostra in un unico posto stato, tipo di avvio, account e dipendenze. PowerShell serve per ripetere e verificare.

**Attenzione:** su DNS guardate soltanto. Non cambiate nulla dei servizi del DC.

# C

# Stato e tipo di avvio

Che cosa fa adesso, e che cosa farà al prossimo avvio **13 minuti** 14 / 26

# Stato e tipo di avvio Due domande diverse

## Stato (State) - «Adesso è in esecuzione?» - Running, Stopped, Paused - Cambia con Start-Service e Stop-Service - Si perde al riavvio di Windows

## Tipo di avvio (StartMode) - «Che cosa farà Windows all'avvio?» - Auto, Manual, Disabled

- L'avvio ritardato è una proprietà a parte: DelayedAutoStart - Cambia con Set-Service o sc.exe

**Nota:** Get-CimInstance Win32_Service mostra le due cose insieme, compreso l'avvio ritardato.

**PASSO 4** **Leggere il tipo di avvio** **Output atteso · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 16 / 26 **Si esegue su: SVR-LAB-01**

PS C:> Get-CimInstance Win32_Service -Filter "Name='DNS'" \| >> Select-Object Name, State, StartMode, DelayedAutoStart

```text
Name State  StartMode DelayedAutoStart
---- -----  --------- ----------------
DNS  Running Auto  False
```

| Campo | Significato |
|:--- |:--- |
| State | stato adesso (come Status di Get-Service) |
| StartMode | Auto · Manual · Disabled |
| DelayedAutoStart | True = automatico ritardato (parte poco dopo gli altri) |

**Nota:** -Filter "Name='DNS'" usa il nome interno. Il filtro si applica prima di restituire i dati, come un WHERE.

# Leggere le combinazioni

## Che cosa si osserva, e quali ipotesi verificare

**State** **StartMode** **Che cosa si osserva** **Ipotesi da verificare**

**Running** Auto attivo; configurato per l'avvio automatico situazione attesa

**Running** Manual attivo; parte solo su richiesta richiesta di un utente, di un altro servizio o di un programma: normale per molti servizi

**Stopped** Auto doveva partire, ora è fermo errore, arresto voluto o arresto automatico: leggere gli eventi (5.B)

**Stopped** Disabled fermo e non avviabile scelta intenzionale? Verificare chi l'ha decisa

**Attenzione:** la combinazione è un indizio, non una diagnosi. La causa si cerca negli eventi e nelle dipendenze.

**PASSO 5** **L'avvio ritardato (sul client)** **Output atteso · da collaudare**

**Si esegue su: CLI-W11-01 (come amministratore)**

| Spooler Auto | True |
|:--- |:--- |
| PS C:> sc.exe config Spooler start= auto |  |
| PS C:> Get-CimInstance Win32_Service -Filter "Name='Spooler'" \| |  |
| >> Select-Object Name, StartMode, DelayedAutoStart |  |
| Spooler Auto | False |

**Attenzione:** dopo start= c'è uno spazio. In PowerShell scrivete sc.exe: sc da solo è un altro comando (alias di Set-Content).

**Nota:** Set-Service di Windows PowerShell 5.1 non ha un valore per l'avvio ritardato. L'ultima riga ripristina il tipo di avvio originale.

# D

# Lo Spooler Disattivarlo sul DC, gestirlo sul client **12 minuti** 19 / 26

# Perché sul DC lo Spooler va spento

Una raccomandazione Microsoft, applicata in laboratorio

## Sul Domain Controller

- Microsoft raccomanda di disattivare lo Spooler sui DC e sui  sistemi di amministrazione di AD - Un DC non deve fare da server di stampa

- Meno servizi attivi = meno superficie di attacco

## Sul client - Lo Spooler serve per stampare: resta attivo

- È un servizio sicuro per esercitarsi con arresto, avvio e riavvio - Lo useremo anche nel laboratorio finale

**Nota:** in produzione la disattivazione si distribuisce con una GPO (Modulo 6.c). Qui la facciamo a mano, per imparare a cambiare il tipo di avvio.

**PASSO 6** **Disattivare lo Spooler sul DC** **Output atteso · da collaudare**

Modulo 5.A · Servizi · Windows Server 2022 21 / 26 **Si esegue su: SVR-LAB-01**

PS C:> Stop-Service -Name Spooler PS C:> Set-Service -Name Spooler -StartupType Disabled

| Name | State | StartMode |
|:--- |:--- |:--- |
| ---- | ----- | --------- | **Comando** **Effetto**

**Stop-Service** arresta il servizio adesso (cambia lo stato)

**Set-Service -StartupType ** **Disabled** impedisce l'avvio, anche al riavvio (cambia il tipo)

**Attenzione:** servono entrambi. Cambiare il tipo di avvio non arresta un servizio già in esecuzione; arrestarlo senza Disabled lo lascia ripartire al riavvio.

**PASSO 7** **Il ciclo completo, sul client** **Output atteso · da collaudare**

**Si esegue su: CLI-W11-01 (come amministratore)**

| Running | Spooler | Spooler di stampa |
|:--- |:--- |:--- |
| PS C:> Stop-Service -Name Spooler |  |  |
| PS C:> Get-Service -Name Spooler |  |  |
| Stopped | Spooler | Spooler di stampa |
| PS C:> Start-Service -Name Spooler |  |  |
| PS C:> Get-Service -Name Spooler |  |  |
| Running | Spooler | Spooler di stampa |

| Verifica | lo stato prima di agire |
|:--- |:--- |
| Azione | un comando alla volta: nessun output = richiesta accettata |
| Verifica | lo stato dopo: è cambiato davvero? |

**PASSO 8** **Restart-Service** **Output atteso · da collaudare**

**Si esegue su: CLI-W11-01 (come amministratore)**

PS C:> Restart-Service -Name Spooler PS C:> Get-Service -Name Spooler Running  Spooler  Spooler di stampa

## Restart-Service arresta e riavvia in un solo comando

## Quando usarlo - Dopo aver letto eventi e dipendenze - Su un servizio bloccato non critico

- Dopo una modifica di configurazione che lo richiede

## Sul DC - NTDS, DNS, Netlogon, Kdc: mai «al volo»

- Riavvio solo in una finestra di manutenzione

- Il riavvio interrompe per qualche istante il servizio agli utenti

# Errori comuni Da riconoscere prima che diventino guasti

**Running = funziona** in esecuzione non vuol dire che risponda: serve una prova

**Manual = avviato da una persona** parte su richiesta: anche di un servizio o di un programma

**Set-Service per l'avvio ritardato** in PS 5.1 non esiste: sc.exe config... start= delayed-auto

**sc invece di sc.exe in PowerShell** sc è un alias di Set-Content

**Riavviare prima di guardare** prima eventi e dipendenze, poi l'azione

**Nome visualizzato dopo -Name** -Name vuole il nome interno, che non cambia con la lingua

# Riferimento rapido

| Comando | Uso | Dove |
|:--- |:--- |:--- |
| Get-Service \| Sort-Object Status, DisplayName | elenco dei servizi | tutti |
| Get-Service -Name DNS | un servizio | tutti |
| Get-Service -Name X -RequiredServices | da chi dipende | tutti |
| Get-Service -Name X -DependentServices | chi dipende da lui | tutti |
| Get-CimInstance Win32_Service -Filter "Name='X'" | stato, tipo, ritardo | tutti |
| Stop-Service / Start-Service -Name X | arresta / avvia | tutti |
| Restart-Service -Name X | arresta e riavvia | tutti |
| Set-Service -Name X -StartupType Disabled | tipo di avvio | tutti |
| sc.exe config X start= delayed-auto | avvio ritardato | tutti |
| services.msc | console Servizi | tutti |

**Get-Service \| Sort-Object Status, DisplayName** elenco dei servizi tutti **Get-Service -Name DNS** un servizio tutti

**Get-Service -Name X -RequiredServices** da chi dipende tutti

**Get-Service -Name X -DependentServices** chi dipende da lui tutti

**Get-CimInstance Win32_Service -Filter "Name='X'"** stato, tipo, ritardo tutti

**Stop-Service / Start-Service -Name X** arresta / avvia tutti

**Restart-Service -Name X** arresta e riavvia tutti

**Set-Service -Name X -StartupType Disabled** tipo di avvio tutti

**sc.exe config X start= delayed-auto** avvio ritardato tutti **services.msc** console Servizi tutti

**Che cosa abbiamo fatto** arrestato e disattivato lo Spooler del DC; arrestato, avviato e riavviato quello del client

**Che cosa cercheremo** le tracce che il Service Control Manager lascia nel registro eventi per le operazioni eseguite

**Nel 5.B** ritroveremo quelle tracce con il Visualizzatore eventi e Get-WinEvent

**Nota:** non chiudete le sessioni. Il 5.B riparte da SVR-LAB-01 con PowerShell come amministratore.
