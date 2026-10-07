# Manuale di Markdown

**Livello:** Principiante → Intermedio  
**Obiettivo:** imparare a scrivere e formattare documenti utilizzando Markdown.

---

## Indice

1. [Cos'è Markdown](#1-cosè-markdown)
2. [Come funziona](#2-come-funziona)
3. [Titoli](#3-titoli)
4. [Paragrafi e interruzioni di riga](#4-paragrafi-e-interruzioni-di-riga)
5. [Grassetto, corsivo e barrato](#5-grassetto-corsivo-e-barrato)
6. [Elenchi puntati](#6-elenchi-puntati)
7. [Elenchi numerati](#7-elenchi-numerati)
8. [Elenchi annidati](#8-elenchi-annidati)
9. [Link](#9-link)
10. [Immagini](#10-immagini)
11. [Citazioni](#11-citazioni)
12. [Codice inline](#12-codice-inline)
13. [Blocchi di codice e code fences](#13-blocchi-di-codice-e-code-fences)
14. [Linee orizzontali](#14-linee-orizzontali)
15. [Tabelle](#15-tabelle)
16. [Checklist](#16-checklist)
17. [Caratteri speciali ed escape](#17-caratteri-speciali-ed-escape)
18. [Collegamenti interni e ancore](#18-collegamenti-interni-e-ancore)
19. [HTML in Markdown](#19-html-in-markdown)
20. [Commenti HTML](#20-commenti-html)
21. [Emoji](#21-emoji)
22. [Markdown avanzato](#22-markdown-avanzato)
23. [GitHub Flavored Markdown](#23-github-flavored-markdown)
24. [Buone pratiche](#24-buone-pratiche)
25. [Esercizi](#25-esercizi)
26. [Progetto finale](#26-progetto-finale)
27. [Cheat sheet](#27-cheat-sheet)

---

# 1. Cos'è Markdown

**Markdown** è un linguaggio di marcatura leggero utilizzato per formattare testo usando una sintassi semplice e leggibile.

Un file Markdown normalmente utilizza l'estensione:

```text
.md
```

Esempio:

```text
lezione.md
README.md
appunti.md
documentazione.md
```

Markdown viene utilizzato in moltissimi contesti, tra cui:

- documentazione tecnica;
- README di progetti;
- appunti;
- blog;
- documentazione software;
- piattaforme collaborative;
- GitHub;
- sistemi di gestione della documentazione;
- generatori di siti statici.

## Perché impararlo?

Markdown permette di scrivere testo strutturato senza dover utilizzare un editor complesso.

Per esempio:

```markdown
# Il mio documento

## Introduzione

Questo è un testo di esempio.

**Questo testo è importante.**
```

Il risultato sarà un documento con un titolo, un sottotitolo, un paragrafo e una frase in grassetto.

---

# 2. Come funziona

Markdown utilizza caratteri normali della tastiera per descrivere la struttura del documento.

Per esempio:

```markdown
# Titolo
```

indica un titolo principale.

```markdown
**testo importante**
```

indica normalmente del testo in grassetto.

```markdown
- primo elemento
- secondo elemento
```

crea un elenco puntato.

Il testo Markdown viene poi interpretato da un'applicazione, che può visualizzarlo in forma formattata.

## Markdown sorgente e risultato

È importante imparare a distinguere:

1. **sorgente Markdown** → ciò che viene scritto;
2. **rendering** → ciò che viene visualizzato.

Esempio sorgente:

```markdown
# Animali

- Cane
- Gatto
- Cavallo
```

Risultato:

# Animali

- Cane
- Gatto
- Cavallo

---

# 3. Titoli

I titoli vengono creati usando il carattere `#`.

Il numero di `#` determina il livello del titolo.

```markdown
# Titolo 1

## Titolo 2

### Titolo 3

#### Titolo 4

##### Titolo 5

###### Titolo 6
```

È possibile arrivare fino a sei livelli.

## Regola pratica

Utilizzare una struttura gerarchica coerente.

Esempio:

```markdown
# Corso di programmazione

## Python

### Variabili

### Funzioni

## JavaScript

### Variabili

### Funzioni
```

Evitare di scegliere il livello del titolo soltanto in base alla dimensione visualizzata.

Il titolo deve rappresentare la **gerarchia logica** del contenuto.

---

# 4. Paragrafi e interruzioni di riga

Un paragrafo normale non richiede una sintassi particolare.

```markdown
Questo è un paragrafo.

Questo è un altro paragrafo.
```

Una riga vuota separa normalmente due paragrafi.

## Nuova riga

Il comportamento dell'interruzione di riga può variare leggermente tra le implementazioni Markdown.

Un metodo comune consiste nell'inserire due spazi alla fine della riga:

```markdown
Prima riga.  
Seconda riga.
```

In molte implementazioni è possibile utilizzare anche un tag HTML:

```markdown
Prima riga.<br>
Seconda riga.
```

Per i documenti normali è generalmente preferibile utilizzare nuovi paragrafi quando il contenuto rappresenta idee separate.

## Esempio completo

```markdown
Il corso si svolge in presenza.

Sede del corso:  
Via Roma 10  
20100 Milano
```

Le prime due righe formano paragrafi distinti; le righe dell'indirizzo utilizzano le interruzioni di riga (due spazi finali) per rimanere in un unico blocco.

---

# 5. Grassetto, corsivo e barrato

## Corsivo

Si può utilizzare un singolo asterisco oppure un singolo underscore.

```markdown
*testo in corsivo*

_testo in corsivo_
```

Risultato:

*testo in corsivo*

## Grassetto

Si utilizzano normalmente due asterischi oppure due underscore.

```markdown
**testo in grassetto**

__testo in grassetto__
```

Risultato:

**testo in grassetto**

## Grassetto + corsivo

```markdown
***testo importante***
```

Risultato:

***testo importante***

## Testo barrato

Il barrato non appartiene al Markdown originale, ma è supportato da molte implementazioni moderne, tra cui GFM.

```markdown
~~testo barrato~~
```

Risultato:

~~testo barrato~~

## Esempio in contesto

```markdown
La versione **2.0** introduce la modalità *scura* e abbandona il supporto ~~obsoleto~~.
```

Risultato:

La versione **2.0** introduce la modalità *scura* e abbandona il supporto ~~obsoleto~~.

---

# 6. Elenchi puntati

Un elenco puntato può essere creato utilizzando `-`, `*` oppure `+`.

Esempio:

```markdown
- Primo elemento
- Secondo elemento
- Terzo elemento
```

Risultato:

- Primo elemento
- Secondo elemento
- Terzo elemento

È consigliabile scegliere un simbolo e utilizzarlo in modo coerente.

## Esempio pratico

```markdown
## Materiale necessario

- Computer
- Connessione Internet
- Editor di testo
- Browser
```

---

# 7. Elenchi numerati

Gli elenchi numerati utilizzano un numero seguito da un punto.

```markdown
1. Primo passaggio
2. Secondo passaggio
3. Terzo passaggio
```

Risultato:

1. Primo passaggio
2. Secondo passaggio
3. Terzo passaggio

Sono particolarmente utili per:

- procedure;
- istruzioni;
- sequenze temporali;
- classificazioni.

## Esempio

```markdown
## Installazione

1. Scaricare il programma.
2. Avviare l'installazione.
3. Accettare le condizioni.
4. Completare la procedura.
```

---

# 8. Elenchi annidati

Gli elenchi possono contenere altri elenchi.

Esempio:

```markdown
- Programmazione
  - Python
  - JavaScript
  - Java
- Database
  - SQL
  - PostgreSQL
  - SQLite
```

Risultato:

- Programmazione
  - Python
  - JavaScript
  - Java
- Database
  - SQL
  - PostgreSQL
  - SQLite

L'indentazione è importante.

Per maggiore compatibilità è consigliabile utilizzare spazi in modo coerente.

## Elenco numerato con sottoelementi

```markdown
1. Preparazione
   - Installare il software
   - Creare la cartella
2. Sviluppo
   - Scrivere il codice
   - Eseguire i test
3. Pubblicazione
```

---

# 9. Link

I collegamenti utilizzano questa sintassi:

```markdown
[Testo del link](https://esempio.com)
```

Esempio:

```markdown
Consultare il [sito ufficiale di Python](https://www.python.org/).
```

## Link con titolo

È possibile aggiungere un titolo opzionale:

```markdown
[Python](https://www.python.org/ "Sito ufficiale di Python")
```

## Link automatici

Alcune implementazioni riconoscono automaticamente gli URL:

```markdown
https://www.python.org/
```

Una forma esplicita molto semplice è:

```markdown
<https://www.python.org/>
```

## Link di riferimento

È possibile separare il testo del link dalla definizione dell'URL:

```markdown
Il progetto è ospitato su [GitHub][gh] e documentato sul [sito ufficiale][docs].

[gh]: https://github.com/
[docs]: https://docs.github.com/ "Documentazione di GitHub"
```

Le definizioni possono essere raccolte in fondo al documento: il testo risulta più leggibile e gli URL più semplici da aggiornare.

---

# 10. Immagini

La sintassi delle immagini è simile a quella dei link, ma inizia con `!`.

```markdown
![Testo alternativo](immagine.png)
```

Il testo alternativo è importante perché descrive l'immagine quando questa non può essere visualizzata e migliora l'accessibilità.

## Immagine da URL

```markdown
![Logo](https://esempio.com/logo.png)
```

## Immagine con titolo

```markdown
![Logo](logo.png "Logo dell'azienda")
```

## Buona pratica

Preferire un testo alternativo descrittivo.

Meglio:

```markdown
![Diagramma che mostra il flusso di un programma](diagramma.png)
```

che:

```markdown
![immagine](diagramma.png)
```

---

# 11. Citazioni

Le citazioni si creano utilizzando `>`.

```markdown
> Questa è una citazione.
```

Risultato:

> Questa è una citazione.

## Citazione su più righe

```markdown
> Prima riga.
> Seconda riga.
> Terza riga.
```

## Citazioni annidate

```markdown
> Citazione principale.
>
> > Citazione interna.
```

## Uso pratico

Le citazioni sono utili per:

- riportare una frase;
- evidenziare una definizione;
- riportare una nota;
- distinguere una comunicazione dal testo principale.

Non è consigliabile utilizzarle soltanto per ottenere un effetto grafico.

## Citazione con contenuto strutturato

Una citazione può contenere altra formattazione, per esempio un elenco:

```markdown
> **Nota operativa:**
>
> - salvare il file;
> - riavviare il servizio;
> - verificare il log.
```

---

# 12. Codice inline

Quando si deve inserire una piccola porzione di codice all'interno di una frase si utilizza il carattere backtick:

```markdown
La funzione `print()` visualizza un valore.
```

Risultato:

La funzione `print()` visualizza un valore.

È particolarmente utile per:

- nomi di funzioni;
- variabili;
- comandi;
- nomi di file;
- parole chiave;
- percorsi.

Esempio:

```markdown
Il file `config.json` contiene le impostazioni dell'applicazione.
```

## Backtick all'interno del codice inline

Per mostrare un backtick letterale si utilizzano due backtick come delimitatori, con uno spazio di separazione:

```markdown
Per aprire un code fence si usa il carattere `` ` `` ripetuto tre volte.
```

---

# 13. Blocchi di codice e code fences

Per inserire più righe di codice è consigliabile utilizzare un **code fence**.

Un code fence è normalmente delimitato da tre backtick di apertura e da tre backtick di chiusura.

Esempio:

````markdown
```
print("Ciao!")
print("Benvenuto")
```
````

Il risultato sarà:

```text
print("Ciao!")
print("Benvenuto")
```

## Specificare il linguaggio

È buona pratica indicare il linguaggio dopo i tre backtick iniziali.

Python:

````markdown
```python
nome = "Anna"
print(nome)
```
````

JavaScript:

````markdown
```javascript
const nome = "Anna";
console.log(nome);
```
````

Bash:

````markdown
```bash
echo "Ciao"
```
````

HTML:

````markdown
```html
<h1>Ciao</h1>
```
````

JSON:

````markdown
```json
{
  "nome": "Anna",
  "eta": 25
}
```
````

Questo permette a molti strumenti di applicare la **syntax highlighting**, cioè la colorazione del codice.

## Code fence con quattro o più backtick

Se il codice contiene già tre backtick, si può utilizzare un delimitatore più lungo.

Esempio:

`````markdown
````
```python
print("Ciao")
```
````
`````

Questa tecnica è particolarmente importante quando si sta scrivendo un **manuale su Markdown**, perché permette di mostrare la sintassi Markdown senza farla interpretare accidentalmente.

---

# 14. Linee orizzontali

Una linea orizzontale può essere creata con tre o più caratteri `-`, `*` oppure `_`.

Esempio:

```markdown
---
```

Oppure:

```markdown
***
```

Risultato:

---

È consigliabile utilizzare le linee orizzontali per separare sezioni concettualmente distinte, senza abusarne.

## Attenzione alla riga precedente

Una riga di `---` posta immediatamente sotto una riga di testo, senza riga vuota di separazione, viene interpretata come titolo di livello 2 (sintassi *setext*):

```markdown
Questo testo diventa un titolo
---
```

Per ottenere una linea orizzontale è quindi necessario lasciare una riga vuota prima di `---`.

---

# 15. Tabelle

Le tabelle sono una delle funzionalità più utilizzate nelle implementazioni moderne di Markdown.

Sintassi:

```markdown
| Nome | Età | Città |
|---|---|---|
| Anna | 25 | Roma |
| Luca | 31 | Milano |
| Sara | 28 | Torino |
```

Risultato:

| Nome | Età | Città |
|---|---|---|
| Anna | 25 | Roma |
| Luca | 31 | Milano |
| Sara | 28 | Torino |

## Allineamento

La posizione dei due punti nella riga separatrice determina l'allineamento.

Sinistra:

```markdown
| Nome |
|:---|
| Anna |
```

Destra:

```markdown
| Prezzo |
|---:|
| 19,90 € |
```

Centrato:

```markdown
| Stato |
|:---:|
| OK |
```

## Esempio

```markdown
| Linguaggio | Tipologia | Difficoltà |
|:---|:---|---:|
| Python | General purpose | Facile |
| JavaScript | Web | Media |
| C | General purpose | Alta |
```

## Formattazione nelle celle

Le celle possono contenere formattazione inline:

```markdown
| Comando | Descrizione |
|---|---|
| `ls -l` | Elenco **dettagliato** |
| `rm -r` | Rimozione *ricorsiva* |
```

Le tabelle Markdown non supportano invece celle su più righe né celle unite; per esigenze di questo tipo è necessario ricorrere all'HTML.

---

# 16. Checklist

Le checklist non fanno parte del Markdown originale, ma sono supportate da diverse implementazioni moderne, in particolare GitHub Flavored Markdown.

Sintassi:

```markdown
- [ ] Attività da completare
- [x] Attività completata
```

Risultato:

- [ ] Attività da completare
- [x] Attività completata

Esempio:

```markdown
## Preparazione del corso

- [x] Creare il materiale
- [x] Preparare gli esempi
- [ ] Preparare gli esercizi
- [ ] Verificare il laboratorio
```

Le checklist sono molto utili per:

- attività;
- esercizi;
- procedure;
- attività di progetto;
- controllo del lavoro.

---

# 17. Caratteri speciali ed escape

Alcuni caratteri hanno un significato speciale in Markdown.

Per visualizzare un carattere senza farlo interpretare come sintassi Markdown, si può utilizzare il backslash `\`.

Esempio:

```markdown
\*Questo non sarà in corsivo\*
```

Risultato:

\*Questo non sarà in corsivo\*

## Caratteri che possono richiedere escape

Tra quelli più comuni:

```text
\
`
*
_
{}
[]
()
#
+
-
.
!
|
>
```

Non è necessario fare l'escape indiscriminatamente. Va utilizzato quando si vuole impedire che un carattere venga interpretato come sintassi.

## Esempio

Per scrivere un asterisco letterale:

```markdown
\*
```

Per scrivere una parentesi quadra letterale:

```markdown
\[
```

## Numero a inizio riga

Un numero seguito da un punto a inizio riga viene interpretato come elenco numerato. Per evitarlo si applica l'escape al punto:

```markdown
2026\. Anno di pubblicazione del manuale.
```

---

# 18. Collegamenti interni e ancore

È possibile creare collegamenti verso altre parti dello stesso documento.

Esempio:

```markdown
[Vai alla sezione Tabelle](#tabelle)
```

Se il titolo è:

```markdown
## Tabelle
```

molti renderer Markdown generano automaticamente un'ancora simile a:

```text
#tabelle
```

La trasformazione esatta degli identificatori può variare a seconda dell'implementazione.

In generale, la regola tipica (in stile GitHub) è:

1. convertire il titolo in minuscolo;
2. sostituire gli spazi con trattini;
3. rimuovere la punteggiatura.

Esempio:

```markdown
## Configurazione avanzata del server
```

diventa:

```text
#configurazione-avanzata-del-server
```

## Perché usarli?

I collegamenti interni sono molto utili nei documenti lunghi:

- manuali;
- guide;
- documentazione;
- FAQ;
- tutorial.

Un indice Markdown può quindi essere realizzato con link interni.

Esempio:

```markdown
## Indice

- [Introduzione](#introduzione)
- [Installazione](#installazione)
- [Configurazione](#configurazione)
- [Conclusioni](#conclusioni)
```

---

# 19. HTML in Markdown

Molti parser Markdown consentono di utilizzare HTML direttamente nel documento.

Esempio:

```html
<p>Questo è un paragrafo HTML.</p>
```

Oppure:

```html
<strong>Testo importante</strong>
```

Questo può essere utile quando Markdown non offre una funzionalità sufficiente per una determinata esigenza.

Tuttavia, l'utilizzo dell'HTML può ridurre la portabilità del documento.

## Regola pratica

Preferire Markdown quando è sufficiente.

Utilizzare HTML quando:

- serve una funzionalità non disponibile;
- il renderer lo supporta;
- si conoscono le conseguenze sulla portabilità.

## Esempio: sezione espandibile

Alcune piattaforme (per esempio GitHub) supportano l'elemento `<details>`:

```html
<details>
<summary>Mostra la soluzione</summary>

Contenuto visualizzato dopo il clic.

</details>
```

Le righe vuote intorno al contenuto sono necessarie affinché questo venga interpretato come Markdown.

---

# 20. Commenti HTML

I commenti HTML possono essere utilizzati per inserire informazioni che non vengono visualizzate nel rendering.

```html
<!-- Questo testo non viene normalmente visualizzato -->
```

Sono utili, ad esempio, per:

- note interne;
- promemoria;
- informazioni per chi modifica il documento;
- marcatori tecnici.

Attenzione: il fatto che un commento non sia visibile nel rendering **non significa necessariamente che sia segreto**. Il contenuto rimane normalmente presente nel file sorgente.

---

# 21. Emoji

Molti ambienti Markdown supportano le emoji direttamente.

Esempio:

```markdown
Buongiorno! 👋

Attenzione: ⚠️

Operazione completata: ✅
```

Alcune piattaforme supportano anche shortcode, per esempio:

```markdown
:smile:
:warning:
:rocket:
```

Il supporto agli shortcode dipende dalla piattaforma.

Per documenti destinati a diversi ambienti, le emoji Unicode sono generalmente più semplici da comprendere, ma anche il loro rendering può variare tra sistemi.

---

# 22. Markdown avanzato

Una volta imparata la sintassi di base, è possibile utilizzare tecniche più avanzate.

## 22.1 Testo combinato

È possibile combinare più formattazioni:

```markdown
***testo in grassetto e corsivo***
```

Oppure:

```markdown
**testo con `codice` incorporato**
```

## 22.2 Link con testo formattato

Esempio:

```markdown
[**Documentazione ufficiale**](https://example.com)
```

## 22.3 Immagine collegata

È possibile inserire un'immagine all'interno di un link:

```markdown
[![Logo](logo.png)](https://example.com)
```

Quando l'utente clicca sull'immagine, viene aperto il collegamento.

## 22.4 Strutture complesse

Markdown può essere combinato con:

- HTML;
- codice;
- tabelle;
- link;
- immagini;
- elementi specifici della piattaforma.

È però importante ricordare che **non esiste un unico Markdown identico in ogni ambiente**.

## 22.5 Testo colorato ed evidenziato

Markdown non prevede una sintassi nativa per il colore del testo. Dove il renderer lo consente, si ricorre all'HTML inline.

### Colore del testo (foreground)

```html
Il valore <span style="color:#c00000">non è valido</span> e va corretto.
```

Il colore può essere espresso con un nome (`red`, `green`, `blue`) oppure in notazione esadecimale (`#c00000`).

### Colore di sfondo (background)

```html
Ricordare la <span style="background-color:yellow">scadenza del 30 giugno</span>.
```

### Combinazione foreground e background

```html
<span style="color:white; background-color:#006600">Operazione completata</span>
```

### Evidenziazione semantica con `<mark>`

L'elemento HTML `<mark>` evidenzia il testo con lo stile predefinito del renderer (tipicamente sfondo giallo), senza specificare colori:

```html
Il termine <mark>idempotente</mark> viene definito nel glossario.
```

### Sintassi `==testo==`

Alcuni editor, tra cui Typora e Obsidian, supportano un'estensione dedicata all'evidenziazione:

```markdown
Testo ==evidenziato== con l'estensione highlight.
```

In Typora la funzionalità va abilitata nelle preferenze (sezione Markdown, opzione *Highlight*). Non fa parte di CommonMark né di GFM: su altri renderer i simboli `==` restano visibili come testo normale.

### Portabilità

Il supporto varia sensibilmente tra gli ambienti:

| Tecnica | Editor locali (es. Typora) | GitHub |
|---|---|---|
| `<span style="...">` | Sì | No: l'attributo `style` viene rimosso |
| `<mark>` | Sì | Sì (stile predefinito, colori non personalizzabili) |
| `==testo==` | Se abilitata | No |

Per documenti destinati a GitHub, `<mark>` è quindi l'unica opzione affidabile tra quelle elencate. In generale, se il colore veicola un'informazione essenziale, è preferibile affiancare sempre un'indicazione testuale: il colore potrebbe non essere visualizzato.

---

# 23. GitHub Flavored Markdown

Una delle varianti più conosciute è **GitHub Flavored Markdown (GFM)**.

GFM estende Markdown con funzionalità come:

- tabelle;
- checklist;
- barrato;
- alcune funzionalità relative a riferimenti e link;
- blocchi di codice;
- funzionalità specifiche dell'ambiente GitHub.

Per esempio:

```markdown
- [x] Task completato
- [ ] Task da completare
```

è una sintassi molto comune nei repository GitHub.

## Note a piè di pagina

GFM supporta le note a piè di pagina:

```markdown
Il progetto adotta una licenza permissiva[^1].

[^1]: Licenza MIT.
```

Il rendering colloca automaticamente la nota in fondo alla pagina, con collegamenti in entrambe le direzioni.

## Markdown non è sempre uguale

Quando si scrive Markdown, bisogna chiedersi:

> "Dove verrà visualizzato questo file?"

Un documento può essere visualizzato con parser differenti, per esempio:

- GitHub;
- GitLab;
- editor locali;
- generatori di siti;
- sistemi di documentazione;
- applicazioni per prendere appunti.

Una funzionalità supportata da un ambiente potrebbe non essere supportata da un altro.

---

# 24. Buone pratiche

## 24.1 Usare titoli coerenti

Preferire:

```markdown
# Manuale

## Installazione

## Configurazione

## Utilizzo
```

a una struttura casuale di titoli.

## 24.2 Non usare la formattazione solo per estetica

Il grassetto dovrebbe avere un significato.

```markdown
**Attenzione:** eliminare il file cancella definitivamente i dati.
```

È più utile di un uso casuale del grassetto.

## 24.3 Usare code fence per il codice

Per esempio:

````markdown
```python
print("Ciao")
```
````

invece di inserire codice lungo dentro un normale paragrafo.

## 24.4 Specificare il linguaggio

Preferire:

````markdown
```python
print("Ciao")
```
````

a:

````markdown
```
print("Ciao")
```
````

quando si conosce il linguaggio.

## 24.5 Scrivere testo alternativo per le immagini

Preferire:

```markdown
![Schema del database](database.png)
```

a:

```markdown
![](database.png)
```

quando è possibile fornire una descrizione utile.

## 24.6 Non creare documenti troppo complessi

Markdown nasce per essere semplice.

Se una soluzione richiede moltissimo HTML e numerose eccezioni, può essere opportuno chiedersi se Markdown sia davvero il formato più adatto.

---

# 25. Esercizi

## Esercizio 1 — Titoli

Trasformare il seguente testo in una struttura Markdown:

```text
Corso di informatica
Introduzione
Hardware
Software
Reti
```

### Soluzione

```markdown
# Corso di informatica

## Introduzione

## Hardware

## Software

## Reti
```

---

## Esercizio 2 — Formattazione

Scrivere una frase contenente:

- una parola in grassetto;
- una parola in corsivo;
- una parola barrata.

### Soluzione

```markdown
Questo è **importante**, questo è *secondario* e questo è ~~da eliminare~~.
```

---

## Esercizio 3 — Elenco

Creare un elenco degli strumenti necessari per una lezione di programmazione.

### Soluzione possibile

```markdown
## Strumenti

- Computer
- Editor di testo
- Browser
- Terminale
```

---

## Esercizio 4 — Procedura

Scrivere una procedura numerata per creare un nuovo file Markdown.

### Soluzione possibile

```markdown
## Creare un file Markdown

1. Aprire l'editor di testo.
2. Creare un nuovo documento.
3. Scrivere il contenuto.
4. Salvare il file con estensione `.md`.
```

---

## Esercizio 5 — Link

Creare un link al sito ufficiale di Python.

### Soluzione

```markdown
[Python](https://www.python.org/)
```

---

## Esercizio 6 — Immagine

Scrivere la sintassi per inserire un'immagine chiamata `logo.png`.

### Soluzione

```markdown
![Logo dell'applicazione](logo.png)
```

---

## Esercizio 7 — Codice

Inserire questo codice Python in un code fence:

```python
nome = "Luca"
print(nome)
```

### Soluzione

````markdown
```python
nome = "Luca"
print(nome)
```
````

---

## Esercizio 8 — Tabella

Creare una tabella con:

- Nome;
- Linguaggio;
- Livello.

### Soluzione possibile

```markdown
| Nome | Linguaggio | Livello |
|---|---|---|
| Anna | Python | Principiante |
| Luca | JavaScript | Intermedio |
| Marco | Java | Avanzato |
```

---

# 26. Progetto finale

Realizzare un piccolo **manuale tecnico in Markdown** di almeno una pagina.

Il documento deve contenere:

- [ ] un titolo principale;
- [ ] almeno tre sezioni;
- [ ] almeno un elenco puntato;
- [ ] almeno un elenco numerato;
- [ ] almeno un link;
- [ ] almeno un'immagine;
- [ ] almeno una citazione;
- [ ] almeno un esempio di codice;
- [ ] almeno una tabella;
- [ ] almeno una checklist;
- [ ] un piccolo indice;
- [ ] un riepilogo finale.

## Traccia suggerita

Argomento:

> **Introduzione a Python**

Struttura:

```markdown
# Introduzione a Python

## Indice

- [Cos'è Python](#cosè-python)
- [Variabili](#variabili)
- [Primo programma](#primo-programma)
- [Riepilogo](#riepilogo)

## Cos'è Python

Spiegazione...

## Variabili

Spiegazione...

## Primo programma

Esempio di codice...

## Tabella dei concetti

Tabella...

## Checklist

- [ ] Ho installato Python
- [ ] Ho scritto il primo programma
- [ ] Ho eseguito il codice

## Riepilogo

Conclusioni...
```

L'obiettivo non è soltanto completare gli elementi richiesti, ma creare un documento **leggibile, ordinato e facile da mantenere**.

---

# 27. Cheat sheet

Questa sezione può essere utilizzata come riferimento rapido.

## Testo

```markdown
*corsivo*
**grassetto**
***grassetto + corsivo***
~~barrato~~
`codice`
```

## Titoli

```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

## Elenchi

```markdown
- elemento
- elemento
```

```markdown
1. elemento
2. elemento
```

## Link

```markdown
[Testo](https://example.com)
```

## Immagini

```markdown
![Descrizione](immagine.png)
```

## Citazione

```markdown
> Citazione
```

## Codice

````markdown
```python
print("Ciao")
```
````

## Linea orizzontale

```markdown
---
```

## Tabella

```markdown
| Colonna 1 | Colonna 2 |
|---|---|
| Valore 1 | Valore 2 |
```

## Checklist

```markdown
- [ ] Da fare
- [x] Fatto
```

## Escape

```markdown
\*asterisco letterale\*
```

## Commento

```html
<!-- Commento non visualizzato -->
```

---

# Conclusione

Markdown è un linguaggio semplice, ma permette di creare documenti strutturati e professionali.

Per iniziare è sufficiente imparare bene:

1. titoli;
2. paragrafi;
3. grassetto e corsivo;
4. elenchi;
5. link;
6. immagini;
7. citazioni;
8. codice inline;
9. code fences;
10. tabelle.

Successivamente si possono aggiungere:

- checklist;
- collegamenti interni;
- HTML;
- funzionalità specifiche di GFM;
- tecniche di organizzazione dei documenti.

## La regola più importante

> **Scrivere Markdown significa prima di tutto descrivere la struttura del contenuto, non decorare il testo.**

Un buon documento Markdown dovrebbe essere:

- leggibile anche nel sorgente;
- strutturato;
- coerente;
- facilmente modificabile;
- comprensibile anche da chi non lo ha scritto.

---

## Riferimento rapido finale

| Obiettivo | Sintassi |
|---|---|
| Titolo | `# Titolo` |
| Corsivo | `*testo*` |
| Grassetto | `**testo**` |
| Barrato | `~~testo~~` |
| Codice inline | `` `codice` `` |
| Link | `[testo](url)` |
| Immagine | `![alt](url)` |
| Citazione | `> testo` |
| Elenco | `- elemento` |
| Numerazione | `1. elemento` |
| Linea | `---` |
| Checklist | `- [ ] attività` |
| Code fence | ```` ```linguaggio ```` |
| Tabella | `| A | B |` |

**Fine del manuale.**
