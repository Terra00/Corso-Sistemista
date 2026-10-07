# MODULO 2 - NETWORKING **Configurare la rete e il Desktop remoto**

## Da indirizzo automatico a indirizzo statico, dalla prova di rete alla connessione RDP reale tra CLI-W11-01 e SVR-LAB-01

Rete IP Indirizzo statico Raggiungibilità e DNS Desktop remoto

# Obiettivi e prerequisiti

Il primo modulo dopo l'installazione: le macchine devono parlarsi prima di diventare un dominio

## Alla fine saprete - Spiegare indirizzo, prefisso, gateway e DNS - Passare da indirizzo automatico a statico - Verificare raggiungibilità, porte e nomi

- Abilitare il Desktop remoto e collegarvi dal client - Trovare il guasto livello per livello

## Prerequisiti

- SVR-LAB-01 e CLI-W11-01 installati (prime 6 ore) - Rete VMware predefinita, senza modifiche

- Accesso come amministratore locale su entrambe - Console VMware a portata di mano

| A | Come funziona una rete IP | 14′ |
|:--- |:--- |:--- |
| B | Leggere e configurare l'indirizzo | 30′ |
| C | Raggiungibilità e DNS | 22′ |
| D | Desktop remoto: abilitare e collegarsi | 28′ |

**B** **Leggere e configurare l'indirizzo** **30′** **C** **Raggiungibilità e DNS** **22′**

**D** **Desktop remoto: abilitare e collegarsi** **28′**

**Chiusura:** troubleshooting a livelli, ponte verso il Modulo 3, laboratorio «trova il guasto» e quiz (circa 20 minuti).

# Lo stato del laboratorio

## Da dove partiamo, dove arriviamo, che cosa resta al Modulo 3

Modulo 2 · Networking · Windows Server 2022 4 / 38 **Momento** **SVR-LAB-01** **CLI-W11-01**

**Inizio del modulo** indirizzo, gateway e DNS dal DHCP di VMware · RDP disabilitato indirizzo, gateway e DNS dal DHCP di VMware

**Fine del modulo** 192.168.134.130/24 · gateway.2 · DNS.2 · RDP abilitato 192.168.134.129/24 · gateway.2 · DNS.2

**Modulo 3 (dopo la promozione)** DNS.130 (se stesso) · Domain Controller di lab.local DNS.130 · unito al dominio

**Attenzione:** il DNS.2 è temporaneo. Su.130 un server DNS esisterà solo dopo la promozione a Domain Controller (Modulo 3).

**Regola d'aula:** sulla rete NAT di VMware (rete predefinita) restano accese solo SVR-LAB-01 e CLI-W11-01. Il.2 è il dispositivo NAT di VMware.

# A

# Come funziona una rete IP

Indirizzo, prefisso, gateway, DNS: i quattro valori che ogni macchina deve avere **14 minuti** 5 / 38

# Indirizzo IP e prefisso

Ogni macchina ha un indirizzo; il prefisso dice quale parte indica la rete

```text
192.168.134   .130
^^^^^^^^^^   ^^^^^
parte di RETE  parte di HOST
(uguale per    (la macchina)
 tutta la rete)
```

| Valore | Significato |
| --- | --- |
| /24 | prefisso: i primi 24 bit (bit = cifra binaria 0 o 1), cioÃ¨ i primi tre numeri, indicano la rete |
| 255.255.255.0 | subnet mask: lo stesso prefisso /24 scritto in un altro modo |
| .129 e .130 | stessa rete 192.168.134: client e server si parlano direttamente |

# 192.168.134 **.130**

**parte di RETE (uguale per tutta la rete)** **parte di HOST (la macchina)** **Valore** **Significato**

**/24** prefisso: i primi 24 bit (bit = cifra binaria 0 o 1), cioè i primi tre numeri, indicano la rete

**255.255.255.0** subnet mask: lo stesso prefisso /24 scritto in un altro modo

**.129 e.130** stessa rete 192.168.134: client e server si parlano direttamente

# Il gateway

Dentro la rete si comunica direttamente; per uscire serve il gateway

# CLI-W11-01 192.168.134.129

# SVR-LAB-01 192.168.134.130

## Gateway.2 dispositivo NAT di VMware

## Altre reti rete del PC fisico, Internet

**Nota:** il gateway predefinito è l'indirizzo a cui una macchina invia tutto ciò che non appartiene alla propria rete.

# Il DNS: dal nome all'indirizzo

Le persone usano i nomi, le macchine gli indirizzi: il DNS traduce

## Record A

Nome → indirizzo. Esempio: svr-lab-01.lab.local → 192.168.134.130

## Record PTR

Indirizzo → nome (ricerca inversa). Serve una zona di ricerca inversa sul server DNS.

| Fase | DNS da usare | Perché |
|:--- |:--- |:--- |
| Modulo 2 (ora) | 192.168.134.2 | su.130 non c'è ancora un server DNS |
| Modulo 3, dopo la promozione | 192.168.134.130 | il primo DC con DNS usa se stesso |
| Su un Domain Controller | mai DNS pubblici | i record di AD non verrebbero registrati |

# DHCP o indirizzo statico

Chi assegna i quattro valori, e perché nel corso il server ha un indirizzo statico

Modulo 2 · Networking · Windows Server 2022 9 / 38 **DHCP (automatico)** **Statico (manuale)**

| Chi assegna | il server DHCP (qui quello di VMware) | l'amministratore, sulla scheda di rete |
|:--- |:--- |:--- |
| L'indirizzo può cambiare? | sì, alla scadenza del lease | no |
| Nel corso | punto di partenza delle due VM | server e client dal Modulo 2 (scelta del laboratorio) |

**Attenzione:** il DHCP di VMware assegna indirizzi da.128 in su. Regola d'aula: sulla rete NAT solo SVR-LAB-01 e CLI-W11-01; prima del PASSO 4,.129 e.130 non devono essere in uso.

**Nota:** lease = periodo per cui il DHCP «presta» un indirizzo alla macchina.

# Porte e servizi in ascolto

Collegamento: serve per capire il Desktop remoto e i test sulle porte

## 1 · Host raggiungibile la macchina risponde in rete (ping)

## 2 · Porta aperta una porta TCP accetta connessioni (Test- NetConnection -Port)

## 3 · Servizio usabile il servizio risponde e vi lascia entrare (connessione reale)

**Nota:** un servizio «ascolta» su una porta numerata. Il Desktop remoto usa la porta TCP 3389.

**Attenzione:** ogni livello dimostra solo se stesso. Una porta aperta non prova che potrete accedere.

# B

# Leggere e configurare l'indirizzo

Prima si legge lo stato di partenza, poi lo si cambia, poi si verifica **30 minuti** 11 / 38

**PASSO 1** **Lo stato di partenza: ipconfig /all** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 12 / 38 **Si esegue su: SVR-LAB-01** C:> ipconfig /all

Scheda Ethernet Ethernet0: DHCP abilitato............: Sì Indirizzo IPv4............: 192.168.134.128(Preferenziale) Subnet mask.............: 255.255.255.0 Gateway predefinito.........: 192.168.134.2 Server DHCP.............: 192.168.134.254 Server DNS..............: 192.168.134.2 **Campo** **Che cosa ci dice**

**DHCP abilitato: Sì** l'indirizzo è assegnato in automatico

**Indirizzo IPv4** l'indirizzo attuale (qui uno qualsiasi dato dal DHCP)

**Gateway / Server DNS** entrambi.2: il dispositivo NAT di VMware **Server DHCP** chi ha assegnato l'indirizzo

**PASSO 2** **Come si chiama la scheda: Get-NetAdapter** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 13 / 38 **Si esegue su: SVR-LAB-01** PS C:> Get-NetAdapter

```text
Name  InterfaceDescription  ifIndex Status  LinkSpeed
----  --------------------  ------- ------  ---------
Ethernet0  Intel(R) 82574L Gigabit Network Conn  4 Up  1 Gbps
```

| Campo | Significato |
|:--- |:--- |
| Name | nome della scheda (InterfaceAlias): lo userete in tutti i comandi successivi |
| ifIndex | numero della scheda: alternativa al nome |
| Status Up | la scheda è collegata alla rete virtuale |

**Suggerimento:** se il nome non è Ethernet0, sostituitelo in tutti i comandi dei passi seguenti.

**PASSO 3** **Leggere con PowerShell** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 14 / 38 **Si esegue su: SVR-LAB-01**

## Riepilogo: Get-NetIPConfiguration PS C:> Get-NetIPConfiguration

| InterfaceAlias |: Ethernet0 |
|:--- |:--- |
| InterfaceIndex |: 4 |
| IPv4Address |: 192.168.134.128 |
| IPv4DefaultGateway: 192.168.134.2 |  |
| DNSServer |: 192.168.134.2 |

## Solo indirizzi: Get-NetIPAddress

PS C:> Get-NetIPAddress \` >> -InterfaceAlias Ethernet0 -AddressFamily IPv4

| IPAddress |: 192.168.134.128 |
|:--- |:--- |
| PrefixLength |: 24 |
| PrefixOrigin |: Dhcp |
| AddressState |: Preferred |

## Attenzione: Get-NetIPAddress non è «l'ipconfig di PowerShell»: mostra solo gli indirizzi, non gateway e DNS.

## Nota: PrefixOrigin Dhcp conferma che l'indirizzo arriva dal DHCP.

**PASSO 4** **IP statico sul server: New-NetIPAddress** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 15 / 38 **Si esegue su: SVR-LAB-01 (console VMware)**

PS C:> New-NetIPAddress -InterfaceAlias "Ethernet0" \` >> -IPAddress 192.168.134.130 -PrefixLength 24 ` >> -DefaultGateway 192.168.134.2

| IPAddress |: 192.168.134.130 |
|:--- |:--- |
| InterfaceAlias: Ethernet0 |  |
| PrefixLength |: 24 |
| PrefixOrigin |: Manual |
| AddressState |: Tentative |
| PolicyStore |: ActiveStore |

## Nota: il comando restituisce l'oggetto dell'indirizzo creato. Se il DHCP era attivo sulla scheda, lo disattiva.

## Attenzione: la connessione può cadere per qualche secondo.

Lavorate dalla console VMware, mai da una sessione remota.

**PASSO 4 bis** **Se il PASSO 4 dà errore** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 16 / 38 **Si esegue su: SVR-LAB-01 (console VMware)**

PS C:> Get-NetIPConfiguration -InterfaceAlias "Ethernet0" **Che cosa vedete** **Che cosa fare**

$$ IPv4Address = 192.168.134.130 $$ il PASSO 4 era già riuscito: passate alla verifica (PASSO 6)

**resta l'indirizzo DHCP o il vecchio gateway** comandi qui sotto, poi ripetete il PASSO 4

**la rete non risponde più** SConfig › Impostazioni di rete: indirizzo statico e DNS

PS C:> Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 \` >> -Confirm:$false PS C:> Remove-NetRoute -InterfaceAlias "Ethernet0" -DestinationPrefix "0.0.0.0/0" \` >> -Confirm:$false

**PASSO 5** **Il server DNS: Set-DnsClientServerAddress** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 17 / 38 **Si esegue su: SVR-LAB-01**

PS C:> Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.134.2

PS C:> (Get-NetIPConfiguration -InterfaceAlias "Ethernet0").DNSServer... ServerAddresses: {192.168.134.2} **Valore** **Quando e perché**

**192.168.134.2** ora: è il DNS della rete VMware, inoltra le richieste

**192.168.134.130** dopo la promozione (Modulo 3): il DC usa se stesso

**1.1.1.1 o altri DNS pubblici** mai su un DC: errore del deck precedente

**PASSO 6** **Verifica, poi lo stesso sul client** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 18 / 38 **Si esegue su: SVR-LAB-01, poi CLI-W11-01**

PS C:> Get-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 \| >> Format-List IPAddress, PrefixOrigin, AddressState

IPAddress: 192.168.134.130 PrefixOrigin: Manual AddressState: Preferred

PS C:> (Get-NetIPConfiguration -InterfaceAlias "Ethernet0").IPv4DefaultGateway.NextHop 192.168.134.2

## Risultato: AddressState Preferred = indirizzo utilizzabile

(Tentative = controllo non ancora finito). Solo ora si prosegue.

## Sul client CLI-W11-01

## PowerShell come amministratore: stessi passi 2, 4, 5 e questa verifica, con indirizzo 192.168.134.129.

# C

# Raggiungibilità e DNS

Le macchine si vedono? Le porte rispondono? I nomi si risolvono? **22 minuti** 19 / 38

**PASSO 7** **ping tra client e server** **Output atteso · da collaudare**

**ping: da CLI-W11-01 · regola: su SVR-LAB-01**

## Prima: risposta assente

C:> ping 192.168.134.130 Richiesta scaduta. Richiesta scaduta. Pacchetti: Trasmessi = 4, Ricevuti = 0, Persi = 4

## Sul server: profilo di rete e regola ICMP

```powershell
PS C:> Get-NetConnectionProfile | Select-Object NetworkCategory
```

```text
NetworkCategory
---------------
Public
```

```powershell
PS C:> Get-NetFirewallRule -DisplayName "*echo*ICMPv4-In*" | Select-Object DisplayName, Enabled
PS C:> Get-NetFirewallRule -DisplayName "*echo*ICMPv4-In*" | Enable-NetFirewallRule
```

## Dopo: il server risponde

C:> ping 192.168.134.130 $$ Risposta da 192.168.134.130: byte=32 durata<1ms TTL=128 $$

## Attenzione: un ping senza risposta non prova che la macchina sia spenta. Il firewall può bloccare il ping (ICMP).

**PASSO 8** **Test-NetConnection: host e porta** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 21 / 38 **Si esegue su: CLI-W11-01** PS C:> Test-NetConnection 192.168.134.130

| SourceAddress |: 192.168.134.129 |
|:--- |:--- |
| PingSucceeded |: True |

| RemotePort |: 3389 |
|:--- |:--- |
| PingSucceeded |: True |
| TcpTestSucceeded |: False |

| Campo | Che cosa dimostra |
|:--- |:--- |
| PingSucceeded: True | il server è raggiungibile in rete |
| TcpTestSucceeded: False | la porta 3389 non risponde: il Desktop remoto non è ancora abilitato |

## PingSucceeded: True il server è raggiungibile in rete

## TcpTestSucceeded: False

la porta 3389 non risponde: il Desktop remoto non è ancora abilitato

**PASSO 9** **Risolvere un nome: nslookup** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 22 / 38 **Si esegue su: CLI-W11-01 o SVR-LAB-01**

| Server: | UnKnown |
|:--- |:--- |
| Address: | 192.168.134.2 |

Risposta da un server non autorevole: Nome:    (nome canonico) Addresses:  (uno o più indirizzi) Aliases:  learn.microsoft.com

| Riga | Significato |
|:--- |:--- |
| Server / Address | il server DNS interrogato (.2); «UnKnown» = il DNS non ha un nome per se stesso (PTR) |
| Risposta non autorevole | risposta inoltrata o dalla cache, non dal server responsabile del nome |
| Aliases | il nome richiesto è un alias di un altro nome |

**PASSO 10** **Lo stesso con PowerShell: Resolve-DnsName** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 23 / 38 **Si esegue su: CLI-W11-01 o SVR-LAB-01**

PS C:> Resolve-DnsName learn.microsoft.com -Type A -Server 192.168.134.2

```text
Name  Type  TTL  Section  IPAddress
----  ----  ---  -------  ---------
(nome canonico)  A 20 Answer (indirizzo IP)
```

| Campo / parametro | Significato |
|:--- |:--- |
| -Type A | chiede il record nome → indirizzo |
| -Server | interroga un server DNS preciso, senza cambiare la configurazione |
| TTL | secondi per cui la risposta può restare in cache |
| Righe CNAME | alias: il nome rimanda a un altro nome | **-Type A** chiede il record nome → indirizzo

**PASSO 11** **Ricerca inversa: non ancora** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 24 / 38 **Si esegue su: CLI-W11-01**

| Server: | UnKnown |
|:--- |:--- |
| Address: | 192.168.134.2 |

## Perché fallisce

Il DNS.2 non conosce le nostre macchine: non esistono ancora la zona lab.local con il record A (nome → indirizzo) né la zona inversa con il record PTR (indirizzo → nome).

## Quando funzionerà

Modulo 3: dopo la promozione, record A e PTR. Il nome breve svrlab-01 funzionerà dopo il join, quando il client avrà il suffisso DNS lab.local.

# D

# Desktop remoto: abilitare e collegarsi

Dal registro alla connessione reale, un livello alla volta **28 minuti** 25 / 38

# Il Desktop remoto: chi si collega e chi può entrare

Il client avvia la connessione; il server ascolta e decide chi entra

# CLI-W11-01 client: Connessione Desktop remoto (mstsc)

# TCP 3389

# SVR-LAB-01 server: servizio TermService in ascolto

# NLA

| Autenticazione prima di aprire la | sessione. |
|:--- |:--- |
| Chi | Può collegarsi? |
| Administrators | sì, per impostazione predefinita |
| Remote Desktop Users | sì, se aggiunti al gruppo (server non DC) |
| Dopo la promozione a DC | di base solo gli amministratori: Mario e Luca no | **Chi** **Può collegarsi?**

**PASSO 12** **Il Desktop remoto è abilitato?** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 27 / 38 **Si esegue su: SVR-LAB-01**

PS C:> Get-ItemProperty \` >> -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" \` >> -Name fDenyTSConnections fDenyTSConnections: 1

| Valore | Significato |
|:--- |:--- |
| fDenyTSConnections = 1 | Desktop remoto disabilitato (stato di partenza) |
| fDenyTSConnections = 0 | Desktop remoto abilitato |

**Nota:** si legge soltanto. Il comando non cambia nulla: l'abilitazione è il passo successivo.

**PASSO 13** **Abilitare con SConfig** **Console · da collaudare**

Modulo 2 · Networking · Windows Server 2022 28 / 38 **Si esegue su: SVR-LAB-01** **PowerShell come amministratore → SConfig**

PS C:> SConfig **› voce 7: Desktop remoto** **› E: abilita** **› scegliere: solo client con Autenticazione a livello di rete (NLA)** Tornare al menu: Desktop remoto risulta abilitato

## Altre strade

Impostazioni › Sistema › Desktop remoto Server Manager › Server locale › Desktop remoto

PS C:> (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server").fDenyTSConnections

**PASSO 14** **Verifiche sul server: un livello alla volta** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 29 / 38 **Si esegue su: SVR-LAB-01**

| Running  TermService | Servizi Desktop remoto |
|:--- |:--- |
| PS C:> qwinsta rdp-tcp |  |
| rdp-tcp | 65536  Listen |
| PS C:> Get-NetTCPConnection -LocalPort 3389 -State Listen \| Format-Table LocalAddress, State |  |
| 0.0.0.0 | Listen |
| PS C:> Get-NetFirewallRule -DisplayGroup "Desktop remoto" \| Format-Table DisplayName, Enabled |  |
| Desktop remoto - Modalità utente (TCP-In) | True | **Prova** **Che cosa dimostra (e basta)** **TermService Running** il servizio è avviato

**qwinsta: rdp-tcp Listen · porta 3389 Listen** il listener RDP è attivo sulla porta

**Regole «Desktop remoto» abilitate** esistono regole che consentono il traffico

**PASSO 15** **Dal client: la porta ora risponde?** **Output atteso · da collaudare**

Modulo 2 · Networking · Windows Server 2022 30 / 38 **Si esegue su: CLI-W11-01**

PS C:> Test-NetConnection 192.168.134.130 -Port 3389

| RemotePort |: 3389 |
|:--- |:--- |
| SourceAddress |: 192.168.134.129 |
| TcpTestSucceeded |: True |

## Risultato: rispetto al PASSO 8 la porta 3389 ora risponde: rete, listener e firewall lasciano passare la connessione.

## Attenzione: True non dice ancora se potrete entrare. Lo dice solo la connessione reale.

**PASSO 16** **La connessione reale** **Console · da collaudare**

**mstsc: da CLI-W11-01 · qwinsta: su SVR-LAB-01** PS C:> mstsc /v:192.168.134.130 **Connessione Desktop remoto**

Nome utente: SVR-LAB-01\Administrator Password: quella dell'Administrator (in aula: P@ssw0rd!, segnaposto fittizio) **› avviso sul certificato del server: Sì** Si apre il desktop del server

**Sul server: la sessione è attiva (estratto)**

| NOME SESSIONE  NOME UTENTE | ID | STATO |
|:--- |:--- |:--- |
| rdp-tcp#0 | Administrator  2 | Attivo |
| rdp-tcp | 65536  Listen |  |

## Risultato: tutti i livelli superati. Il Desktop remoto è abilitato e utilizzato.

# Troubleshooting a livelli

## Dal basso verso l'alto: ogni prova dimostra un livello solo

| Livello | Comando di verifica |
| --- | --- |
| Indirizzo | ipconfig /all, Get-NetIPConfiguration · su ciascuna macchina |
| Raggiungibilità | ping, Test-NetConnection · dal client (ICMP può essere bloccato) |
| Nomi | nslookup, Resolve-DnsName · quale DNS risponde? |
| Porta | Test-NetConnection -Port 3389 · dal client |
| Servizio e listener | Get-Service TermService, qwinsta · sul server |
| Firewall | Get-NetFirewallRule · sul server (un blocco esplicito prevale) |
| Autorizzazione | connessione reale con mstsc · chi può entrare? |

# Errori comuni

Quelli corretti rispetto ai deck precedenti, e altri frequenti

**DNS pubblico (1.1.1.1) su un DC** i record di Active Directory non vengono registrati

**Ping senza risposta = macchina spenta** il firewall può bloccare ICMP

**Porta trovata con netstat = in ascolto** serve lo stato LISTENING / Listen

**Regole abilitate = traffico consentito** una regola di blocco esplicita prevale

**// nei comandi PowerShell o CMD** i commenti PowerShell iniziano con #

**Get-Command -Modulo** il parametro è -Module

**IP statico dentro l'intervallo DHCP** possibile conflitto con un'altra macchina

# DNS

Dopo la promozione: DNS.130 su server e client; zona di ricerca inversa e record PTR.

## Nomi del dominio

Resolve-DnsName svr-lab-01.lab.local (record A) e nslookup dell'indirizzo (PTR); il nome breve svr-lab-01 dopo il join (suffisso DNS).

## RDP verso il DC

Di base solo gli amministratori: Mario e Luca, utenti di dominio, saranno respinti.

| Comando | Uso | Dove |
|:--- |:--- |:--- |
| ipconfig /all | configurazione completa | entrambe |
| Get-NetAdapter · Get-NetIPConfiguration | nome scheda · riepilogo | entrambe |
| New-NetIPAddress | indirizzo statico e gateway | entrambe |
| Set-DnsClientServerAddress | server DNS | entrambe |
| ping · Test-NetConnection [-Port] | raggiungibilità · porta | client |
| nslookup · Resolve-DnsName | risoluzione dei nomi | entrambe |
| Get-ItemProperty... fDenyTSConnections | RDP abilitato? | server |
| SConfig (voce 7) | abilitare RDP | server |
| Get-Service TermService · qwinsta | servizio e listener | server |
| Get-NetFirewallRule -DisplayGroup | regole firewall | server |
| mstsc /v:192.168.134.130 | connessione reale | client |

# Laboratorio finale · 10 minuti

Trova il guasto: dal client non riuscite più a collegarvi in RDP

| Controllo | Da dove iniziare |
| --- | --- |
| Dove si ferma? | Test-NetConnection 192.168.134.130 -Port 3389 dal client |
| Rete | se il ping o l'indirizzo non tornano: ipconfig /all |
| Server | fDenyTSConnections, TermService, qwinsta |
| Firewall | Get-NetFirewallRule -DisplayGroup "Desktop remoto" |
| In chat | scrivete la causa e il comando che la dimostra |

**Nota:** il guasto lo introduce il formatore. Serve trovarlo e dimostrarlo, non solo correggerlo.

# Quiz di verifica

Rispondete in chat con il numero della domanda e la lettera

A che cosa serve il gateway? A) a tradurre i nomi in indirizzi  B) a raggiungere le altre reti  C) ad assegnare gli indirizzi

Quale DNS deve usare SVR-LAB-01 dopo la promozione a DC? A) 1.1.1.1  B) 192.168.134.2  C) 192.168.134.130

| Il ping verso il server non risponde, ma Test-NetConnection -Port 3389 riesce. Causa probabile? A) ICMP bloccato dal firewall | B) server |
|:--- |:--- |
| spento | C) RDP disabilitato |

qwinsta mostra rdp-tcp in stato Listen. Che cosa dimostra? A) che il client può entrare  B) che il listener RDP è attivo sul server  C) che il firewall consente la porta 3389

Dopo la promozione, Mario Rossi può collegarsi in RDP al DC? A) sì, è nel dominio  B) sì, se la porta è aperta  C) no, di base possono solo gli amministratori

# Correzione del quiz Risposta corretta e perché

Modulo 2 · Networking · Windows Server 2022 38 / 38 **N.** **Risposta** **Perché**

| B | il gateway inoltra il traffico destinato ad altre reti |
|:--- |:--- |
| C | il primo DC con DNS usa se stesso; mai DNS pubblici su un DC |
| A | la porta risponde, quindi il server è acceso: causa probabile il blocco ICMP, da verificare |
| B | Listen riguarda il server; l'accesso si prova solo con la connessione reale |
| C | il diritto RDP su un DC spetta di base al gruppo Administrators |

C il primo DC con DNS usa se stesso; mai DNS pubblici su un DC

A la porta risponde, quindi il server è acceso: causa probabile il blocco ICMP, da verificare

B Listen riguarda il server; l'accesso si prova solo con la connessione reale

C il diritto RDP su un DC spetta di base al gruppo Administrators

**Prossimo passo:** Modulo 3 Active Directory. Si parte dagli indirizzi statici configurati oggi.
