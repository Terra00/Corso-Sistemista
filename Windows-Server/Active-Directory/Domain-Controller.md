# MODULO 3.a - ACTIVE DIRECTORY **Il Domain Controller: da server a dominio**

## Promozione di SVR-LAB-01, DNS del dominio lab.local e ingresso di CLI-W11-01 nel dominio

Dominio e DC Promozione DNS del dominio Join del client

# Obiettivi e prerequisiti

Dal server in gruppo di lavoro al primo dominio del corso

## Alla fine saprete - Spiegare dominio e Domain Controller - Installare il ruolo e promuovere il server - Configurare il DNS del dominio - Unire il client al dominio - Leggere gli errori DNS più comuni

## Prerequisiti

- Modulo 2 completato: IP statici.130 e.129, DNS.2

- CLI-W11-01 con Windows 11 Pro (l'edizione Home non può unirsi a un dominio) - Accesso come amministratore locale - Console VMware aperta (ci saranno riavvii)

**Nota:** questo è il passaggio più delicato del corso. Tutti i moduli successivi usano il dominio lab.local creato oggi.

# Il percorso

Tre blocchi, dalla teoria al primo accesso di dominio

| A   | Dominio e Domain Controller       | 12′ |
| :-- | :-------------------------------- | :-- | ------------------------------------------ |
| B   | La promozione del server          | 30′ |
| C   | DNS del dominio e join del client | 32′ | **B** **La promozione del server** **30′** |

**C** **DNS del dominio e join del client** **32′**

**Chiusura:** errori DNS, errori comuni, laboratorio «trova il guasto» e quiz (circa 25 minuti).

# Lo stato del laboratorio

## Da dove partiamo e dove arriviamo

| Momento | SVR-LAB-01 | CLI-W11-01 |
| :------ | :--------- | :--------- |

## Gruppo di lavoro - Ogni computer ha i propri account locali - Nessun database condiviso

- Le credenziali vanno create su ogni macchina - Nessun criterio centralizzato

## Dominio

- Un solo account per i computer membri (dove è autorizzato)

- Autenticazione centralizzata (Active Directory)

- Criteri di gruppo gestiti centralmente, con ambito e filtri (Modulo 6.c) - Gestione da un'unica console

**Nota:** fino a ora SVR-LAB-01 e CLI-W11-01 erano in un gruppo di lavoro. Oggi diventano un dominio.

# Che cos'è un Domain Controller

Il dominio è una struttura logica; il DC è il server che la fa funzionare

## Dominio lab.local

struttura logica: utenti, gruppi, computer, criteri di sicurezza

## Domain Controller SVR-LAB-01

server che ospita il database di AD (NTDS.dit) e autentica utenti e computer

**Nota:** un dominio può avere più Domain Controller, ognuno con una copia del database. Nel laboratorio ce n'è uno solo.

**Attenzione:** il DC non è un «livello» del dominio (come le OU): è il server che lo ospita.

# Che cosa fa la promozione

Installare il ruolo non basta: la promozione crea il dominio

| Foresta e dominio                                                                                                                           | crea la foresta (l'insieme dei domini: qui uno solo), il dominio lab.local e NTDS.dit |
| ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Server DNS                                                                                                                                  | installa il DNS con la zona lab.local e i record SRV                                  |
| SYSVOL                                                                                                                                      | cartella condivisa con criteri e script (Moduli 4 e 6)                                |
| Kerberos                                                                                                                                    | avvia il centro distribuzione chiavi (KDC) per l'autenticazione                       |
| Nota: i computer trovano i Domain Controller chiedendo al DNS i record SRV (record che indicano quale server offre un servizio). Senza DNS, | niente dominio.                                                                       |

**Foresta e dominio** crea la foresta (l'insieme dei domini: qui uno solo), il dominio lab.local e NTDS.dit

**Server DNS** installa il DNS con la zona lab.local e i record SRV

**SYSVOL** cartella condivisa con criteri e script (Moduli 4 e 6)

**Kerberos** avvia il centro distribuzione chiavi (KDC) per l'autenticazione

**Nota:** i computer trovano i Domain Controller chiedendo al DNS i record SRV (record che indicano quale server offre un servizio). Senza DNS, niente dominio.

# Il nome del dominio Tre nomi da non confondere

| Nome                                                                                                                                           | Che cos'Ã¨                                            |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ---------------------- |
| Nome                                                                                                                                           | Che cos'è                                             |
| lab.local                                                                                                                                      | nome DNS del dominio Active Directory                 |
| LAB                                                                                                                                            | nome NetBIOS (breve): si usa in LAB\Administrator     |
| svr-lab-01.lab.local                                                                                                                           | FQDN del server: nome del computer + nome del dominio |
| Attenzione: il nome del dominio si sceglie durante la promozione e non si cambia facilmente. lab.local è una scelta del laboratorio: Microsoft | sconsiglia.local.                                     | **Nome** **Che cos'è** |

**Attenzione:** il nome del dominio si sceglie durante la promozione e non si cambia facilmente. lab.local è una scelta del laboratorio: Microsoft sconsiglia.local.

# B

# La promozione del server Ruolo, promozione, riavvio, verifiche **30 minuti** 10 / 31

**PASSO 1** **Lo stato iniziale** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 11 / 31 **Si esegue su: SVR-LAB-01**

```text
C:> hostname
SVR-LAB-01
```

```text
C:> ipconfig /all
DHCP abilitato....: No
Indirizzo IPv4....: 192.168.134.130(Preferenziale)
Gateway predefinito.: 192.168.134.2
Server DNS......: 192.168.134.2
```

## Attenzione: il nome del computer va deciso prima della promozione. Se non è SVR-LAB-01, cambiatelo ora (SConfig, voce nome computer) e riavviate.

## Nota: la rete è quella del Modulo 2. Qui si verifica soltanto.

**PASSO 2** **Installare il ruolo AD DS** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 12 / 31 **Si esegue su: SVR-LAB-01**

PS C:> Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Success Restart Needed Exit Code Feature Result ------- -------------- ---------------------True No Success {Servizi di dominio Active Directory,...}

| Campo                   | Significato                                            |
| :---------------------- | :----------------------------------------------------- | ---------------------------------------- |
| Success: True           | installazione riuscita                                 |
| Restart Needed: No      | non serve riavviare ora                                |
| -IncludeManagementTools | installa anche le console e il modulo PowerShell di AD | **Success: True** installazione riuscita |

**Attenzione:** il ruolo installa i componenti, ma il server non è ancora un Domain Controller.

**PASSO 3** **Promuovere con PowerShell** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 13 / 31 **Si esegue su: SVR-LAB-01**

PS C:> Install-ADDSForest -DomainName "lab.local" \` >> -DomainNetbiosName "LAB" -InstallDns

SafeModeAdministratorPassword: ****\***** Confirm SafeModeAdministratorPassword: ****\*****

# AVVISO:...

| Parametro          | Significato                                                                 |
| :----------------- | :-------------------------------------------------------------------------- |
| -DomainName        | nome DNS della nuova foresta e del dominio                                  |
| -DomainNetbiosName | nome breve LAB                                                              |
| -InstallDns        | installa il server DNS insieme al dominio                                   |
| Password DSRM      | modalità di ripristino dei servizi directory (in aula: P@ssw0rd!, fittizia) |

**Nota:** due avvisi sono attesi (delega DNS, impostazioni di sicurezza predefinite) e si prosegue; un errore invece ferma il comando.

**PASSO 3 bis** **In alternativa: la procedura guidata** **Console · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 14 / 31 **Si esegue su: SVR-LAB-01** **Server Manager**

**› bandierina delle notifiche → Promuovi il server a controller di dominio** **› Configurazione distribuzione: Aggiungi una nuova foresta → nome dominio radice: lab.local** **› Opzioni controller di dominio: lasciare DNS e catalogo globale (indice degli oggetti della foresta); password DSRM** **› Opzioni DNS: avviso sulla delega DNS → normale, si prosegue** **› Opzioni aggiuntive: nome NetBIOS LAB** **› Percorsi e Revisione opzioni: valori predefiniti** **› Controllo dei prerequisiti → Installa (il server si riavvia)**

**Nota:** stessa operazione del PASSO 3. Scegliete una sola strada.
| Elemento | Descrizione |
| --- | --- |

# Dopo il riavvio Il server è diventato un Domain Controller

| Elemento        | Descrizione                                                              |
| --------------- | ------------------------------------------------------------------------ |
| Riavvio         | automatico; il primo avvio da DC richiede qualche minuto in piÃ¹         |
| Accesso         | LAB\Administrator, con la password che aveva l'Administrator locale      |
| Che cosa cambia | sul DC non si usano piÃ¹ account locali: valgono gli account del dominio |

**Riavvio** automatico; il primo avvio da DC richiede qualche minuto in più

**Accesso** LAB\Administrator, con la password che aveva l'Administrator locale

**Che cosa cambia** sul DC non si usano più account locali: valgono gli account del dominio

**Suggerimento:** in GoTo chiedete a tutti di scrivere «fatto» in chat quando vedono di nuovo il desktop del server.

**PASSO 4** **I servizi del Domain Controller** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 16 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-Service ADWS, DNS, NTDS, Kdc, Netlogon
```

```text
Status   Name      DisplayName
------   ----      ----------
Running  ADWS      Servizi Web Active Directory
Running  DNS       Server DNS
Running  Kdc       Centro distribuzione chiavi Kerberos
Running  Netlogon  Accesso rete
Running  NTDS      Servizi di dominio Active Directory
```

| Servizio       | A che cosa serve                                                        |
| :------------- | :---------------------------------------------------------------------- |
| NTDS           | il database di Active Directory                                         |
| Kdc · Netlogon | autenticazione Kerberos · ricerca del DC e registrazione dei record DNS |
| ADWS           | strumenti di amministrazione: lo usano i cmdlet di AD (modulo 3.b)      |

**PASSO 5** **Il DNS risponde davvero?** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 17 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Resolve-DnsName _ldap._tcp.dc._msdcs.lab.local -Type SRV -Server 127.0.0.1
```

```text
Name                            Type TTL Section NameTarget            Port
----                            ---- --- ------- --------            ----
_ldap._tcp.dc._msdcs.lab.local  SRV  600 Answer  svr-lab-01.lab.local  389
```

## Risultato: il DNS del DC conosce il dominio e il record SRV che dice ai client dove trovare il Domain Controller (LDAP, porta 389).

## Attenzione: Test-NetConnection -Port 53 o -Port 389 prova solo che la porta TCP risponde. Per il DNS serve una query vera, come questa.

# C

# DNS del dominio e join del client

Il DNS passa da.2 a.130, poi il client entra nel dominio **32 minuti** 18 / 31

**PASSO 6** **Il DNS del server: da.2 a.130** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 19 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-DnsClientServerAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4
ServerAddresses: {127.0.0.1, 192.168.134.2}
PS C:> Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" `
  -ServerAddresses 192.168.134.130
PS C:> Get-DnsServerForwarder
IPAddress: {192.168.134.2}
PS C:> Resolve-DnsName learn.microsoft.com -Type A
```

```text
Nome canonico    A     20   Answer   (indirizzo IP)
```

| Che cosa           | Perché                                                            |
| :----------------- | :---------------------------------------------------------------- |
| DNS client =.130   | il primo e unico DC con DNS usa se stesso                         |
| Server d'inoltro.2 | i nomi esterni passano dal DNS di VMware: lo prova l'ultima query |

## DNS client =.130 il primo e unico DC con DNS usa se stesso

## Server d'inoltro.2

i nomi esterni passano dal DNS di VMware: lo prova l'ultima query

**Nota:** se il server d'inoltro manca, si aggiunge con Add-DnsServerForwarder -IPAddress 192.168.134.2.

**PASSO 7** **La zona di ricerca inversa** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 20 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Add-DnsServerPrimaryZone -NetworkId "192.168.134.0/24" -ReplicationScope Domain
PS C:> ipconfig /registerdns
```

```text
C:> nslookup 192.168.134.130
Server:  svr-lab-01.lab.local
Address:  192.168.134.130

Nome:    svr-lab-01.lab.local
Address:  192.168.134.130
```

**Si esegue su: CLI-W11-01 (come amministratore)**

```powershell
PS C:> Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.134.130
PS C:> (Get-NetIPConfiguration -InterfaceAlias "Ethernet0").DNSServer
ServerAddresses: {192.168.134.130}
```

**Nota:** l'indirizzo.129 resta quello del Modulo 2. L'IP statico del client è una scelta del laboratorio: un client con DHCP entra nel dominio se riceve il DNS giusto.

**Attenzione:** un client che usa un DNS pubblico o.2 non trova il dominio lab.local.

**PASSO 9** **Il client trova il dominio?** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 22 / 31 **Si esegue su: CLI-W11-01**

```text
C:> nslookup lab.local
Server:  svr-lab-01.lab.local
Address:  192.168.134.130

Nome:    lab.local
Address:  192.168.134.130
```

| Name                   | Type TTL Section IPAddress  |
| :--------------------- | :-------------------------- |
| svr-lab-01.lab.local A | 1200 Answer 192.168.134.130 |

## Nota: usiamo il nome completo (FQDN). Il nome breve svr-lab-01 lo proveremo dopo il join, quando il client avrà il suffisso DNS lab.local.

**PASSO 10** **Unire il client al dominio** **Output atteso · da collaudare**

**Si esegue su: CLI-W11-01 (come amministratore)**

PS C:> Add-Computer -DomainName "lab.local" -Credential LAB\Administrator -Restart **In alternativa: dalla console**

sysdm.cpl → Nome computer → Cambia **› Membro di: Dominio → lab.local** **› credenziali LAB\Administrator** **› conferma → riavvio**

**Nota:** si apre la richiesta della password di LAB\Administrator, poi il client si riavvia. L'account computer CLI-W11-01 viene creato in Active Directory durante il join. Il riavvio completa la configurazione del client, non crea l'account.

**PASSO 11** **Il computer nel dominio** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 24 / 31 **Si esegue su: SVR-LAB-01**

```powershell
PS C:> Get-ADComputer CLI-W11-01
```

```text
DNSHostName   : CLI-W11-01.lab.local
Enabled       : True
SamAccountName: CLI-W11-01$
```

## Nota: il client è nel contenitore predefinito Computers. A un contenitore non si collegano GPO: per questo nel Modulo 6.c la GPO dei computer si collegherà al dominio.

**PASSO 12** **Il primo accesso di dominio** **Output atteso · da collaudare**

Modulo 3.a · Domain Controller · Windows Server 2022 25 / 31 **Si esegue su: CLI-W11-01**

```text
C:> whoami
lab\administrator
```

```powershell
PS C:> Resolve-DnsName svr-lab-01
```

```text
Nome       Type TTL Section IPAddress
----       ---- --- ------- ---------
svr-lab-01 A    1200 Answer  192.168.134.130
svr-lab-01.lab.local A 1200 Answer  192.168.134.130
```

## Risultato: il client autentica un account del dominio e ora risolve anche il nome breve, grazie al suffisso lab.local.

| Messaggio             | Ipotesi più probabile                       | Che cosa verificare                    |
| :-------------------- | :------------------------------------------ | :------------------------------------- |
| DNS request timed out | il server DNS non risponde                  | rete, indirizzo del DNS, servizio DNS  |
| Server failure        | il server ha risposto con un errore interno | zona e servizio sul DC                 |
| Non-existent domain   | il nome non esiste nella zona               | nome scritto, registrazione del record |

**Server: UnKnown** il DNS non trova un nome per il proprio indirizzo (ricerca inversa) zona inversa e PTR (PASSO 7)

**FQDN sì, nome breve no** suffisso DNS assente o diverso suffisso e lista dei suffissi (ipconfig /all)

# Errori comuni

Quelli corretti rispetto al deck precedente, e altri frequenti

**DNS pubblico su un DC** i record di Active Directory non vengono registrati

**Test-NetConnection -Port 53 = DNS funziona** prova solo la porta: serve una query

**Computer assente in ADUC = «non ha riavviato»** controllate esito del join, nome, contenitore

**IP statico del client obbligatorio** è una scelta del laboratorio

**Nome breve = NetBIOS** spesso lo risolve il DNS con il suffisso

**Mario o Luca in RDP sul DC** di base solo il gruppo Administrators

# Riferimento rapido

## Da consultare durante il laboratorio

| Comando                                    | Uso                    | Dove     |
| :----------------------------------------- | :--------------------- | :------- |
| Install-WindowsFeature AD-Domain-Services  | installa il ruolo      | server   |
| Install-ADDSForest -DomainName -InstallDns | promuove a DC          | server   |
| Get-Service ADWS, DNS, NTDS, Kdc, Netlogon | servizi del DC         | server   |
| Resolve-DnsName... -Type SRV -Server       | query DNS vera         | entrambe |
| Set-DnsClientServerAddress                 | DNS.130                | entrambe |
| Add-DnsServerPrimaryZone -NetworkId        | zona inversa           | server   |
| nslookup                                   | risoluzione e messaggi | entrambe |
| Add-Computer -DomainName -Restart          | join al dominio        | client   |
| Get-ADComputer                             | computer nel dominio   | server   |
| whoami                                     | chi sono               | client   |

# Laboratorio finale · 12 minuti

| Verifica | Che cosa verificare |
| -------- | ------------------- |

Trova il guasto: il client non trova più il dominio

| Verifica       | Che cosa verificare                                                 |
| -------------- | ------------------------------------------------------------------- |
| Dove si ferma? | nslookup lab.local dal client: quale messaggio compare?             |
| Configurazione | ipconfig /all sul client: quale DNS usa?                            |
| Server         | Get-Service DNS sul DC; Resolve-DnsName lab.local -Server 127.0.0.1 |
| Prova finale   | dopo la correzione: nslookup lab.local dal client deve rispondere   |
| In chat        | la causa e il comando che la dimostra                               |

**Nota:** il guasto lo introduce il formatore. Usate la tabella «Leggere gli errori DNS».

# Quiz di verifica

## Rispondete in chat con il numero della domanda e la lettera

Che cosa fa Install-WindowsFeature AD-Domain-Services? A) crea il dominio B) installa il ruolo, senza creare il dominio C) unisce il server a un dominio

Quale DNS usa SVR-LAB-01 dopo la promozione? A) 192.168.134.130 B) 192.168.134.2 C) 1.1.1.1

nslookup risponde «Server failure». Significa: A) server DNS spento B) il nome non esiste C) il server ha risposto con un errore interno

Il FQDN si risolve, il nome breve no. Che cosa controlli? A) il suffisso DNS del client B) NetBIOS sul server C) il gateway

Quando viene creato l'account computer di CLI-W11-01? A) al riavvio del client B) durante il join C) alla promozione del DC

Test-NetConnection -Port 53 restituisce True. Che cosa sai? A) che il DNS risolve lab.local B) che la porta TCP 53 risponde C) che il client è nel dominio

# Correzione e ponte verso il Modulo 3.b

## Risposta corretta e perché

Modulo 3.a · Domain Controller · Windows Server 2022 31 / 31 **N.** **Risposta** **Perché**

| B   | il ruolo installa i componenti; il dominio nasce con la promozione |
| :-- | :----------------------------------------------------------------- | ------------------------------------------- |
| A   | il primo e unico DC con DNS usa se stesso                          |
| C   | Server failure è un errore restituito dal server, non un timeout   |
| A   | senza suffisso DNS il nome breve non viene completato              |
| B   | l'account nasce durante il join; il riavvio completa il client     |
| B   | è solo un test TCP: per il DNS serve una query                     | A il primo e unico DC con DNS usa se stesso |

C Server failure è un errore restituito dal server, non un timeout

A senza suffisso DNS il nome breve non viene completato

B l'account nasce durante il join; il riavvio completa il client

B è solo un test TCP: per il DNS serve una query

**Modulo 3.b:** OU, utenti (Mario e Luca), gruppi, delega, foreste e trust. Si parte dal dominio creato oggi.
