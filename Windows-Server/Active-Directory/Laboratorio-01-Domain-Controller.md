# Laboratorio 01

| Installazione di un Domain Controller Windows Server 2022 | e Join di Windows 11 al Dominio |
|:--- |:--- |
| DOMINIO | lab.local |
| SERVER | SVR-LAB-01   192.168.134.130 |
| CLIENT | CLI-W11-01   192.168.134.129 |
| RETE | VMware Workstation - 192.168.134.0/24 | **DOMINIO** lab.local

# SERVER SVR-LAB-01   192.168.134.130

# CLIENT CLI-W11-01   192.168.134.129

**RETE** VMware Workstation - 192.168.134.0/24

*Dispensa ufficiale del corso - Livello junior / intermedio*

Windows Server 2022 Standard  ·  Windows 11 Pro  ·  VMware Workstation

# Indice del laboratorio 1. Introduzione 2. Architettura del laboratorio 3. Configurazione iniziale 4. Installazione di Active Directory 5. Promozione a Domain Controller 6. Verifica servizi 7. Configurazione DNS del server 8. Configurazione del client Windows 11 9. Verifica della rete 10. Errori realmente riscontrati 11. Join del client al dominio 12. Verifica finale 13. Tabella riassuntiva dei comandi 14. Errori comuni 15. Cosa deve ricordare uno studente 16. Quiz finale 17. Conclusione Introduzione Lab 01 - AD DS & Join al dominio

# Introduzione

I concetti fondamentali prima di mettere le mani sulla console: dominio, Domain Controller e perché servono in un'infrastruttura Windows.

## INTRODUZIONE

# Dominio e Domain Controller: i concetti chiave

## Cos'è un dominio Windows

Un dominio è un raggruppamento logico di computer, utenti e risorse che condividono uno stesso database di sicurezza centralizzato, gestito da Active Directory Domain Services (AD DS).

## Cos'è un Domain Controller (DC)

È il server che ospita il database di Active Directory e fornisce i servizi di autenticazione, autorizzazione e directory a tutti i membri del dominio.

## Perché serve un Domain Controller

Centralizza la gestione di utenti, gruppi, computer e criteri di sicurezza (Group Policy), evitando di configurare ogni singola macchina in modo isolato e incoerente.

## INTRODUZIONE

# Workgroup vs Dominio

# WORKGROUP

**•** • Ogni PC gestisce localmente i propri account utente

**•** • Nessun database di sicurezza condiviso tra i computer

**•** • Adatto a reti molto piccole (indicativamente fino a 10-20 PC) **•** • Nessuna Group Policy centralizzata

**•** • Le credenziali vanno create su ogni singola macchina

# DOMINIO

**•** • Autenticazione centralizzata tramite Active Directory

**•** • Un solo account utente valido su tutti i computer membri

**•** • Scalabile a migliaia di computer e utenti

**•** • Criteri di sicurezza (GPO) applicati centralmente

**•** • Gestione unificata da un'unica console amministrativa

# Architettura del laboratorio

## Una singola rete privata VMware ospita il Domain Controller e il client Windows 11 che verrà unito al dominio.

## ARCHITETTURADELLABORATORIO

# Schema logico della rete di laboratorio

**VMware -** ** ** **Rete privata 192.168.134.0/24**

## \|

**Windows Server 2022** **SVR-LAB-01  ·  192.168.134.130**

## \|

**Active Directory  ·  DNS  ·  Domain Controller**

## \|

**Windows 11** **CLI-W11-01  ·  192.168.134.129**

*Server e client condividono la stessa rete privata isolata creata da VMware Workstation.* *Il client utilizza il Domain Controller come unico riferimento DNS per raggiungere il dominio.*

# Configurazione iniziale

## Prima di installare qualsiasi ruolo, il server riceve un indirizzamento IP statico coerente con la rete di laboratorio.

## CONFIGURAZIONEINIZIALE

# Configurazione IP del server SVR-LAB-01 **Parametro** **Valore** **A cosa serve**

Nome computer SVR-LAB-01 Identifica il server nella rete; diventerà parte del nome DNS completo (FQDN) una volta promosso a Domain Controller.

Indirizzo IP 192.168.134.130 Indirizzo statico del server: un DC deve avere sempre un IP fisso, mai assegnato via DHCP, per essere raggiungibile in modo stabile.

| Subnet mask | 255.255.255.0 | Definisce l'ampiezza della rete locale (/24): identifica quali indirizzi appartengono alla rete 192.168.134.0/24. |
|:--- |:--- |:--- |
| Gateway predefinito | 192.168.134.2 | Indirizzo del router usato per instradare il traffico verso reti esterne alla subnet locale. |
| DNS preferito | 192.168.134.130 | Il server punta a se stesso come server DNS: requisito essenziale per Active Directory (vedi sezione 7). |

Gateway predefinito 192.168.134.2 Indirizzo del router usato per instradare il traffico verso reti esterne alla subnet locale.

DNS preferito 192.168.134.130 Il server punta a se stesso come server DNS: requisito essenziale per Active Directory (vedi sezione 7).

# Installazione di Active Directory

Il ruolo Active Directory Domain Services (AD DS) è il componente software che rende possibile trasformare il server in un Domain Controller.

## INSTALLAZIONEDIACTIVEDIRECTORY

# Cosa fa il ruolo AD DS e perché installarlo

## Cosa fa il ruolo AD DS

Installa i binari e i servizi necessari a ospitare un database di directory (NTDS), i protocolli LDAP e Kerberos, e gli strumenti di amministrazione di Active Directory.

## Perché installarlo

È il prerequisito obbligatorio: senza questo ruolo non è possibile eseguire la promozione del server a Domain Controller, descritta nella sezione successiva.

## INSTALLAZIONEDIACTIVEDIRECTORY

# Comando di installazione del ruolo

## Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

| Elemento | Descrizione |
|:--- |:--- |
| Descrizione | Installa il ruolo Active Directory Domain Services e i relativi strumenti di gestione (snap-in MMC, moduli PowerShell). |
| Sintassi | Install-WindowsFeature <NomeFeature> [-IncludeManagementTools] |

**Parametro -IncludeManagementTools** Installa anche la console di gestione e i cmdlet PowerShell dedicati (es. Active Directory Users and Computers); senza questo parametro il ruolo sarebbe installato senza strumenti di gestione grafici.

| Output atteso | Success: True   Restart Needed: No   Exit Code: Success   Feature Result: Active Directory Domain Services |

# Promozione del server a Domain Controller

Installare il ruolo non basta: il server deve essere promosso per diventare effettivamente un Domain Controller e creare una nuova foresta.

## PROMOZIONEDELSERVERADOMAINCONTROLLER

# Cosa comporta la promozione

## Cosa significa promuovere un server

La promozione (Active Directory Domain Services Configuration Wizard / Install-ADDSForest) trasforma il server da semplice membro a controller di dominio, creando il database di Active Directory e la nuova foresta lab.local.

## Cosa viene installato automaticamente

Durante la promozione vengono configurati in automatico: il servizio DNS Server, la cartella condivisa SYSVOL, il database NTDS e il servizio Kerberos KDC.

## Perché viene installato il DNS

Active Directory dipende interamente dal DNS per individuare i Domain Controller (record SRV) e risolvere i nomi delle risorse di dominio: senza DNS funzionante, il dominio non è utilizzabile.

## PROMOZIONEDELSERVERADOMAINCONTROLLER

# Il dominio: lab.local

## Nome DNS del dominio

lab.local è il nome DNS univoco che identifica il dominio Active Directory creato in laboratorio.

## Suffisso.local

È un suffisso non instradabile su Internet, tipico di laboratori didattici e reti isolate: evita conflitti con domini pubblici reali.

## Nomi completi (FQDN)

Tutti i computer membri diventano raggiungibili tramite FQDN nella forma nome-host.lab.local, ad esempio SVR-LAB-01.lab.local.

# Verifica servizi

## Dopo la promozione è indispensabile verificare che i servizi critici di Active Directory e DNS siano attivi e raggiungibili.

## VERIFICASERVIZI

# Stato dei servizi ADWS e DNS

```powershell
PS C:\> Get-Service ADWS,DNS
```

| Elemento | Descrizione |
| --- | --- |
| A cosa serve | Verifica lo stato (Running / Stopped) dei servizi Active Directory Web Services (ADWS) e DNS Server, entrambi indispensabili per il funzionamento del dominio. |
| Output atteso | Status: Running   Name: ADWS   DisplayName: Active Directory Web Services Status: Running   Name: DNS    DisplayName: DNS Server |
| Interpretazione | Se lo Status non Ã¨ Running, il servizio va avviato manualmente (Start-Service) e la causa va indagata nel Visualizzatore eventi. |

| Elemento | Descrizione |
| --- | --- |
| A cosa serve | Verifica lo stato (Running / Stopped) dei servizi Active Directory Web Services (ADWS) e DNS Server, entrambi indispensabili per il funzionamento del dominio. |

| Output atteso | Status: Running   Name: ADWS   DisplayName: Active Directory Web Services Status: Running   Name: DNS    DisplayName: DNS Server |

| Interpretazione | Se lo Status non è Running, il servizio va avviato manualmente (Start-Service) e la causa va indagata nel Visualizzatore eventi. |

# Verifica della porta LDAP (389)

## Test-NetConnection localhost -Port 389

| Elemento | Descrizione |
|:--- |:--- |
| A cosa serve | Verifica che la porta 389 (LDAP), usata per le query e l'autenticazione su Active Directory, sia in ascolto sul server. |
| Output atteso | ComputerName: localhost   RemotePort: 389   TcpTestSucceeded: True |
| Interpretazione | TcpTestSucceeded: True conferma che il servizio LDAP risponde; False indica che AD DS non è attivo o è bloccato dal firewall. |

# Verifica della porta DNS (53)

## Test-NetConnection localhost -Port 53

| Elemento | Descrizione |
|:--- |:--- |
| A cosa serve | Verifica che la porta 53 (DNS) sia in ascolto e che il servizio DNS Server risponda correttamente alle query. |
| Output atteso | ComputerName: localhost   RemotePort: 53   TcpTestSucceeded: True |
| Interpretazione | Se il test fallisce, il servizio DNS non è avviato oppure il firewall del server blocca la porta 53. |

# Configurazione DNS del server

Il Domain Controller deve essere il proprio primo riferimento DNS: è la regola fondamentale su cui si basa il funzionamento di Active Directory.

## CONFIGURAZIONEDNSDELSERVER

# Perché il DNS del DC deve puntare a se stesso

## Auto-risoluzione

Un Domain Controller deve usare se stesso (il proprio IP, 192.168.134.130) come server DNS primario, perché è lui a ospitare la zona DNS del dominio lab.local con i record SRV necessari a localizzare i servizi di AD.

## Cosa succede se punta altrove

Se il DC utilizza un DNS esterno che non conosce la zona lab.local, i client non riusciranno a individuare il Domain Controller e l'autenticazione di dominio fallirà.

## CONFIGURAZIONEDNSDELSERVER

# Verifica del DNS configurato sul server

## Get-DnsClientServerAddress

| Elemento | Descrizione |
|:--- |:--- |
| A cosa serve | Mostra quale server DNS è configurato su ciascuna interfaccia di rete del server. |
| Output atteso | InterfaceAlias: Ethernet0   AddressFamily: IPv4   ServerAddresses: {192.168.134.130} |
| Interpretazione | Il server deve avere come unico DNS il proprio indirizzo IP (192.168.134.130): qualsiasi altro valore è un errore di configurazione da correggere. |

# Configurazione del client Windows 11

## Anche il client richiede una configurazione di rete statica e coerente con il DNS del dominio, condizione indispensabile per il join.

## CONFIGURAZIONEDELCLIENTWINDOWS11

# Perché il client deve usare il DNS del Domain Controller

## IP statico

Anche il client riceve un indirizzo IP statico (192.168.134.129) nella stessa subnet del server, per garantire una comunicazione stabile e prevedibile durante i test.

## DNS del dominio

Il DNS del client deve puntare a 192.168.134.130 (il DC): è l'unico modo per risolvere correttamente i nomi del dominio lab.local e individuare i servizi di Active Directory.

## CONFIGURAZIONEDELCLIENTWINDOWS11

# Impostazione dell'IP statico

## netsh interface ip set address name="Ethernet0" static 192.168.134.129 255.255.255.0 192.168.134.2

| Elemento | Descrizione |
|:--- |:--- |
| Cosa fa | Imposta un indirizzo IP statico sull'interfaccia di rete specificata, sostituendo l'eventuale configurazione DHCP. |
| Sintassi | netsh interface ip set address name="<Interfaccia>" static <IP> <SubnetMask> <Gateway> |
| Esempio | netsh interface ip set address name="Ethernet0" static 192.168.134.129 255.255.255.0 192.168.134.2 |
| Risultato | L'interfaccia Ethernet0 del client riceve l'IP 192.168.134.129/24 con gateway 192.168.134.2, verificabile con ipconfig. |

## CONFIGURAZIONEDELCLIENTWINDOWS11

# Impostazione del DNS statico

## netsh interface ip set dns name="Ethernet0" static 192.168.134.130

| Elemento | Descrizione |
|:--- |:--- |
| Cosa fa | Configura il server DNS statico dell'interfaccia di rete, in modo che il client interroghi sempre il Domain Controller. |
| Sintassi | netsh interface ip set dns name="<Interfaccia>" static <IP-DNS> |
| Esempio | netsh interface ip set dns name="Ethernet0" static 192.168.134.130 |
| Risultato | Il client utilizza 192.168.134.130 come unico server DNS, requisito indispensabile per risolvere lab.local ed eseguire il join. |

# Verifica della rete

Prima del join, ogni parametro di rete e DNS del client viene verificato singolarmente per evitare errori difficili da diagnosticare in seguito.

## VERIFICADELLARETE

# Configurazione IP completa del client

## ipconfig /all

| Elemento | Descrizione |
|:--- |:--- |
| Spiegazione | Mostra la configurazione IP completa di tutte le interfacce di rete del client: indirizzo, subnet, gateway, DNS e altro. |
| Output atteso | IPv4 Address: 192.168.134.129   Subnet Mask: 255.255.255.0   Default Gateway: 192.168.134.2   DNS Servers: 192.168.134.130 |
| Interpretazione | Conferma che il client ha effettivamente ricevuto la configurazione statica impostata al passo precedente. |

# Test di connettività verso il server

## ping 192.168.134.130

| Elemento | Descrizione |
|:--- |:--- |
| Spiegazione | Verifica la connettività di rete di livello IP tra il client e il Domain Controller. |
| Output atteso | Reply from 192.168.134.130: bytes=32 time<1ms TTL=128   (0% perdita pacchetti) |
| Interpretazione | Risposte regolari confermano che il percorso di rete funziona; timeout o 'Richiesta scaduta' indicano un problema di rete o firewall. |

# Risoluzione del nome di dominio

## nslookup lab.local **Elemento** **Descrizione**

**Spiegazione** Interroga il server DNS configurato per risolvere il nome del dominio lab.local.

| Output atteso | Server: SVR-LAB-01.lab.local   Address: 192.168.134.130 Name: lab.local   Address: 192.168.134.130 |

| Interpretazione | Conferma che il client sta usando il DNS corretto (192.168.134.130) e che la zona lab.local è risolvibile. |

# Risoluzione del FQDN del server

## Resolve-DnsName SVR-LAB-01.lab.local

| Elemento | Descrizione |
|:--- |:--- |
| Spiegazione | Cmdlet PowerShell equivalente a nslookup: risolve il nome FQDN del Domain Controller in un indirizzo IP. |
| Output atteso | Name: SVR-LAB-01.lab.local   Type: A   Address: 192.168.134.130 |
| Interpretazione | Se la risoluzione fallisce, il problema è quasi sempre nella configurazione DNS del client o nella registrazione del record sul server. |

# Errori realmente riscontrati

La diagnostica è parte integrante del lavoro con Active Directory: ecco i problemi reali incontrati durante il laboratorio e come sono stati risolti.

# 1 0. E R R O R I R E A L M E N T E R I S C O N T R A T I

# Diagnostica DNS (1/3) **nslookup: "Server failed"**

# SINTOMO

Il comando nslookup restituisce l'errore 'Server failed' invece di risolvere il nome richiesto.

# CAUSA

Il server DNS configurato sul client non è raggiungibile o non risponde alle query (es. servizio DNS non ancora avviato sul DC).

# SOLUZIONE

Verificato lo stato del servizio con Get-Service DNS e riavviato il servizio; ripetuto il test dopo l'avvio. **"DNS request timed out"**

# SINTOMO

La query DNS non riceve alcuna risposta entro il timeout previsto.

# CAUSA

Problema di connettività di rete tra client e server, oppure IP del DNS configurato in modo errato sul client.

# SOLUZIONE

Verificata la connettività con ping verso il DC e corretto l'indirizzo DNS del client con netsh interface ip set dns.

# 1 0. E R R O R I R E A L M E N T E R I S C O N T R A T I

# Diagnostica DNS (2/3) **"Server Unknown"**

# SINTOMO

nslookup non riesce nemmeno a identificare il server DNS interrogato.

# CAUSA

Il client non ha nessun server DNS valido configurato, oppure manca il record per la risoluzione inversa sul server.

# SOLUZIONE

Impostato esplicitamente il DNS del client su 192.168.134.130 e verificata la zona di ricerca inversa sul server. **Record DNS non registrato**

# SINTOMO

Resolve-DnsName o nslookup non trovano il record A del server o del client nella zona lab.local.

# CAUSA

Il record non è stato ancora creato o aggiornato, spesso perché il computer non ha eseguito la registrazione dinamica dopo un cambio di IP.

# SOLUZIONE

Eseguito ipconfig /registerdns sul computer interessato e verificata la comparsa del record in DNS Manager.

# 1 0. E R R O R I R E A L M E N T E R I S C O N T R A T I

# Diagnostica DNS (3/3) **DNS del client errato**

# SINTOMO

Il client risolve i nomi Internet ma non riesce a raggiungere lab.local o a completare il join al dominio.

# CAUSA

Il client stava ancora utilizzando un DNS pubblico (es. quello del router) invece del Domain Controller.

# SOLUZIONE

Riconfigurato il DNS del client con netsh interface ip set dns name="Ethernet0" static 192.168.134.130. **Nome breve vs FQDN**

# SINTOMO

Un comando funziona con SVR-LAB-01.lab.local ma fallisce con solo SVR-LAB-01, o viceversa.

# CAUSA

Il nome breve viene risolto tramite NetBIOS/broadcast locale, mentre il FQDN viene risolto sempre tramite DNS: i due meccanismi si comportano diversamente.

# SOLUZIONE

Preferito l'uso del FQDN (nome-host.lab.local) in tutti i test e nelle configurazioni, per affidarsi esclusivamente al DNS di dominio.

# 1 0. E R R O R I R E A L M E N T E R I S C O N T R A T I

# Strumento di risoluzione: ipconfig /registerdns

## ipconfig /registerdns

| Elemento | Descrizione |
|:--- |:--- |
| Cosa fa | Forza la registrazione (o l'aggiornamento) dei record DNS del computer presso il server DNS configurato, senza attendere il refresh automatico. |
| Quando usarlo | Dopo aver cambiato IP o nome del computer, o quando un record DNS risulta mancante/obsoleto durante i test di risoluzione. |

| Output atteso | Registration of the DNS resource records for all adapters of this computer has been initiated. Any errors will be reported in the Event Viewer in 15 minutes. |

**Nel laboratorio** Utilizzato per risolvere il problema 'record DNS non registrato' riscontrato durante la verifica della rete (sezione 9).

# Join del client al dominio

## Con rete e DNS correttamente configurati, il client Windows 11 può finalmente essere unito al dominio lab.local.

# 1 1. J O I N D E L C L I E N T A L D O M I N I O

# Procedura di join

## 1. Apertura delle proprietà del sistema

Sul client, eseguire sysdm.cpl (Start → Esegui) per aprire la finestra Proprietà del sistema, scheda 'Nome computer'.

## 2. Selezione del dominio

Cliccare su 'Cambia', selezionare l'opzione 'Dominio' e digitare lab.local.

## 3. Autenticazione

Al prompt delle credenziali, inserire un account con diritti di join, ad esempio LAB\Administrator oppure Administrator@lab.local.

## 4. Riavvio

Al termine, Windows richiede il riavvio del client per completare l'ingresso nel dominio e applicare la nuova identità di sicurezza.

# Verifica finale

## L'ultimo passo del laboratorio conferma che dominio, DNS e autenticazione funzionano correttamente end-to-end.

# 1 2. V E R I F I C A F I N A L E

# Checklist di verifica

## Login con account di dominio

Dalla schermata di accesso, selezionare 'Altro utente' e accedere con LAB\nomeutente per confermare che l'autenticazione di dominio funziona.

## Computer presente nel dominio

Su sysdm.cpl il client mostra ora 'Dominio: lab.local' invece di 'Gruppo di lavoro'.

## Connettività (ping)

ping SVR-LAB-01.lab.local dal client conferma la raggiungibilità del Domain Controller via FQDN.

## Risoluzione DNS

nslookup e Resolve-DnsName restituiscono correttamente gli indirizzi delle risorse di dominio.

## Active Directory Users and Computers

Sul server, la console ADUC mostra il nuovo oggetto computer CLI-W11-01 nel container Computers del dominio lab.local.

# 1 3. T U T T I I C O M A N D I U T I L I Z Z A T I

# Tabella riassuntiva dei comandi (1/2) **Comando** **Descrizione** **Quando usarlo**

Install-WindowsFeature AD-Domain-Services -IncludeManagementTools Installa il ruolo AD DS e gli strumenti di gestione Prima di promuovere il server a DC

| Get-Service ADWS,DNS | Verifica lo stato dei servizi AD e DNS | Dopo la promozione, per controllare i | servizi |
|:--- |:--- |:--- |:--- |
| Test-NetConnection localhost -Port 389 | Verifica la porta LDAP | Per diagnosticare problemi di | autenticazione |
| Test-NetConnection localhost -Port 53 | Verifica la porta DNS | Per diagnosticare problemi di | risoluzione nomi |
| Get-DnsClientServerAddress | Mostra il DNS configurato sul server | Per controllare che il DC punti a se | stesso |
| netsh interface ip set address... | Imposta l'IP statico sul client | Durante la configurazione di rete del | client |
| netsh interface ip set dns... | Imposta il DNS statico sul client | Durante la configurazione di rete del | client |

Test-NetConnection localhost -Port 389 Verifica la porta LDAP Per diagnosticare problemi di autenticazione

Test-NetConnection localhost -Port 53 Verifica la porta DNS Per diagnosticare problemi di risoluzione nomi

Get-DnsClientServerAddress Mostra il DNS configurato sul server Per controllare che il DC punti a se stesso

netsh interface ip set address... Imposta l'IP statico sul client Durante la configurazione di rete del client

netsh interface ip set dns... Imposta il DNS statico sul client Durante la configurazione di rete del client

# 1 3. T U T T I I C O M A N D I U T I L I Z Z A T I

# Tabella riassuntiva dei comandi (2/2)

| Comando | Descrizione | Quando usarlo |
|:--- |:--- |:--- |
| ipconfig /all | Mostra la configurazione IP completa | Per verificare la rete del client |
| ping 192.168.134.130 | Testa la connettività verso il DC | Per diagnosticare problemi di rete |
| nslookup lab.local | Risolve il nome del dominio | Per verificare la risoluzione DNS |
| Resolve-DnsName SVR-LAB-01.lab.local | Risolve il FQDN del server | Alternativa PowerShell a nslookup |

ipconfig /all Mostra la configurazione IP completa Per verificare la rete del client

ping 192.168.134.130 Testa la connettività verso il DC Per diagnosticare problemi di rete

nslookup lab.local Risolve il nome del dominio Per verificare la risoluzione DNS

Resolve-DnsName SVR-LAB-01.lab.local Risolve il FQDN del server Alternativa PowerShell a nslookup

ipconfig /registerdns Forza la registrazione DNS Quando un record DNS manca o è obsoleto

sysdm.cpl Apre le proprietà del sistema Per effettuare il join al dominio

# 1 4. E R R O R I C O M U N I

# Errori comuni (1/3)

| Errore | Sintomo | Causa | Soluzione |
|:--- |:--- |:--- |:--- |
| "Server failed" (nslookup) | Nessuna risoluzione del nome | DNS non raggiungibile o servizio fermo | Verificare/avviare il servizio DNS sul DC |
| "DNS request timed out" | Query DNS senza risposta | DNS client errato o rete non raggiungibile | Correggere il DNS del client e testare con ping |
| "Server Unknown" | DNS non identificato | Nessun DNS valido configurato | Impostare esplicitamente il DNS sul client |
| Record DNS mancante | Nome non risolvibile | Registrazione dinamica non avvenuta | Eseguire ipconfig /registerdns |

"Server failed" (nslookup) Nessuna risoluzione del nome DNS non raggiungibile o servizio fermo Verificare/avviare il servizio DNS sul DC

"DNS request timed out" Query DNS senza risposta DNS client errato o rete non raggiungibile Correggere il DNS del client e testare con ping

"Server Unknown" DNS non identificato Nessun DNS valido configurato Impostare esplicitamente il DNS sul client

Record DNS mancante Nome non risolvibile Registrazione dinamica non avvenuta Eseguire ipconfig /registerdns

# 1 4. E R R O R I C O M U N I

# Errori comuni (2/3)

**Errore** **Sintomo** **Causa** **Soluzione**

Join fallito: dominio non contattabile Errore durante l'inserimento di lab.local DNS del client non punta al DC Correggere il DNS con netsh e ritestare

Join fallito: credenziali rifiutate Autenticazione negata Formato utente errato (manca LAB\) Usare LAB\Administrator o Administrator@lab.local

Promozione a DC fallita Wizard non completa la configurazione Ruolo AD DS non ancora installato Installare AD-Domain-Services prima della promozione

Ping fallito verso il DC Richiesta scaduta Firewall blocca ICMP o IP errato Verificare regole firewall e configurazione IP

# 1 4. E R R O R I C O M U N I

# Errori comuni (3/3)

**Errore** **Sintomo** **Causa** **Soluzione**

Login di dominio negato Accesso rifiutato Formato account errato o account bloccato Usare dominio\utente e verificare l'account in ADUC

Computer assente in ADUC dopo il join Oggetto computer non visibile Mancato riavvio del client dopo il join Riavviare il client e ricontrollare ADUC

Test-NetConnection: TcpTestSucceeded False Porta 389/53 non raggiungibile Servizio non avviato o firewall attivo Avviare il servizio e controllare il firewall

Nome breve non risolto (FQDN sì) Comando funziona solo con FQDN Risoluzione NetBIOS non disponibile Usare sempre il FQDN nei test e nelle configurazioni

# 1 5. C O S A D E V E R I C O R D A R E U N O S T U D E N T E

# 15 punti da ricordare

Un dominio centralizza autenticazione e gestione di utenti e computer.

Un Domain Controller ospita il database di Active Directory.

Il ruolo AD DS va installato prima di poter promuovere il server.

La promozione crea la foresta, installa DNS e configura SYSVOL.

Il DC deve sempre usare se stesso come server DNS.

Un client deve puntare al DNS del dominio, mai a un DNS pubblico.

IP statici (server e client) evitano instabilità nella risoluzione dei nomi.

LDAP (389) e DNS (53) sono le porte critiche da verificare sempre.

nslookup e Resolve-DnsName sono gli strumenti principali di diagnosi DNS.

ipconfig /registerdns forza la registrazione dei record DNS mancanti.

Il join richiede credenziali nel formato DOMINIO\utente o utente@dominio.

Dopo il join è sempre necessario riavviare il client.

FQDN e nome breve si risolvono con meccanismi diversi: preferire il FQDN.

ADUC è la console per verificare gli oggetti del dominio.

La diagnostica sistematica (rete → DNS → autenticazione) risolve la maggior parte dei problemi.

# 1 6. Q U I Z F I N A L E

# Domande 1-2 **Domanda 1**

**Cosa distingue principalmente un dominio da un workgroup?**

**A.** Il dominio richiede internet, il workgroup no

**B.** Il dominio ha un'autenticazione centralizzata, il workgroup no

**C.** Il workgroup supporta più utenti del dominio **D.** Non c'è nessuna differenza pratica **Domanda 2**

**Quale comando installa il ruolo Active Directory Domain Services?** **A.** Add-ADDSForest

**B.** Install-WindowsFeature AD-Domain-Services -IncludeManagementTools **C.** netsh interface ip set address **D.** Get-Service ADWS,DNS

# 1 6. Q U I Z F I N A L E

# Domande 3-4 **Domanda 3**

**Perché un Domain Controller deve usare se stesso come server DNS?** **A.** Per motivi di licenza Microsoft

**B.** Perché ospita la zona DNS con i record del dominio, necessari a localizzare i servizi AD

**C.** Perché altrimenti internet non funziona

**D.** Non è necessario, è solo una best practice opzionale **Domanda 4**

**Quale porta viene usata per verificare il servizio LDAP?**

# A.

# B.

# C.

# D.

# 1 6. Q U I Z F I N A L E

# Domande 5-6 **Domanda 5**

**Cosa indica un output "TcpTestSucceeded: False"?** **A.** Il test è stato eseguito con successo

**B.** La porta non è raggiungibile o il servizio non è attivo **C.** Il comando non esiste **D.** La rete non ha alcun problema **Domanda 6**

**Quale comando forza la registrazione dei record DNS di un computer?** **A.** ipconfig /flushdns **B.** ipconfig /registerdns **C.** nslookup /register **D.** netsh dns register

# 1 6. Q U I Z F I N A L E

# Domande 7-8 **Domanda 7**

**Qual è il formato corretto delle credenziali per il join al dominio?** **A.** lab.local\utente

**B.** LAB\Administrator oppure Administrator@lab.local **C.** SVR-LAB-01\Administrator **D.** Administrator (senza dominio) **Domanda 8**

**Dopo il join di un client al dominio, cosa è necessario fare?** **A.** Nulla, è tutto immediato **B.** Reinstallare Windows **C.** Riavviare il computer **D.** Disattivare il firewall

# 1 6. Q U I Z F I N A L E

# Domande 9-10 **Domanda 9**

**Quale strumento verifica, lato server, che un computer sia entrato nel ** **dominio?** **A.** Task Manager

**B.** Active Directory Users and Computers (ADUC) **C.** Disk Management **D.** Solo il Visualizzatore eventi **Domanda 10**

**Qual è la differenza tra nome breve e FQDN?**

**A.** Sono identici, cambia solo la lunghezza

**B.** Il nome breve usa DNS, il FQDN usa NetBIOS

**C.** Il nome breve si risolve spesso via NetBIOS, il FQDN sempre via DNS **D.** Il FQDN funziona solo su Windows 11

# 1 6. Q U I Z F I N A L E

# Soluzioni (per il docente)

| Q1 | B |
|:--- |:--- |
| Q2 | B |
| Q3 | B |
| Q4 | C |
| Q5 | B |
| Q6 | B |
| Q7 | B |
| Q8 | C |
| Q9 | B |
| Q10 | C |

# Q2 **B**

# Q3 **B**

# Q4 **C**

# Q5 **B**

# Q6 **B**

# Q7 **B**

# Q8 **C**

# Q9 **B**

# Q10 **C**

# Conclusione

## Un bilancio di ciò che è stato costruito, passo dopo passo, in questo primo laboratorio del corso.

# 1 7. C O N C L U S I O N E

# Cosa è stato costruito

## Un dominio Active Directory funzionante

lab.local, ospitato su un Domain Controller Windows Server 2022 (SVR-LAB-01), completo di database AD e servizio DNS integrato.

## Un client unito al dominio

Windows 11 (CLI-W11-01) correttamente configurato in rete, unito al dominio e in grado di autenticare utenti di dominio.

## Un percorso di diagnostica completo

Verifica di rete, DNS, servizi e autenticazione basata su problemi realmente incontrati e risolti durante il laboratorio.

# Grazie.

Laboratorio 01 completato - Domain Controller e Join al dominio lab.local

*Corso Windows Server 2022  ·  Dispensa ufficiale del corso*
