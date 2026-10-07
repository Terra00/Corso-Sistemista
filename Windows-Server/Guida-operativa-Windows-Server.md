# Percorso operativo Windows Server

Questa guida mette in ordine i moduli e i laboratori della raccolta. I nomi, gli indirizzi e i comandi di esempio si riferiscono al laboratorio `lab.local`: adattarli prima dell'uso in un ambiente diverso.

## Ordine di studio e dipendenze

1. [Networking e Desktop remoto](Rete/Networking-e-Desktop-remoto.md): indirizzi IP, gateway, DNS temporaneo e RDP.
2. [Domain Controller](Active-Directory/Domain-Controller.md): installazione AD DS, promozione, DNS del dominio e join del client.
3. [Oggetti Active Directory](Active-Directory/Oggetti-Active-Directory.md): OU, utenti, gruppi, delega e trust.
4. [Storage, NTFS e condivisioni](Storage-e-file-server/Storage-NTFS-e-condivisioni.md): disco dati, filesystem, condivisioni e autorizzazioni.
5. [Laboratorio GPO](Criteri-di-gruppo/Laboratorio-GPO.md): ciclo di vita e verifica delle Group Policy.

## Configurazione di riferimento del laboratorio

| Componente                 | Valore                              |
| -------------------------- | ----------------------------------- |
| Dominio                    | `lab.local`                         |
| Domain Controller          | `SVR-LAB-01` — `192.168.134.130/24` |
| Client                     | `CLI-W11-01` — `192.168.134.129/24` |
| Gateway VMware             | `192.168.134.2`                     |
| DNS prima della promozione | `192.168.134.2`                     |
| DNS dopo la promozione     | `192.168.134.130`                   |

## Regole tecniche da non confondere

- Prima della promozione, il server non ospita ancora il DNS di `lab.local`; dopo la promozione il Domain Controller deve usare il proprio DNS, non un DNS pubblico.
- Un test TCP della porta 53 prova il listener TCP, non la qualità della risoluzione DNS. Per il DNS eseguire anche `Resolve-DnsName` o `nslookup`.
- `PingSucceeded` e `TcpTestSucceeded` rispondono a domande diverse. Il primo dipende anche dalle regole ICMP; il secondo non dimostra che l'account possa accedere al servizio.
- Gli utenti, i gruppi e i computer sono oggetti distinti. Gli utenti locali restano locali al singolo computer; gli utenti di dominio risiedono in Active Directory.
- Applicare i permessi alle risorse tramite gruppi, seguendo AGDLP quando il modello è adatto, anziché assegnarli direttamente agli utenti.
- Per una condivisione Windows si valutano sia i permessi di condivisione sia quelli NTFS: durante un accesso via rete prevale il risultato più restrittivo.
- Le GPO hanno ambito e precedenza; verificare il risultato effettivo con `gpresult` o RSoP, non solo la loro esistenza.

## Documenti di supporto

- [Comandi di rete](Rete/Comandi-di-rete.md): riferimento conciso e corretto per diagnostica e configurazione.
- [Laboratorio 01: Domain Controller e join](Active-Directory/Laboratorio-01-Domain-Controller.md): procedura guidata estesa.
- [Materiale integrativo su Active Directory](Active-Directory/Materiale-integrativo-Active-Directory.md): spiegazioni ed esercizi aggiuntivi; i moduli 3a e 3b restano il percorso didattico principale.
