# Diagramma delle Classi - Sistema SignalR iWine

```mermaid
classDiagram
    class IwineHub {
        <<Hub~IClient~>>
        +SendMessage(Message message) Task
        +SendConnectionId(string receiverId, string senderId) Task
        -Clients: IHubCallerClients~IClient~
        -Context: HubCallerContext
    }
    
    class IClient {
        <<interface>>
        +GetMessage(Message message) Task
        +GetConnectionId(string connectionId) Task
    }
    
    class Message {
        <<data>>
        +string Action
        +string Name
        +string Route
        +object Params
    }
    
    class Hub~T~ {
        <<abstract>>
        #Clients: IHubCallerClients~T~
        #Context: HubCallerContext
        #Groups: IGroupManager
        +OnConnectedAsync() Task
        +OnDisconnectedAsync(Exception) Task
    }
    
    class Program {
        <<static>>
        +Main(string[] args) void
        -ConfigureServices(WebApplicationBuilder)
        -ConfigureMiddleware(WebApplication)
    }
    
    class IServiceCollectionExtensions {
        <<static>>
        +AddCustomAuthentication(IServiceCollection) IServiceCollection
        +AddCustomAuthorization(IServiceCollection) IServiceCollection
    }
    
    %% Relazioni
    IwineHub --|> Hub~T~ : extends
    IwineHub ..> IClient : uses
    IwineHub ..> Message : uses
    IClient ..> Message : receives
    Program ..> IServiceCollectionExtensions : uses
    Program ..> IwineHub : registers

    note for IwineHub "Hub principale per la gestione\ndelle comunicazioni real-time\ntra i client iWine"
    note for Message "DTO per il trasferimento\ndi messaggi tra client\ncon routing e parametri"
```

## Descrizione delle Classi

### IwineHub
Il componente centrale del sistema che gestisce tutte le comunicazioni SignalR. Estende `Hub<IClient>` per fornire funzionalità type-safe di comunicazione con i client.

**Responsabilità:**
- Broadcasting di messaggi a tutti i client connessi
- Invio di notifiche point-to-point tra client specifici
- Gestione del ciclo di vita delle connessioni

### IClient
Interfaccia che definisce i metodi che il server può invocare sui client connessi. Implementa il pattern Strongly-Typed Hub.

**Metodi:**
- `GetMessage`: Riceve messaggi broadcast o diretti
- `GetConnectionId`: Riceve l'identificativo di connessione di un altro client

### Message
Data Transfer Object (DTO) che incapsula tutte le informazioni necessarie per il routing e l'elaborazione dei messaggi.

**Proprietà:**
- `Action`: Tipo di azione da eseguire
- `Name`: Nome del messaggio/evento
- `Route`: Percorso di destinazione del messaggio
- `Params`: Parametri dinamici associati al messaggio
