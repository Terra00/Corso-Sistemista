# Comandi di rete per Windows Server

Riferimento operativo per leggere la configurazione IP, impostare indirizzo e DNS, quindi verificare connettività, nomi e porte. Eseguire i comandi di configurazione da PowerShell avviato come amministratore.

## Sequenza consigliata

1. Leggere la configurazione locale con `ipconfig /all` oppure `Get-NetIPConfiguration`.
2. Verificare la scheda con `Get-NetAdapter`.
3. Configurare indirizzo, gateway e DNS se necessario.
4. Testare prima il raggiungimento dell'host, poi la risoluzione DNS, infine una porta del servizio.

## Consultazione e configurazione

| Comando | Ambiente | Scopo |
|---|---|---|
| `ipconfig /all` | Prompt o PowerShell | Mostra indirizzo, subnet mask, gateway, DNS e stato DHCP. |
| `Get-NetIPConfiguration` | PowerShell | Mostra in modo sintetico indirizzi, gateway e DNS per interfaccia. |
| `Get-NetIPAddress -AddressFamily IPv4` | PowerShell | Mostra gli indirizzi IPv4; non è un riepilogo completo di gateway e DNS. |
| `Get-NetAdapter` | PowerShell | Elenca schede e stato; il valore `Name` è l'`InterfaceAlias` usato nei cmdlet successivi. |
| `New-NetIPAddress` | PowerShell | Aggiunge un indirizzo IPv4, prefisso e gateway a una scheda. |
| `Set-DnsClientServerAddress` | PowerShell | Imposta i server DNS della scheda. |

### Esempio: configurazione statica

Sostituire i valori con quelli dell'ambiente. L'esempio usa la rete didattica `192.168.134.0/24`.

```powershell
Get-NetAdapter
New-NetIPAddress -InterfaceAlias "Ethernet0" `
  -IPAddress 192.168.134.130 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.134.2
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" `
  -ServerAddresses 192.168.134.2
```

> La configurazione può interrompere una sessione remota. Usare la console della VM quando si modifica l'indirizzo della macchina corrente.

## Verifica della rete e dei servizi

| Comando | Cosa dimostra | Esempio |
|---|---|---|
| `ping` | L'host risponde a ICMP, se firewall e rete lo consentono. | `ping 192.168.134.130` |
| `Test-NetConnection` | Raggiungibilità dell'host e, con `-Port`, di una porta TCP. | `Test-NetConnection 192.168.134.130 -Port 3389` |
| `nslookup` | Risoluzione diretta o inversa attraverso il DNS configurato. | `nslookup svr-lab-01.lab.local` |
| `Resolve-DnsName` | Query DNS dettagliata da PowerShell. | `Resolve-DnsName svr-lab-01.lab.local -Type A` |
| `netstat -an` | Porte in ascolto e connessioni attive. | `netstat -an \| find "3389"` |
| `Get-NetTCPConnection` | Stato delle connessioni TCP locali. | `Get-NetTCPConnection -LocalPort 3389` |

Un risultato `TcpTestSucceeded: True` conferma che la porta TCP risponde, ma non conferma da solo che l'utente disponga dell'autorizzazione per usare il servizio. La prova definitiva è una connessione reale.

## Checklist di diagnosi

- [ ] La scheda corretta è `Up` e usa l'alias atteso.
- [ ] Indirizzo, prefisso, gateway e DNS corrispondono alla rete prevista.
- [ ] L'host remoto è raggiungibile, tenendo conto che ICMP può essere bloccato.
- [ ] Il nome viene risolto dal DNS corretto.
- [ ] La porta del servizio è raggiungibile.
- [ ] Il servizio è attivo e l'account è autorizzato.
