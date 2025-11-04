# Diagramma di Flusso Dati - Elaborazione Messaggi

```mermaid
flowchart TD
    START([Client Invia Messaggio]) --> SERIALIZE[Serializzazione Messaggio<br/>Client-side]
    
    SERIALIZE --> TRANSPORT{Tipo di<br/>Trasporto?}
    
    TRANSPORT -->|WebSocket| WS[Invio via WebSocket<br/>Bidirezionale Full-Duplex]
    TRANSPORT -->|SSE| SSE_SEND[Invio via HTTP POST<br/>Ricezione via SSE]
    TRANSPORT -->|Long Polling| LP[Invio via HTTP POST<br/>Polling per risposta]
    
    WS --> SERVER_RECEIVE[Server Riceve Messaggio]
    SSE_SEND --> SERVER_RECEIVE
    LP --> SERVER_RECEIVE
    
    SERVER_RECEIVE --> DESERIALIZE[Deserializzazione<br/>JSON → Message Object]
    
    DESERIALIZE --> VALIDATE{Validazione<br/>Messaggio}
    
    VALIDATE -->|Invalid| ERROR_RESPONSE[Genera Errore<br/>400 Bad Request]
    ERROR_RESPONSE --> SEND_ERROR[Invia Errore al Client]
    SEND_ERROR --> END_ERROR([Fine - Errore])
    
    VALIDATE -->|Valid| ROUTE_CHECK{Analisi<br/>Route}
    
    ROUTE_CHECK -->|/broadcast| BROADCAST[Clients.All.GetMessage]
    ROUTE_CHECK -->|/direct/:id| EXTRACT_ID[Estrai receiverId<br/>da Params]
    ROUTE_CHECK -->|/group/:name| EXTRACT_GROUP[Estrai groupName<br/>da Params]
    
    EXTRACT_ID --> CHECK_CLIENT{Client<br/>Connesso?}
    
    CHECK_CLIENT -->|Yes| DIRECT[Clients.Client(id)<br/>.GetMessage]
    CHECK_CLIENT -->|No| QUEUE{Coda<br/>Messaggi?}
    
    QUEUE -->|Enabled| SAVE_QUEUE[Salva in Coda<br/>per Delivery Posticipato]
    QUEUE -->|Disabled| NOTIFY_OFFLINE[Notifica Mittente<br/>Client Offline]
    
    SAVE_QUEUE --> END_QUEUED([Fine - In Coda])
    NOTIFY_OFFLINE --> END_OFFLINE([Fine - Offline])
    
    EXTRACT_GROUP --> GROUP[Clients.Group(name)<br/>.GetMessage]
    
    BROADCAST --> PARALLEL_SEND[Invio Parallelo<br/>a tutti i Client]
    DIRECT --> SINGLE_SEND[Invio a<br/>Client Specifico]
    GROUP --> GROUP_SEND[Invio a<br/>Membri del Gruppo]
    
    PARALLEL_SEND --> CLIENT_RECEIVE[Client Riceve<br/>GetMessage Event]
    SINGLE_SEND --> CLIENT_RECEIVE
    GROUP_SEND --> CLIENT_RECEIVE
    
    CLIENT_RECEIVE --> CLIENT_DESERIALIZE[Deserializzazione<br/>Client-side]
    
    CLIENT_DESERIALIZE --> PROCESS{Tipo di<br/>Action?}
    
    PROCESS -->|notifica| UPDATE_UI[Aggiorna UI<br/>Mostra Notifica]
    PROCESS -->|message| DISPLAY_MSG[Visualizza Messaggio<br/>in Chat]
    PROCESS -->|update| REFRESH_DATA[Aggiorna Dati<br/>in Cache/Store]
    PROCESS -->|command| EXECUTE_CMD[Esegui Comando<br/>Applicazione]
    
    UPDATE_UI --> SEND_ACK{Richiesta<br/>ACK?}
    DISPLAY_MSG --> SEND_ACK
    REFRESH_DATA --> SEND_ACK
    EXECUTE_CMD --> SEND_ACK
    
    SEND_ACK -->|Yes| ACK_SERVER[Invia ACK al Server]
    SEND_ACK -->|No| END_SUCCESS([Fine - Success])
    
    ACK_SERVER --> END_SUCCESS
    
    style START fill:#e1f5ff
    style END_ERROR fill:#ffcdd2
    style END_QUEUED fill:#fff9c4
    style END_OFFLINE fill:#ffe0b2
    style END_SUCCESS fill:#c8e6c9
    
    style VALIDATE fill:#fff3e0
    style ROUTE_CHECK fill:#fff3e0
    style CHECK_CLIENT fill:#fff3e0
    style PROCESS fill:#fff3e0
    style SEND_ACK fill:#fff3e0
```

## Fasi del Flusso Dati

### 1. Client-Side: Preparazione e Invio

#### Serializzazione
```typescript
const message: Message = {
    Action: "notifica",
    Name: "nuovo_ordine",
    Route: "/orders/broadcast",
    Params: {
        orderId: 123,
        status: "pending"
    }
};

// SignalR automaticamente serializza in JSON
await connection.invoke("SendMessage", message);
```

#### Selezione Trasporto
SignalR sceglie automaticamente il miglior trasporto disponibile:
1. **WebSocket** (preferito): Full-duplex, bassa latenza
2. **Server-Sent Events**: Fallback per browser senza WebSocket
3. **Long Polling**: Fallback finale per massima compatibilità

### 2. Server-Side: Ricezione e Validazione

#### Deserializzazione
```csharp
public async Task SendMessage(Message message)
{
    // SignalR automaticamente deserializza da JSON a Message object
    // Binding automatico delle proprietà
}
```

#### Validazione
Controlli eseguiti:
- ✅ Message non null
- ✅ Action è una stringa valida
- ✅ Route è presente e ben formata
- ✅ Params è un oggetto valido (può essere null)

Se validazione fallisce:
```csharp
throw new HubException("Invalid message format");
// Client riceve errore e può gestirlo
```

### 3. Routing del Messaggio

#### Tipi di Route Supportate

**Broadcast (`/broadcast`)**
```csharp
await Clients.All.GetMessage(message);
```
- Invia a tutti i client connessi
- Uso: Notifiche globali, aggiornamenti di sistema

**Direct (`/direct/:connectionId`)**
```csharp
string receiverId = message.Params?.receiverId;
await Clients.Client(receiverId).GetMessage(message);
```
- Invia a un client specifico
- Uso: Chat 1-to-1, notifiche personali

**Group (`/group/:groupName`)**
```csharp
string groupName = message.Params?.groupName;
await Clients.Group(groupName).GetMessage(message);
```
- Invia a tutti i membri di un gruppo
- Uso: Chat di gruppo, team notifications

### 4. Gestione Client Offline

#### Strategia 1: Message Queue (da implementare)
```csharp
if (await IsClientConnected(receiverId))
{
    await Clients.Client(receiverId).GetMessage(message);
}
else
{
    await messageQueue.EnqueueAsync(receiverId, message);
    // Delivery alla prossima connessione
}
```

#### Strategia 2: Notifica Mittente
```csharp
if (!await IsClientConnected(receiverId))
{
    await Clients.Caller.GetMessage(new Message
    {
        Action = "error",
        Name = "recipient_offline",
        Params = new { receiverId, originalMessage = message }
    });
}
```

### 5. Client-Side: Ricezione e Elaborazione

#### Registrazione Handler
```typescript
connection.on("GetMessage", (message: Message) => {
    switch (message.Action) {
        case "notifica":
            showNotification(message);
            break;
        case "message":
            displayChatMessage(message);
            break;
        case "update":
            refreshData(message);
            break;
        case "command":
            executeCommand(message);
            break;
        default:
            console.warn("Unknown action:", message.Action);
    }
});
```

#### Elaborazione per Action Type

**Action: "notifica"**
```typescript
function showNotification(message: Message) {
    const notification = new Notification(message.Name, {
        body: message.Params?.text,
        icon: "/icon.png"
    });
}
```

**Action: "message"**
```typescript
function displayChatMessage(message: Message) {
    const chatWindow = document.getElementById("chat");
    chatWindow.innerHTML += `
        <div class="message">
            <span class="sender">${message.Params?.sender}</span>
            <p>${message.Params?.content}</p>
        </div>
    `;
}
```

**Action: "update"**
```typescript
function refreshData(message: Message) {
    // Aggiorna stato applicazione
    store.dispatch({
        type: "UPDATE_DATA",
        payload: message.Params
    });
}
```

**Action: "command"**
```typescript
function executeCommand(message: Message) {
    // Esegui comando applicativo
    const command = message.Params?.command;
    const handler = commandHandlers[command];
    if (handler) {
        handler(message.Params);
    }
}
```

### 6. Acknowledgment (Opzionale)

#### Server-Side Configuration
```csharp
public async Task<bool> SendMessageWithAck(Message message)
{
    await Clients.All.GetMessage(message);
    return true; // Conferma invio
}
```

#### Client-Side
```typescript
try {
    const ack = await connection.invoke("SendMessageWithAck", message);
    if (ack) {
        console.log("Messaggio inviato con successo");
    }
} catch (error) {
    console.error("Errore invio:", error);
}
```

## Ottimizzazioni e Best Practices

### 1. Message Batching
Raggruppa messaggi multipli in un unico invio:
```typescript
const messages = [msg1, msg2, msg3];
await connection.invoke("SendMessageBatch", messages);
```

### 2. Compressione
Per messaggi grandi, considera MessagePack invece di JSON:
```typescript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
```

### 3. Throttling/Debouncing
Limita la frequenza di invio messaggi:
```typescript
const throttledSend = throttle(
    (msg) => connection.invoke("SendMessage", msg),
    1000 // Max 1 messaggio/secondo
);
```

### 4. Error Handling
Gestisci errori in modo robusto:
```typescript
connection.on("GetMessage", async (message) => {
    try {
        await processMessage(message);
    } catch (error) {
        await connection.invoke("ReportError", {
            messageId: message.Params?.id,
            error: error.message
        });
    }
});
```
