# MODULO 4 - STORAGE E FILE SERVER

**Dischi, volumi NTFS e condivisioni di ** **cartelle**

## Dal disco appena aggiunto alla cartella condivisa con i permessi giusti, un comando alla volta su SVR-LAB-01

A · Disco, volume, file system B · Cartella e condivisione C · Identità e permessi Laboratorio finale

# Obiettivi e prerequisiti

Che cosa saprete fare alla fine del modulo e che cosa serve per partire

## Alla fine saprete

- Preparare un nuovo disco: inizializzarlo, partizionarlo e  formattarlo in NTFS

- Condividere una cartella in rete e raggiungerla dal client

- Leggere e modificare i permessi di condivisione e NTFS

- Assegnare i permessi con il modello AGDLP e verificarli

## Prerequisiti

- Modulo 3 completato: LAB\m.rossi (Mario Rossi) in  IT_Admin, LAB\l.bianchi (Luca Bianchi) non membro, entrambi nella OU IT, primo accesso già fatto

- Poter aggiungere a SVR-LAB-01 un disco virtuale da 20 GB;  lettere E: ed F: libere - Snapshot della VM prima di iniziare

**Durata:** circa 3 ore, con una pausa di 10 minuti dopo il blocco A. Ogni comando si esegue in diretta, uno alla volta.

## Il percorso del modulo

Tre blocchi che si appoggiano l'uno sull'altro, poi il laboratorio

# A **Disco, volume, file system** Dal disco RAW al volume E: in NTFS **44′**

# B **Cartella e condivisione**

Dalla cartella locale al percorso \\SVR-LAB-01\Documenti **26′**

# C **Identità e permessi**

Perché Mario viene respinto e come farlo entrare con AGDLP **55′**

# L **Laboratorio finale**

Una nuova condivisione in sola lettura, in autonomia **25′**

# L'ambiente di laboratorio

Dove lavoriamo e con quali identità: fissiamolo prima di eseguire qualsiasi comando

# SVR-LAB-01

Windows Server 2022 Domain Controller del dominio LAB Disco 0 (80 GB): sistema Disco 1 (20 GB): dati, da aggiungere nel blocco A

## Client

Windows 11 unito al dominio LAB Da qui si prova l'accesso alla condivisione

| Oggetto | Ruolo nel modulo |
| --- | --- |
| LAB\Administrator | amministrazione |
| LAB\m.rossi (Mario) | in IT_Admin |
| LAB\l.bianchi (Luca) | NON in IT_Admin |

| Livello | Che cos'è | Blocco |
| --- | --- | --- |
| ACL (liste dei permessi) | chi può fare che cosa su file e cartelle | Blocco C |
| Cartelle e file | ciò che gli utenti vedono e condividono | Blocco B |
| File system (NTFS) | organizza dati, nomi e permessi: si crea formattando il volume | Blocco A |
| Volume | la partizione come la vede Windows; può avere una lettera (es. E:) | Blocco A |
| Partizione | una porzione del disco, descritta dalla tabella delle partizioni | Blocco A |
| Tabella delle partizioni (GPT) | l'indice che dice dove inizia e finisce ogni partizione | Blocco A |
| Disco | il dispositivo fisico o virtuale | Blocco A |

# MBR contro GPT

Le due tabelle delle partizioni: su un server moderno si sceglie GPT

| Caratteristica | MBR (Master Boot Record) | GPT (GUID Partition Table) |
|:--- |:--- |:--- |
| Nato con | i PC con BIOS (anni '80) | lo standard UEFI |
| Dimensione massima del disco | 2 TB | oltre 2 TB (limite pratico molto più alto) |
| Partizioni | 4 primarie (di più solo con trucchi) | fino a 128 in Windows |
| Protezione dell'indice | una sola copia | copia di riserva a fine disco |
| Avvio del sistema | con BIOS | con UEFI |

**Suggerimento:** per i dischi dati di Windows Server 2022 usiamo GPT. MBR serve solo per compatibilità con sistemi o strumenti datati.

## PASSO 1 **Quali dischi vede il server: Get-Disk**

PS C:> Get-Disk

```text
Number Friendly Name              HealthStatus OperationalStatus Total Size Partition Style
------ -------------              ------------ ----------------- ---------- ---------------
0      VMware Virtual NVMe Disk   Healthy        Online             80 GB     GPT
```

| Campo | Che cosa significa |
| --- | --- |
| Number | numero del disco: lo useremo nei comandi successivi |
| HealthStatus | stato di salute (Healthy = nessun problema rilevato) |
| OperationalStatus | Online = disco attivo e utilizzabile da Windows |
| Partition Style | tabella delle partizioni: GPT, MBR o RAW (nessuna) |

**Nota:** per ora c'è un solo disco, il disco 0, che contiene Windows Server.

# Perché serve un secondo disco

Il disco 0 contiene il sistema operativo: in laboratorio non si tocca

## Disco 0 · 80 GB · sistema

Contiene Windows Server, Active Directory e il registro. Un errore su questo disco può rendere il server inutilizzabile.

## Disco 1 · 20 GB · dati

| Disco virtuale nuovo, dedicato ai dati. | Su questo creeremo partizioni, volumi e condivisioni. |
|:--- |:--- |
| Impostazione del nuovo disco virtuale | Valore consigliato |
| Dimensione | 20 GB |
| Allocazione immediata dello spazio | No |
| Nome del file | Disk1-Data-20GB.vmdk |

**PASSO 2** **Il disco nuovo è visibile ma RAW** **Output atteso · da collaudare**

PS C:> Get-Disk \| Where-Object PartitionStyle -eq 'RAW'

```text
Number Friendly Name  HealthStatus OperationalStatus Total Size Partition Style
------ -------------  ------------ ----------------- ---------- ---------------
1  VMware Virtual disk Healthy Online 20 GB RAW
```

## Che cosa vuol dire RAW

Windows vede il disco ma non riconosce una tabella delle partizioni. Su un disco nuovo: niente partizioni, volumi o file system.

## Perché filtrare

Il filtro mostra solo i dischi senza tabella delle partizioni. Numero, nome e dimensione confermano che è il disco appena aggiunto.

**Controllate anche OperationalStatus:** se il disco 1 è Offline, portatelo Online (Gestione disco › clic destro › Online) prima del PASSO 3.

**Attenzione:** RAW non significa vuoto. Anche un disco con dati non riconosciuti appare RAW: inizializzate solo il disco appena aggiunto.

# Il ciclo di vita di un disco

Quattro passi, ognuno possibile solo dopo il precedente

# RAW spazio senza indice

# GPT **Initialize-Disk** crea la tabella

## Partizione **New-Partition** ritaglia lo spazio

## Volume NTFS **Format-Volume** crea il file system

# E:\ pronto per le cartelle

**Nota:** senza inizializzazione non esiste GPT; senza partizione non esiste un volume; senza formattazione il volume non può contenere file. L'ordine non si può cambiare.

## PASSO 3 **Inizializzare il disco: Initialize-Disk**

```powershell
PS C:> Initialize-Disk -Number 1 -PartitionStyle GPT
PS C:>
```

| Parte del comando | Significato |
|---|---|
| -Number 1 | il disco da inizializzare: quello trovato al passo 2 |
| -PartitionStyle GPT | scrive sul disco una tabella delle partizioni GPT |
| nessun output | in PowerShell un comando riuscito spesso non stampa nulla |

**Nota:** dopo questo passo il disco ha la tabella GPT e, probabilmente, già la piccola partizione riservata MSR; nessun volume per i dati, nessuna lettera. Non è ancora utilizzabile.

**PASSO 4** **Verificare: da RAW a GPT** **Output atteso · da collaudare**

```text
Number Friendly Name  HealthStatus OperationalStatus Total Size Partition Style
------ -------------  ------------ ----------------- ---------- ---------------
1  VMware Virtual disk Healthy Online 20 GB GPT
```

```text
PartitionNumber  DriveLetter Offset  Size Type
---------------  ----------- ------  ---- ----
1  7408 15.98 MB Reserved
```

## Risultato: Partition Style è passato da RAW a GPT. Oltre all'indice c'è solo la piccola partizione riservata (MSR, tipo Reserved): lo spazio per i dati è ancora tutto non allocato.

## Suggerimento: dopo ogni modifica, verificate. È l'abitudine che distingue un sistemista attento.

**PASSO 5** **Creare la partizione: New-Partition** **Output atteso · da collaudare**

PS C:> New-Partition -DiskNumber 1 -Size 10GB -DriveLetter E

```text
PartitionNumber  DriveLetter Offset  Size Type
---------------  ----------- ------  ---- ----
2  E 16777216 10 GB Basic  **Parte del comando**
```

**-Size 10GB** usa 10 GB: circa 10 GB restano liberi per un esercizio facoltativo

**-DriveLetter E** assegna la lettera E:, verificata libera nei prerequisiti

**PartitionNumber 2** la numero 1 è una piccola partizione riservata creata da Windows su GPT

**Nota:** la partizione esiste e ha una lettera, ma non ha ancora un file system. Se ora apriste E:, Windows vi chiederebbe di formattarla.

# Quale file system scegliere

Prima di formattare, decidiamo come Windows organizzerà i dati

| File system | Punti di forza | Limiti e uso tipico |
|:--- |:--- |:--- |
| FAT32 | compatibile con quasi tutti i dispositivi | file di massimo 4 GB, nessun permesso: chiavette datate |
| exFAT | file grandi, adatto ai supporti rimovibili | nessun permesso NTFS: memorie USB e schede |
| NTFS | permessi (ACL), quote, compressione, journaling | standard per i volumi di Windows Server, anche di avvio |
| ReFS | integrità dei dati, grandi volumi (Hyper-V, backup) | non si usa per il volume di avvio |

**Scelta per il nostro volume:** NTFS, perché nel blocco C ci serviranno i permessi su cartelle e file.

**PASSO 6** **Formattare in NTFS: Format-Volume** **Output atteso · da collaudare**

```powershell
PS C:> Format-Volume -DriveLetter E -FileSystem NTFS `
  -NewFileSystemLabel "DATI" -Confirm:$false
```

```text
DriveLetter FriendlyName FileSystemType DriveType HealthStatus SizeRemaining  Size
----------- ------------ -------------- --------- ------------ -------------  ----
E           DATI         NTFS           Fixed     Healthy      9.95 GB      10 GB
```

| Parte del comando | Significato |
|---|---|
| -FileSystem NTFS | il file system scelto nella slide precedente |
| -NewFileSystemLabel "DATI" | l'etichetta, cioè il nome che si vede accanto alla lettera |
| -Confirm:$false | non chiede conferma: usatelo solo dopo aver controllato la lettera |

## Risultato: da questo momento E: può contenere cartelle e file, e accettare permessi NTFS.

**PASSO 7** **Verificare i volumi: Get-Volume** **Output atteso · da collaudare**

```powershell
PS C:> Get-Volume
```

```text
DriveLetter FriendlyName FileSystemType DriveType HealthStatus SizeRemaining      Size
----------- ------------ -------------- --------- ------------ -------------      ---
D           VMware Tools Unknown        CD-ROM    Healthy                0 B
C                        NTFS           Fixed     Healthy          60.04 GB    79.68 GB
E           DATI         NTFS           Fixed     Healthy         9.95 GB     10 GB
```

| Volume | Ruolo |
|:--- |:--- |
| C: | sistema operativo: non ospita dati aziendali |
| senza lettera | partizione EFI in FAT32: avvio UEFI, non va mai modificata |
| D: | CD-ROM virtuale degli strumenti VMware: non lo useremo |
| E: (DATI) | il nostro volume: qui costruiremo il file server |

**CONSOLE** **La stessa cosa da Gestione disco** **Console · da collaudare**

Modulo 4 · Storage e condivisioni · Windows Server 2022 18 / 48 **Si esegue su: SVR-LAB-01** **Win+R › diskmgmt.msc**

## Disco 1 · Base · 20 GB · Online

**› E: DATI · 10 GB · NTFS** **› circa 10 GB · Non allocato** Disco 0: sistema, EFI e ripristino **› non si toccano** **PowerShell** **Gestione disco**

**Initialize-Disk** clic destro sul disco › Inizializza disco (GPT predefinito) **New-Partition + ** **Format-Volume**

clic destro su «Non allocato» › Nuovo volume semplice

**Get-Disk · Get-Volume** elenco grafico dei dischi e dei volumi

## Attenzione: sul disco 1 non si rifà nulla, E: esiste già. La console qui serve a leggere il risultato dei passi 1-7.

## Suggerimento: la proverete nel laboratorio facoltativo, creando il volume F: con «Nuovo volume semplice».

# Riepilogo del blocco A Dal disco RAW al volume E: in NTFS

Modulo 4 · Storage e condivisioni · Windows Server 2022 19 / 48 **Comando** **A che cosa è servito**

**Get-Disk** vedere i dischi e trovare quello RAW **Initialize-Disk** creare la tabella GPT

**New-Partition** ritagliare 10 GB con lettera E: **Format-Volume** creare il file system NTFS **Get-Volume** verificare il risultato

**diskmgmt.msc** vedere lo stesso risultato nella console Gestione disco

**Nota:** dopo la pausa daremo un uso al volume E:, creando una cartella e rendendola raggiungibile dalla rete.

# B

# Cartella e condivisione

Rendere raggiungibile dalla rete una cartella del volume E: **26 minuti** 20 / 48

# Cartella locale, condivisione e percorso UNC

Una cartella esiste sul server; una condivisione la rende raggiungibile dalla rete

## Cartella locale

Esiste solo sul server. Si indica con un percorso locale: E:\Documenti

## Condivisione

Pubblica in rete una cartella, con un nome. Si indica con un percorso UNC: \\SVR-LAB-01\Documenti

| Parte del percorso UNC | Significato |
|:--- |:--- |
| \\ | inizio di un percorso di rete |
| SVR-LAB-01 | nome del server (risolto dal DNS) |
| Documenti | nome della condivisione, non della cartella | **** inizio di un percorso di rete

## PASSO 8 **Creare la cartella: New-Item**

PS C:> New-Item -Path "E:\Documenti" -ItemType Directory Directory: E:\

Mode                 LastWriteTime         Length Name --------------------- ---d----11/07/2026     22:55                Documenti

| Campo | Significato |
|:--- |:--- |
| Mode d----- | la d indica una directory, cioè una cartella |
| LastWriteTime | data e ora di creazione o ultima modifica |
| Name | il nome della cartella creata |

**Nota:** la cartella esiste solo localmente. Dal client \\SVR-LAB-01\Documenti non funziona ancora.

## PASSO 9 **Condividere la cartella: New-SmbShare**

```powershell
PS C:> New-SmbShare -Name "Documenti" -Path "E:\Documenti" `
  -FullAccess "LAB\Administrator"
```

```text
Name  ScopeName Path  Description
----  --------- ----  -----------
Documenti *  E:\Documenti
```

## SMB, il protocollo

SMB (Server Message Block) è il protocollo di Windows per le cartelle condivise. Usa la porta TCP 445 del server.

## I parametri

-Name: nome della condivisione -Path: cartella pubblicata -FullAccess: chi ha il controllo completo della condivisione

**PASSO 10** **Chi può accedere: Get-SmbShareAccess** **Output atteso · da collaudare**

PS C:> Get-SmbShareAccess -Name "Documenti"

```text
Name  ScopeName AccountName  AccessControlType AccessRight
----  --------- -----------  ----------------- -----------
Documenti *  LAB\Administrator Allow  Full
```

| Campo | Significato |
|:--- |:--- |
| AccountName | l'utente o il gruppo a cui si applica il permesso |
| AccessControlType | Allow (consenti) o Deny (nega) |
| AccessRight | Read (lettura), Change (modifica) o Full (completo) |

## Attenzione: nell'elenco non c'è né Mario Rossi né il suo gruppo. A livello di condivisione, per ora, Mario non ha alcun permesso.

**CONSOLE** **La condivisione da Esplora file** **Console · da collaudare**

Modulo 4 · Storage e condivisioni · Windows Server 2022 25 / 48 **Si esegue su: SVR-LAB-01** **Esplora file › E:\Documenti › Proprietà**

## scheda Condivisione

**› percorso di rete: \\SVR-LAB-01\Documenti** **› Condivisione avanzata › Autorizzazioni** LAB\Administrator: Controllo completo scheda Sicurezza **› permessi NTFS (blocco C)** **PowerShell** **Console**

**New-SmbShare** Condivisione avanzata › Condividi la cartella

**Get-/Grant-SmbShareAccess** Condivisione avanzata › Autorizzazioni

**Get-ACL · icacls** scheda Sicurezza (blocco C)

## Attenzione: in una condivisione NUOVA creata da Condivisione avanzata compare Everyone tra le autorizzazioni (livello da collaudare; le fonti

vecchie indicano Lettura). Controllate sempre e togliete ciò che non serve: con Everyone in Lettura Mario leggerebbe i file dalla rete.

## Nota: su Documenti si guarda soltanto, senza cambiare nulla.

**PASSO 11** **Tutte le condivisioni: Get-SmbShare** **Output atteso · da collaudare**

PS C:> Get-SmbShare \| Select-Object Name, Path, Description

```text
Name  Path  Description
----  ----  -----------
ADMIN$  C:\Windows  Remote Admin
C$  C:\  Default share
Documenti E:\Documenti
E$  E:\  Default share
IPC$  Remote IPC
NETLOGON  C:\Windows\SYSVOL\SYSVOL\lab.local\SCRIPTS Logon server share
SYSVOL  C:\Windows\SYSVOL\SYSVOL  Logon server share
```

## Risultato: Documenti compare accanto ad altre condivisioni che non abbiamo creato noi. Da dove vengono? Lo vediamo subito.

# Le tre categorie di condivisioni Spiegano le righe dell'output precedente

## Amministrative

# ADMIN$, C$, E$, IPC$

Create da Windows. Il simbolo $ le nasconde nella navigazione di rete. E$ è comparsa da sola con il volume E:.

## Active Directory

# NETLOGON, SYSVOL

Create dalla promozione a Domain Controller (Modulo 3). Contengono script di accesso e Criteri di gruppo.

## Create da noi Documenti

Pubblicate dall'amministratore per gli utenti. Sono le uniche su cui interveniamo in questo modulo.

**Attenzione:** le condivisioni amministrative e di Active Directory non si cancellano né si modificano.

**PASSO 12** **Prima prova dal client Windows 11** **Output atteso · da collaudare**

Modulo 4 · Storage e condivisioni · Windows Server 2022 28 / 48 **Sessione: LAB\Administrator**

```powershell
PS C:> Test-Path "\\SVR-LAB-01\Documenti"
True
```

**Sessione: LAB\m.rossi (Mario Rossi)**

```powershell
PS C:> Test-Path "\\SVR-LAB-01\Documenti"
False
PS C:> Get-ChildItem "\\SVR-LAB-01\Documenti"
Get-ChildItem: Access is denied.
```

## Che cosa dimostra True

Per l'amministratore il percorso esiste ed è raggiungibile: un buon indizio che nome, rete e condivisione funzionano.

## Perché Mario riceve False

False da solo non dice il perché. Il messaggio di Get-ChildItem (accesso negato) e il passo 10 indicano un problema di permessi.

**Domanda per il blocco C:** che cosa serve perché Mario possa lavorare in Documenti?

# C

# Identità e permessi

Perché Mario è stato respinto, e come farlo entrare nel modo corretto **55 minuti** 29 / 48

# Due livelli di sicurezza

Dalla rete servono due autorizzazioni: vince sempre la più restrittiva

**Utente** **Permessi di condivisione** **Permessi NTFS** **Cartella**

| Condivisione | NTFS | Risultato dalla rete |
|:--- |:--- |:--- |
| Full | Read | Read |
| Read | Full Control | Read |
| Change | Modify | Change / Modify |
| nessuno | Modify | accesso negato | **Full** Read Read **Read** Full Control Read **Change** Modify Change / Modify **nessuno** Modify accesso negato

**Nota:** Change (condivisione) e Modify (NTFS) hanno nomi diversi ma un effetto simile: leggere, scrivere ed eliminare, senza cambiare i permessi.

# PASSO 13 **Il secondo livello: Get-ACL**

PS C:> Get-ACL "E:\Documenti" \| Format-List

| Owner |: BUILTIN\Administrators |  |
|:--- |:--- |:--- |
| Group |: LAB\Domain Users |  |
| Access: BUILTIN\Administrators Allow | FullControl |  |
|  | BUILTIN\Administrators Allow  FullControl |  |
|  | NT AUTHORITY\SYSTEM | Allow  FullControl |
|  | CREATOR OWNER | Allow  268435456 |
|  | BUILTIN\Users | Allow  ReadAndExecute, Synchronize |
|  | BUILTIN\Users | Allow  AppendData |
|  | BUILTIN\Users | Allow  CreateFiles |

| Campo | Significato |
|:--- |:--- |
| Owner | proprietario: può sempre cambiare i permessi della cartella |
| Access | l'elenco delle voci di permesso (ACE) della cartella |

## Owner

proprietario: può sempre cambiare i permessi della cartella

## Access

l'elenco delle voci di permesso (ACE) della cartella

## Nota: Get-ACL mostra i permessi NTFS, non quelli di condivisione. Sono due elenchi separati.

# Leggere l'ACL di E:\Documenti Chi c'è nell'elenco, e chi manca

| Voce | Permesso | Che cosa vuol dire |
|:--- |:--- |:--- |
| BUILTIN\Administrators | Full Control | amministratori del server: tutto, permessi compresi |
| NT AUTHORITY\SYSTEM | Full Control | il sistema operativo stesso: servizi, backup, antivirus |
| CREATOR OWNER | segnaposto | sui nuovi file, assegna i permessi a chi li crea |
| BUILTIN\Users | lettura e creazione | leggere, entrare, creare nuovi file e sottocartelle |

**Attenzione:** in NTFS gli utenti possono creare nuovi file, ma manca un permesso per modificare ed eliminare quelli degli altri. E dalla rete Mario è comunque bloccato dalla condivisione.

**PASSO 14** **Stessa verifica con icacls** **Output atteso · da collaudare**

PS C:> icacls "E:\Documenti" E:\Documenti BUILTIN\Administrators:(F) BUILTIN\Administrators:(I)(OI)(CI)(F) NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F) CREATOR OWNER:(I)(OI)(CI)(IO)(F) BUILTIN\Users:(I)(OI)(CI)(RX) BUILTIN\Users:(I)(CI)(AD) BUILTIN\Users:(I)(CI)(WD) Successfully processed 1 files

| Sigla | Significato |
|:--- |:--- |
| F | Full Control, completo |
| M | Modify, modifica |
| RX | lettura ed esecuzione |
| I | ereditato dal padre |
| OI | vale per i file |
| CI | vale per le sottocartelle |
| IO | solo per gli oggetti figli |
| AD | crea sottocartelle (append data) |
| WD | crea file (write data) |
| Suggerimento: (I) indica un permesso ereditato. Per cambiarlo si agisce sulla cartella padre, oppure si aggiunge un permesso esplicito sulla | cartella. | **F** Full Control, completo **M** Modify, modifica **RX** lettura ed esecuzione **I** ereditato dal padre **OI** vale per i file **CI** vale per le sottocartelle **IO** solo per gli oggetti figli **AD** crea sottocartelle (append data) **WD** crea file (write data)

**Suggerimento:** (I) indica un permesso ereditato. Per cambiarlo si agisce sulla cartella padre, oppure si aggiunge un permesso esplicito sulla cartella.

# Permessi espliciti, ereditati e Deny

Come Windows decide quando le voci si contraddicono

**Deny esplicito** negazione assegnata direttamente all'oggetto

**Allow esplicito** consenso assegnato direttamente all'oggetto

**Ereditate dal padre** prima il Deny, poi l'Allow

**Ereditate più in alto** un livello alla volta (nonno, bisnonno...): in ognuno prima il Deny, poi l'Allow

**Attenzione:** non è vero che «Deny vince sempre». Un Allow esplicito prevale su un Deny ereditato.

**Suggerimento:** preferite non concedere invece di negare, ma verificate che nessun altro gruppo dell'utente conceda già quel diritto.

# A chi si assegnano i permessi: AGDLP

Nel corso i permessi degli utenti vanno ai gruppi, in una catena precisa

# A **Account** Mario Rossi

# G **Gruppo globale** IT_Admin

# DL **Gruppo locale di dominio** DL_Documenti_RW

# P **Permessi** Modify su Documenti

| Gruppo | A che cosa serve |
|:--- |:--- |
| Globale (G) | raccoglie le persone per ruolo: chi lavora nell'IT |
| Locale di dominio (DL) | rappresenta un accesso a una risorsa: modificare Documenti |

**Perché funziona:** se domani un nuovo collega entra nell'IT, basta aggiungerlo a IT_Admin. Permessi e condivisione non si toccano.

**PASSO 15** **Creare il gruppo locale di dominio** **Output atteso · da collaudare**

```powershell
PS C:> New-ADGroup -Name "DL_Documenti_RW" -GroupScope DomainLocal -GroupCategory Security
PS C:> Add-ADGroupMember -Identity "DL_Documenti_RW" -Members "IT_Admin"
PS C:> Get-ADGroupMember -Identity "DL_Documenti_RW" | Select-Object Name, objectClass
```

| Name | objectClass |
|:--- |:--- |
| ---- | ----------- |

| Parte del comando | Significato |
|---|---|
| -GroupScope DomainLocal | ambito locale di dominio: il DL di AGDLP |
| -GroupCategory Security | gruppo di sicurezza: può ricevere permessi |
| -Identity / -Members | il gruppo che contiene / chi viene aggiunto |

## Risultato: IT_Admin, e quindi Mario Rossi, ora fa parte di DL_Documenti_RW.

**PASSO 16** **Dare i permessi al gruppo, su due livelli** **Output atteso · da collaudare**

## Livello 1 · condivisione: Change

```powershell
PS C:> Grant-SmbShareAccess -Name "Documenti" `
  -AccountName "LAB\DL_Documenti_RW" -AccessRight Change -Force
```

| Name | ScopeName AccountName | AccessControlType AccessRight |
|:--- |:--- |:--- |
| Documenti * | LAB\Administrator | Allow Full |
| Documenti * | LAB\DL_Documenti_RW Allow | Change |

## Livello 2 · NTFS: Modify, ereditato da file (OI) e sottocartelle (CI)

PS C:> icacls "E:\Documenti" /grant "LAB\DL_Documenti_RW:(OI)(CI)(M)" processed file: E:\Documenti Successfully processed 1 files; Failed processing 0 files

## Perché Modify e non Full Control: chi lavora sui documenti deve creare, modificare ed eliminare, ma non cambiare i permessi o il proprietario.

**PASSO 17** **Verificare prima di provare** **Output atteso · da collaudare**

```powershell
PS C:> Get-SmbShareAccess -Name "Documenti" | Select-Object AccountName, AccessRight
```

```text
AccountName         AccessRight
--------------------
LAB\Administrator   Full
LAB\DL_Documenti_RW Change
```

```powershell
PS C:> icacls "E:\Documenti"
```

```text
E:\Documenti LAB\DL_Documenti_RW:(OI)(CI)(M) BUILTIN\Administrators:(F)...
```

## Condivisione: solo Administrator e DL_Documenti_RW. Chi non è nel gruppo non ha alcun permesso di rete.

## NTFS: compare la nuova voce Modify, non ereditata (manca la I), con (OI)(CI).

**CONSOLE** **Accesso effettivo: il risultato per Mario** **Console · da collaudare**

Modulo 4 · Storage e condivisioni · Windows Server 2022 39 / 48 **Si esegue su: SVR-LAB-01** **Esplora file › E:\Documenti › Proprietà**

scheda Sicurezza › Avanzate **› scheda Accesso effettivo** **› Seleziona un utente: LAB\m.rossi** **› Visualizza accesso effettivo** Atteso: modifica, scrittura ed eliminazione consentite Atteso: modifica delle autorizzazioni non consentita

## Provate anche con Luca

LAB\l.bianchi: in NTFS può leggere e creare file, grazie a BUILTIN\Users (slide 32). Eppure dalla rete è respinto: lo blocca la condivisione (PASSO 17).

**Nota:** qui valutiamo NTFS sul percorso locale, per l'identità selezionata. Non è la sessione di Mario sul client: dalla rete contano anche la condivisione e l'autenticazione già in uso.

**Consultazione:** non modificate i permessi da questa finestra. È il controllo più rapido quando un utente dice «non riesco a entrare».

# Perché Mario non entra ancora

Il permesso c'è, ma il client usa ancora un «biglietto» vecchio

Modulo 4 · Storage e condivisioni · Windows Server 2022 40 / 48 **Output atteso · da collaudare**

Quando Mario accede a SVR-LAB-01, il client ottiene un ticket Kerberos di servizio (un biglietto che riporta i suoi gruppi). L'appartenenza ai gruppi locali di dominio viene scritta nel ticket solo quando il ticket viene emesso. Il ticket e la connessione aperti alla prima prova (passo 12) non contengono DL_Documenti_RW.

## Soluzione consigliata · nuovo accesso

Mario si disconnette completamente dal client e accede di nuovo: nuovi ticket e nuove connessioni. Non basta bloccare e sbloccare.

## Alternativa · klist purge

Cancella i ticket in memoria, ma non chiude una connessione già aperta verso il server: da sola può non bastare.

```powershell
PS C:> klist purge
Ticket(s) purged!
```

**PASSO 18** **La prova finale dal client** **Output atteso · da collaudare**

## Sessione: LAB\m.rossi (dopo un nuovo accesso)

|  | Directory: \\SVR-LAB-01\Documenti |  |
|:--- |:--- |:--- |
| Mode | LastWriteTime | Length Name |
| -a---- |... | 0 prova.txt |

## Sessione: LAB\l.bianchi (Luca Bianchi)

```powershell
PS C:> Test-Path "\\SVR-LAB-01\Documenti"
False
PS C:> Get-ChildItem "\\SVR-LAB-01\Documenti"
Access is denied.
```

## Risultato: Mario crea ed elimina, ma non cambia i permessi. Luca, fuori dal gruppo, è respinto.

## La catena AGDLP funziona: Mario Rossi → IT_Admin → DL_Documenti_RW → Change + Modify su Documenti.

# Troubleshooting a livelli

## Se l'accesso non funziona, si controlla dal basso verso l'alto

| Nome | Resolve-DnsName SVR-LAB-01 | il nome si risolve in un indirizzo IP? |
|:--- |:--- |:--- |
| Porta | Test-NetConnection SVR-LAB-01 -Port 445 | SMB raggiungibile? (solo rete, non permessi) |
| Condivisione | Get-SmbShare -Name Documenti | la condivisione esiste? |
| Permessi di rete | Get-SmbShareAccess -Name Documenti | il gruppo dell'utente compare? |
| Permessi NTFS | icacls E:\Documenti | il gruppo ha il permesso giusto? |
| Autenticazione | nuovo accesso (sul client) | la sessione usa già i gruppi nuovi? |

# Errori comuni

Gli sbagli che abbiamo incontrato, o evitato, durante il modulo

**Inizializzare il disco sbagliato** controllare Number e Total Size, filtrare i dischi RAW

**Usare il percorso locale dal client** dal client si usa \\SVR-LAB-01\Documenti, non E:\Documenti

**Creare la condivisione e dimenticare i permessi** servono condivisione E NTFS; dalla console controllate le autorizzazioni proposte

**Dare i permessi agli utenti uno per uno** nel corso: permessi degli utenti ai gruppi locali di dominio (AGDLP)

**Dare Full Control a chi deve solo lavorare** Modify basta: Full Control permette di cambiare i permessi

**Provare subito dopo aver cambiato i gruppi** serve un nuovo accesso completo del client

# Riferimento rapido dei comandi

## Da consultare durante il laboratorio: non serve leggerlo in aula

Modulo 4 · Storage e condivisioni · Windows Server 2022 44 / 48 **Comando** **Uso** **Get-Disk** elencare i dischi

**Initialize-Disk -Number N -PartitionStyle GPT** inizializzare un disco RAW

**New-Partition -DiskNumber N -Size 10GB -DriveLetter E** creare una partizione

**Format-Volume -DriveLetter E -FileSystem NTFS** formattare il volume

**New-SmbShare / Get-SmbShare** creare / elencare le condivisioni

**Get-SmbShareAccess / Grant-SmbShareAccess** leggere / concedere permessi di condivisione

**Get-ACL / icacls** leggere / modificare permessi NTFS

**New-ADGroup / Add-ADGroupMember** creare un gruppo / aggiungere membri

**klist purge** cancellare i ticket Kerberos sul client

**diskmgmt.msc · Proprietà › Condivisione avanzata** le stesse operazioni dalle console grafiche

**Sicurezza › Avanzate › Accesso effettivo** permessi NTFS effettivi di un utente

# Laboratorio finale · 25 minuti

Una nuova condivisione in sola lettura, in autonomia, sullo stesso volume E:

**Cartella e condivisione** E:\Progetti con un file Leggimi.txt, condivisa come Progetti

**Gruppo locale di dominio** DL_Progetti_R, con IT_Admin come membro

**Permessi su due livelli** Read sulla condivisione, (OI)(CI)(RX) su NTFS

**Verifica sul server** Get-SmbShareAccess e icacls

**Prove dal client** Mario legge Leggimi.txt ma non crea file; Luca è respinto

**Attenzione:** prima delle prove Mario si disconnette e accede di nuovo.

**Facoltativo, dopo la lezione:** con Gestione disco › Nuovo volume semplice create F: (se libera) sui circa 10 GB liberi del disco 1.

# Quiz di verifica

## Rispondete in chat con il numero della domanda e la lettera

Il disco da 20 GB appena aggiunto compare RAW. Significa che: A) è danneggiato  B) non ha ancora una tabella delle partizioni  C) è già NTFS

Quale comando crea il file system? A) New-Partition  B) Initialize-Disk  C) Format-Volume

Condivisione Read e NTFS Modify, accesso dalla rete: A) Modify  B) Read  C) nessun accesso

Il percorso UNC corretto è: A) E:\Documenti  B) \\SVR-LAB-01\Documenti  C) //Documenti/SVR-LAB-01

In AGDLP i permessi si assegnano a: A) account utente  B) gruppo globale  C) gruppo locale di dominio

| Passi 15-17 verificati, ma Mario, ancora collegato dalla prima prova, è respinto. Prima cosa: A) riavviare il server | B) disconnettersi e accedere di |
|:--- |:--- |
| nuovo | C) dare Full Control a Mario |

Dopo il PASSO 7 aprite Gestione disco. Sul disco 1: A) lo inizializzate di nuovo  B) verificate E: e lo spazio non allocato, senza rifare nulla  C) formattate di nuovo E:

# Correzione del quiz Risposta corretta e perché

Modulo 4 · Storage e condivisioni · Windows Server 2022 47 / 48 **N.** **Risposta** **Perché**

| B | RAW = nessuna tabella riconosciuta; sul disco nuovo e vuoto si inizializza (passi 2-3) |
|:--- |:--- |
| C | Format-Volume crea il file system; New-Partition crea solo la partizione |
| B | dalla rete vince il più restrittivo tra condivisione e NTFS |
| B | \\server\condivisione; E:\Documenti è un percorso locale |
| C | gli utenti vanno nei gruppi globali, i permessi ai gruppi locali di dominio |
| B | la sessione usa ancora l'autenticazione precedente, senza il nuovo gruppo |
| B | la console serve a leggere il risultato dei passi 1-7: E: esiste già (slide 18) |

C Format-Volume crea il file system; New-Partition crea solo la partizione

B dalla rete vince il più restrittivo tra condivisione e NTFS

B \\server\condivisione; E:\Documenti è un percorso locale

C gli utenti vanno nei gruppi globali, i permessi ai gruppi locali di dominio

B la sessione usa ancora l'autenticazione precedente, senza il nuovo gruppo

B la console serve a leggere il risultato dei passi 1-7: E: esiste già (slide 18)

# RIEPILOGO DEL MODULO 4

**Dal disco RAW alla condivisione con ** **permessi AGDLP**

| A | disco 1 → GPT → partizione → volume E: in NTFS |
|:--- |:--- |
| B | E:\Documenti condivisa come \\SVR-LAB-01\Documenti |
| C | DL_Documenti_RW: Change + Modify, verificati dal client |

**B** E:\Documenti condivisa come \\SVR-LAB-01\Documenti

**C** DL_Documenti_RW: Change + Modify, verificati dal client

*Prossimo modulo · Amministrazione: anche le condivisioni dipendono da un servizio, «Server» (LanmanServer). Lo gestiremo nel Modulo 5.*
