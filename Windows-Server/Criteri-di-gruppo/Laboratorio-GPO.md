# R Y · 2 2

# Group Policy Objects

## Laboratorio pratico guidato: dalla creazione alla rimozione di una GPO

**18 laboratori pratici** **PowerShell + CMD** **Teoria integrata**

Preparato per: simplelogin-newsletter.0gpr6@simplelogin.com 13 luglio 2026

# I O **Cosa vedremo**

## Fondamenti

Cos'è una GPO, dove viene memorizzata, ordine di elaborazione (LSDOU)

## Ciclo di vita

Creazione, collegamento a una OU, verifica dell'ereditarietà

## Targeting

Security Filtering: applicare una GPO a gruppi specifici

## Continuità Backup, elenco backup e ripristino di una GPO

## Applicazione

gpupdate, gpresult e Resultant Set of Policy (RSoP)

## Manutenzione

Disabilitazione, rimozione del collegamento ed eliminazione Laboratorio GPO · Windows Server 2022

# I A **Cos'è una Group Policy Object**

## Una GPO è un insieme di impostazioni di configurazione applicate

centralmente a utenti e computer in un dominio Active Directory: sicurezza, ambiente desktop, software, script di accesso, restrizioni.

## Ogni GPO è composta da due parti fisiche

**GPC** **Group Policy Container** Oggetto in Active Directory (attributi, versione, riferimenti)

**GPT** **Group Policy Template** Cartella su SYSVOL con i file reali delle impostazioni

## Il modulo PowerShell GroupPolicy

Incluso nelle RSAT (Remote Server Administration Tools) o direttamente sul Domain Controller. Espone i cmdlet usati in questo laboratorio. ● Get-GPO ● New-GPO ● New-GPLink / Remove-GPLink ● Get-GPInheritance ● Set-GPPermissions / Get-GPPermission ● Backup-GPO / Restore-GPO / Get-GPOBackup ● Set-GPO ● Remove-GPO Laboratorio GPO · Windows Server 2022

# I A **Ordine di elaborazione: L-S-D-OU**

Le GPO vengono applicate in sequenza; l'ultima elaborata (la più vicina all'oggetto) vince in caso di impostazioni in conflitto.

# L **Local** Criteri di sicurezza locali del singolo computer $$ \to $$

# S **Site** GPO collegate al sito di Active Directory $$ \to $$

# D **Domain** GPO collegate all'intero dominio (es. Default Domain Policy) $$ \to $$

# OU **Organizational Unit** GPO collegate alla OU, dalla più esterna alla più interna **Eccezioni da conoscere**

**Block Inheritance ** - una OU può bloccare le GPO ereditate dai livelli superiori. **Enforced (No Override) ** - una GPO impostata come "enforced" si applica sempre, anche se la OU blocca l'ereditarietà. Laboratorio GPO · Windows Server 2022

# 0 1 · I O

# Visualizzare tutte le GPO Windows PowerShell **PS C:>** Get-GPO -All

# OUTPUT SIMULATO

DisplayName: Default Domain Policy Owner: LAB\Domain Admins

DisplayName: Default Domain Controllers Policy Owner: LAB\Domain Admins

DisplayName: GPO_IT_Admin Owner: LAB\Domain Admins

✓ Elencare tutte le GPO presenti nel dominio, indipendentemente dal collegamento a una OU. ✓

Verificare il proprietario (Owner) di ciascuna GPO.

# DA SAPER E

Get-GPO -All legge direttamente dal container delle Group Policy in Active Directory: mostra anche le GPO create ma non ancora collegate a nulla. Laboratorio GPO · Windows Server 2022

# 0 2 · I O

# Visualizzare una GPO specifica Windows PowerShell **PS C:>** Get-GPO -Name "GPO_IT_Admin"

# OUTPUT SIMULATO

| DomainName |: lab.local |
|:--- |:--- |
| Status |: AllSettingsEnabled |

✓ Recuperare i dettagli di una singola GPO tramite il suo nome.

✓ Verificare lo stato (Status) delle impostazioni utente e computer.

# DA SAPER E

Status può essere AllSettingsEnabled, UserSettingsDisabled, ComputerSettingsDisabled o AllSettingsDisabled: utile per capire subito se una parte della GPO è disattivata. Laboratorio GPO · Windows Server 2022

# 0 3 · T A

# Creare una nuova GPO Windows PowerShell

**PS C:>** New-GPO -Name "Blocco_PannelloControllo"

# OUTPUT SIMULATO

DisplayName: Blocco_PannelloControllo Id: 2c91b2...

✓ Creare una nuova Group Policy Object vuota, pronta per essere configurata.

✓ La GPO esiste in AD ma, finché non viene collegata, non produce alcun effetto.

# DA SAPER E

New-GPO crea sia il Group Policy Container (in AD) sia il Group Policy Template (su SYSVOL) - ma non la collega automaticamente a nessuna OU. Laboratorio GPO · Windows Server 2022

# 0 4 · T A

# Collegare la GPO a una OU Windows PowerShell

**PS C:>** New-GPLink \` -Name "Blocco_PannelloControllo" \` -Target "OU=IT,OU=Azienda,DC=lab,DC=local"

# OUTPUT SIMULATO

| Enabled |: True |
|:--- |:--- |
| Target |: OU=IT,OU=Azienda, | DC=lab,DC=local ✓

Collegare (link) una GPO a una Organizational Unit.

✓ Da questo momento le impostazioni iniziano ad applicarsi agli oggetti contenuti nella OU.

# DA SAPER E

Una stessa GPO può essere collegata a più OU, siti o al dominio: il collegamento è un puntatore, la GPO resta un oggetto unico. Laboratorio GPO · Windows Server 2022

# 0 5 · C A

# Verificare il collegamento (Inheritance) Windows PowerShell

**PS C:>** Get-GPInheritance "OU=IT,OU=Azienda, DC=lab,DC=local"

# OUTPUT SIMULATO Inherited GPOs

| Default Domain Policy | GPO_IT_Admin | Blocco_PannelloControllo |
|:--- |:--- |:--- |
| ✓ | Verificare quali GPO vengono effettivamente | applicate a una OU. |
| ✓ | Controllare l'ordine di ereditarietà (le GPO in | fondo alla lista vincono in caso di conflitto). |

✓ Verificare quali GPO vengono effettivamente applicate a una OU.

✓ Controllare l'ordine di ereditarietà (le GPO in fondo alla lista vincono in caso di conflitto).

# DA SAPER E

Get-GPInheritance mostra anche eventuali blocchi di ereditarietà (GpoInheritanceBlocked) impostati sulla OU. Laboratorio GPO · Windows Server 2022

## I A **Security Filtering: cos'è e a cosa serve**

Per impostazione predefinita ogni GPO si applica a tutti gli "Authenticated Users" della OU collegata. Il Security Filtering permette di restringere l'applicazione a specifici utenti, computer o gruppi, senza dover creare nuove OU.

## ① **Il permesso chiave: Apply Group Policy**

Una GPO si applica solo agli oggetti che hanno contemporaneamente i permessi di lettura (Read) e applicazione (Apply Group Policy) impostati su Allow.

## ② **Come si restringe l'ambito**

Si rimuove il permesso Apply Group Policy da Authenticated Users e lo si assegna a un gruppo di sicurezza specifico (es. IT_Admin), tramite Set-GPPermissions.

## ③ **Perché è utile**

Consente un targeting granulare (es. "solo il team IT") mantenendo un'unica struttura di OU, evitando di moltiplicare le Organizational Unit solo per differenziare l'applicazione delle policy. Laboratorio GPO · Windows Server 2022

# 0 6 · N G

# Applicare Security Filtering Windows PowerShell

**PS C:>** Set-GPPermissions \` -Name "Blocco_PannelloControllo" \` -TargetName "IT_Admin" \` -TargetType Group \` -PermissionLevel GpoApply

# OUTPUT SIMULATO Permissions updated successfully.

✓ Applicare una GPO soltanto a un gruppo di sicurezza specifico.

✓ Restringere l'ambito senza toccare la struttura delle OU.

# DA SAPER E

GpoApply assegna Allow su Read + Apply Group Policy: il gruppo IT_Admin riceverà la GPO, gli altri utenti della OU no. Laboratorio GPO · Windows Server 2022

# 0 7 · N G

# Verificare i permessi Windows PowerShell

**PS C:>** Get-GPPermission \` -Name "Blocco_PannelloControllo" \` -All

# OUTPUT SIMULATO Domain Admins Authenticated Users IT_Admin

✓ Controllare l'elenco completo dei soggetti (Trustee) e dei relativi livelli di permesso sulla GPO.

✓ Confermare che il Security Filtering sia stato applicato correttamente.

# DA SAPER E

Domain Admins mantiene sempre i permessi di gestione (Edit/Delete): non compare per errore, serve per l'amministrazione della GPO stessa. Laboratorio GPO · Windows Server 2022

## I A **Backup e Restore delle GPO**

Le GPO possono modificare in modo profondo sicurezza e comportamento di client e server: un backup regolare è la rete di sicurezza prima di ogni modifica importante.

## ① **Backup-GPO**

Salva su file system sia il Group Policy Container sia il Group Policy Template della GPO, in una cartella di destinazione (es. C:\BackupGPO). Ogni backup è marcato con data e ora.

## ② **Get-GPOBackup**

Elenca i backup disponibili in una cartella, con nome GPO e timestamp: utile per scegliere quale versione ripristinare prima di un rollback.

## ③ **Restore-GPO**

Ripristina la GPO allo stato salvato nel backup, riportando indietro tutte le impostazioni in caso di errore, test falliti o modifiche indesiderate. Laboratorio GPO · Windows Server 2022

## 08-09

# 0 8 - 0 9 · I T À

# Backup della GPO e verifica dei backup Windows PowerShell

**PS C:>** Backup-GPO \` -Name "Blocco_PannelloControllo" \` -Path "C:\BackupGPO"

# OUTPUT · LAB 08 Backup completed successfully. Windows PowerShell

**PS C:>** Get-GPOBackup \` -Path "C:\BackupGPO"

# OUTPUT · LAB 09

DisplayName: Blocco_PannelloControllo BackupTime: 13/07/2026 ✓

Eseguire il backup di una GPO su file system, prima di apportare modifiche rischiose. ✓

Elencare i backup disponibili in una cartella, verificandone data e nome. Laboratorio GPO · Windows Server 2022

# 1 0 · I T À

# Ripristinare una GPO Windows PowerShell

**PS C:>** Restore-GPO \` -Name "Blocco_PannelloControllo" \` -Path "C:\BackupGPO"

# OUTPUT SIMULATO Restore completed successfully.

✓ Ripristinare una GPO allo stato salvato nell'ultimo backup disponibile.

✓ Recuperare rapidamente da una configurazione errata o da un test fallito.

# DA SAPER E

Restore-GPO ripristina anche i collegamenti se la GPO esiste ancora; se era stata eliminata, ne ricrea uno nuovo con lo stesso contenuto. Laboratorio GPO · Windows Server 2022

## I A **Come e quando le GPO vengono applicate**

Le impostazioni di una GPO non sono immediate per tutti i client: Windows le riapplica a intervalli regolari in background.

## ① **Refresh in background**

Per default i computer membri aggiornano le policy ogni 90 minuti (con uno scarto casuale di 0-30 minuti per evitare picchi di carico sui DC); i Domain Controller ogni 5 minuti.

## ② **gpupdate /force**

Forza l'aggiornamento immediato e riapplica tutte le impostazioni, anche quelle non modificate - utile in laboratorio e in troubleshooting per non attendere il ciclo naturale.

## ③ **Riavvio per alcune impostazioni**

Alcune policy (es. software installato via GPO, alcune impostazioni di sicurezza) richiedono un riavvio o un nuovo logon per essere applicate completamente. Laboratorio GPO · Windows Server 2022

# 1 1 · N E

# Forzare l'applicazione della GPO Prompt dei comandi **C:>** gpupdate /force

# OUTPUT SIMULATO Updating policy...

Computer Policy update completed successfully. User Policy update completed successfully.

✓ Forzare l'aggiornamento immediato delle policy su un client, senza attendere il refresh automatico.

✓ Verificare rapidamente l'effetto di una GPO appena modificata o collegata.

# DA SAPER E

gpupdate (senza /force) applica solo le impostazioni cambiate dall'ultimo aggiornamento; /force le riapplica tutte da zero. Laboratorio GPO · Windows Server 2022

## 12-13

# 1 2 - 1 3 · C A

# Verificare le GPO applicate e generare un report Prompt dei comandi **C:>** gpresult /r

# OUTPUT · LAB 12 Applied Group Policy Objects

Default Domain Policy GPO_IT_Admin Blocco_PannelloControllo Prompt dei comandi **C:>** gpresult /h C:\ReportGPO.html

# OUTPUT · LAB 13

| The gpresult report has been | written to | C:\ReportGPO.html |
|:--- |:--- |:--- |
| ✓ | gpresult /r mostra a riga di comando le GPO effettivamente applicate a utente e | computer. |
| ✓ | gpresult /h genera un report HTML dettagliato e navigabile, utile per | documentazione e troubleshooting. |

✓ gpresult /r mostra a riga di comando le GPO effettivamente applicate a utente e computer.

✓ gpresult /h genera un report HTML dettagliato e navigabile, utile per documentazione e troubleshooting. Laboratorio GPO · Windows Server 2022

# 1 4 · C A

# Resultant Set of Policy (RSoP)

## Cos'è

Il RSoP è la combinazione netta e finale di tutte le GPO applicate a un utente e a un computer, dopo aver risolto ereditarietà, blocchi, enforcement e conflitti tra impostazioni.

## A cosa serve

● Analizzare la configurazione realmente applicata a un oggetto, oltre la teoria dei collegamenti.

● Individuare quale GPO ha vinto in caso di impostazioni in conflitto tra più policy. Esegui **C:>** RSoP.msc

# RISULTA T O SIMULAT O

Si apre la console Microsoft Management Console "Resultant Set of Policy", che mostra ad albero tutte le impostazioni effettivamente applicate - sia lato Computer che lato Utente - con l'indicazione della GPO sorgente per ciascuna impostazione. Laboratorio GPO · Windows Server 2022

## 15-16

# 1 5 - 1 6 · N E

# Disabilitare e riabilitare una GPO Windows PowerShell

**PS C:>** Set-GPO \` -Name "Blocco_PannelloControllo" \` -Status UserSettingsDisabled

# OUTPUT · LAB 15 Group Policy updated successfully. Windows PowerShell

**PS C:>** Set-GPO \` -Name "Blocco_PannelloControllo" \` -Status AllSettingsEnabled

# OUTPUT · LAB 16 Group Policy updated successfully.

✓ Disabilitare temporaneamente una parte (o tutta) la GPO, senza eliminarla né rimuovere il collegamento.

✓ Riabilitarla non appena il test o la manutenzione sono conclusi, ripristinando l'applicazione completa. Laboratorio GPO · Windows Server 2022

## 17-18

# 1 7 - 1 8 · N E

# Rimuovere il collegamento ed eliminare la GPO Windows PowerShell

**PS C:>** Remove-GPLink \` -Name "Blocco_PannelloControllo" \` -Target "OU=IT,OU=Azienda,DC=lab,DC=local"

# OUTPUT · LAB 17 Group Policy link removed. Windows PowerShell

**PS C:>** Remove-GPO \` -Name "Blocco_PannelloControllo"

# OUTPUT · LAB 18 Group Policy removed successfully.

✓ Remove-GPLink scollega la GPO dalla OU: la GPO resta nel dominio ma non produce più effetto lì.

✓ Remove-GPO elimina definitivamente la GPO (GPC + GPT): operazione irreversibile senza un backup precedente. Laboratorio GPO · Windows Server 2022

# G O **Competenze acquisite**

Al termine dei 18 laboratori hai coperto le attività operative fondamentali sulla gestione delle GPO in Active Directory.

| ✓ | Creazione di una GPO | ✓ | Collegamento a dominio e OU | ✓ | Verifica dell'ereditarietà (Inheritance) |
|:--- |:--- |:--- |:--- |:--- |:--- |
| ✓ | Security Filtering | ✓ | Gestione delle autorizzazioni | ✓ | Backup e ripristino |
| ✓ | Abilitazione e disabilitazione | ✓ | Aggiornamento delle policy (gpupdate) | ✓ | Verifica dell'applicazione (gpresult) |
| ✓ | Analisi del RSoP (RSoP.msc) | ✓ | Rimozione del collegamento | ✓ | Eliminazione di una GPO |

✓ **Security Filtering** ✓ **Gestione delle autorizzazioni** ✓ **Backup e ripristino**

✓ **Abilitazione e disabilitazione** ✓ **Aggiornamento delle policy (gpupdate)** ✓ **Verifica dell'applicazione (gpresult)**

✓ **Analisi del RSoP (RSoP.msc)** ✓ **Rimozione del collegamento** ✓ **Eliminazione di una GPO** Laboratorio GPO · Windows Server 2022

## I A **Da approfondire: oltre il singolo dominio**

Due argomenti restano fuori dalla portata di questo laboratorio a dominio singolo, perché richiedono almeno due domini o foreste per essere sperimentati concretamente. Meritano comunque un inquadramento teorico.

## ① **Trust relationship (relazioni di trust)**

Una relazione di fiducia che permette a utenti di un dominio di autenticarsi e accedere a risorse in un altro dominio o foresta. Può essere unidirezionale o bidirezionale, transitiva o non transitiva: le GPO non attraversano un trust, ma la delega e l'accesso alle risorse sì.

## ② **Foreste (Forest)**

La foresta è il confine di sicurezza più ampio in Active Directory: raggruppa uno o più domini che condividono schema, catalogo globale e configurazione. Progettare GPO in ambienti multi-forest richiede pianificare trust, replica e, spesso, strumenti come MIM o soluzioni di sincronizzazione.

## ③

**Delega amministrativa (Delegation of Control)**

Consente di assegnare permessi granulari su OU, GPO o oggetti specifici (es. "reset password", "modifica GPO_IT_Admin") a un gruppo, senza concedere i diritti completi di Domain Admin - riducendo la superficie di rischio amministrativo. Laboratorio GPO · Windows Server 2022

# I O

# Hai completato il percorso GPO

Dalla creazione al ripristino, dal Security Filtering al RSoP: 18 laboratori pratici più le basi teoriche di trust, foreste e delega amministrativa per proseguire lo studio. Grazie per l'attenzione

Laboratorio GPO · Windows Server 2022
