# Documentazione Diagrammi - Sistema SignalR iWine

Questa cartella contiene tutti i diagrammi Mermaid per documentare l'architettura e il funzionamento del sistema SignalR iWine, utilizzabili nella tesi di laurea.

## 📋 Indice dei Diagrammi

### 1. Diagramma delle Classi
**File:** `01-class-diagram.md`  
**Tipo:** Class Diagram  
**Descrizione:** Rappresenta la struttura delle classi principali del sistema, le loro relazioni e responsabilità.

**Quando usarlo nella tesi:**
- Capitolo sulla progettazione del sistema
- Sezione "Architettura del Software"
- Per spiegare il pattern Hub di SignalR

---

### 2. Diagramma di Sequenza - Connessione
**File:** `02-sequence-connection.md`  
**Tipo:** Sequence Diagram  
**Descrizione:** Mostra il flusso di connessione di un client al server SignalR, dall'handshake iniziale alla ricezione del ConnectionId.

**Quando usarlo nella tesi:**
- Capitolo sul "Protocollo di Comunicazione"
- Sezione "Gestione delle Connessioni"
- Per spiegare il ciclo di vita della connessione

---

### 3. Diagramma di Sequenza - Broadcasting
**File:** `03-sequence-broadcast.md`  
**Tipo:** Sequence Diagram  
**Descrizione:** Illustra il meccanismo di broadcasting dei messaggi a tutti i client connessi.

**Quando usarlo nella tesi:**
- Sezione "Comunicazione Many-to-Many"
- Capitolo sui "Pattern di Messaging"
- Per spiegare le notifiche globali

---

### 4. Diagramma di Sequenza - Point-to-Point
**File:** `04-sequence-p2p.md`  
**Tipo:** Sequence Diagram  
**Descrizione:** Descrive la comunicazione diretta tra due client specifici tramite il server.

**Quando usarlo nella tesi:**
- Sezione "Comunicazione One-to-One"
- Capitolo sulla "Messaggistica Privata"
- Per spiegare il routing dei messaggi diretti

---

### 5. Diagramma dei Componenti
**File:** `05-component-diagram.md`  
**Tipo:** Component Diagram  
**Descrizione:** Rappresenta l'architettura a livelli del sistema con tutti i componenti e le loro dipendenze.

**Quando usarlo nella tesi:**
- Capitolo "Architettura del Sistema"
- Sezione "Struttura Multi-Layer"
- Panoramica tecnologica del progetto

---

### 6. Diagramma di Deployment
**File:** `06-deployment-diagram.md`  
**Tipo:** Deployment Diagram  
**Descrizione:** Mostra come il sistema viene distribuito su server, inclusi load balancer, reverse proxy e opzioni di hosting.

**Quando usarlo nella tesi:**
- Capitolo "Deployment e Produzione"
- Sezione "Infrastruttura"
- Per spiegare scalabilità e load balancing

---

### 7. Diagramma di Stato
**File:** `07-state-diagram.md`  
**Tipo:** State Diagram  
**Descrizione:** Rappresenta tutti gli stati possibili di una connessione client e le transizioni tra essi.

**Quando usarlo nella tesi:**
- Capitolo "Gestione delle Connessioni"
- Sezione "Resilienza e Reconnection"
- Per spiegare la gestione degli errori

---

### 8. Diagramma di Flusso Dati
**File:** `08-data-flow-diagram.md`  
**Tipo:** Flowchart  
**Descrizione:** Mostra il flusso completo dei dati dall'invio del messaggio client alla ricezione ed elaborazione.

**Quando usarlo nella tesi:**
- Capitolo "Elaborazione dei Messaggi"
- Sezione "Pipeline di Comunicazione"
- Per spiegare validazione e routing

---

## 🎓 Struttura Consigliata per la Tesi

### Capitolo: Progettazione del Sistema

#### 3.1 Architettura Generale
- **Diagramma 5** (Component Diagram): Visione d'insieme dei livelli
- Descrizione dell'approccio multi-layer
- Tecnologie utilizzate per ogni livello

#### 3.2 Modello delle Classi
- **Diagramma 1** (Class Diagram): Struttura del codice
- Pattern architetturali applicati (Hub pattern, DTO)
- Relazioni tra le classi principali

#### 3.3 Protocollo di Comunicazione
- **Diagramma 2** (Sequence - Connection): Stabilire connessione
- **Diagramma 7** (State Diagram): Stati della connessione
- Gestione della resilienza e reconnection

#### 3.4 Pattern di Messaging
- **Diagramma 3** (Sequence - Broadcast): Comunicazione many-to-many
- **Diagramma 4** (Sequence - P2P): Comunicazione one-to-one
- **Diagramma 8** (Data Flow): Elaborazione messaggi

#### 3.5 Deployment e Scalabilità
- **Diagramma 6** (Deployment): Infrastruttura di produzione
- Opzioni di hosting e load balancing
- Strategie di scalabilità orizzontale

---

## 🛠️ Come Usare i Diagrammi

### Opzione 1: Mermaid Live Editor
1. Vai su [mermaid.live](https://mermaid.live)
2. Copia il codice dal file `.md`
3. Esporta come PNG/SVG per inserire nella tesi

### Opzione 2: VS Code
1. Installa l'estensione "Markdown Preview Mermaid Support"
2. Apri il file `.md`
3. Usa preview Markdown per vedere il diagramma
4. Screenshot o export

### Opzione 3: Integrazione LaTeX
Se usi LaTeX per la tesi:
```latex
\usepackage{mermaid}

\begin{figure}[h]
\centering
\includegraphics[width=\textwidth]{diagrams/01-class-diagram.pdf}
\caption{Diagramma delle classi del sistema SignalR iWine}
\label{fig:class-diagram}
\end{figure}
```

### Opzione 4: Word/Google Docs
1. Esporta diagrammi come immagini PNG ad alta risoluzione
2. Inserisci come figure nella tesi
3. Aggiungi didascalie descrittive

---

## 📊 Personalizzazione dei Diagrammi

Tutti i diagrammi sono completamente modificabili. Puoi:

- Cambiare colori modificando le classi CSS
- Aggiungere/rimuovere componenti
- Modificare testi e note
- Adattare il livello di dettaglio

### Esempio di Modifica Colori
```mermaid
classDef myStyle fill:#f9f,stroke:#333,stroke-width:2px
class MyClass myStyle
```

---

## 📖 Legenda Simboli

### Diagrammi delle Classi
- `+` : public
- `-` : private
- `#` : protected
- `<<interface>>` : interfaccia
- `<<abstract>>` : classe astratta
- `--|>` : ereditarietà
- `..>` : dipendenza/uso

### Diagrammi di Sequenza
- `→` : chiamata sincrona
- `--→` : risposta
- `-.→` : chiamata asincrona
- `rect` : raggruppamento logico
- `par` : esecuzione parallela

### Diagrammi di Stato
- `[*]` : stato iniziale/finale
- `→` : transizione
- `state X {}` : stato composito

---

## 💡 Suggerimenti per la Tesi

1. **Non sovraccaricare**: Usa 4-6 diagrammi chiave, non tutti e 8
2. **Contestualizza**: Ogni diagramma dovrebbe avere 1-2 pagine di spiegazione
3. **Progressive disclosure**: Inizia con overview (Component), poi dettagli (Sequence, Class)
4. **Coerenza**: Usa stessa terminologia in tutti i diagrammi
5. **Qualità**: Esporta immagini ad alta risoluzione (300 DPI minimo)

---

## 🔗 Riferimenti Esterni

- [Documentazione SignalR](https://docs.microsoft.com/aspnet/core/signalr)
- [Mermaid Documentation](https://mermaid.js.org/)
- [UML Best Practices](https://www.uml-diagrams.org/)

---

## ✅ Checklist per la Tesi

- [ ] Diagramma componenti per architettura generale
- [ ] Diagramma classi per design del codice
- [ ] Almeno un diagramma di sequenza per flussi principali
- [ ] Diagramma di stato per gestione connessioni
- [ ] Diagramma deployment per infrastruttura
- [ ] Tutti i diagrammi hanno didascalie chiare
- [ ] Diagrammi citati e spiegati nel testo
- [ ] Immagini ad alta risoluzione

---

**Versione:** 1.0  
**Data:** Novembre 2025  
**Autore:** Sistema SignalR iWine  
**Licenza:** Uso interno per tesi
