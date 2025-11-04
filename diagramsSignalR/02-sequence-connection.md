# Diagramma di Sequenza - Connessione Client

```mermaid
sequenceDiagram
    autonumber
    actor Client as Client Application
    participant SignalR as SignalR Client SDK
    participant Server as ASP.NET Core Server
    participant Hub as IwineHub
    participant Context as Hub Context
    
    Note over Client,Context: Fase di Connessione
    
    Client->>SignalR: Inizializza connessione
    SignalR->>Server: HTTP Negotiate Request
    Server-->>SignalR: Connection Token + TransportId
    
    SignalR->>Server: WebSocket Upgrade Request
    Server-->>SignalR: WebSocket Connection Established
    
    Server->>Hub: OnConnectedAsync()
    Hub->>Context: Ottieni ConnectionId
    Context-->>Hub: ConnectionId
    
    Hub->>Client: GetConnectionId(connectionId)
    Client-->>Hub: ACK
    
    Note over Client,Context: Client Connesso - Pronto per Messaggi
    
    rect rgb(200, 220, 240)
        Note over Client,Hub: Fase Operativa
        Client->>Hub: SendMessage(message)
        Hub->>Hub: Valida messaggio
        Hub->>Client: GetMessage(message)
        Client-->>Hub: Message Processed
    end
    
    rect rgb(240, 200, 200)
        Note over Client,Context: Disconnessione
        Client->>SignalR: Disconnect()
        SignalR->>Server: Close WebSocket
        Server->>Hub: OnDisconnectedAsync(exception)
        Hub->>Context: Rimuovi connessione
        Context-->>Hub: Connection Removed
    end
```

## Fasi della Connessione

### 1. Negoziazione (Steps 1-3)
Il client SignalR avvia una richiesta HTTP di negoziazione per ottenere:
- Token di connessione
- Tipo di trasporto disponibile (WebSocket, Server-Sent Events, Long Polling)
- Endpoint del server

### 2. Upgrade WebSocket (Steps 4-5)
Se supportato, il client richiede l'upgrade della connessione HTTP a WebSocket per comunicazione bidirezionale full-duplex.

### 3. Registrazione Hub (Steps 6-9)
Il metodo `OnConnectedAsync()` viene invocato automaticamente:
- Genera un ConnectionId univoco per il client
- Invia il ConnectionId al client tramite `GetConnectionId()`
- Il client può usare questo ID per comunicazioni dirette

### 4. Fase Operativa (Steps 10-13)
Il client può ora:
- Inviare messaggi tramite `SendMessage()`
- Ricevere messaggi tramite `GetMessage()`
- Mantenere la connessione persistente

### 5. Disconnessione (Steps 14-17)
Quando il client si disconnette:
- `OnDisconnectedAsync()` viene invocato
- Le risorse vengono rilasciate
- Il ConnectionId viene rimosso dal contesto
