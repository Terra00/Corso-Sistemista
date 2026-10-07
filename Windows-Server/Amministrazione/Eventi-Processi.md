# Modulo 5.B: Eventi e Processi in Windows Server

Questo modulo approfondisce l'analisi diagnostica tramite il registro eventi di Windows e l'ispezione dei processi in esecuzione, consentendo di risalire dalle anomalie di sistema alle cause sottostanti.

---

## 1. Obiettivi e Prerequisiti

### Alla fine saprete
- Distinguere i registri di sistema e i livelli di gravità degli eventi.
- Filtrare gli eventi tramite la console grafica e il cmdlet `Get-WinEvent`.
- Identificare le tracce lasciate da avvii, arresti e variazioni di stato dei servizi.
- Leggere le informazioni sui processi tramite Task Manager e `Get-Process`.
- Risalire da un processo (inclusi i contenitori `svchost.exe`) ai servizi specifici ospitati.

### Prerequisiti
- Modulo 5.A completato con successo.
- `SVR-LAB-01` con PowerShell attivo come Amministratore e credenziali `LAB\Administrator`.

---

## 2. Il Visualizzatore Eventi (`eventvwr.msc`)

### Registri Principali di Windows
| Registro | Contenuto Principale | Utilizzo Tipico nel Corso |
| :--- | :--- | :--- |
| **Sistema** | Eventi dei componenti di sistema e dei driver di dispositivo, inclusi i servizi gestiti dall'SCM. | Monitoraggio di avvii, arresti e variazioni di stato dei servizi. |
| **Applicazione** | Eventi registrati da programmi e applicazioni installate nel sistema. | Diagnostica applicativa generale. |
| **Sicurezza** | Tracce di audit relative agli accessi e alle autorizzazioni (`Audit Logon`). | Controllo di accessi riusciti e falliti. |
| **Registri applicazioni e servizi** | Sottocategorie specifiche per componente (es. `Operational` per Utilità di pianificazione o GPO). | Analisi dettagliata del comportamento dei sottosistemi. |

### Livelli di Gravità degli Eventi
- **Critical (Critico):** Errore grave che ha provocato l'arresto anomalo del sistema o di un componente essenziale.
- **Error (Errore):** Un'operazione o un servizio non è riuscito a completarsi.
- **Warning (Avviso):** Condizione non immediatamente critica ma che potrebbe evolvere in un malfunzionamento.
- **Informational (Informazioni):** Registrazione di un evento di routine (es. avvio completato, servizio modificato).
- **Verbose (Dettagliato):** Dati diagnostici granulari utili al troubleshooting avanzato.

---

## 3. Analisi degli Eventi con PowerShell (`Get-WinEvent`)

### Lettura degli ultimi eventi di sistema
```powershell
Get-WinEvent -LogName System -MaxEvents 10
```

### Filtri avanzati basati su Hashtable (`-FilterHashtable`)
L'uso delle hashtable consente di pre-filtrare gli eventi direttamente all'origine nel motore di log, ottenendo prestazioni superiori:
```powershell
$Filtro = @{ LogName = 'System'; ProviderName = 'Service Control Manager' }
Get-WinEvent -FilterHashtable $Filtro -MaxEvents 20
```

### Ricerca testuale mirata nel messaggio
```powershell
$Filtro = @{ LogName = 'System'; ProviderName = 'Service Control Manager' }
Get-WinEvent -FilterHashtable $Filtro -MaxEvents 50 | Where-Object { $_.Message -like "*Spooler di stampa*" }
```

### Event ID Fondamentali per la Diagnostica
| Event ID | Registro | Provider (ProviderName) | Significato / Evento di Riferimento |
| :--- | :--- | :--- | :--- |
| **6005 / 6008** | Sistema | `EventLog` | Avvio del servizio Registro eventi / Arresto anomalo precedente. |
| **1074** | Sistema | `User32` | Riavvio o arresto di sistema avviato da un processo o utente. |
| **41** | Sistema | `Microsoft-Windows-Kernel-Power` | Riavvio di sistema senza arresto pulito precedente (*Kernel-Power*). |
| **4624 / 4625** | Sicurezza | `Microsoft-Windows-Security-Auditing` | Accesso riuscito (`4624`) o fallito (`4625`), se l'auditing è attivo. |
| **7036 / 7031** | Sistema | `Service Control Manager` | Variazione di stato di un servizio (`7036`) o arresto anomalo (`7031`). |

---

## 4. Gestione Attività e Analisi dei Processi

### Utilizzo di `Get-Process`
Per visualizzare i processi ordinati per consumo cumulativo di CPU:
```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, Id, CPU, WorkingSet
```
> **Attenzione:** La proprietà `CPU` rappresenta il tempo totale di processore (in secondi) accumulato dall'avvio del processo, non la percentuale di carico istantanea.

### Il ruolo dei processi contenitore (`svchost.exe`)
Molti servizi di Windows non dispongono di un eseguibile dedicato, ma vengono eseguiti all'interno di istanze condivise denominate `svchost.exe`.

#### Risalire dal PID ai servizi ospitati
Se un processo `svchost.exe` (identificato dal PID tramite Task Manager o `Get-Process`) mostra un consumo anomalo, è possibile individuare esattamente quali servizi ospita:
```powershell
# Sostituire 1234 con il PID effettivo letto dal sistema
Get-CimInstance Win32_Service -Filter "ProcessId=1234" | Select-Object Name, DisplayName, State
```
In alternativa da prompt dei comandi (`cmd.exe`):
```cmd
tasklist /svc
```

---

## 5. Riferimento Rapido dei Comandi

| Comando PowerShell / Utility | Scopo |
| :--- | :--- |
| `Get-WinEvent -LogName System -MaxEvents 10` | Visualizza gli ultimi eventi del registro di sistema |
| `$Filtro = @{ LogName='...'; ProviderName='...' }` | Crea una hashtable di filtro per i log |
| `Get-WinEvent -FilterHashtable $Filtro -MaxEvents 20` | Estrae eventi filtrati ad alta efficienza |
| `Get-Process | Sort-Object CPU -Descending` | Elenca i processi ordinati per consumo di CPU |
| `Get-CimInstance Win32_Service -Filter "ProcessId=<PID>"` | Elenca i servizi ospitati da uno specifico PID |
| `eventvwr.msc` | Apre la console grafica del Visualizzatore eventi |
| `taskmgr` (Ctrl+Shift+Esc) | Apre Gestione attività |
