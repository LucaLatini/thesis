# 🔄 Guida ai Diagrammi di Sequenza

Questa cartella contiene **5 diagrammi di sequenza** che mostrano i flussi temporali e l'interazione tra componenti del sistema MCP 3D Configurator.

---

## 📋 Indice dei Diagrammi

| # | File | Titolo | Complessità | Focus |
|---|------|--------|-------------|-------|
| 1 | `sequence_1_login_getprojects.mmd` | Login e Recupero Progetti | ⭐⭐ Media | Autenticazione + API REST |
| 2 | `sequence_2_create_open_project.mmd` | Creazione e Apertura Progetto | ⭐⭐⭐ Alta | SignalR + Dual-channel |
| 3 | `sequence_3_add_article_variants.mmd` | Aggiunta Articolo con Varianti | ⭐⭐⭐⭐ Molto Alta | GET-MODIFY-SAVE pattern |
| 4 | `sequence_4_signalr_bidirectional.mmd` | Comunicazione Bidirezionale | ⭐⭐⭐⭐ Molto Alta | SignalR full-duplex |
| 5 | `sequence_5_error_handling.mmd` | Gestione Errori e Casi Limite | ⭐⭐⭐ Alta | Error handling |

---

## 🎯 Come Usare i Diagrammi nella Tesi

### **Capitolo: Flussi di Lavoro e Use Cases**

---

### **1. Sequence Diagram: Login e Recupero Progetti** ⭐⭐
*Pagine suggerite: 3-4*

**File:** `sequence_1_login_getprojects.mmd`

**Cosa spiega:**
- Flusso autenticazione con Bearer Token
- Gestione credenziali (fornite vs. configurate)
- Auto-login con credenziali da appsettings.json
- Chiamate API autenticate
- Persistenza token per richieste successive

**Concetti da evidenziare:**
- **Pattern**: Bearer Token Authentication
- **Security**: Token memorizzato in variabile statica (trade-off)
- **User Experience**: Auto-login trasparente
- **Error Handling**: Credenziali invalide (401)

**Esempio testo per la tesi:**
> "Il flusso di autenticazione implementa il pattern Bearer Token, dove le credenziali vengono inviate alla API `/api/login` e la risposta contiene un token JWT. Questo token viene memorizzato in una variabile statica `_authToken` e riutilizzato per tutte le richieste successive tramite header HTTP `Authorization: Bearer <token>`. Il sistema supporta anche un meccanismo di auto-login che utilizza credenziali configurate nel file `appsettings.json`, permettendo un'esperienza utente più fluida quando il token non è disponibile..."

---

### **2. Sequence Diagram: Creazione e Apertura Progetto** ⭐⭐⭐
*Pagine suggerite: 4-5*

**File:** `sequence_2_create_open_project.mmd`

**Cosa spiega:**
- Integrazione dual-channel (API REST + SignalR)
- Comando "create" via SignalR con conferma lato client
- Flusso "open" con ricerca progetto + notifica client
- Interazione asincrona con configurator 3D
- Separazione responsabilità (backend vs. frontend)

**Concetti da evidenziare:**
- **Pattern**: Command Pattern (SignalR messages)
- **Pattern**: Repository Pattern (API per dati)
- **Architecture**: Separation of Concerns
- **Communication**: Asynchronous bidirectional

**Esempio testo:**
> "L'apertura di un progetto esemplifica l'architettura dual-channel del sistema. Prima, viene effettuata una ricerca nel database tramite API REST (`/muconf/plist`) per verificare l'esistenza del progetto e ottenere il suo ID. Successivamente, viene inviato un comando SignalR al client 3D con `action: 'open'` e `name: projectId`. Questa separazione permette al backend di gestire la logica di business (autenticazione, ricerca), mentre il frontend si occupa della visualizzazione 3D..."

---

### **3. Sequence Diagram: Aggiunta Articolo con Varianti** ⭐⭐⭐⭐
*Pagine suggerite: 5-6*

**File:** `sequence_3_add_article_variants.mmd`

**Cosa spiega:**
- Flusso completo a 3 fasi (Insert → Modify → Save)
- Pattern GET-MODIFY-SAVE per applicare varianti
- Parsing response per estrarre rowId
- Modifica del campo `pars` con valori ruleset
- Notifica client 3D per aggiornamento vista

**Concetti da evidenziare:**
- **Pattern**: GET-MODIFY-SAVE (Update Pattern)
- **Data Structure**: Campo `pars` come dizionario chiave-valore
- **Atomicity**: Operazione divisa in step separati (non transazionale)
- **Idempotency**: Possibilità di riapplicare varianti

**Esempio testo:**
> "L'aggiunta di un articolo con varianti rappresenta il flusso più complesso del sistema. Dopo l'inserimento dell'articolo base via `/muconf/radd`, che restituisce il `rowId` della riga creata, il sistema esegue un pattern GET-MODIFY-SAVE per applicare le varianti. Prima recupera lo stato corrente dell'articolo (`/muconf/rget`), poi modifica il campo `pars` applicando i valori specificati nel `ruleset` (es. `{cod: 'str', opz: 'c.01'}` per impostare il colore struttura a Bianco), infine salva le modifiche (`/muconf/rsave`). Questo approccio a tre fasi permette di sovrascrivere selettivamente solo i parametri desiderati, mantenendo invariati gli altri..."

---

### **4. Sequence Diagram: Comunicazione Bidirezionale SignalR** ⭐⭐⭐⭐
*Pagine suggerite: 5-6*

**File:** `sequence_4_signalr_bidirectional.mmd`

**Cosa spiega:**
- Lifecycle completo connessione SignalR
- Tentativo multipli endpoint (fallback strategy)
- Registrazione handler per messaggi in arrivo
- Comunicazione bidirezionale (AI → Client e Client → AI)
- Auto-reconnect con retry policy
- Gestione eventi (Closed, Reconnecting, Reconnected)

**Concetti da evidenziare:**
- **Pattern**: Observer Pattern (event handlers)
- **Pattern**: Retry Pattern (auto-reconnect)
- **Architecture**: Full-Duplex Communication
- **Resilience**: Automatic recovery

**Esempio testo:**
> "Il servizio SignalR implementa una comunicazione full-duplex che permette al sistema di inviare comandi al configurator 3D e ricevere risposte o notifiche spontanee. Il diagramma mostra il caso d'uso `ListElements`, dove l'AI invia una richiesta di ricerca nel catalogo, il client 3D elabora la query localmente e risponde con i risultati. Questa bidirezionalità è resa possibile dalla registrazione di handler tramite `connection.On('ReceiveMessage', handler)`, che implementa il pattern Observer. In caso di disconnessione, il sistema attiva automaticamente una strategia di retry con intervalli progressivi (0s, 2s, 5s, 10s), garantendo resilienza..."

---

### **5. Sequence Diagram: Gestione Errori e Casi Limite** ⭐⭐⭐
*Pagine suggerite: 4-5*

**File:** `sequence_5_error_handling.mmd`

**Cosa spiega:**
- 6 scenari di errore comuni:
  1. Token scaduto durante operazione
  2. Progetto non trovato
  3. SignalR offline
  4. Errore validazione API
  5. Errore applicazione varianti
  6. Network timeout
- Messaggi di errore strutturati con hint
- Graceful degradation
- Feedback utile per debugging

**Concetti da evidenziare:**
- **Pattern**: Fail-Fast with Helpful Messages
- **UX**: Error messages con suggerimenti contestuali
- **Resilience**: Partial success handling
- **Debugging**: Structured error responses

**Esempio testo:**
> "Il sistema implementa una gestione errori robusta che fornisce feedback dettagliato all'utente. Ad esempio, quando un token JWT scade durante un'operazione, il sistema non esegue automaticamente un re-login, ma informa l'utente con un messaggio strutturato che include status code (401), descrizione dell'errore e un hint operativo ('Effettua nuovo login'). Questo approccio fail-fast permette all'AI assistant di comprendere il problema e intraprendere l'azione correttiva appropriata. Un altro esempio è la gestione del caso 'progetto non trovato', dove la risposta include non solo l'errore, ma anche una lista dei primi 10 progetti disponibili come suggerimento..."

---

## 🎨 Elementi Grafici nei Diagrammi

### **Box colorati (rect):**
- 🟢 **Verde chiaro** (`rgb(240, 255, 240)`): Operazioni riuscite, flusso normale
- 🟡 **Giallo chiaro** (`rgb(255, 255, 240)`): Warning, riconnessione, fallback
- 🟠 **Arancio chiaro** (`rgb(255, 248, 240)`): Operazioni speciali, parsing
- 🔵 **Blu chiaro** (`rgb(240, 248, 255)`): Operazioni di salvataggio, persistenza
- 🔴 **Rosso chiaro** (`rgb(255, 240, 240)`): Errori, eccezioni, casi limite

### **Note:**
- Note in alto/basso per separare fasi logiche
- Note laterali per spiegare dettagli tecnici
- Note con emoji per identificare attori esterni (👤 👨‍💻 🌐 🔌 🎨)

### **Alternative (alt):**
- Mostrano branch condizionali (if/else)
- Evidenziano gestione errori
- Spiegano logica di fallback

### **Loop:**
- Mostrano iterazioni (es. per ogni variante)
- Mostrano retry logic

---

## 💡 Consigli per la Presentazione nella Tesi

### **Per ogni diagramma:**

1. **Introduzione (1 paragrafo)**
   ```
   "Il flusso di [operazione] coinvolge [N] componenti principali 
   e si articola in [N] fasi logiche..."
   ```

2. **Descrizione fasi (2-3 paragrafi)**
   - Fase 1: Cosa succede, quali componenti interagiscono
   - Fase 2: Elaborazione, trasformazione dati
   - Fase 3: Risultato, notifiche, persistenza

3. **Dettagli tecnici (1-2 paragrafi)**
   - Format dei messaggi JSON
   - Header HTTP utilizzati
   - Codici di stato rilevanti
   - Timeout e retry policy

4. **Pattern di design (1 paragrafo)**
   - Quali pattern sono implementati
   - Perché sono stati scelti
   - Vantaggi per manutenibilità/scalabilità

5. **Casi limite (1 paragrafo, se rilevante)**
   - Cosa succede in caso di errore
   - Strategie di recovery
   - Feedback all'utente

### **Esempio di struttura testo:**

```markdown
## 4.3 Caso d'Uso: Aggiunta Articolo con Varianti

### 4.3.1 Overview del Flusso
[Introduzione + Figura X: Sequence Diagram]

Il processo di aggiunta di un articolo con varianti rappresenta...

### 4.3.2 Fase 1: Inserimento Articolo Base
[Descrizione dettagliata + snippet JSON request]

L'operazione inizia con una chiamata POST a `/muconf/radd`...

### 4.3.3 Fase 2: Applicazione Varianti (Pattern GET-MODIFY-SAVE)
[Descrizione del pattern + snippet JSON pars]

Il sistema applica le varianti attraverso un pattern GET-MODIFY-SAVE...

**Sub-fase 2.1: GET - Recupero Stato Corrente**
...

**Sub-fase 2.2: MODIFY - Modifica Parametri**
...

**Sub-fase 2.3: SAVE - Persistenza**
...

### 4.3.4 Fase 3: Notifica Client 3D
[Descrizione interazione SignalR]

### 4.3.5 Considerazioni Tecniche
[Pattern di design, trade-off, performance]

### 4.3.6 Gestione Errori
[Riferimento a Sequence Diagram 5 - scenario 4 e 5]
```

---

## 📊 Tabella Riepilogativa Utilizzo

| Diagramma | Capitolo Tesi | Sezione | Obiettivo Didattico |
|-----------|---------------|---------|---------------------|
| Seq 1 | Cap. 4 - Use Cases | 4.1 Autenticazione | Spiegare Bearer Token Auth |
| Seq 2 | Cap. 4 - Use Cases | 4.2 Gestione Progetti | Mostrare dual-channel integration |
| Seq 3 | Cap. 4 - Use Cases | 4.3 Gestione Articoli | Spiegare pattern GET-MODIFY-SAVE |
| Seq 4 | Cap. 5 - Comunicazione | 5.1 SignalR Real-time | Spiegare full-duplex communication |
| Seq 5 | Cap. 6 - Quality Attributes | 6.1 Resilienza | Mostrare error handling robusto |

---

## ✅ Checklist per ogni Sequence Diagram

Prima di inserire un diagramma nella tesi, verifica:

- [ ] Il diagramma ha un titolo chiaro e numerazione figura
- [ ] La didascalia spiega cosa rappresenta il diagramma
- [ ] Nel testo c'è almeno un riferimento alla figura
- [ ] Le fasi principali sono evidenziate con note colorate
- [ ] I messaggi HTTP/SignalR hanno payload mostrato
- [ ] Gli errori sono gestiti con blocchi alt
- [ ] Le decisioni condizionali sono esplicite
- [ ] Gli attori sono identificati chiaramente (emoji)
- [ ] Il tempo scorre dall'alto verso il basso
- [ ] Le frecce indicano correttamente la direzione del messaggio
- [ ] I return sono rappresentati con linee tratteggiate
- [ ] I loop sono numerati o hanno condizione chiara

---

## 🎓 Ordine Consigliato nella Tesi

**Per progressione logica:**

1. Start → **Seq 1** (Login) - Base per tutto
2. Then → **Seq 2** (Create/Open) - Introduce SignalR
3. Then → **Seq 3** (Add Article) - Flusso più complesso
4. Then → **Seq 4** (Bidirectional) - Approfondisce SignalR
5. Finally → **Seq 5** (Errors) - Mostra robustezza

**Alternativa per topic:**
- **Cap. Autenticazione**: Seq 1
- **Cap. Gestione Progetti**: Seq 2
- **Cap. Gestione Articoli**: Seq 3
- **Cap. Comunicazione Real-time**: Seq 4
- **Cap. Quality Attributes**: Seq 5

---

## 📝 Snippet di Codice da Abbinare

Per rendere i diagrammi ancora più chiari, abbina snippet di codice reale:

### **Esempio per Seq 1 (Login):**
```csharp
// MarkunoApiTools.cs - Login method
var loginRequest = new { name = username, password = password };
var response = await _httpClient.PostAsJsonAsync($"{baseUrl}api/login", loginRequest);
var loginResponse = await response.Content.ReadFromJsonAsync<LoginResponse>();
_authToken = loginResponse.Data.User.Token;  // Salva token
```

### **Esempio per Seq 3 (Add Article - GET-MODIFY-SAVE):**
```csharp
// STEP 1: GET
var rgetResponse = await _httpClient.PostAsJsonAsync("muconf/rget", 
    new { id = projectId, idr = rowId, noeval = true });
var data = await rgetResponse.Content.ReadFromJsonAsync<dynamic>();

// STEP 2: MODIFY
var pars = data.pars as Dictionary<string, object>;
foreach (var rule in ruleset)
{
    pars[rule.Cod] = rule.Opz;  // Applica variante
}

// STEP 3: SAVE
await _httpClient.PostAsJsonAsync("muconf/rsave", 
    new { id = projectId, idr = rowId, pars = pars });
```

---

## 🚀 Export dei Diagrammi

Per inserirli nella tesi:

1. **Mermaid Live Editor**: https://mermaid.live/
2. **Copy code** → **Paste** nel editor
3. **Export as SVG** (vettoriale, migliore qualità) o **PNG** (300+ DPI)
4. Inserisci in Word/LaTeX con didascalia

### **Template didascalia:**
```
Figura 4.3 - Diagramma di sequenza: Aggiunta articolo con varianti.
Il diagramma mostra il flusso completo dall'invio della richiesta da parte 
dell'AI assistant fino alla visualizzazione dell'articolo nel configurator 3D, 
evidenziando il pattern GET-MODIFY-SAVE per l'applicazione delle varianti.
```

---

**Buona scrittura! 🎓📝**
