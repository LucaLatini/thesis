# Diagramma dei Componenti - Architettura Sistema

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web Application<br/>TypeScript/JavaScript]
        MOBILE[Mobile Application<br/>Native/Hybrid]
        DESKTOP[Desktop Application<br/>.NET/Electron]
    end
    
    subgraph "Transport Layer"
        WS[WebSocket Protocol<br/>ws://]
        SSE[Server-Sent Events<br/>http://]
        LP[Long Polling<br/>http://]
    end
    
    subgraph "SignalR Core Layer"
        SDK[SignalR Client SDK<br/>@microsoft/signalr]
        CONN[Connection Manager]
        PROTO[Protocol Handler<br/>JSON/MessagePack]
    end
    
    subgraph "Application Layer"
        CORS[CORS Middleware]
        AUTH[Authentication<br/>JWT/Bearer]
        HUB[IwineHub<br/>SignalR Hub]
        INTERFACE[IClient Interface<br/>Strongly-Typed]
    end
    
    subgraph "Business Layer"
        MSG[Message Handler]
        ROUTE[Routing Engine]
        VALID[Validation Service]
    end
    
    subgraph "Infrastructure Layer"
        CONFIG[Configuration<br/>appsettings.json]
        EXT[Service Extensions]
        LOG[Logging<br/>ILogger]
    end
    
    subgraph "Data Layer"
        DTO[Message DTO]
        CONTEXT[Hub Context]
    end
    
    %% Client to Transport
    WEB -.->|Prefer| WS
    WEB -.->|Fallback| SSE
    WEB -.->|Fallback| LP
    MOBILE -.->|Prefer| WS
    DESKTOP -.->|Prefer| WS
    
    %% Transport to SignalR Core
    WS --> SDK
    SSE --> SDK
    LP --> SDK
    SDK --> CONN
    CONN --> PROTO
    
    %% SignalR Core to Application
    PROTO --> CORS
    CORS --> AUTH
    AUTH --> HUB
    HUB -.->|Implements| INTERFACE
    
    %% Application to Business
    HUB --> MSG
    MSG --> ROUTE
    MSG --> VALID
    
    %% Business to Data
    ROUTE --> DTO
    VALID --> DTO
    HUB --> CONTEXT
    
    %% Infrastructure Dependencies
    CONFIG --> EXT
    EXT -.->|Configures| AUTH
    EXT -.->|Configures| CORS
    LOG -.->|Monitors| HUB
    LOG -.->|Monitors| MSG
    
    %% Styling
    classDef clientStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef transportStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef coreStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef appStyle fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef businessStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef infraStyle fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef dataStyle fill:#e0f2f1,stroke:#004d40,stroke-width:2px
    
    class WEB,MOBILE,DESKTOP clientStyle
    class WS,SSE,LP transportStyle
    class SDK,CONN,PROTO coreStyle
    class CORS,AUTH,HUB,INTERFACE appStyle
    class MSG,ROUTE,VALID businessStyle
    class CONFIG,EXT,LOG infraStyle
    class DTO,CONTEXT dataStyle
```

## Descrizione dei Livelli

### 1. Client Layer
Applicazioni client che utilizzano il sistema SignalR:
- **Web Application**: Client browser-based con TypeScript/JavaScript
- **Mobile Application**: App native o hybrid (React Native, Flutter, Xamarin)
- **Desktop Application**: Applicazioni desktop .NET o Electron

### 2. Transport Layer
Protocolli di trasporto supportati da SignalR con fallback automatico:
- **WebSocket**: Protocollo preferito per comunicazione bidirezionale full-duplex
- **Server-Sent Events**: Fallback per browser che non supportano WebSocket
- **Long Polling**: Fallback finale per massima compatibilità

### 3. SignalR Core Layer
Componenti core del framework SignalR:
- **Client SDK**: Libreria `@microsoft/signalr` per gestione connessioni
- **Connection Manager**: Gestione del ciclo di vita delle connessioni
- **Protocol Handler**: Serializzazione/deserializzazione messaggi (JSON/MessagePack)

### 4. Application Layer
Logica applicativa del server:
- **CORS Middleware**: Gestione Cross-Origin Resource Sharing
- **Authentication**: Validazione JWT token e autenticazione utenti
- **IwineHub**: Hub principale per routing messaggi
- **IClient Interface**: Contratto strongly-typed per comunicazione server-to-client

### 5. Business Layer
Logica di business per elaborazione messaggi:
- **Message Handler**: Gestione del ciclo di vita dei messaggi
- **Routing Engine**: Instradamento messaggi basato su Action/Route
- **Validation Service**: Validazione struttura e contenuto messaggi

### 6. Infrastructure Layer
Servizi di supporto e configurazione:
- **Configuration**: Gestione impostazioni da appsettings.json
- **Service Extensions**: Extension methods per configurazione servizi
- **Logging**: Sistema di logging centralizzato

### 7. Data Layer
Modelli dati e contesto:
- **Message DTO**: Data Transfer Object per messaggi
- **Hub Context**: Contesto runtime dell'hub con informazioni connessioni

## Flusso dei Dati

1. **Client → Server**: Client invia messaggio tramite SDK
2. **Transport**: Messaggio trasmesso via WebSocket (o fallback)
3. **Protocol**: Deserializzazione da JSON a oggetto Message
4. **Security**: Validazione CORS e autenticazione
5. **Hub**: Routing al metodo appropriato di IwineHub
6. **Business**: Elaborazione e validazione del messaggio
7. **Distribution**: Invio ai client destinatari tramite IClient interface
