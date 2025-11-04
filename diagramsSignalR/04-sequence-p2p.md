# Diagramma di Sequenza - Comunicazione Point-to-Point

```mermaid
sequenceDiagram
    autonumber
    actor Sender as Client A (Mittente)
    participant Hub as IwineHub
    participant Context as Connection Manager
    participant Receiver as Client B (Destinatario)
    
    Note over Sender,Receiver: Scenario: Invio messaggio diretto tra due client
    
    rect rgb(230, 240, 255)
        Note over Sender,Receiver: Fase 1: Richiesta ConnectionId del destinatario
        
        Sender->>Hub: SendConnectionId(receiverId, senderConnectionId)
        activate Hub
        
        Hub->>Context: Verifica esistenza receiverId
        activate Context
        Context-->>Hub: Client trovato
        deactivate Context
        
        Hub->>Receiver: GetConnectionId(senderConnectionId)
        activate Receiver
        
        Note over Receiver: Client B riceve l'ID di Client A<br/>e può salvarlo per comunicazioni future
        
        Receiver-->>Hub: ACK (ConnectionId ricevuto)
        deactivate Receiver
        Hub-->>Sender: Success
        deactivate Hub
    end
    
    rect rgb(240, 255, 230)
        Note over Sender,Receiver: Fase 2: Invio messaggio diretto
        
        Sender->>Hub: SendMessage(message)
        activate Hub
        
        Note over Hub: Estrae receiverId dai Params del messaggio
        
        Hub->>Context: GetClient(receiverId)
        activate Context
        
        alt Client connesso
            Context-->>Hub: Client proxy
            deactivate Context
            
            Hub->>Receiver: GetMessage(message)
            activate Receiver
            
            Note over Receiver: Elabora messaggio:<br/>- Verifica mittente<br/>- Processa Action<br/>- Aggiorna UI
            
            Receiver-->>Hub: Message processed
            deactivate Receiver
            
            Hub-->>Sender: Delivered (200)
            
        else Client disconnesso
            Context-->>Hub: Client non trovato
            deactivate Context
            Hub-->>Sender: Client offline (404)
            
            Note over Hub: Possibile implementazione:<br/>- Salva in coda messaggi<br/>- Notifica mittente<br/>- Retry automatico
        end
        
        deactivate Hub
    end
```

## Comunicazione Point-to-Point

### Vantaggi del P2P via SignalR
1. **Bassa Latenza**: Comunicazione diretta senza intermediari
2. **Real-time**: Delivery istantaneo quando entrambi i client sono online
3. **Scalabilità**: Il server gestisce solo il routing, non la business logic

### Flusso di Scambio ConnectionId

#### Perché scambiare ConnectionId?
- SignalR genera un ConnectionId univoco per ogni connessione
- I client devono conoscere il ConnectionId del destinatario per inviare messaggi diretti
- Il metodo `SendConnectionId()` facilita questo scambio

#### Esempio di Scambio
```typescript
// Client A richiede il ConnectionId di Client B
await connection.invoke("SendConnectionId", 
    "client-b-connection-id",  // receiverId
    myConnectionId             // senderId
);

// Client B riceve:
connection.on("GetConnectionId", (senderConnectionId) => {
    // Salva per comunicazioni future
    contactsConnectionMap.set("Client A", senderConnectionId);
});
```

### Gestione Messaggi Diretti

#### Struttura Message per P2P
```json
{
  "Action": "message",
  "Name": "chat_privato",
  "Route": "/chat/private",
  "Params": {
    "receiverId": "connection-id-destinatario",
    "senderId": "connection-id-mittente",
    "content": "Messaggio privato",
    "timestamp": "2025-11-03T10:30:00Z"
  }
}
```

### Gestione Disconnessioni
- Se il destinatario è offline, il messaggio non viene consegnato
- Implementazioni possibili:
  - **Message Queue**: Salvare messaggi per delivery posticipato
  - **Notification System**: Notificare via email/push
  - **Retry Logic**: Tentare reinvio automatico
