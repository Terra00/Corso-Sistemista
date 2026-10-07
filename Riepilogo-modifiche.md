# Riepilogo delle modifiche

## Struttura creata

La raccolta è stata distribuita in cartelle descrittive e stabili:

- `Linux/`: terminale, file system e storage, utenti e permessi, rete e accesso, amministrazione, distribuzioni;
- `Reti/`: concetti di rete trasversali ai sistemi operativi;
- `Cybersecurity/`: cifratura e risorse di sicurezza;
- `Windows-Server/`: rete, Active Directory, storage e criteri di gruppo;
- `Strumenti-e-metodologie/`: Markdown, Git, VMware e gestione dei servizi.

L'[Indice](Indice.md) contiene i collegamenti relativi a tutti i documenti risultanti.

## Spostamenti e rinomini

I nomi sono stati resi descrittivi senza prefissi numerici. In particolare:

- gli appunti Linux sono stati separati per area funzionale; `Working-with-files.md` è diventato `Linux/File-system-e-storage/Gestione-file-e-collegamenti.md`;
- i moduli Windows Server sono stati riuniti in `Windows-Server/`, con una cartella specifica per Active Directory, rete, storage e GPO;
- le risorse sulla sicurezza sono ora in `Cybersecurity/`; le risorse trasversali sulle reti sono in `Reti/`;
- Git, Markdown, VMware e ITIL sono stati raccolti in `Strumenti-e-metodologie/`.

## Fusioni e doppioni rimossi

- `Terzo Giorno.md` era un duplicato sostanziale di `root danneggiata.md`: il contenuto è stato conservato e riscritto in `Linux/Amministrazione/Ripristino-accesso-root-e-pacchetti-Alpine.md`.
- `umask.md` e `privilegi.md` ripetevano materiale già presente, più completo, nella guida ai file e ai permessi. Le nozioni uniche sull'ordine di scelta tra proprietario, gruppo e altri sono state integrate nella guida principale.
- `creare-user.md` è stato incorporato in `Linux/Utenti-e-permessi/Account-e-gruppi-locali.md`.
- `Comandi.md` è stato trasformato nel riferimento breve `Linux/Terminale-e-shell/Comandi-di-base.md`, con rinvii al documento di approfondimento per evitare una seconda guida estesa sugli stessi comandi.

I documenti che trattano un argomento comune ma hanno finalità diverse — per esempio moduli, laboratori e schede rapide di Windows Server, oppure `top` e gestione dei processi — sono stati mantenuti distinti.

## Correzioni e integrazioni (revisione precedente)

- Corrette imprecisioni e refusi in VMware, `vi`, Alpine, LVM e ITIL.
- Riorganizzata e chiarita la procedura di ripristino dell'accesso root su Alpine, distinguendo i valori esemplificativi da quelli da verificare sulla macchina reale.
- Corretto il comando per assegnare proprietario e gruppo a una directory LVM (`chown`, non `chmod`).
- Aggiunta una nota sulla scelta effettiva delle classi di permesso Unix e una sezione pratica per creare e verificare un account locale.
- Rimossa dalla guida Tailscale una chiave di autorizzazione esposta e sostituita con un segnaposto. L'utente ha confermato che la chiave era già scaduta.
- Aggiunti collegamenti relativi fra il riferimento rapido dei comandi, la guida ai file e la procedura VMware/di ripristino.

## Revisione tecnica corrente (ottobre 2026)

### Deduplicazione paragrafi-tabelle (224 paragrafi, 13 file)
Rimossi i paragrafi che ripetevano esattamente il contenuto delle tabelle (etichetta in grassetto identica alla prima colonna, testo coperto dalle celle). File interessati: tutti i moduli Windows Server.

### Rimozione piedi di pagina slide nelle intestazioni tabella (13 casi)
Eliminati i residui di numerazione slide (es. `Modulo 3.a · Domain Controller · Windows Server 2022 19 / 31`) finiti dentro le intestazioni di tabella.

### Conversione output console in blocchi di codice (24 casi)
Le tabelle usate per mostrare output di comandi sono state convertite in blocchi `text` o `powershell`/`bat` per leggibilità e copiabilità.

### Unificazione codifica e fine riga (48 file)
Tutti i file in UTF-8 senza BOM con fine riga LF (nessun file misto).

### Correzione link interrotti e Indice.md (11 casi)
Riparati 2 link interrotti e aggiunti 9 documenti mancanti nell'Indice.md.

### Correzione blocchi di codice appiattiti (~16 righe in 9 file)
Separati comandi PowerShell/Batch/cmd su righe distinte, ricostruiti blocchi con linguaggio appropriato (`powershell`, `bat`, `text`), associato output al comando che lo produce. File corretti:
- `Windows-Server/Batch/Automazione-con-file-batch-Windows-Server.md` (5 casi)
- `Windows-Server/Active-Directory/Modulo-3a-Domain-Controller.md` (7 casi)
- `Windows-Server/Active-Directory/Modulo-3b-Oggetti-Active-Directory.md` (4 casi)
- `Windows-Server/Active-Directory/Materiale-integrativo-Active-Directory.md` (1 caso)
- `Windows-Server/GPO/CriteridiGruppo-e-GPO-Windows-Server.md` (7 casi)
- `Windows-Server/Prestazioni-attivita-pianificate/Monitoraggioeprestazioni-Windows-Server.md` (6 casi)
- `Windows-Server/Rete/Modulo-2-Networking-e-Desktop-remoto.md` (1 caso)
- `Windows-Server/Storage-e-file-server/Modulo-4-Storage-NTFS-e-condivisioni.md` (6 casi)

### Verifica integrità contenuto (735 token)
Confrontato token-per-token ogni file con il backup (`C:\Users\Raffaele\AppData\Local\Temp\opencode\backup-md`). Le differenze sono attribuibili a: deduplicazione paragrafi-tabelle, rimozione piedi di pagina slide, normalizzazione escape (`\\`→`\`, `\|`→`|`), correzione blocchi appiattiti. **Nessuna perdita di contenuto unico confermata.**

## Verifiche e punti da completare

- I collegamenti Markdown creati nell'indice e nelle guide aggiornate usano percorsi relativi e sono stati controllati rispetto alla struttura finale.
- La sezione Windows Server dispone ora di una guida operativa che riordina il percorso, le dipendenze, la configurazione di laboratorio e le regole tecniche trasversali. Il riferimento ai comandi di rete è stato riscritto in forma consultabile.
- Alcuni output, indirizzi IP, date, versioni di software e comandi presenti nei laboratori sono esempi dell'ambiente didattico. Non sono stati eseguiti su macchine reali durante questa revisione.
- Le affermazioni tecniche lasciate come note di cautela nei documenti originali sono state mantenute; dove l'ambiente effettivo può cambiare il risultato (LVM, cifratura, filesystem, firmware), è stato esplicitato che serve una verifica prima dell'intervento.
- **Priorità 3 (scelte di stile):** mantenuti separatori tabella `:---` (allineamento sinistro, retaggio estrazione) per coerenza interna; intestazioni tabella ricostruite dove mancanti con etichette descrittive (`Elemento | Descrizione`); ripetizioni deliberate (es. `Accesso effettivo di Luca` in Modulo-3b) mantenute come struttura didattica.
