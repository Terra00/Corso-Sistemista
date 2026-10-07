# Calcolo degli Host nelle Classi di Rete IPv4

In una rete IPv4 con indirizzamento di tipo *classful*, il numero massimo di indirizzi e di host utilizzabili per singola rete varia a seconda della classe (A, B o C).

## Riepilogo rapido

| Classe | Bit host | Indirizzi totali | Host utilizzabili (IP validi) | Maschera predefinita |
| :---: | :---: | :---: | :---: | :---: |
| **Classe A** | 24 bit | 16.777.216 | **16.777.214** | `255.0.0.0` (`/8`) |
| **Classe B** | 16 bit | 65.536 | **65.534** | `255.255.0.0` (`/16`) |
| **Classe C** | 8 bit | 256 | **254** | `255.255.255.0` (`/24`) |

*Nota: il numero di host utilizzabili segue la formula $2^n - 2$, dove $n$ è il numero di bit dedicati all'host. Si sottraggono sempre due indirizzi per ogni rete: l'indirizzo di rete (tutti i bit host a 0) e l'indirizzo di broadcast (tutti i bit host a 1).*

---

## Dettaglio delle classi

### Classe A
* **Spazio host**: dedica 24 bit alla porzione host ($2^{24}$).
* **Capacità**: genera 16.777.216 indirizzi totali, di cui 16.777.214 assegnabili a dispositivi reali.
* **Uso tipico**: grandissime organizzazioni o enti governativi.

### Classe B
* **Spazio host**: dedica 16 bit alla porzione host ($2^{16}$).
* **Capacità**: genera 65.536 indirizzi totali, di cui 65.534 assegnabili a dispositivi reali.
* **Uso tipico**: medie e grandi aziende, università.

### Classe C
* **Spazio host**: dedica 8 bit alla porzione host ($2^{8}$).
* **Capacità**: genera 256 indirizzi totali, di cui 254 assegnabili a dispositivi reali.
* **Uso tipico**: piccole imprese, reti domestiche e locali.
