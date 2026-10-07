# MODULO 3.b - ACTIVE DIRECTORY

**Oggetti di Active Directory: OU, utenti, ** **gruppi, delega**

## La struttura che useranno tutti i moduli successivi, più foreste e trust in teoria OU Utenti Gruppi e AGDLP Delega e trust

# Obiettivi e prerequisiti

## Alla fine saprete - Leggere dominio e foresta da PowerShell - Creare OU, utenti e gruppi - Gestire il ciclo di vita di un account - Distinguere e gestire gli account locali - Delegare un compito su una OU - Spiegare che cos'è un trust

## Prerequisiti - Modulo 3.a completato: lab.local attivo - CLI-W11-01 unito al dominio

- Accesso a SVR-LAB-01 come LAB\Administrator

**Nota:** nomi e password sono fittizi. In aula la password iniziale è il segnaposto P@ssw0rd!.

# Il percorso Sei blocchi brevi

| A | Architettura logica | 8′ |
|:--- |:--- |:--- |
| B | Unità organizzative | 13′ |
| C | Utenti di dominio e locali | 30′ |
| D | Gruppi e AGDLP | 13′ |
| E | Delega | 15′ |
| F | Foreste e trust (teoria) | 11′ | **B** **Unità organizzative** **13′** **C** **Utenti di dominio e locali** **30′** **D** **Gruppi e AGDLP** **13′** **E** **Delega** **15′** **F** **Foreste e trust (teoria)** **11′**

**Chiusura:** errori comuni, riferimento, laboratorio, quiz e ponte verso i Moduli 4 e 6 (circa 19 minuti).

# La struttura che costruiremo

## L'obiettivo concreto, prima dei comandi

| └─ Azienda | (OU) |  |
|:--- |:--- |:--- |
|  | └─ IT | (OU) |
|  | ├─ Mario Rossi | m.rossi |
|  | ├─ Luca Bianchi  l.bianchi |  |
|  | ├─ IT_Admin | gruppo: Mario |
|  | └─ Helpdesk_IT | gruppo: Luca |

## Perché proprio così

- Modulo 4: permessi su cartelle con Mario e Luca

- Modulo 6.c: GPO sulla OU IT applicata solo a IT_Admin

- Delega: Helpdesk_IT reimposta le password nella OU IT

**Nota:** Mario e Luca stanno nella stessa OU apposta: a distinguerli saranno i gruppi, non la posizione.

# A

# Architettura logica

Foresta, dominio, OU e il server che li ospita **8 minuti** 5 / 43

# Foresta, dominio, OU - e il Domain Controller

Contenitori logici da una parte, server dall'altra

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 6 / 43 **Foresta lab.local** **Dominio lab.local** **OU Azienda**

# OU IT

# SVR-LAB-01

Domain Controller: il server che ospita una copia del database del dominio

**Attenzione:** il DC non è un livello tra dominio e OU. Con un solo dominio, foresta e dominio hanno lo stesso nome ma restano oggetti diversi.

# Gli oggetti e il database Che cosa contiene il dominio

| Oggetto | Cos'Ã¨ |
| --- | --- |
| Utenti | persone che accedono: nome di accesso, password, attributi |
| Gruppi | insiemi di utenti (o di altri gruppi) usati per assegnare permessi |
| Computer | client e server membri del dominio, come CLI-W11-01 |
| OU | contenitori creati dall'amministratore per organizzare, delegare, collegare GPO |

**Nota:** tutto è conservato in NTDS.dit sul Domain Controller. PowerShell e la console ADUC (dsa.msc) sono due strade verso gli stessi oggetti.

**PASSO 1** **Il dominio: Get-ADDomain** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 8 / 43 **Si esegue su: SVR-LAB-01** PS C:> Get-ADDomain

| DNSRoot |: lab.local |
|:--- |:--- |
| NetBIOSName |: LAB |
| DomainMode |: Windows2016Domain |
| RIDMaster |: SVR-LAB-01.lab.local |
| UsersContainer |: CN=Users,DC=lab,DC=local |

| Campo | Significato |
|:--- |:--- |
| DNSRoot · NetBIOSName | i due nomi del dominio (Modulo 3.a) |
| DomainMode | livello funzionale: Windows2016Domain è il massimo con DC Windows Server 2022 |
| RIDMaster | distribuisce ai DC i blocchi di identificatori (RID) per i nuovi oggetti |
| UsersContainer | contenitore predefinito per i nuovi utenti |

**PASSO 2** **La foresta: Get-ADForest** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 9 / 43 **Si esegue su: SVR-LAB-01** PS C:> Get-ADForest

| Name |: lab.local |
|:--- |:--- |
| RootDomain |: lab.local |
| Domains |: {lab.local} |
| ForestMode |: Windows2016Forest |
| GlobalCatalogs |: {SVR-LAB-01.lab.local} |
| SchemaMaster |: SVR-LAB-01.lab.local |

| Campo | Significato |
|:--- |:--- |
| Domains | i domini della foresta: qui soltanto lab.local |
| GlobalCatalogs | DC con il catalogo globale: un indice di tutti gli oggetti della foresta |
| SchemaMaster | DC che gestisce lo schema, cioè la definizione dei tipi di oggetto |

# B

# Unità organizzative

Contenitori per organizzare, delegare e collegare criteri **13 minuti** 10 / 43

# OU e contenitori predefiniti

Non tutte le «cartelle» di Active Directory sono OU

## OU (Organizational Unit)

Create dall'amministratore. Contengono utenti, gruppi, computer e altre OU. Si possono delegare e vi si collegano le GPO.

## Contenitori (CN)

Creati da Windows: Users, Computers, Builtin. Non si possono collegare GPO a un contenitore.

**CN=Mario Rossi,** **OU=IT,** **OU=Azienda,** **DC=lab,DC=local**

| Parte | Significato |
|:--- |:--- |
| CN | Common Name: nome dell'oggetto o di un contenitore predefinito |
| OU | unità organizzativa |
| DC | Domain Component: una parte del nome DNS (lab, local) |

**Suggerimento:** si legge da sinistra a destra, dall'oggetto verso la radice del dominio.

**PASSO 3** **Creare le OU Azienda e IT** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 13 / 43 **Si esegue su: SVR-LAB-01**

| PS C:> New-ADOrganizationalUnit -Name "Azienda" -Path "DC=lab,DC=local" | PS C:> New-ADOrganizationalUnit -Name "IT" -Path "OU=Azienda,DC=lab,DC=local" |
|:--- |:--- |
| Parametro | Significato |
| -Name | nome della nuova OU |
| -Path | dove crearla: la radice del dominio oppure dentro un'altra OU | **Parametro** **Significato** **-Name** nome della nuova OU

**Nota:** nessun output significa che il comando è riuscito. Le OU create così sono protette dalla cancellazione accidentale.

**PASSO 4** **Verificare la struttura** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 14 / 43 **Si esegue su: SVR-LAB-01**

PS C:> Get-ADOrganizationalUnit -Filter * \| Select-Object Name, DistinguishedName

```text
Name  DistinguishedName
----  -----------------
Domain Controllers  OU=Domain Controllers,DC=lab,DC=local
Azienda  OU=Azienda,DC=lab,DC=local
IT  OU=IT,OU=Azienda,DC=lab,DC=local
```

## Nota: perché non creare utenti nella radice o in Users? Per organizzare per reparto, delegare per OU e collegare GPO solo dove servono.

# C

# Utenti di dominio e locali

Creazione, primo accesso, ciclo di vita, account locali **30 minuti** 15 / 43

**PASSO 5** **Creare Mario Rossi** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 16 / 43 **Si esegue su: SVR-LAB-01**

PS C:> New-ADUser -Name "Mario Rossi" -GivenName "Mario" -Surname "Rossi" \` >> -SamAccountName "m.rossi" -UserPrincipalName "m.rossi@lab.local" \` >> -Path "OU=IT,OU=Azienda,DC=lab,DC=local" \` >> -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) \` >> -Enabled $true -ChangePasswordAtLogon $true

| Parametro | Significato |
|:--- |:--- |
| -Path | OU in cui creare l'utente |
| -AccountPassword | password iniziale (P@ssw0rd!: segnaposto fittizio) |
| -ChangePasswordAtLogon $true | al primo accesso Windows chiede una nuova password | **-Path** OU in cui creare l'utente

-ChangePasswordAtLogon $true al primo accesso Windows chiede una nuova password

## Nota: nessun output = utente creato. Il primo accesso lo faremo al PASSO 8.

# Due nomi di accesso Stesso account, due formati

## LAB\m.rossi

SamAccountName in formato dominio\utente. È il formato classico per accedere ai computer del dominio.

## m.rossi@lab.local

UPN (User Principal Name), in formato simile a un indirizzo email. Anche con questo si accede al dominio.

**Nota:** l'UPN non è un indirizzo email e non serve una casella di posta per usarlo.

**PASSO 6** **Creare Luca Bianchi** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 18 / 43 **Si esegue su: SVR-LAB-01**

PS C:> New-ADUser -Name "Luca Bianchi" -GivenName "Luca" -Surname "Bianchi" \` >> -SamAccountName "l.bianchi" -UserPrincipalName "l.bianchi@lab.local" \` >> -Path "OU=IT,OU=Azienda,DC=lab,DC=local" \` >> -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) \` >> -Enabled $true -ChangePasswordAtLogon $true

## Attenzione: Luca NON va aggiunto al gruppo IT_Admin. Stessa OU di Mario, gruppi diversi: è la base delle prove dei Moduli 4 e 6.c.

**PASSO 7** **Verificare gli utenti** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 19 / 43 **Si esegue su: SVR-LAB-01**

PS C:> Get-ADUser -Filter * -SearchBase "OU=IT,OU=Azienda,DC=lab,DC=local" \| >> Select-Object Name, SamAccountName, Enabled

```text
Name  SamAccountName Enabled
----  -------------- -------
Luca Bianchi  l.bianchi  True
Mario Rossi  m.rossi True
```

## Nota: -SearchBase limita la ricerca alla OU IT. Senza -Properties, Get-ADUser restituisce solo un insieme di proprietà predefinito.

**PASSO 8** **Il primo accesso sul client** **Console · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 20 / 43 **Si esegue su: CLI-W11-01** **Schermata di accesso di CLI-W11-01**

**› Altro utente → LAB\m.rossi → password iniziale (segnaposto)** **› Windows chiede di cambiare la password: nuova password di laboratorio** **› primo accesso: Windows prepara il profilo (qualche minuto)** **› disconnettersi e ripetere con LAB\l.bianchi**

**Risultato:** Mario e Luca hanno una password personale e un profilo sul client. I Moduli 4 e 6 partono puliti.

**PASSO 9** **Il ciclo di vita di un account** **Output atteso · da collaudare**

**Si esegue su: SVR-LAB-01, prova su CLI-W11-01**

```powershell
PS C:> Disable-ADAccount -Identity "l.bianchi"
PS C:> Get-ADUser l.bianchi | Select-Object SamAccountName, Enabled
```

```text
SamAccountName Enabled
-------------- ------
l.bianchi      False
```

| Prova | Luca si disconnette da CLI-W11-01 e riprova ad accedere: accesso rifiutato |
|:--- |:--- |
| Riattivare | solo dopo la prova: Enable-ADAccount -Identity "l.bianchi" |
| Solo citati | Move-ADObject (spostare) · Remove-ADUser (eliminare): Luca serve ancora |

## Riattivare

solo dopo la prova: Enable-ADAccount -Identity "l.bianchi"

## Solo citati

Move-ADObject (spostare) · Remove-ADUser (eliminare): Luca serve ancora

# Utente di dominio o utente locale

## Due archivi di account diversi: Active Directory e il singolo computer

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 22 / 43 **Utente di dominio** **Utente locale**

**Dove è registrato** in Active Directory, sui domain controller nel computer stesso (è lui l'autorità)

**Dove vale** riconosciuto dai computer del dominio; l'accesso dipende dai diritti solo su quel computer

**Con che cosa si gestisce** ADUC (dsa.msc) · modulo ActiveDirectory Gestione computer · modulo LocalAccounts

**Nome di accesso** LAB\m.rossi CLI-W11-01\tecnico

**Attenzione:** su un domain controller gli account non si gestiscono con Utenti e gruppi locali. Per questo la pratica si fa su CLI-W11-01, non su SVR-LAB01.

**Si esegue su: CLI-W11-01 (amministratore)** **Windows PowerShell a 64 bit, aperta come amministratore**

PS C:> Get-LocalUser \| Select-Object Name, Enabled

```text
Name  Enabled
----  -------
Administrator  False
DefaultAccount  False
Guest  False
```

```text
Name  PrincipalSource
----  ---------------
CLI-W11-01\Administrator  Local
CLI-W11-01<account dell'installazione> Local
LAB\Domain Admins  ActiveDirectory
```

**Nota:** Administrators (nome della documentazione: in italiano è tradotto, per questo si usa il SID) contiene Domain Admins, aggiunto dal join al dominio: per questo LAB\Administrator amministra il client.

**Attenzione:** è un gruppo locale del computer, non un gruppo Domain Local di Active Directory (AGDLP, blocco D).

**PASSO 11** **Creare un utente locale** **Output atteso · da collaudare**

**Si esegue su: CLI-W11-01 (amministratore)** **Windows PowerShell a 64 bit, aperta come amministratore**

PS C:> New-LocalUser -Name "tecnico" -FullName "Tecnico locale" \` >> -Description "Account locale di prova (fittizio)" \` >> -Password (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)

```text
Name  Enabled Description
----  ------- -----------
tecnico True  Account locale di prova (fittizio)
```

| Parametro | Significato |
|:--- |:--- |
| -Name | fino a 20 caratteri; qui un nome diverso dagli utenti di dominio, per evitare ambiguità |
| -Password | password iniziale (P@ssw0rd!: segnaposto fittizio) |
| nessun -ChangePasswordAtLogon | New-LocalUser non ha un parametro equivalente a quello di New-ADUser |

**Nota:** tecnico può accedere al client perché il gruppo locale Users contiene Authenticated Users (da collaudare).

**PASSO 12** **Verificare e chiudere il ciclo** **Console · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 25 / 43 **Dimostrazione del formatore · CLI-W11-01** **Gestione computer (compmgmt.msc)**

## › Utenti e gruppi locali

**› Utenti** tecnico compare nell'elenco, accanto agli account predefiniti

**Console della VM (non RDP): accesso come CLI-W11-01\tecnico**

```text
PS C:> whoami
cli-w11-01\tecnico
```

**Uscita da tecnico, nuovo accesso come LAB\Administrator: l'account non serve ai moduli successivi**

```powershell
PS C:> Disable-LocalUser -Name "tecnico"
PS C:> Get-LocalUser -Name "tecnico" | Select-Object Name, Enabled
```

```text
Name  Enabled
----  -------
tecnico  False
```

# D

# Gruppi e AGDLP

Gli utenti si raggruppano, i permessi si danno ai gruppi **13 minuti** 26 / 43

# Tipi e ambiti dei gruppi

## Due scelte: a che cosa serve il gruppo e fin dove arriva

**Ambito** **Membri ammessi** **Dove riceve permessi**

| Global | account e altri gruppi Global dello stesso dominio | qualsiasi dominio della foresta e domini o foreste che si | fidano |
|:--- |:--- |:--- |:--- |
| Domain Local | account e gruppi Global di domini attendibili; Universal della foresta; Domain | Local dello stesso dominio | solo nel proprio dominio |
| Universal | account, gruppi Global e Universal dei domini della foresta (non i Domain | Local) | qualsiasi dominio della foresta e foreste che si fidano |

## Nota: Security = gruppo di sicurezza (riceve permessi). Distribution = lista di distribuzione per la posta (non riceve permessi).

**PASSO 13** **Il gruppo IT_Admin e Mario** **Output atteso · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 28 / 43 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> New-ADGroup -Name "IT_Admin" -GroupCategory Security -GroupScope Global `
  -Path "OU=IT,OU=Azienda,DC=lab,DC=local"
PS C:> Add-ADGroupMember -Identity "IT_Admin" -Members "m.rossi"
PS C:> Get-ADGroupMember "IT_Admin" | Select-Object Name, ObjectClass
```

```text
Name  ObjectClass
----  -----------
Mario Rossi  user
```

## Nota: -Identity è il gruppo, -Members chi entra. IT_Admin è un nome scelto da noi: non dà nessun diritto amministrativo da solo.

# AGDLP: il modello del corso Account → Global → Domain Local → Permissions

# A **Account** Mario Rossi

# G **Global** IT_Admin

# DL **Domain Local** riceve i permessi

## Perché

Non tutti devono essere Domain Admins. Con la delega un gruppo può fare solo ciò che gli serve, solo nella OU indicata.

## Compiti comuni della procedura guidata - Creare e gestire account utente - Reimpostare le password e forzare il cambio - Leggere le informazioni degli utenti - Modificare l'appartenenza a un gruppo - Unire computer al dominio

**Nota:** la delega vale per la OU scelta e per gli oggetti al suo interno. Qui la diamo direttamente a un gruppo globale (semplificazione): AGDLP completo lo usiamo per i permessi sulle cartelle nel Modulo 4.

**PASSO 14** **Delegare la reimpostazione delle password** **Console · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 32 / 43 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> New-ADGroup -Name "Helpdesk_IT" -GroupCategory Security -GroupScope Global `
  -Path "OU=IT,OU=Azienda,DC=lab,DC=local"
PS C:> Add-ADGroupMember -Identity "Helpdesk_IT" -Members "l.bianchi"
```

**Utenti e computer di Active Directory (dsa.msc)**

## › clic destro sulla OU IT → Delega controllo

**› Utenti o gruppi: Aggiungi → Helpdesk_IT** **› Attività da delegare: «Reimposta le password utente e forza il cambio della password all'accesso successivo»** **› Fine**

**PASSO 15** **Verificare la delega: Accesso effettivo** **Console · da collaudare**

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 33 / 43 **Si esegue su: SVR-LAB-01**

**Utenti e computer di Active Directory (dsa.msc)**

**› Visualizza → Funzionalità avanzate** **› OU IT → Mario Rossi → Proprietà → Sicurezza → Avanzate** **› scheda Accesso effettivo → Seleziona un utente: Luca Bianchi** **› Visualizza accesso effettivo → «Reimposta password» consentito**

**Risultato:** Luca ha il diritto «Reimposta password» sull'account di Mario, nella OU IT. Non è diventato amministratore.

**Nota:** il controllo su un utente fuori dalla OU IT lo faremo nel laboratorio finale, con Anna Verdi (OU HR).

# F

# Foreste e trust

Solo teoria: quando più domini devono fidarsi tra loro **11 minuti** 34 / 43

# Che cos'è un trust Una relazione di fiducia tra due domini

## Dominio A si fida (trusting): ha le risorse **la fiducia va da A verso B** **Dominio B** è fidato (trusted): ha gli utenti **l'accesso va da B verso A**

**Nota:** A accetta l'autenticazione fatta da B (chi sei). Che cosa possono fare gli utenti di B lo decidono i permessi in A (autorizzazione).

**Attenzione:** un trust non dà permessi da solo. Rende soltanto possibile assegnarli.

# Tipi di trust e transitività

## Transitivo = la fiducia si estende ad altri domini collegati, ma solo entro i limiti del tipo di trust

| Tipo | Creazione | Direzione | Transitività |
|:--- |:--- |:--- |:--- |
| Padre-figlio | automatica | bidirezionale | transitivo |
| Radice di albero (nuovo albero nella foresta) | automatica | bidirezionale | transitivo |
| Foresta | manuale | uni o bidirezionale | tra le due foreste, non verso una terza |
| Esterno (dominio di un'altra foresta) | manuale | uni o bidirezionale | non transitivo |
| Collegamento (shortcut) | manuale | uni o bidirezionale | fonti Microsoft in contrasto |
| Area (realm: Kerberos non Windows) | manuale | uni o bidirezionale | transitivo o non transitivo |

## Attenzione: sulla transitività dei trust di collegamento le fonti Microsoft non concordano. Non presentatela come un fatto certo.

# Che cosa servirebbe, che cosa vediamo

## Perché non lo facciamo in laboratorio

## Per un trust tra foreste - Una seconda foresta (un altro server)

- Risoluzione DNS reciproca (inoltro condizionale)
- Porte aperte tra i DC (es. 53, 88, 445)
- Console Domini e trust di Active Directory (netdom trust non crea trust tra foreste)

**Nel nostro laboratorio**

```powershell
PS C:> Get-ADTrust -Filter *
PS C:>
```

**Nota:** nessun risultato: lab.local è l'unico dominio della sua foresta e non ha trust con altri domini.

# Errori comuni

Quelli corretti rispetto al deck precedente, e altri frequenti

**OU e contenitore come la stessa cosa** alle OU si collegano GPO e si delega

**DN letto al contrario** dall'oggetto verso la radice

**Utenti creati in Users o nella radice** usate -Path con la OU giusta

**Luca aggiunto a IT_Admin** salterebbero le prove dei Moduli 4 e 6.c

**Delega data a una persona** delegate a un gruppo

**Direzione del trust confusa con l'accesso** fiducia A→B, accesso B→A

**Cercare gli utenti locali sul DC** lì non si gestiscono: si lavora sul client

**Nome del gruppo locale in inglese** su Windows in italiano è tradotto: usate il SID

# Riferimento rapido

## Da consultare durante il laboratorio

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 39 / 43 **Comando** **Uso**

**Get-ADDomain · Get-ADForest** dominio e foresta

**New-ADOrganizationalUnit -Name -Path** creare una OU

**Get-ADOrganizationalUnit -Filter *** elencare le OU

**New-ADUser... -Path -AccountPassword -Enabled** creare un utente

**Get-ADUser -Filter * -SearchBase** cercare utenti in una OU

**Disable-ADAccount · Enable-ADAccount** disattivare · riattivare

**New-ADGroup -GroupScope Global** creare un gruppo

**Add-ADGroupMember · Get-ADGroupMember** appartenenza ai gruppi

**dsa.msc › Delega controllo** delega su una OU **Get-ADTrust -Filter *** trust del dominio

**Get-LocalUser · New-LocalUser · Disable-LocalUser** account locali (sul client)

**Get-LocalGroupMember -SID S-1-5-32-544** membri di Administrators (sul client)

# Laboratorio finale · 10 minuti

| Passaggio | Cosa fare |
## Da soli: un nuovo reparto, poi il limite della delega

| Passaggio | Cosa fare |
| --- | --- |
| OU | creare la OU HR dentro Azienda |
| Utente | creare Anna Verdi (a.verdi, nome fittizio) nella OU HR |
| Gruppo | creare il gruppo globale HR_Staff nella OU HR e aggiungere Anna |
| Verifica | Get-ADGroupMember HR_Staff e la console ADUC |
| Delega | Accesso effettivo di Luca sull'account di Anna: Â«Reimposta passwordÂ» non consentito |
| Ciclo di vita | disattivare l'account di Anna e verificarlo |

**Delega** Accesso effettivo di Luca sull'account di Anna: «Reimposta password» non consentito

**Ciclo di vita** disattivare l'account di Anna e verificarlo

**Nota:** soluzione nelle note del relatore. Il laboratorio si può ripetere: usate nomi nuovi se HR esiste già.

# Quiz di verifica

## Rispondete in chat con il numero della domanda e la lettera

Differenza tra una OU e il contenitore Users? A) nessuna  B) a una OU si puÃ² collegare una GPO, al contenitore no  C) il contenitore ha solo computer

Come si legge CN=Mario Rossi,OU=IT,OU=Azienda,DC=lab,DC=local? A) dall'oggetto verso la radice  B) dalla radice verso l'oggetto C) in qualsiasi ordine

Differenza tra una OU e il contenitore Users? A) nessuna  B) a una OU si può collegare una GPO, al contenitore no  C) il contenitore ha solo computer

Come si legge CN=Mario Rossi,OU=IT,OU=Azienda,DC=lab,DC=local? A) dall'oggetto verso la radice  B) dalla radice verso l'oggetto C) in qualsiasi ordine

m.rossi@lab.local è: A) il SamAccountName  B) un indirizzo email obbligatorio  C) l'UPN

Un gruppo Global contiene di norma: A) account del proprio dominio  B) permessi sulle cartelle  C) computer di altre foreste

La delega su OU IT a Helpdesk_IT permette a Luca di: A) diventare Domain Admin  B) reimpostare le password degli utenti della OU IT  C) creare GPO

Il dominio A si fida del dominio B. Chi può accedere alle risorse di chi? A) utenti di B alle risorse di A, se autorizzati  B) utenti di A alle risorse di B - nessuno

| L'identità locale CLI-W11-01\tecnico può essere usata per autenticarsi su SVR-LAB-01? A) sì, il client è nel dominio | B) no, vale solo su CLI-W11- |
| ---: |:--- |
| 01 | C) sì, se è in Administrators |

# Correzione del quiz

## Risposta corretta e perché

Modulo 3.b · Oggetti di Active Directory · Windows Server 2022 42 / 43 **N.** **Risposta** **Perché**

| B | le GPO si collegano a OU, dominio e siti; non ai contenitori |
|:--- |:--- |
| A | il DN parte dall'oggetto e arriva alla radice del dominio |
| C | è lo User Principal Name; non serve una casella di posta |
| A | i gruppi globali raccolgono account del proprio dominio |
| B | la delega vale solo per il compito e la OU scelti |
| A | fiducia da A a B; gli utenti di B accedono alle risorse di A, se autorizzati |
| B | un account locale vale solo sul computer in cui è stato creato |

A il DN parte dall'oggetto e arriva alla radice del dominio

C è lo User Principal Name; non serve una casella di posta

A i gruppi globali raccolgono account del proprio dominio

B la delega vale solo per il compito e la OU scelti

A fiducia da A a B; gli utenti di B accedono alle risorse di A, se autorizzati

B un account locale vale solo sul computer in cui è stato creato

# Ponte verso i moduli successivi

La struttura di oggi lavora per tutto il corso

## Modulo 4 · Storage

Gruppi Domain Local e permessi NTFS sulle cartelle condivise: Mario (IT_Admin) e Luca avranno accessi diversi.

## Modulo 6.c · GPO

GPO collegata alla OU IT e applicata solo a IT_Admin: Mario la riceve, Luca no.

**Nota:** non cancellate né spostate Mario, Luca, IT_Admin e Helpdesk_IT.
