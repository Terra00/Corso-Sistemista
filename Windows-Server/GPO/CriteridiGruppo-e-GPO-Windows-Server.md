# MODULO 6.c - AUTOMAZIONE

**Group Policy: creare, filtrare e verificare le ** **GPO**

## Tre GPO e uno script di accesso su SVR-LAB-01, verificati sul client Windows 11 con due utenti diversi

Creare e configurare Filtri: sicurezza e WMI Verifica utente e computer Script e guasto

# Obiettivi e prerequisiti

Dopo PowerShell (6a) e Batch (6b): distribuire impostazioni e script a utenti e computer del dominio

## Alla fine saprete - Creare, configurare e collegare una GPO

- Applicarla solo a un gruppo (filtro di sicurezza)

- Applicarla solo a certi computer (filtro WMI)

- Verificare il risultato per l'utente e per il computer - Distribuire uno script di accesso - Trovare perché una GPO non si applica

## Prerequisiti - Dominio lab.local con OU IT (Modulo 3)

- Mario Rossi in IT_Admin, Luca Bianchi non membro, entrambi  nella OU IT - Client CLI-W11-01 unito al dominio

- Console gpmc.msc e modulo GroupPolicy sul server - Ciao.cmd del 6b in C:\Scripts

**Nota:** GPO = Group Policy Object, oggetto Criteri di gruppo. Nella console italiana le GPO si chiamano anche «oggetti Criteri di gruppo».

# Il percorso dell'unità

Sei blocchi, dalla teoria alla prova sul client

| A | Che cos'è una GPO e come si applica | 12′ |
|:--- |:--- |:--- |
| B | Creare, configurare e collegare tre GPO | 30′ |
| C | Filtrare: sicurezza e WMI | 32′ |
| D | Verificare: utente e computer | 31′ |
| E | Distribuire uno script di accesso (dal 6b) | 10′ |
| F | Manutenzione, mentre lo script attende | 8′ |

**B** **Creare, configurare e collegare tre GPO** **30′** **C** **Filtrare: sicurezza e WMI** **32′**

**D** **Verificare: utente e computer** **31′**

**E** **Distribuire uno script di accesso (dal 6b)** **10′**

**F** **Manutenzione, mentre lo script attende** **8′**

**Apertura e controlli:** 8 minuti. Chiusura (troubleshooting, laboratorio «trova il guasto», quiz): 19 minuti. Totale: 150 minuti.

# Lo scenario: tre GPO, tre casi diversi

## Che cosa costruiremo e che cosa ci aspettiamo · nomi e account sono fittizi, a scopo didattico

| GPO | Parte | Collegata a | Filtro | Risultato atteso |
|:--- |:--- |:--- |:--- |:--- |
| Blocco_PannelloControllo | Utente | OU IT | sicurezza: IT_Admin | Mario: blocco e script; Luca no |
| Sfondo_Aziendale | Utente | OU IT | nessuno | sfondo per Mario e Luca |
| Messaggio_Accesso | Computer | dominio, attivato dopo il filtro | WMI: solo client | messaggio sul client, non sul DC |

**Messaggio_Accesso** Computer dominio, attivato dopo il filtro WMI: solo client messaggio sul client, non sul DC **Identità e computer** **Ruolo**

**Mario Rossi (m.rossi)** OU IT, membro di IT_Admin

**Luca Bianchi (l.bianchi)** OU IT, non membro di IT_Admin

**CLI-W11-01 / SVR-LAB-01** client Windows 11 (192.168.134.129) / domain controller (192.168.134.130)

**PASSO 1** **Prima di iniziare: i controlli** **Output atteso · da collaudare**

```powershell
PS C:> Get-ADUser -Filter 'SamAccountName -eq "m.rossi" -or SamAccountName -eq "l.bianchi"' | Select-Object SamAccountName, DistinguishedName
```

```text
SamAccountName DistinguishedName
-------------- ----------------
m.rossi        CN=Mario Rossi,OU=IT,OU=Azienda,DC=lab,DC=local
l.bianchi      CN=Luca Bianchi,OU=IT,OU=Azienda,DC=lab,DC=local
```

```powershell
PS C:> Get-ADGroupMember IT_Admin | Select-Object SamAccountName
```

```text
SamAccountName
--------------
m.rossi
```

```powershell
PS C:> Get-ADComputer CLI-W11-01 | Select-Object DistinguishedName
```

```text
DistinguishedName
-----------------
CN=CLI-W11-01,CN=Computers,DC=lab,DC=local
```

**Risultato atteso:** Mario e Luca nella OU IT, solo Mario in IT_Admin, il client nel contenitore Computers.

**Attenzione:** non modificate la OU Neoassunti, i file in C:\Scripts e l'attività ScriviLog: sono lo stato lasciato dai moduli precedenti.

# A

# Che cos'è una GPO e come si applica

L'oggetto, l'ordine in cui le GPO si sommano e i tempi di applicazione **12 minuti** 6 / 46

# Che cos'è una GPO

Un insieme di impostazioni con due metà: una per i computer, una per gli utenti

## GPC · in Active Directory

Il Group Policy Container: l'oggetto della GPO nel database di AD (versione, stato).

# GPT · in SYSVOL

Il Group Policy Template: la cartella in SYSVOL con i file delle impostazioni e degli script, replicata sui DC. **Parte** **Quando si applica e a chi**

**Configurazione computer** all'avvio e ai refresh: vale per il computer, chiunque acceda

**Configurazione utente** all'accesso e ai refresh: segue l'utente su ogni computer

**Nota:** il collegamento non sta nella GPO. È un attributo (GPLink) del sito, del dominio o della OU a cui la GPO è collegata.

# Chi vince: ordine di applicazione

Le GPO si sommano; in caso di conflitto conta l'ordine

# L **Locale**

# S **Sito**

# D **Dominio**

# OU **Unità organizzativa**

**Regola 1:** le GPO si applicano nell'ordine L → S → D → OU; in caso di conflitto vince l'ultima applicata, la più vicina all'oggetto.

**Regola 2:** tra più GPO collegate allo stesso contenitore vince quella con il numero d'ordine di collegamento (link order) più basso.

## Block Inheritance

Su un dominio o una OU: blocca le GPO ereditate dai livelli superiori (non quelle con collegamento Enforced).

## Enforced

Proprietà di un collegamento: le sue impostazioni prevalgono su quelle in conflitto dei livelli inferiori e non si possono bloccare. I filtri continuano a valere.

# Quando arrivano le impostazioni

Le GPO non sono istantanee: Windows le riapplica a intervalli

Modulo 6.c · Group Policy · Windows Server 2022 9 / 46 **Momento** **Che cosa succede**

**Avvio del computer** si applica la parte Computer

**Accesso dell'utente** si applica la parte Utente

**Refresh in background** ogni 90 minuti più un ritardo casuale fino a 30 (90-120)

**Domain Controller** criteri computer ogni 5 minuti

**gpupdate /force** riapplica subito tutte le impostazioni

**Script di accesso** partono solo all'accesso dell'utente (dopo circa 5 minuti), non ai refresh

**Suggerimento:** alcune impostazioni (il messaggio di accesso, lo script) si vedono solo al successivo accesso o riavvio, anche dopo gpupdate /force.

**PASSO 2** **Che cosa c'è già: Get-GPO -All** **Output atteso · da collaudare**

PS C:> Get-GPO -All \| Select-Object DisplayName, GpoStatus

```text
DisplayName  GpoStatus
-----------  ---------
Default Domain Policy  AllSettingsEnabled
```

| Campo | Significato |
|:--- |:--- |
| Default Domain Policy | GPO predefinita collegata al dominio (password, account) |
| Default Domain Controllers Policy | GPO predefinita collegata alla OU Domain Controllers |
| GpoStatus | quali parti sono attive: AllSettingsEnabled = entrambe |

# B

# Creare, configurare e collegare

Tre GPO, dalla creazione all'impostazione al collegamento **30 minuti** 11 / 46

**PASSO 3** **Creare le tre GPO: New-GPO** **Output atteso · da collaudare**

```powershell
PS C:> New-GPO -Name "Blocco_PannelloControllo"
PS C:> New-GPO -Name "Sfondo_Aziendale"
PS C:> New-GPO -Name "Messaggio_Accesso"
```

| DisplayName |: Messaggio_Accesso |
|:--- |:--- |
| DomainName |: lab.local |
| GpoStatus |: AllSettingsEnabled |
| WmiFilter |: |

## Attenzione: New-GPO crea la GPO vuota e NON la collega. Finché non è configurata e collegata, non produce alcun effetto.

**Nota:** l'output mostrato è un estratto (ultima GPO creata). WmiFilter è vuoto: lo useremo nel blocco C.

**PASSO 4** **Configurare: blocco del Pannello di controllo** **Console · da collaudare**

**Console gpmc.msc → Blocco_PannelloControllo → Modifica**

Configurazione utente **› Criteri** **› Modelli amministrativi** **› Pannello di controllo** Impedisci l'accesso al Pannello di controllo e alle impostazioni del PC = Attivata

## In inglese (fonte)

User Configuration > Control Panel > Prohibit access to Control Panel and PC settings

**In alternativa alla console (una sola delle due strade):**

PS C:> Set-GPRegistryValue -Name "Blocco_PannelloControllo" \` >> -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" \` >> -ValueName "NoControlPanel" -Type DWord -Value 1

**PASSO 5** **Configurare: lo sfondo aziendale** **Console · da collaudare**

PS C:> Copy-Item "C:\Windows\Web\Wallpaper\Windows\img0.jpg" \` >> -Destination "\\SVR-LAB-01\NETLOGON\sfondo.jpg"

**Console gpmc.msc → Sfondo_Aziendale → Modifica**

Configurazione utente **› Criteri** **› Modelli amministrativi** **› Desktop** **› Desktop** Sfondo del desktop = Attivata Nome sfondo: \\SVR-LAB-01\NETLOGON\sfondo.jpg

## Perché NETLOGON

**Console gpmc.msc → Messaggio_Accesso → Modifica**

Configurazione computer **› Criteri** **› Impostazioni di Windows** **› Impostazioni di sicurezza** **› Criteri locali** **› Opzioni di sicurezza** Accesso interattivo: titolo del messaggio = «Laboratorio LAB»    ·    testo del messaggio = «Accesso riservato ai partecipanti del corso»

**Nota:** questa è un'impostazione del COMPUTER. Compare nella schermata di accesso, prima che l'utente inserisca la password.

**Attenzione:** collegata al dominio, la GPO raggiungerebbe anche il DC. Per questo la collegheremo disattivata e la attiveremo dopo il filtro WMI.

**PASSO 7** **Collegare le GPO: New-GPLink** **Output atteso · da collaudare**

```powershell
PS C:> New-GPLink -Name "Blocco_PannelloControllo" -Target "OU=IT,OU=Azienda,DC=lab,DC=local"
PS C:> New-GPLink -Name "Sfondo_Aziendale" -Target "OU=IT,OU=Azienda,DC=lab,DC=local"
PS C:> New-GPLink -Name "Messaggio_Accesso" -Target "DC=lab,DC=local" -LinkEnabled No
```

| Enabled |: False |
|:--- |:--- |
| Enforced |: False |
| Target |: DC=lab,DC=local |
| Order |: 2 | **Collegamento** **Perché lì**

**due GPO utente → OU IT** contiene gli utenti Mario e Luca: collegamenti attivi

**Messaggio_Accesso → dominio** contiene (anche) il computer CLI-W11-01: collegamento disattivato

**Attenzione:** -LinkEnabled No. Il collegamento esiste ma non agisce: lo abiliteremo al PASSO 14, dopo il filtro WMI. Order 2 vale se sul dominio c'è solo la Default Domain Policy.

**PASSO 8** **Verificare lato server** **Output atteso · da collaudare**

```powershell
PS C:> (Get-GPInheritance -Target "OU=IT,OU=Azienda,DC=lab,DC=local").GpoLinks | Select-Object DisplayName, Enabled, Order
```

```text
DisplayName              Enabled Order
-----------              ------- ----
Blocco_PannelloControllo    True     1
Sfondo_Aziendale            True     2
```

```powershell
PS C:> (Get-GPInheritance -Target "DC=lab,DC=local").GpoLinks | Select-Object DisplayName, Enabled, Order
```

```text
DisplayName           Enabled Order
-----------           ------- ----
Default Domain Policy    True     1
Messaggio_Accesso       False     2
```

```powershell
PS C:> Get-GPOReport -Name "Blocco_PannelloControllo" -ReportType Html -Path C:\Report_Blocco.html
```

**Attenzione:** Get-GPInheritance mostra come sono configurati i contenitori, non che cosa arriva davvero a un utente. Quello lo dice il client (blocco D).

**Suggerimento:** aprite Report_Blocco.html e controllate che l'impostazione del Pannello di controllo risulti «Attivata».

# C

# Filtrare: sicurezza e WMI

Decidere a chi e su quali computer si applica una GPO, senza spostare oggetti tra OU **32 minuti** 18 / 46

# Il filtro di sicurezza

Una GPO si applica a chi ha due permessi: Read e Apply Group Policy

## Situazione predefinita

Authenticated Users (utenti e computer autenticati) ha Read + Apply: la GPO si applica a tutti nel contenitore collegato.

## Che cosa vogliamo

Blocco_PannelloControllo solo per IT_Admin: Apply a IT_Admin, solo Read ad Authenticated Users.

**Attenzione:** non togliete anche la lettura ad Authenticated Users. Read serve perché la GPO venga letta; è Apply che decide a chi si applica.

**Nota:** l'elenco delle GPO si legge con l'account del computer (aggiornamento MS16-072): anche il computer deve poter leggere la GPO. Authenticated Users comprende anche i computer.

**Suggerimento:** il filtro si gestisce nella scheda Ambito della GPO oppure con Set-GPPermission.

**PASSO 9** **Apply al gruppo IT_Admin** **Output atteso · da collaudare**

```powershell
PS C:\> Set-GPPermission -Name "Blocco_PannelloControllo" `
    -TargetName "IT_Admin" -TargetType Group -PermissionLevel GpoApply
```

| Parametro | Significato |
| --- | --- |
| -TargetName / -TargetType | chi riceve il permesso: il gruppo IT_Admin |
| -PermissionLevel GpoApply | Read + Apply Group Policy: la GPO si applica ai membri |

PS C:> Set-GPPermission -Name "Blocco_PannelloControllo" \` >> -TargetName "IT_Admin" -TargetType Group -PermissionLevel GpoApply **Parametro** **Significato**

**-TargetName / -TargetType** chi riceve il permesso: il gruppo IT_Admin

**-PermissionLevel GpoApply** Read + Apply Group Policy: la GPO si applica ai membri

**Nota:** dopo questo passo Authenticated Users ha ancora Apply: la GPO arriverebbe ancora anche a Luca. Il passo successivo completa il filtro.

**PASSO 10** **Authenticated Users: solo lettura** **Output atteso · da collaudare**

```powershell
PS C:> (Get-GPPermission -Name "Blocco_PannelloControllo" -All).Trustee.Name
Domain Admins
Enterprise Admins
SYSTEM
Authenticated Users
ENTERPRISE DOMAIN CONTROLLERS
IT_Admin
```

```powershell
PS C:> Set-GPPermission -Name "Blocco_PannelloControllo" `
  -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead -Replace
```

**Attenzione:** in -TargetName scrivete il nome esatto letto nel primo comando. Su un DC in italiano potrebbe essere tradotto.

## PermissionLevel GpoRead

Authenticated Users continua a leggere la GPO, ma non la applica più.

## Replace

Sostituisce il permesso esistente (Apply) con Read. Senza -Replace un permesso più alto non verrebbe ridotto.

**PASSO 11** **Verificare i permessi: Get-GPPermission** **Output atteso · da collaudare**

```powershell
PS C:> Get-GPPermission -Name "Blocco_PannelloControllo" -All | Select-Object @{n="Trustee";e={$_.Trustee.Name}}, Permission
```

```text
Trustee  Permission
-------  ----------
Domain Admins  GpoEditDeleteModifySecurity
Enterprise Admins  GpoEditDeleteModifySecurity
SYSTEM  GpoEditDeleteModifySecurity
Authenticated Users  GpoRead
ENTERPRISE DOMAIN CONTROLLERS  GpoRead
IT_Admin  GpoApply
```

## Risultato: solo IT_Admin ha GpoApply. Authenticated Users legge ma non applica: il filtro di sicurezza è completo.

# Il filtro WMI

## Applicare una GPO solo ai computer che rispondono "vero" a una domanda

Un filtro WMI è una query (in linguaggio WQL) valutata sul computer di destinazione: se il risultato è vero la GPO si applica, se è falso no. Una query che non si riesce a valutare conta come falsa. A una GPO si collega un solo filtro WMI.

**Proprietà di Win32_OperatingSystem** **Valori utili**

**ProductType** 1 = client · 2 = domain controller · 3 = server non DC

**BuildNumber** TESTO, non numero: 20348 = Windows Server 2022; tra i client, da 22000 = Windows 11 (non usatelo da solo)

**Version** 10.0 per Windows 10, Windows 11 e Server 2022: da sola non li distingue

**Attenzione:** non copiate le query vecchie con Version like "6.%": valgono per sistemi precedenti a Windows 10.

**PASSO 12** **Che cosa risponde ogni computer** **Output atteso · da collaudare**

Modulo 6.c · Group Policy · Windows Server 2022 24 / 46 **Sul client CLI-W11-01**

```powershell
PS C:> Get-CimInstance Win32_OperatingSystem | Select-Object ProductType, BuildNumber
```

```text
ProductType BuildNumber
----------- -----------
1           22631
```

**Sul domain controller SVR-LAB-01**

```powershell
PS C:> Get-CimInstance Win32_OperatingSystem | Select-Object ProductType, BuildNumber
```

```text
ProductType BuildNumber
----------- -----------
2           20348
```

## Risultato: ProductType distingue il client (1) dal DC (2). La query «ProductType = "1"» sarà vera solo sul client.

**PASSO 13** **Creare e associare il filtro WMI** **Console · da collaudare**

Modulo 6.c · Group Policy · Windows Server 2022 25 / 46 **Console gpmc.msc**

Foresta › Domini › lab.local **› Filtri WMI → Nuovo** Nome: Solo client Aggiungi → spazio dei nomi root\CIMv2 Query: SELECT * FROM Win32_OperatingSystem WHERE ProductType = "1" Salva Messaggio_Accesso → scheda Ambito **› Filtro WMI: Solo client**

**Nota:** per i filtri WMI non c'è un cmdlet nel modulo GroupPolicy: si usa la console.

**Attenzione:** un solo filtro WMI per GPO. Lo stesso filtro può servire più GPO.

**PASSO 14** **Ora il collegamento può agire: Set-GPLink** **Output atteso · da collaudare**

PS C:> Set-GPLink -Name "Messaggio_Accesso" -Target "DC=lab,DC=local" -LinkEnabled Yes

| Enabled |: True |
|:--- |:--- |
| Enforced |: False |
| Order |: 2 |

## Risultato: il collegamento al dominio è attivo, ma il filtro «Solo client» lascia passare solo i computer con ProductType 1.

**Nota:** l'ordine conta. Prima il filtro, poi il collegamento attivo: partendo dallo stato iniziale del corso, il DC non riceve il messaggio.

# D · Verificare: chi, dove, con quale sessione

## Ogni prova ha la sua macchina, il suo account e il suo ambito

**Passo** **Macchina** **Account e sessione** **Comando** **Che cosa si vede**

| CLI-W11-01 | LAB\m.rossi · standard | gpresult /r /scope user | due GPO utente applicate |
|:--- |:--- |:--- |:--- |
| CLI-W11-01 | LAB\l.bianchi · standard | gpresult /r /scope user | blocco filtrato (sicurezza) |
| CLI-W11-01 | LAB\Administrator · amministrativa | gpresult /r /scope computer | messaggio applicato |

CLI-W11-01 LAB\l.bianchi · standard gpresult /r /scope user blocco filtrato (sicurezza)

CLI-W11-01 LAB\Administrator · amministrativa gpresult /r /scope computer messaggio applicato

CLI-W11-01 LAB\Administrator · amministrativa gpresult /h <file> /user LAB\m.rossi /scope user report di Mario

SVR-LAB-01 LAB\Administrator gpresult /r /scope computer messaggio filtrato (WMI)

**Suggerimento:** utente e computer si verificano separatamente. Una sessione standard mostra bene la parte utente; per la parte computer serve una sessione da amministratore.

**PASSO 15** **Mario su CLI-W11-01: la parte utente** **Output atteso · da collaudare**

```powershell
PS C:> gpupdate /target:user /force
Updating policy...
User Policy update has completed successfully.
```

```text
PS C:> gpresult /r /scope user

Applied Group Policy Objects
----------------------------
        Blocco_PannelloControllo
        Sfondo_Aziendale
```

## Che cosa si vede

Dopo disconnessione e nuovo accesso: - c'è lo sfondo aziendale - il Pannello di controllo è bloccato

*Testo dell'output in inglese, come nelle fonti: sul laboratorio in italiano è tradotto. Da sostituire con l'output del collaudo.*

## Risultato: le due GPO utente agiscono su Mario, membro di IT_Admin e utente della OU IT.

**Nota:** /target:user aggiorna solo la parte utente. La parte computer si verifica al PASSO 17, in una sessione da amministratore.

**PASSO 16** **Luca sullo stesso client** **Output atteso · da collaudare**

Modulo 6.c · Group Policy · Windows Server 2022 29 / 46

```text
PS C:> gpresult /r /scope user

Applied Group Policy Objects
-----------------------------
Sfondo_Aziendale

The following GPOs were not applied because they were filtered out
-------------------------------------------
Blocco_PannelloControllo
Filtering:  Denied (Security)
```

## Che cosa si vede - sfondo: sì - Pannello di controllo: si apre

*Testo dell'output in inglese, come nelle fonti: sul laboratorio in italiano è tradotto. Da sostituire con l'output del collaudo.*

## Risultato: stessa OU di Mario, ma Luca non è in IT_Admin. Il filtro di sicurezza fa la differenza: GPO filtrata per sicurezza.

**PASSO 17** **Il client come computer: il messaggio** **Output atteso · da collaudare**

**Su CLI-W11-01, PowerShell aperto come LAB\Administrator (sessione amministrativa)**

```powershell
PS C:> gpupdate /force
Updating policy...
Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

```text
PS C:> gpresult /r /scope computer

Applied Group Policy Objects
-----------------------------
Messaggio_Accesso
```

## Che cosa si vede

Al successivo accesso, per chiunque acceda al client, compare il messaggio «Laboratorio LAB».

*Testo dell'output in inglese, come nelle fonti: sul laboratorio in italiano è tradotto. Da sostituire con l'output del collaudo.*

## Risultato: il filtro WMI è vero sul client (ProductType 1): Messaggio_Accesso si applica al computer CLI-W11-01.

**PASSO 18** **Il report di Mario e il registro** **Output atteso · da collaudare**

**Stessa sessione amministrativa su CLI-W11-01: il report riguarda Mario, il registro il computer**

```powershell
PS C:> gpresult /h "$env:USERPROFILE\Report_Mario.html" /user LAB\m.rossi /scope user /f
```

```powershell
PS C:> Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 5 | Select-Object TimeCreated, Id
```

| Strumento | A che cosa serve |
|---|---|

**/user... /scope user** report HTML della sola parte utente di Mario: GPO applicate, filtrate e impostazioni **/f** sovrascrive il report se lo rifate

**registro Operational** sequenza dell'elaborazione dei criteri, con Event ID: guardate l'orario

**RSoP.msc** vista grafica, ma NON mostra tutte le impostazioni: preferire gpresult

**PASSO 19** **Sul domain controller: niente messaggio** **Output atteso · da collaudare**

```text
PS C:> gpresult /r /scope computer

Applied Group Policy Objects
----------------------------
        Default Domain Controllers Policy
        Default Domain Policy

The following GPOs were not applied because they were filtered out
Messaggio_Accesso
Filtering:  Denied (WMI Filter)
```

## Che cosa si vede

All'accesso al DC il messaggio non compare: per il DC la query ProductType = "1" è falsa (vale 2).

*Testo dell'output in inglese, come nelle fonti: sul laboratorio in italiano è tradotto. Da sostituire con l'output del collaudo.*

## Risultato: stessa GPO collegata al dominio, ma il filtro WMI la esclude dal DC.

# E

# Distribuire uno script di accesso

Nel 6b avete scritto Ciao.cmd: ora lo fa partire una GPO. Mentre lo script attende, il blocco F **10 minuti** 33 / 46

**PASSO 20** **Lo script nella GPO Blocco_PannelloControllo** **Console · da collaudare**

Modulo 6.c · Group Policy · Windows Server 2022 34 / 46 **File: C:\Scripts\AccessoGPO.cmd**

```bat
@echo off
rem Script di accesso distribuito con la GPO Blocco_PannelloControllo
echo Accesso di %USERNAME% > "%USERPROFILE%\AccessoGPO.txt"
```

**Console gpmc.msc → Blocco_PannelloControllo → Modifica**

Configurazione utente **› Criteri** **› Impostazioni di Windows** **› Script (Accesso/Disconnessione)** Accesso → Mostra file: copiate AccessoGPO.cmd nella cartella che si apre Aggiungi → Sfoglia → AccessoGPO.cmd → OK → OK

## Perché un file

Gli script di accesso delle GPO girano nascosti e partono dopo circa 5 minuti: il file nel profilo è la prova che lo script è partito.

**Poi, su CLI-W11-01 come Mario:** gpupdate /target:user /force, disconnessione e nuovo accesso. Lasciate la sessione aperta: intanto, blocco F.

# F

# Manutenzione

Mentre lo script di Mario attende: backup, ripristino e rimozione, li mostra il formatore **8 minuti** 35 / 46

**PASSO 21** **Backup delle GPO** **Output atteso · da collaudare**

```powershell
PS C:> New-Item -Path "C:\BackupGPO" -ItemType Directory
PS C:> Backup-GPO -All -Path "C:\BackupGPO"
```

| DisplayName |: Blocco_PannelloControllo |
|:--- |:--- |
| BackupDirectory: C:\BackupGPO |  |
| CreationTime |:... |

## Che cosa salva

Ogni GPO del dominio, comprese le due predefinite. Restituisce un oggetto di backup per ciascuna: qui un estratto.

## Elenco dei backup

Console gpmc.msc → Oggetti Criteri di gruppo → Gestisci backup. Il cmdlet Get-GPOBackup NON esiste.

**PASSO 22** **Ripristinare una GPO** **Output atteso · da collaudare**

```powershell
PS C:> Restore-GPO -Name "Blocco_PannelloControllo" -Path "C:\BackupGPO"
```

```text
DisplayName: Blocco_PannelloControllo GpoStatus: AllSettingsEnabled
```

```powershell
PS C:> (Get-GPInheritance -Target "OU=IT,OU=Azienda,DC=lab,DC=local").GpoLinks
```

**Attenzione:** nessuna fonte ufficiale afferma che il ripristino rimetta anche i collegamenti. Dopo un ripristino, verificateli sempre.

**Nota:** Restore-GPO usa il backup più recente e funziona solo se la GPO esiste ancora. Una GPO eliminata si ricrea con Import-GPO CreateIfNeeded: poi vanno rifatti collegamenti e filtri.

**PASSO 23** **Sospendere e rimuovere (a fine corso)** **Output atteso · da collaudare**

Modulo 6.c · Group Policy · Windows Server 2022 38 / 46 **Sospendere (console gpmc.msc)**

Sfondo_Aziendale → scheda Dettagli **› Stato GPO: Impostazioni configurazione utente disattivate**

```powershell
PS C:> Remove-GPLink -Name "Sfondo_Aziendale" `
  -Target "OU=IT,OU=Azienda,DC=lab,DC=local"
PS C:> Remove-GPO -Name "Sfondo_Aziendale"
```

| Operazione | Effetto |
|:--- |:--- |
| Stato GPO | la GPO resta collegata ma una sua parte non si applica |
| Remove-GPLink | la GPO resta nel dominio ma non agisce più su quella OU |
| Remove-GPO | elimina la GPO: Restore-GPO non la recupera più |

**PASSO 24** **Lo script è partito? Solo per Mario** **Output atteso · da collaudare**

**CLI-W11-01 · Mario, nella sessione rimasta aperta**

```powershell
PS C:> Get-ChildItem "$env:USERPROFILE\AccessoGPO.txt" | Select-Object Name, LastWriteTime
```

```text
Name  LastWriteTime
----  -------------
```

**CLI-W11-01 · Luca, dopo il suo accesso**

```powershell
PS C:> Test-Path "$env:USERPROFILE\AccessoGPO.txt"
False
```

**Risultato:** il file di Mario è stato scritto dopo il suo ultimo accesso. Luca non riceve la GPO (PASSO 16), quindi nemmeno lo script.

**Nota:** gpresult dice che la GPO è applicata, non che lo script sia partito: per questo si guarda il file. Se dopo 5 minuti manca, controllate assegnazione dello script e gpresult.

# Troubleshooting: la GPO non si applica

Si controlla nell'ordine in cui Windows decide

| Controllo | Domanda da porsi |
| --- | --- |
| Collegamento | la GPO è collegata al contenitore giusto e il collegamento è abilitato? |
| Parte giusta | impostazione utente dove sta l'utente, impostazione computer dove sta il computer? |
| Stato GPO | la parte che serve è attiva? |
| Filtro di sicurezza | l'utente o il computer ha Read + Apply? |
| Filtro WMI | la query è vera su quel computer? |
| Tempi | gpupdate /force, nuovo accesso o riavvio fatti? |
| Diagnosi | gpresult con l'ambito giusto (user o computer) e registro Operational |

# Errori comuni

Quelli corretti nelle versioni precedenti del deck, e altri frequenti

**GPO creata ma non configurata** senza impostazioni non succede nulla

**Apply lasciato ad Authenticated Users** il filtro di sicurezza non restringe

**Read tolto a tutti** la GPO non viene nemmeno letta

**GPO computer collegata a una OU di soli utenti** non agisce su nessun computer

**GPO computer attiva prima del filtro WMI** arriva anche ai computer da escludere

**Query WMI con Version like "6.%" o "10.%"** non distingue Windows 10, 11 e Server 2022

**Remove-GPO e poi Restore-GPO** Restore-GPO non ricrea una GPO eliminata

**Get-GPOBackup, Set-GPO** non esistono nel modulo GroupPolicy

# Collegamenti con il resto del corso

Le GPO distribuiscono ciò che abbiamo imparato altrove

## Script di accesso · 6b

Fatto nel blocco E: AccessoGPO.cmd distribuito con Blocco_PannelloControllo. Lo script sta nella GPO; quello assegnato nel profilo dell'account si legge da Netlogon.

## Execution Policy · 6a

Una GPO può impostare l'Execution Policy di PowerShell: MachinePolicy vale per il computer, UserPolicy per l'utente.

## Delega e trust · Modulo 3

Delegare la gestione delle GPO e lavorare tra più domini sono argomenti del Modulo 3.

# Riferimento rapido

## Da consultare durante il laboratorio

Modulo 6.c · Group Policy · Windows Server 2022 43 / 46 **Comando** **Uso** **Get-GPO -All** elencare le GPO **New-GPO -Name** creare una GPO vuota

**Set-GPRegistryValue** configurare un'impostazione basata sul Registro

**New-GPLink -Name -Target [-LinkEnabled No]** collegare (anche disattivato)

**Set-GPLink... -LinkEnabled Yes \| No** abilitare o disattivare un collegamento

**(Get-GPInheritance -Target...).GpoLinks** collegamenti di un dominio o di una OU

**Set-GPPermission... -Replace** filtro di sicurezza

**Get-GPPermission -All** verificare i permessi

**Get-CimInstance Win32_OperatingSystem** valori per il filtro WMI

**gpresult /r /scope user \| computer** verificare utente o computer

**Backup-GPO · Restore-GPO · Import-GPO** backup, ripristino, ricreazione

**Remove-GPLink · Remove-GPO** scollegare ed eliminare (a fine corso)

# Laboratorio finale · 10 minuti

Trova il guasto: Blocco_PannelloControllo non arriva più a Mario

Come Mario su CLI-W11-01: gpupdate /target:user /force, poi gpresult /r /scope user. La GPO è applicata, filtrata o assente?

Se è assente: sul DC, (Get-GPInheritance -Target "OU=IT,OU=Azienda,DC=lab,DC=local").GpoLinks. Il collegamento c'è? È abilitato?

Se è filtrata per sicurezza: sul DC, Get-GPPermission -Name "Blocco_PannelloControllo" -All

Scrivete in chat la causa e il comando che la dimostra

**Nota:** il guasto lo introduce il formatore; non serve correggerlo, serve trovarlo e dimostrarlo con il comando giusto.

# Quiz di verifica

Rispondete in chat con il numero della domanda e la lettera

Quale cmdlet NON esiste nel modulo GroupPolicy? A) Get-GPOBackup  B) Backup-GPO  C) Restore-GPO

Per applicare una GPO solo a IT_Admin: A) Read + Apply a IT_Admin e solo Read ad Authenticated Users  B) solo Apply a IT_Admin  C) togliere tutto ad Authenticated Users

Una GPO con impostazioni computer va collegata: A) alla OU degli utenti  B) al gruppo IT_Admin  C) a un sito, dominio o OU nel cui ambito rientra il computer

In un filtro WMI, ProductType = 1 indica: A) un domain controller  B) un client  C) un server membro

Nel nostro scenario gpresult sul DC mostra Messaggio_Accesso filtrata dal filtro WMI. Significa che: A) la GPO non è collegata  B) la query è falsa per quel computer  C) manca il permesso Read

# Correzione del quiz Risposta corretta e perché

Modulo 6.c · Group Policy · Windows Server 2022 46 / 46 **N.** **Risposta** **Perché**

| A | l'elenco ufficiale del modulo non contiene Get-GPOBackup |
|:--- |:--- |
| A | senza togliere Apply ad Authenticated Users il filtro non restringe; la lettura resta (qui ad Authenticated Users) |
| C | conta l'ambito: sito, dominio o OU che contiene il computer (Computers non si può collegare) |
| B | ProductType: 1 client, 2 domain controller, 3 server non DC |
| B | il filtro WMI è stato valutato e la query è falsa (il DC ha ProductType 2) |
| Fine del Modulo 6: PowerShell automatizza, il Batch esegue controlli semplici, le GPO distribuiscono impostazioni e script agli utenti e ai | computer destinatari. |

A senza togliere Apply ad Authenticated Users il filtro non restringe; la lettura resta (qui ad Authenticated Users)

C conta l'ambito: sito, dominio o OU che contiene il computer (Computers non si può collegare)

B ProductType: 1 client, 2 domain controller, 3 server non DC

B il filtro WMI è stato valutato e la query è falsa (il DC ha ProductType 2)

**Fine del Modulo 6:** PowerShell automatizza, il Batch esegue controlli semplici, le GPO distribuiscono impostazioni e script agli utenti e ai computer destinatari.
