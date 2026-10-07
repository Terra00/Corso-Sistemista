# MODULO 3 - ACTIVE DIRECTORY

# Dal Dominio alla Gestione di Utenti e Gruppi

Percorso guidato completo: Get-ADDomain, struttura OU, utenti, gruppi e verifica grafica con ADUC

Dominio & Foresta Organizational Unit Utenti & Gruppi PowerShell + ADUC

# Cos'è Active Directory

Il punto di partenza prima di eseguire il primo comando.

## Active Directory (AD) è il servizio di directory di Microsoft: una base dati centralizzata che conserva utenti, computer, gruppi e le

## regole di sicurezza di un'intera organizzazione. **Autenticare gli utenti** **Autorizzare gli accessi** **Gestire gli utenti** **Gestire i computer** **Gestire i gruppi** **Centralizzare l'amministrazione**

*Questa è la base su cui costruiremo tutto il modulo.*

# Architettura di Active Directory

Foresta, Dominio, Domain Controller e OU: quattro livelli da non confondere.

# FORESTA

Il livello più alto: può contenere uno o più domini che condividono schema e configurazione.

# DOMINIO

L'unità amministrativa di base: contiene utenti, gruppi, computer e criteri di sicurezza.

# DOMAIN CONTROLLER

Il server che ospita il database di Active Directory (NTDS.dit) e risponde alle richieste di autenticazione.

# OU (ORGANIZATIONAL UNIT)

Contenitori logici, creati dall'amministratore dentro il dominio, per organizzare gli oggetti e applicare le GPO.

*Nel nostro laboratorio: Foresta lab.local  →  Dominio lab.local  →  Domain Controller SVR-LAB-01  →  OU Azienda/IT*

# Gli oggetti di Active Directory

## Quattro tipi di oggetti che incontreremo per tutto il modulo.

## Utenti Le persone che accedono alla rete: hanno un login, una password e attributi anagrafici.

## Gruppi Insiemi di utenti usati per assegnare permessi in modo centralizzato (mai agli utenti singoli).

## Computer Workstation e server uniti al dominio, gestiti e autenticati come oggetti di Active Directory.

# OU Contenitori organizzativi creati

dall'amministratore: possono ricevere le Group Policy (GPO).

*Questi quattro tipi di oggetto compongono la quasi totalità di un'infrastruttura Active Directory aziendale.*

# NTDS.dit: il database dietro le quinte

PowerShell e ADUC sono due porte diverse sullo stesso database. **PowerShell** **ADUC (dsa.msc)**

## NTDS.dit il database di Active Directory

*Ogni Domain Controller conserva una copia del database NTDS.dit. PowerShell e ADUC sono due interfacce diverse - testuale e grafica - che *

*leggono e scrivono sugli stessi dati. Non esistono "oggetti creati con PowerShell" e "oggetti creati dalla GUI": sono gli stessi oggetti.*

## STEP 1 **Informazioni del dominio: Get-ADDomain**

| DNSRoot |: lab.local |
|:--- |:--- |
| NetBIOSName |: LAB |
| DomainMode |: Windows2016Domain |
| PDCEmulator |: SVR-LAB-01.lab.local |
| RIDMaster |: SVR-LAB-01.lab.local |
| InfrastructureMaster |: SVR-LAB-01.lab.local |
| UsersContainer |: CN=Users,DC=lab,DC=local |

## Dominio identificato correttamente

## Get-ADDomain mostra le informazioni principali del dominio a cui appartiene il Domain Controller: nome, NetBIOS, livello funzionale e ruoli FSMO. È

## spesso il primo comando da eseguire per verificare di essere collegati al dominio corretto.

# Analisi dell'output - Get-ADDomain

| DNSRoot | lab.local | Nome DNS del dominio Active Directory |
|:--- |:--- |:--- |
| NetBIOSName | LAB | Nome breve, usato come LAB\Administrator |
| DomainMode | Windows2016Domain | Livello funzionale massimo per Windows Server 2022 |
| PDCEmulator | SVR-LAB-01 | Ruolo FSMO: Primary Domain Controller Emulator |
| RIDMaster | SVR-LAB-01 | Assegna gli identificatori univoci (RID) ai nuovi oggetti |
| InfrastructureMaster | SVR-LAB-01 | Coerenza dei riferimenti tra domini di una foresta |
| UsersContainer | CN=Users,... | Contenitore predefinito per utenti e gruppi |

*Con Get-ADDomain ottieni l'identità e la configurazione del dominio: è il primo comando per orientarsi in un'infrastruttura AD.*

## STEP 2 **Informazioni della foresta: Get-ADForest**

| Name |: lab.local |
|:--- |:--- |
| RootDomain |: lab.local |
| Domains |: {lab.local} |
| ForestMode |: Windows2016Forest |
| GlobalCatalogs |: {SVR-LAB-01.lab.local} |
| SchemaMaster |: SVR-LAB-01.lab.local |
| DomainNamingMaster |: SVR-LAB-01.lab.local |

## Foresta configurata come previsto

## Get-ADForest mostra le informazioni del livello logico superiore al dominio: domini appartenenti alla foresta, Global Catalog, livello funzionale e i ruoli

## FSMO a livello di foresta (Schema Master e Domain Naming Master).

# Dominio vs Foresta: la gerarchia di Active Directory

# DOMINIO

Contiene utenti, gruppi, computer e criteri di sicurezza. È l'unità amministrativa di base.

# FORESTA

Il livello più alto di AD: può contenere uno o più domini che condividono schema e configurazione.

*Nel laboratorio: Foresta lab.local  →  Dominio lab.local  →  Domain Controller SVR-LAB-01  (un solo dominio, quindi dominio e foresta coincidono)*

In aziende di grandi dimensioni una foresta può contenere più domini (es. azienda.local, eu.azienda.local, usa.azienda.local).

## STEP 3

**Esplorare la struttura: quante OU esistono?**

| Name |: Domain Controllers |
|:--- |:--- |
| ObjectClass |: organizationalUnit | **Esiste una sola OU: Domain Controllers**

Il dominio è ancora "vuoto": contiene solo le strutture create automaticamente da Active Directory. Le OU sono i contenitori logici usati per organizzare

utenti e computer, delegare l'amministrazione e applicare le Group Policy (GPO).

# OU (Organizational Unit) vs Container (CN)

"Le cartelle che vedo in Active Directory sono tutte OU" - falso: esistono due tipi di contenitori.

# ORGANIZATIONAL UNIT (OU)

Creata dall'amministratore. Può contenere utenti, gruppi, computer e altre OU. ✓ **Riceve le GPO** Esempio:  IT · HR · Commerciale · Server

*Vengono create appositamente per organizzare l'azienda secondo * *reparti o funzioni.*

# CONTAINER (CN)

Creati automaticamente da Windows durante l'installazione di Active Directory. ✗ **Non riceve GPO**

Esempio:  CN=Users · CN=Computers · CN=Builtin · CN=System

*Per questo motivo, nelle aziende si creano sempre OU personalizzate al * *posto di usare i container predefiniti.*

# Cosa costruiremo: la struttura obiettivo

Una gerarchia di OU come farebbe un sistemista in un'azienda reale. **lab.local** **Azienda**

# IT

# HR Commerciale Direzione **Computer** **Server**

*Questa sarà la base per organizzare utenti, gruppi, computer e Group Policy nelle prossime lezioni.*

## STEP 4 **Creare la prima OU: Azienda**

PS C:\Users\Administrator> New-ADOrganizationalUnit -Name "Azienda" -Path "DC=lab,DC=local" (nessun output = operazione riuscita) **OU "Azienda" creata nella radice**

-Name specifica il nome della OU; -Path indica dove crearla - qui nella radice del dominio lab.local. Questa sarà la OU principale che conterrà tutte le

altre OU aziendali (IT, HR, Commerciale, Direzione).

## STEP 5

**Verificare la OU e capire perché organizzarla**

| Name | DistinguishedName |
|:--- |:--- |
| Domain Controllers | OU=Domain Controllers,DC=lab,DC=local |
| Azienda | OU=Azienda,DC=lab,DC=local |

## Perché non creare utenti direttamente nella radice del dominio? - Migliore leggibilità della struttura

- Applicazione selettiva delle Group Policy (GPO) - Delega dell'amministrazione per reparto - Gestione più semplice di utenti e computer

## STEP 6 **Creare la OU "IT" dentro Azienda**

PS C:\Users\Administrator> New-ADOrganizationalUnit -Name "IT" -Path "OU=Azienda,DC=lab,DC=local"

## Differenza tra creare nella radice e creare dentro un'altra OU **-Path "DC=lab,DC=local"**

Crea l'oggetto nella radice del dominio (usato per Azienda). **-Path "OU=Azienda,DC=lab,DC=local"**

Crea l'oggetto come figlio della OU Azienda (usato per IT). **lab.local** **└ Azienda**

# └ IT

## STEP 7 **Verificare l'intera struttura delle OU**

PS C:\Users\Administrator> Get-ADOrganizationalUnit -Filter * \| Sort-Object DistinguishedName \|

| Name | DistinguishedName |
|:--- |:--- |
| Azienda | OU=Azienda,DC=lab,DC=local |
| Domain Controllers | OU=Domain Controllers,DC=lab,DC=local |
| IT | OU=IT,OU=Azienda,DC=lab,DC=local | **Struttura confermata: Azienda > IT**

La struttura è corretta. Da questo momento smettiamo di costruire l'infrastruttura e iniziamo ad amministrarla, esattamente come farebbe un sistemista:

creando utenti e gruppi all'interno delle OU appena definite.

# Il Distinguished Name (DN)

Il "percorso completo" di un oggetto nell'albero di Active Directory.

| OU=IT, | OU=Azienda, | DC=lab,DC=local |
|:--- |:--- |:--- |
| DC | Domain Component | Ogni parte del nome DNS del dominio (lab, local) |
| OU | Organizational Unit | Contenitore creato dall'amministratore (Azienda, IT) |
| CN | Common Name | Nome di oggetti e container predefiniti (es. CN=Users, CN=Mario Rossi) |

*Si legge dal basso verso l'alto: prima l'oggetto, poi i contenitori via via più esterni, fino alla radice del dominio. È uno dei concetti più ostici per chi inizia: vale la pena * *tornarci più volte.*

## STEP 8 **Creare il primo utente: Mario Rossi** PS C:\Users\Administrator> New-ADUser \`

-Name "Mario Rossi" -GivenName "Mario" -Surname "Rossi" \` -DisplayName "Mario Rossi" -SamAccountName "m.rossi" \` -UserPrincipalName "m.rossi@lab.local" \` -Path "OU=IT,OU=Azienda,DC=lab,DC=local" \` -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) \` -Enabled $true -ChangePasswordAtLogon $true **Utente creato correttamente**

*Con un unico comando vengono introdotti: nome, nome visualizzato, SAM Account, UPN, OU di destinazione, password iniziale e abilitazione dell'account.*

# Gli attributi di un utente Active Directory

## Un utente non è solo "nome + password": è un oggetto con decine di attributi.

| Name | Mario Rossi | Nome dell'oggetto in Active Directory |
|:--- |:--- |:--- |
| GivenName / Surname | Mario / Rossi | Informazioni anagrafiche |
| DisplayName | Mario Rossi | Nome visualizzato nelle console |
| SamAccountName | m.rossi | Login legacy (LAB\m.rossi) |
| UserPrincipalName (UPN) | m.rossi@lab.local | Login moderno |
| Path (OU) | OU=IT,OU=Azienda | Posizione nell'albero di Active Directory |
| Enabled | True | Account attivo |

## lab.local → Azienda → IT → Mario Rossi

# SamAccountName vs UserPrincipalName

Due login diversi per lo stesso utente: quale usare, e quando. **SamAccountName**

# LAB\m.rossi

Login "legacy" in formato NetBIOS. Usato per accedere a

workstation, condivisioni e applicazioni più datate. **UserPrincipalName (UPN)**

# m.rossi@lab.local

Login "moderno", in formato email. Usato per l'accesso a servizi cloud, Microsoft 365 e applicazioni recenti. $$ = $$

*Sono due nomi di accesso diversi per lo stesso account: entrambi autenticano Mario Rossi in Active Directory. Una delle domande più frequenti nei corsi su * *Windows Server.*

## STEP 9 **Verificare l'utente e il ruolo dei gruppi**

PS C:\Users\Administrator> Get-ADUser "m.rossi" -Properties DisplayName,Enabled \|

| SamAccountName |: m.rossi |
|:--- |:--- |
| DisplayName |: Mario Rossi |
| UserPrincipalName: m.rossi@lab.local | Enabled: True |

**Concetto fondamentale: un utente non possiede permessi direttamente** **Utente** **Gruppo** **Permessi**

*I permessi vengono assegnati tramite i gruppi: gestione più semplice, maggiore sicurezza, amministrazione centralizzata. Questo rispetta il principio * *AGDLP che vedremo a breve.*

## STEP 10 **Creare il primo gruppo di sicurezza** PS C:\Users\Administrator> New-ADGroup \`

-Name "IT_Admin" -SamAccountName "IT_Admin" \` -GroupCategory Security -GroupScope Global \` -Path "OU=IT,OU=Azienda,DC=lab,DC=local" (nessun output = operazione riuscita) **Gruppo "IT_Admin" creato nella OU IT**

Crea un gruppo di sicurezza globale chiamato IT_Admin all'interno della OU IT. Con questo comando introduciamo la differenza tra Security e Distribution

Group, e tra Global, Domain Local e Universal Scope.

# Security vs Distribution & Group Scope

Con -GroupCategory Security abbiamo creato un gruppo di sicurezza; con -GroupScope Global il tipo più comune.

# TIPO DI GRUPPO

**Security** Assegna permessi a cartelle, stampanti, GPO

**Distribution** Solo lista email (Exchange), non assegna permessi

# GROUP SCOPE

| Global | Raggruppa utenti dello stesso dominio (usato oggi) |
|:--- |:--- |
| Domain Local | Assegna permessi alle risorse |
| Universal | Foreste con più domini |

*Nel 99% dei laboratori useremo Security Group con scope Global. Il concetto sarà approfondito con il modello AGDLP.*

# OU IT **└ IT_Admin   (Security · Global)**

## STEP 11-12

**Aggiungere e verificare il membro del gruppo**

PS C:\Users\Administrator> Add-ADGroupMember -Identity "IT_Admin" -Members "m.rossi" (nessun output = operazione riuscita)

| Name | ObjectClass |
|:--- |:--- |
| Mario Rossi | user | **Mario Rossi è membro di IT_Admin**

*Dopo ogni modifica ad Active Directory è buona pratica verificarne il risultato: è una regola che riduce gli errori e rende più affidabile il lavoro del sistemista.*

# Microsoft Best Practice: il modello AGDLP

I permessi non vengono mai assegnati direttamente agli utenti.

# A **Account** Mario Rossi

# G **Global Group** IT_Admin

# DL **Domain Local Group** Riceve i permessi

# P **Permissions** Cartelle, stampanti

*Nel nostro laboratorio siamo al primo passaggio: Mario Rossi → IT_Admin. Quando parleremo di File Server e Condivisioni, saranno i gruppi ad avere * *i permessi sulle cartelle.* **lab.local › Azienda › IT** **Mario Rossi ** ∈ **IT_Admin**

## STEP 13 **Verifica grafica con ADUC** **Percorso** Server Manager Tools Active Directory Users and Computers **oppure:  Win + R  →  dsa.msc**

# STRUTTURA ATTESA **lab.local** └ Azienda

# └ IT └ Mario Rossi └ IT_Admin └ Domain Controllers

*Obiettivo: la OU IT contiene sia utenti sia gruppi. PowerShell e la console grafica leggono/scrivono sugli stessi dati (NTDS.dit).*

# Riepilogo - Modulo 3: Active Directory

| ✓ | Active Directory | Cos'è, architettura, oggetti, NTDS.dit |
|:--- |:--- |:--- |
| ✓ | Domain Controller | Già trattato nel Laboratorio 01 |
| ✓ | Join dominio | Richiamato dal Laboratorio 01 |
| ✓ | OU vs Container | Struttura logica, Distinguished Name, OU create da PowerShell |
| ✓ | Utenti | Creazione, attributi, SamAccountName, UPN |
| ✓ | Gruppi | Security/Distribution, Scope, AGDLP, ADUC |

✓ **Domain Controller** Già trattato nel Laboratorio 01

✓ **Join dominio** Richiamato dal Laboratorio 01

✓ **OU vs Container** Struttura logica, Distinguished Name, OU create da PowerShell

✓ **Utenti** Creazione, attributi, SamAccountName, UPN

✓ **Gruppi** Security/Distribution, Scope, AGDLP, ADUC

*Modulo completo al 100% rispetto al programma previsto.*

# C E

# Materiale per il docente

Contenuti di supporto alla progettazione del corso: esercizi di estensione, struttura del percorso completo e laboratorio

finale - non fanno parte della lezione allo studente.

# Per completare: il ciclo di vita dell'utente

## Quattro esercizi finali (15-20 minuti) per chiudere il modulo in modo professionale. **Ricerca oggetti**

```powershell
Get-ADUser -Filter *
Get-ADGroup -Filter *
Get-ADComputer -Filter *
```

**Spostamento oggetto**

```powershell
Move-ADObject
```

**Disabilitare / riabilitare**

```powershell
Disable-ADAccount
Enable-ADAccount
```

**Eliminare un utente**

```powershell
Remove-ADUser
```

**Creazione** **Modifica** **Gruppi** **Disabilitazione** **Eliminazione**

*Questo rappresenta il flusso reale seguito nelle aziende per la gestione delle identità.*

# Come strutturare il percorso completo

Una progressione a tre parti per trasformarlo in un corso professionale. **PARTE 1** **Infrastruttura**

Installazione AD DS, DNS, Domain Controller, Join al dominio **~2 ore**

**PARTE 2** **Amministrazione (questo modulo)**

Dominio, Foresta, OU, Utenti, Gruppi, PowerShell, ADUC, Best Practice **~3 ore** **PARTE 3** **Group Policy**

GPO, OU, Ereditarietà, Filtri, gpupdate, gpresult, RSoP **~3 ore**

# Laboratorio finale integrato (30-40 min)

Da svolgere senza guida passo-passo, per verificare l'acquisizione delle competenze. **SCENARIO -** ** ** **Azienda Contoso**

Creare la OU Azienda con le sotto-OU IT, HR e Commerciale Creare tre utenti Creare due gruppi di sicurezza Assegnare gli utenti ai gruppi

Verificare il risultato sia da PowerShell sia da Active Directory Users and Computers

*L'esercizio consolida tutti i concetti del modulo e dimostra le competenze operative richieste a un amministratore di dominio.*
