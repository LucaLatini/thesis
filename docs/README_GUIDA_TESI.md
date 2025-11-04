# 📖 Guida alla Documentazione UML per la Tesi

Questo documento spiega come utilizzare i diagrammi UML/Mermaid nella tesi, suddivisi per complessità crescente.

---

## 🎯 Struttura Consigliata per la Tesi

### **Capitolo: Architettura del Sistema**

#### **1. Introduzione all'Architettura** 
*Pagine suggerite: 2-3*

**Diagramma da usare:** `diagram_1_overview.mmd`

**Cosa spiegare:**
- Visione generale del sistema a 3 livelli:
  1. **MCP Server** (MarkunoApiTools) - espone funzionalità all'AI
  2. **Service Layer** (SignalRService) - comunicazione real-time
  3. **Sistemi Esterni** (API REST + Configurator 3D)
- Pattern architetturale: **Layered Architecture** + **Dependency Injection**
- Protocolli di comunicazione: HTTP/REST (API) + WebSocket (SignalR)

**Esempio testo:**
> "L'architettura del sistema si basa su una struttura a livelli che separa le responsabilità. Il componente centrale `MarkunoApiTools` espone gli strumenti MCP che permettono all'assistente AI di interagire con il sistema Markuno. La comunicazione con il configurator 3D avviene tramite `SignalRService` che gestisce una connessione WebSocket bidirezionale..."

---

#### **2. Layer di Configurazione e Bootstrap**
*Pagine suggerite: 2-3*

**Diagramma da usare:** `diagram_2_configuration.mmd`

**Cosa spiegare:**
- Processo di startup dell'applicazione
- Pattern **Dependency Injection** con .NET
- Configurazione esternalizzata (appsettings.json)
- Registrazione servizi (Singleton, Transient, Scoped)
- MCP Server setup con trasporto STDIO

**Esempio testo:**
> "Il punto di ingresso dell'applicazione (`Program.cs`) implementa il pattern Host Builder di .NET, configurando tutti i servizi necessari. La configurazione è esternalizzata nel file `appsettings.json`, che contiene le credenziali di accesso, l'URL dell'API Markuno e i parametri per la connessione SignalR. Il `SignalRService` viene registrato come Singleton per mantenere una singola connessione persistente..."

---

#### **3. Servizio SignalR per Comunicazione Real-Time**
*Pagine suggerite: 3-4*

**Diagramma da usare:** `diagram_3_signalr_service.mmd`

**Cosa spiegare:**
- Cos'è SignalR e perché è stato scelto
- Gestione della connessione (lifecycle)
- Pattern **Retry** e riconnessione automatica
- Gestione eventi (Closed, Reconnecting, Reconnected)
- Supporto messaggi bidirezionali
- Strategia multi-endpoint fallback

**Esempio testo:**
> "SignalR è un framework per comunicazione real-time che astrae il protocollo WebSocket. Il `SignalRService` implementa una logica robusta di connessione che tenta automaticamente multipli endpoint configurati. In caso di disconnessione, il servizio implementa una strategia di retry con intervalli progressivi (0s, 2s, 5s, 10s)..."

---

#### **4. Gestione Autenticazione**
*Pagine suggerite: 2-3*

**Diagramma da usare:** `diagram_4_authentication.mmd`

**Cosa spiegare:**
- Flusso di autenticazione con API Markuno
- Pattern **Bearer Token Authentication**
- Gestione token in memoria (variabile statica)
- Auto-login con credenziali configurate
- Modello User e livelli di autorizzazione

**Esempio testo:**
> "L'autenticazione avviene tramite una chiamata POST all'endpoint `/api/login` con username e password. La risposta contiene un token JWT che viene memorizzato in una variabile statica e utilizzato per tutte le richieste successive tramite header `Authorization: Bearer <token>`. Il sistema supporta anche un meccanismo di auto-login che utilizza credenziali configurate in `appsettings.json`..."

---

#### **5. Gestione Progetti**
*Pagine suggerite: 3-4*

**Diagramma da usare:** `diagram_5_projects.mmd`

**Cosa spiegare:**
- Modello dati Project
- Operazioni CRUD sui progetti
- Pattern **DTO (Data Transfer Object)**
- Integrazione dual-channel (API REST + SignalR)
- Filtraggio progetti (onlyMy)

**Esempio testo:**
> "La gestione progetti combina due canali di comunicazione: le API REST per il recupero e la ricerca dei progetti, e SignalR per operazioni che richiedono aggiornamento del client 3D. Il modello `Project` contiene sia informazioni anagrafiche del cliente che metadati del progetto (timestamp, stato, importo). L'operazione `OpenProject` esemplifica questa dual-channel integration..."

---

#### **6. Gestione Articoli e Varianti**
*Pagine suggerite: 4-5*

**Diagramma da usare:** `diagram_6_articles_variants.mmd`

**Cosa spiegare:**
- Sistema di catalogo e articoli
- Concetto di varianti/regole (colori, finiture, materiali)
- Modello RuleSetItem e VariantOptions
- Flusso GET-MODIFY-SAVE per applicare varianti
- Ricerca nel catalogo

**Esempio testo:**
> "Ogni articolo nel sistema può avere varianti che ne modificano l'aspetto o le caratteristiche (colore, finitura, materiale). Il modello `RuleSetItem` rappresenta una singola variante con un codice parametro (`cod`) e un valore (`opz`). L'applicazione di varianti segue un pattern GET-MODIFY-SAVE: prima si recupera lo stato corrente dell'articolo (`/muconf/rget`), poi si modificano i parametri nel campo `pars`, infine si salva (`/muconf/rsave`)..."

---

#### **7. Comunicazione SignalR e Messaggi**
*Pagine suggerite: 3-4*

**Diagramma da usare:** `diagram_7_signalr_messages.mmd`

**Cosa spiegare:**
- Formato standard dei messaggi (Message model)
- Pattern **Command** per azioni sul configurator
- Gestione messaggi in arrivo (handler registration)
- Azioni supportate (open, create, list_elements, add_article)
- Pattern **Observer** per eventi

**Esempio testo:**
> "La comunicazione con il configurator 3D avviene tramite messaggi strutturati che seguono il pattern Command. Ogni messaggio contiene un campo `Action` che specifica l'operazione richiesta, un campo `Name` per parametri semplici, e un campo `Params` per dati complessi. Il servizio SignalR implementa anche un pattern Observer permettendo la registrazione di handler per messaggi in arrivo..."

---

### **Capitolo: Flussi di Lavoro Principali**

#### **8. Sequence Diagram: Aggiunta Articolo con Varianti**
*Pagine suggerite: 2-3*

**Diagramma da usare:** `diagram_8_sequence_add_article.mmd`

**Cosa spiegare:**
- Flusso temporale completo
- Interazione tra componenti
- Gestione errori e casi alternativi
- Validazione CVE delle dipendenze (se applicabile)

---

#### **9. Sequence Diagram: Apertura Progetto**
*Pagine suggerite: 2-3*

**Diagramma da usare:** `diagram_9_sequence_open_project.mmd`

**Cosa spiegare:**
- Coordinazione API REST + SignalR
- Ricerca case-insensitive
- Gestione fallimenti (progetto non trovato, SignalR offline)
- User experience (feedback immediato)

---

## 📊 Riepilogo Ordine Consigliato

| # | Diagramma | Tipo | Complessità | Pagine |
|---|-----------|------|-------------|--------|
| 1 | Overview | Class | ⭐ Bassa | 2-3 |
| 2 | Configuration | Class | ⭐⭐ Media | 2-3 |
| 3 | SignalR Service | Class | ⭐⭐⭐ Media-Alta | 3-4 |
| 4 | Authentication | Class | ⭐⭐ Media | 2-3 |
| 5 | Projects | Class | ⭐⭐⭐ Media-Alta | 3-4 |
| 6 | Articles & Variants | Class | ⭐⭐⭐⭐ Alta | 4-5 |
| 7 | SignalR Messages | Class | ⭐⭐⭐ Media-Alta | 3-4 |
| 8 | Seq: Add Article | Sequence | ⭐⭐⭐⭐ Alta | 2-3 |
| 9 | Seq: Open Project | Sequence | ⭐⭐⭐ Media-Alta | 2-3 |

**Totale stimato:** 25-33 pagine per capitolo architettura

---

## 💡 Consigli Pratici

### **Per ogni diagramma:**

1. **Introduzione (1 paragrafo)**
   - Cosa rappresenta il diagramma
   - Perché è importante
   - Come si collega al diagramma precedente

2. **Spiegazione componenti (2-3 paragrafi)**
   - Descrizione di ogni classe/componente
   - Responsabilità principali
   - Pattern di design utilizzati

3. **Relazioni (1-2 paragrafi)**
   - Come i componenti interagiscono
   - Direzione delle dipendenze
   - Motivazioni delle scelte architetturali

4. **Considerazioni tecniche (1 paragrafo)**
   - Tecnologie utilizzate
   - Vantaggi della soluzione
   - Eventuali limitazioni

### **Formato delle immagini:**
- Esporta i diagrammi come **SVG** (scalabili, perfetti per PDF)
- In alternativa **PNG ad alta risoluzione** (300 DPI minimo)
- Usa didascalie descrittive per ogni figura

### **Stile di scrittura:**
- Usa **tempo presente** per descrivere l'architettura
- Usa **terminologia tecnica** ma spiega acronimi alla prima occorrenza
- Aggiungi **riferimenti incrociati** tra sezioni
- Includi **snippet di codice** dove appropriato (max 10-15 righe)

---

## 🎓 Esempio Struttura Capitolo

```markdown
## 3. Architettura del Sistema

### 3.1 Visione d'Insieme
[Diagramma 1 + spiegazione 2-3 pagine]

### 3.2 Layer di Configurazione
[Diagramma 2 + spiegazione 2-3 pagine]

### 3.3 Servizio di Comunicazione Real-Time
[Diagramma 3 + spiegazione 3-4 pagine]

### 3.4 Sistema di Autenticazione
[Diagramma 4 + spiegazione 2-3 pagine]

### 3.5 Gestione Progetti
[Diagramma 5 + spiegazione 3-4 pagine]

### 3.6 Gestione Articoli e Varianti
[Diagramma 6 + spiegazione 4-5 pagine]

### 3.7 Protocollo di Messaggistica SignalR
[Diagramma 7 + spiegazione 3-4 pagine]

## 4. Flussi di Lavoro Principali

### 4.1 Caso d'Uso: Aggiunta Articolo
[Diagramma 8 + spiegazione 2-3 pagine]

### 4.2 Caso d'Uso: Apertura Progetto
[Diagramma 9 + spiegazione 2-3 pagine]
```

---

## ✅ Checklist Finale

- [ ] Ogni diagramma ha una figura numerata (es. "Figura 3.1")
- [ ] Ogni figura ha una didascalia esplicativa
- [ ] Nel testo c'è almeno un riferimento esplicito alla figura
- [ ] I termini tecnici sono spiegati o nel glossario
- [ ] I pattern di design sono identificati e motivati
- [ ] Le scelte architetturali sono giustificate
- [ ] I diagrammi seguono una progressione logica
- [ ] C'è coerenza nella terminologia usata
- [ ] Le relazioni tra componenti sono chiare
- [ ] Ci sono esempi concreti dove utile

---

**Buona scrittura della tesi! 🎓📝**
