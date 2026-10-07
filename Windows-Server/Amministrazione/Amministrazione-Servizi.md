# Modulo 5.A: Amministrazione dei Servizi di Windows Server

Questo modulo descrive la gestione dei servizi in Windows Server 2022, spiegando la differenza tra processi e servizi, l'interazione con il *Service Control Manager* (SCM) e l'uso di PowerShell e delle console di gestione.

---

## 1. Obiettivi e Prerequisiti

### Alla fine saprete
- Spiegare che cos'è un servizio e chi lo gestisce.
- Leggere lo stato, il tipo di avvio e le dipendenze di un servizio.
- Usare la console di gestione dei servizi (`services.msc`) e i cmdlet di PowerShell (`Get-Service`, `Start-Service`, `Stop-Service`, `Set-Service`).
- Arrestare, avviare e configurare un servizio, verificandone l'esito.

### Prerequisiti
- Moduli precedenti completati: `SVR-LAB-01` configurato come Domain Controller (`lab.local`) e `CLI-W11-01` come client unito al dominio.
- Accesso come amministratore (`LAB\Administrator`).
- PowerShell avviato con privilegi elevati (Amministratore).

---

## 2. Concetti Chiave: Processo, Servizio e SCM

### Processo vs Servizio
- **Processo:** Un programma in esecuzione in memoria, solitamente avviato da un utente (spesso dotato di interfaccia grafica), che termina alla chiusura o alla disconnessione dell'utente (es. `notepad.exe`).
- **Servizio:** Un componente software che opera in background (senza interfaccia grafica) e può essere avviato automaticamente all'avvio del sistema operativo prima ancora che un utente effettui l'accesso. Un servizio è sempre ospitato all'interno di uno o più processi (spesso condivisi tramite `svchost.exe`).

### Il Service Control Manager (SCM)
Il Service Control Manager (`services.exe`) è il componente architetturale di Windows che riceve le richieste di gestione (avvio, arresto, modifica della configurazione) e le applica interagendo direttamente con i servizi.
> **Nota:** Gli strumenti di amministrazione (PowerShell o `services.msc`) non modificano direttamente lo stato del servizio, ma inviano la richiesta all'SCM, garantendo coerenza e tracciabilità nel registro eventi.

### Nomi interni e nomi visualizzati
Ogni servizio dispone di due denominazioni principali:
1. **Nome interno (`Name`):** La chiave identificativa di sistema (es. `Spooler`, `DNS`, `Netlogon`). Non cambia mai, indipendentemente dalla lingua del sistema operativo, e va utilizzata nei comandi PowerShell.
2. **Nome visualizzato (`DisplayName`):** La stringa descrittiva mostrata nelle interfacce grafiche (es. *Spooler di stampa*, *Server DNS*), localizzata in base alla lingua del sistema operativo.

---

## 3. Gestione e Lettura dei Servizi (PowerShell e Console)

### Elenco e ordinamento dei servizi
Per ottenere l'elenco completo dei servizi ordinati per stato e nome visualizzato:
```powershell
Get-Service | Sort-Object Status, DisplayName
```

### Servizi critici su un Domain Controller (`SVR-LAB-01`)
| Servizio | Nome Interno | Funzione Principale | Impatto in caso di arresto anomalo |
| :--- | :--- | :--- | :--- |
| **NTDS** | `NTDS` | Database di Active Directory | Fallimento delle autenticazioni e delle modifiche di dominio |
| **DNS** | `DNS` | Risoluzione dei nomi e record SRV | I client perdono la connettività con il Domain Controller |
| **Netlogon** | `Netlogon` | Ricerca del DC e registrazione record | Compromissione degli accessi di dominio |
| **Kdc** | `Kdc` | Autenticazione Kerberos | Impossibilità di emettere nuovi ticket di accesso |
| **ADWS** | `ADWS` | Web Service per Active Directory | Fallimento dei cmdlet PowerShell di Active Directory |

### Verifica funzionale di un servizio
Verificare che il servizio sia in esecuzione non garantisce che risponda correttamente alle richieste. È sempre opportuno associare lo stato a una verifica funzionale:
```powershell
# Verifica dello stato del servizio DNS
Get-Service -Name DNS

# Verifica funzionale della risposta del server DNS locale
Resolve-DnsName lab.local -Server 127.0.0.1
```

### Gestione delle dipendenze
Per identificare i prerequisiti e i servizi collegati:
```powershell
# Servizi necessari affinché DNS possa avviarsi
Get-Service -Name DNS -RequiredServices

# Servizi che dipendono da DNS (se arrestati o interrotti)
Get-Service -Name DNS -DependentServices
```

---

## 4. Stato e Tipo di Avvio

È fondamentale distinguere lo stato corrente dal comportamento configurato all'avvio del sistema:
- **Stato (`State` / `Status`):** Condizione attuale (`Running`, `Stopped`, `Paused`). Modificabile temporaneamente con `Start-Service` o `Stop-Service`.
- **Tipo di avvio (`StartMode`):** Configurazione persistente al riavvio (`Auto`, `Manual`, `Disabled`). Modificabile con `Set-Service` o utility di sistema.

### Verifica avanzata con WMI/CIM
Per visualizzare contemporaneamente stato, tipo di avvio ed eventuale ritardo:
```powershell
Get-CimInstance Win32_Service -Filter "Name='DNS'" | Select-Object Name, State, StartMode, DelayedAutoStart
```

### Configurazione dell'avvio ritardato (`DelayedAutoStart`)
Poiché PowerShell 5.1 non gestisce nativamente l'avvio ritardato tramite `Set-Service`, si utilizza l'utility di sistema `sc.exe`:
```powershell
sc.exe config Spooler start= delayed-auto
```
*(Nota: prestare attenzione allo spazio obbligatorio dopo il segno `=`).*

---

## 5. Operatività: Gestione dello Spooler e Ciclo dei Servizi

### Disattivazione dello Spooler sul Domain Controller
Le best practice Microsoft raccomandano di disattivare il servizio di stampa (`Spooler`) sui Domain Controller per ridurre la superficie di attacco:
```powershell
Stop-Service -Name Spooler
Set-Service -Name Spooler -StartupType Disabled
```

### Ciclo di gestione sul client (`CLI-W11-01`)
```powershell
# Arresto del servizio
Stop-Service -Name Spooler

# Avvio del servizio
Start-Service -Name Spooler

# Riavvio rapido (arresto e successivo avvio)
Restart-Service -Name Spooler
```

---

## 6. Riferimento Rapido dei Comandi

| Comando PowerShell / Utility | Scopo | Ambito |
| :--- | :--- | :--- |
| `Get-Service | Sort-Object Status, DisplayName` | Elenco completo dei servizi ordinati | Tutti |
| `Get-Service -Name <Nome>` | Informazioni su uno specifico servizio | Tutti |
| `Get-Service -Name <Nome> -RequiredServices` | Visualizza le dipendenze in ingresso | Tutti |
| `Get-Service -Name <Nome> -DependentServices` | Visualizza i servizi dipendenti | Tutti |
| `Get-CimInstance Win32_Service -Filter "Name='<Nome>'"` | Stato dettagliato, tipo di avvio e ritardo | Tutti |
| `Start-Service / Stop-Service -Name <Nome>` | Avvia o arresta un servizio | Tutti |
| `Restart-Service -Name <Nome>` | Riavvia un servizio | Tutti |
| `Set-Service -Name <Nome> -StartupType Disabled` | Modifica il tipo di avvio | Tutti |
| `sc.exe config <Nome> start= delayed-auto` | Configura l'avvio automatico ritardato | Tutti |
| `services.msc` | Apre la console grafica dei servizi | Tutti |
