# Diagramma di Sequenza - Broadcasting Messaggi

```mermaid
sequenceDiagram
    autonumber
    actor Sender as Client Mittente
    participant Hub as IwineHub
    participant Clients as Hub Clients Manager
    participant R1 as Client Ricevente 1
    participant R2 as Client Ricevente 2
    participant R3 as Client Ricevente 3
    
    Note over Sender,R3: Scenario: Broadcasting a tutti i client connessi
    
    Sender->>Hub: SendMessage(message)
    activate Hub
    
    Note over Hub: Validazione del messaggio<br/>- Action valida?<br/>- Route presente?<br/>- Params validi?
    
    Hub->>Clients: Clients.All.GetMessage(message)
    activate Clients
    
    par Invio parallelo a tutti i client
        Clients->>R1: GetMessage(message)
        activate R1
        Clients->>R2: GetMessage(message)
        activate R2
        Clients->>R3: GetMessage(message)
        activate R3
    end
    
    Note over R1,R3: Ogni client elabora il messaggio in modo indipendente
    
    par Elaborazione parallela
        R1->>R1: Processa messaggio<br/>secondo Action e Route
        R2->>R2: Processa messaggio<br/>secondo Action e Route
        R3->>R3: Processa messaggio<br/>secondo Action e Route
    end
    
    par Acknowledgment (opzionale)
        R1-->>Clients: ACK
        deactivate R1
        R2-->>Clients: ACK
        deactivate R2
        R3-->>Clients: ACK
        deactivate R3
    end
    
    Clients-->>Hub: Broadcast completato
    deactivate Clients
    
    Hub-->>Sender: Success (200)
    deactivate Hub
```

## Meccanismo di Broadcasting

### Caratteristiche del Broadcasting
1. **Fire-and-Forget**: Il mittente non attende conferma dai riceventi
2. **Invio Parallelo**: Tutti i client ricevono il messaggio simultaneamente
3. **Indipendenza**: Ogni client elabora il messaggio autonomamente

### Struttura del Messaggio
```json
{
  "Action": "notifica",
  "Name": "nuovo_ordine",
  "Route": "/ordini/123",
  "Params": {
    "orderId": 123,
    "status": "pending",
    "timestamp": "2025-11-03T10:30:00Z"
  }
}
```

### Casi d'Uso
- **Notifiche di sistema**: Aggiornamenti broadcast a tutti gli utenti
- **Eventi globali**: Cambio stato applicazione, manutenzione programmata
- **Sincronizzazione dati**: Aggiornamenti cache condivisa
- **Broadcasting eventi**: Nuovi ordini, messaggi pubblici, alert
