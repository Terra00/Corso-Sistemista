# Corso Sistemista - Repository Materiale Didattico

Questo repository contiene tutto il materiale didattico del corso per Sistemisti, organizzato per argomenti e tecnologie. Copre l'amministrazione di sistemi Windows Server, Linux, reti, cybersecurity, e strumenti di gestione.

## 📁 Struttura del Repository

```
Corso Sistemista/
├── Windows-Server/           # Amministrazione Windows Server
├── Linux/                    # Amministrazione Linux
├── Reti/                     # Networking e protocolli
├── Cybersecurity/            # Sicurezza informatica
├── Strumenti-e-metodologie/  # Tool, virtualizzazione, Git, ITIL
├── Indice.md                 # Indice completo dei contenuti
└── Riepilogo-modifiche.md    # Changelog del materiale
```

---

## 🪟 Windows Server

### Active Directory
- **Domain-Controller.md** - Configurazione e gestione Domain Controller
- **Laboratorio-01-Domain-Controller.md** - Lab pratico: setup Domain Controller
- **Oggetti-Active-Directory.md** - Gestione oggetti AD (utenti, gruppi, OU, computer)
- **Materiale-integrativo-Active-Directory.md** - Risorse aggiuntive e approfondimenti

### Group Policy Objects (GPO)
- **CriteridiGruppo-e-GPO-Windows-Server.md** - Teoria e configurazione GPO
- **Laboratorio-GPO.md** - Esercitazione pratica su Group Policy

### Storage e File Server
- **Storage-NTFS-e-condivisioni.md** - Filesystem NTFS, permessi, condivisioni SMB/NFS

### Servizi e Amministrazione
- **Guida-operativa-Windows-Server.md** - Guida rapida operativa
- **Amministrazione/Amministrazione-Servizi.md** - Gestione servizi Windows
- **Amministrazione/Eventi-Processi.md** - Event Viewer e gestione processi
- **Servizi-e-amministrazione/Amministrazionediservices-Windows-Server.md** - Approfondimento servizi

### Rete e Desktop Remoto
- **Rete/Networking-e-Desktop-remoto.md** - Configurazione rete, RDP, VPN
- **Rete/Comandi-di-rete.md** - Comandi rete essenziali (ipconfig, netsh, ping, tracert, nslookup)

### Monitoraggio e Prestazioni
- **Prestazioni-attivita-pianificate/Monitoraggioeprestazioni-Windows-Server.md** - Performance Monitor, Task Scheduler, Resource Monitor

### Analisi Eventi e Processi
- **Eventi-processi/Analisi-eventie-processi-Windows-Server.md** - Analisi log, troubleshooting eventi

### Automazione
- **Batch/Automazione-con-file-batch-Windows-Server.md** - Script batch, variabili, loop, condizionali

---

## 🐧 Linux

### Terminale e Shell
- **Terminale-e-shell/Comandi-di-base.md** - Comandi fondamentali (ls, cd, cp, mv, rm, mkdir, etc.)
- **Terminale-e-shell/Bash-cheat-sheet.md** - Riferimento rapido comandi Bash
- **Terminale-e-shell/Editor-vi.md** - Guida editor vi/vim
- **Terminale-e-shell/Redirezioni-e-pipe.md** - stdin/stdout/stderr, pipe, redirezioni
- **Terminale-e-shell/Sessione-e-cronologia-Bash.md** - History, alias, variabili d'ambiente

### File System e Storage
- **File-system-e-storage/Filesystem-Hierarchy-Standard.md** - Standard FHS, struttura directory
- **File-system-e-storage/Gestione-file-e-collegamenti.md** - Hard/soft link, find, locate, tar
- **File-system-e-storage/Dischi-e-spazio.md** - lsblk, df, du, fdisk, parted, mount
- **File-system-e-storage/Archivi-tar.md** - Creazione/gestione archivi tar, compressione gzip/bzip2/xz
- **File-system-e-storage/LVM-su-Alpine.md** - Logical Volume Manager su Alpine Linux
- **File-system-e-storage/Estendere-root-con-LVM-su-Rocky.md** - Estensione partizione root LVM su Rocky Linux

### Utenti e Permessi
- **Utenti-e-permessi/Account-e-gruppi-locali.md** - useradd, usermod, passwd, groupadd, sudoers, permessi file

### Rete e Accesso Remoto
- **Rete-e-accesso/Accesso-SSH-con-chiavi.md** - Configurazione SSH key-based authentication
- **Rete-e-accesso/Tailscale.md** - VPN mesh Zero-config con Tailscale

### Amministrazione di Sistema
- **Amministrazione/Journal-e-log.md** - systemd-journald, logrotate, analisi log
- **Amministrazione/Monitoraggio-processi-con-top.md** - top, htop, ps, kill, nice/renice
- **Amministrazione/Processi-e-pianificazione.md** - systemd timer, cron, at, process management
- **Amministrazione/Backup-di-etc-con-cron.md** - Script backup automatico /etc con cron
- **Amministrazione/Avvio-systemd-e-diagnostica-su-Rocky.md** - Boot systemd, target, diagnostica avvio
- **Amministrazione/Ripristino-accesso-root-e-pacchetti-Alpine.md** - Recovery root, gestione pacchetti apk

### Distribuzioni
- **Distribuzioni/Configurare-Bash-e-sudo.md** - Personalizzazione .bashrc, configurazione sudo
- **Distribuzioni/Installare-GNU-tar.md** - Installazione GNU tar su sistemi minimal

---

## 🌐 Reti

- **Indirizzamento-IPv4-classful.md** - Classi IP, subnet mask, calcolo sottoreti, CIDR
- **Dispositivi-di-rete.md** - Switch, router, firewall, access point, VLAN, trunking

---

## 🔒 Cybersecurity

- **Risorse-cybersecurity.md** - Link utili, framework (NIST, MITRE ATT&CK), tool, certificazioni
- **Crittografia-disco-LUKS.md** - Crittografia full-disk con LUKS su Linux, gestione chiavi

---

## 🛠️ Strumenti e Metodologie

### Virtualizzazione
- **Virtualizzazione/VMware-ordine-di-avvio.md** - Configurazione ordine boot VM, BIOS/UEFI

### Markdown e Git
- **Markdown-e-Git/Manuale-Markdown.md** - Sintassi Markdown completa, tabelle, mermaid, GitHub Flavored
- **Markdown-e-Git/Git.md** - Comandi Git, branching, merge, rebase, remote, workflow

### Gestione Servizi (ITIL)
- **Gestione-servizi/ITIL-e-problemi-disco.md** - Framework ITIL, incident/problem/change management, troubleshooting disco

---

## 📋 File di Navigazione

| File | Descrizione |
|------|-------------|
| **Indice.md** | Indice alfabetico e per categoria di tutti i documenti |
| **Riepilogo-modifiche.md** | Storico aggiornamenti e versioni del materiale |

---

## 🎯 Obiettivi del Corso

Al termine del percorso lo studente sarà in grado di:

- ✅ Installare, configurare e gestire **Windows Server** (AD, GPO, File Server, Servizi)
- ✅ Amministrare sistemi **Linux** (shell, storage LVM, utenti, servizi systemd, rete)
- ✅ Configurare e troubleshooting **reti** (IPv4, subnetting, dispositivi, VPN)
- ✅ Applicare principi base di **Cybersecurity** (crittografia, hardening, framework)
- ✅ Utilizzare **strumenti professionali** (Git, Markdown, virtualizzazione, ITIL)
- ✅ Automatizzare task con **script** (Batch, Bash, cron/systemd timer)

---

## 📚 Come Usare Questo Materiale

1. **Studio sequenziale**: Segui l'ordine logico (basi → avanzato) per ogni sezione
2. **Laboratori pratici**: Esegui i lab (es. `Laboratorio-01-Domain-Controller.md`, `Laboratorio-GPO.md`)
3. **Riferimento rapido**: Usa i cheat-sheet (`Bash-cheat-sheet.md`, `Comandi-di-rete.md`)
4. **Approfondimenti**: Consulta il materiale integrativo per temi specifici

---

## 🔧 Prerequisiti Consigliati

- Conoscenza base di sistemi operativi
- Familiarità con riga di comando (Windows CMD/PowerShell o Linux Bash)
- Concetti base di rete (IP, DNS, DHCP)
- Virtualizzazione (VMware Workstation/Player, VirtualBox, Hyper-V) per i lab

---

## 📝 Note

- Il materiale è in continuo aggiornamento (vedi `Riepilogo-modifiche.md`)
- I comandi sono testati su: **Windows Server 2019/2022**, **Rocky Linux 9**, **Alpine Linux**, **Ubuntu 22.04+**
- Per ambienti di prova si consiglia l'uso di macchine virtuali

---

## 📄 Licenza

Materiale didattico per uso interno del corso. Tutti i diritti riservati.

---

*Ultimo aggiornamento: Ottobre 2026*