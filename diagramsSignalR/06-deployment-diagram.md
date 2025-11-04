# Diagramma di Deployment

```mermaid
graph TB
    subgraph "Client Devices"
        BROWSER[Web Browser<br/>Chrome/Firefox/Safari]
        MOBILE_APP[Mobile Device<br/>iOS/Android]
        DESKTOP_APP[Desktop Client<br/>Windows/macOS/Linux]
    end
    
    subgraph "Load Balancer / Reverse Proxy"
        NGINX[NGINX / IIS<br/>Port 80/443]
        SSL[SSL/TLS Termination]
    end
    
    subgraph "Application Server"
        subgraph "ASP.NET Core Runtime"
            KESTREL[Kestrel Web Server<br/>Port 5000]
            SIGNALR_MW[SignalR Middleware]
            HUB_RUNTIME[Hub Runtime]
        end
        
        subgraph "Application Components"
            IWINE_HUB[IwineHub Instance]
            SERVICE_EXT[Service Extensions]
            CONFIG_MGR[Configuration Manager]
        end
        
        subgraph "Static Content"
            WWWROOT[wwwroot Folder<br/>Static Files]
            HTML[index.html]
            JS[JavaScript Bundles]
            CSS[CSS Styles]
        end
    end
    
    subgraph "Configuration & Logging"
        CONFIG_FILE[appsettings.json<br/>Configuration File]
        LOG_FILE[Application Logs<br/>Log Files]
    end
    
    subgraph "Windows Service"
        WIN_SERVICE[Windows Service Host<br/>Background Service]
    end
    
    %% Client Connections
    BROWSER -->|HTTPS/WSS| NGINX
    MOBILE_APP -->|HTTPS/WSS| NGINX
    DESKTOP_APP -->|HTTPS/WSS| NGINX
    
    %% Load Balancer to Server
    NGINX --> SSL
    SSL -->|HTTP/WS| KESTREL
    
    %% Server Internal Flow
    KESTREL --> SIGNALR_MW
    SIGNALR_MW --> HUB_RUNTIME
    HUB_RUNTIME --> IWINE_HUB
    
    KESTREL --> WWWROOT
    WWWROOT --> HTML
    WWWROOT --> JS
    WWWROOT --> CSS
    
    %% Configuration
    CONFIG_FILE -.->|Reads| CONFIG_MGR
    CONFIG_MGR -.->|Configures| SERVICE_EXT
    SERVICE_EXT -.->|Initializes| IWINE_HUB
    
    %% Logging
    IWINE_HUB -.->|Writes| LOG_FILE
    SIGNALR_MW -.->|Writes| LOG_FILE
    
    %% Windows Service
    WIN_SERVICE -.->|Hosts| KESTREL
    
    %% Styling
    classDef clientStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef proxyStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef serverStyle fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef runtimeStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef appStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef staticStyle fill:#e0f2f1,stroke:#004d40,stroke-width:2px
    classDef configStyle fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef serviceStyle fill:#ffebee,stroke:#b71c1c,stroke-width:2px
    
    class BROWSER,MOBILE_APP,DESKTOP_APP clientStyle
    class NGINX,SSL proxyStyle
    class KESTREL,SIGNALR_MW,HUB_RUNTIME runtimeStyle
    class IWINE_HUB,SERVICE_EXT,CONFIG_MGR appStyle
    class WWWROOT,HTML,JS,CSS staticStyle
    class CONFIG_FILE,LOG_FILE configStyle
    class WIN_SERVICE serviceStyle
```

## Architettura di Deployment

### 1. Livello Client
**Dispositivi Supportati:**
- Browser web moderni (Chrome, Firefox, Safari, Edge)
- Applicazioni mobile native (iOS/Android)
- Client desktop multi-piattaforma

**Protocolli:**
- HTTPS per chiamate HTTP
- WSS (WebSocket Secure) per comunicazione real-time

### 2. Load Balancer / Reverse Proxy
**Componenti:**
- **NGINX o IIS**: Reverse proxy per gestione traffico
- **SSL/TLS Termination**: Decrittazione HTTPS/WSS

**Funzionalità:**
- Terminazione SSL/TLS
- Bilanciamento del carico tra più istanze server
- Compressione response
- Caching statico
- Rate limiting

**Configurazione NGINX Esempio:**
```nginx
upstream signalr_backend {
    ip_hash;  # Importante per sticky sessions con SignalR
    server localhost:5000;
}

server {
    listen 443 ssl;
    server_name iwine.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location /hub {
        proxy_pass http://signalr_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 3. Application Server
**Runtime Environment:**
- ASP.NET Core 8.0
- Kestrel Web Server
- Windows Service Host (per deployment come servizio)

**Componenti:**
- **Kestrel**: Server web cross-platform ad alte prestazioni
- **SignalR Middleware**: Pipeline per gestione connessioni SignalR
- **Hub Runtime**: Ambiente di esecuzione per IwineHub
- **Static Files**: Servizio di file statici da wwwroot

### 4. Configuration Management
**File di Configurazione:**
```json
{
  "PathBase": "/",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore.SignalR": "Debug"
    }
  },
  "AllowedHosts": "*"
}
```

**Gestione Environment:**
- `appsettings.json`: Configurazione base
- `appsettings.Development.json`: Override per sviluppo
- `appsettings.Production.json`: Override per produzione

### 5. Windows Service Deployment
**Configurazione come Servizio:**
```powershell
# Pubblicazione applicazione
dotnet publish -c Release -r win-x64

# Installazione come servizio Windows
sc create IwineSignalRService binPath="C:\path\to\iwine-signalr.exe"
sc start IwineSignalRService
```

**Vantaggi:**
- Avvio automatico all'avvio di Windows
- Gestione tramite Services Manager
- Logging integrato con Event Viewer

## Opzioni di Deployment

### Opzione 1: Windows Server + IIS
- Hosting in IIS con modulo ASP.NET Core
- Gestione pool applicazioni
- Integrazione con Windows Authentication

### Opzione 2: Windows Service
- Esecuzione come servizio background
- Avvio automatico
- Gestione tramite sc.exe o PowerShell

### Opzione 3: Docker Container
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY bin/Release/net8.0/publish/ .
EXPOSE 5000
ENTRYPOINT ["dotnet", "iwine-signalr.dll"]
```

### Opzione 4: Azure App Service
- Deployment cloud managed
- Scaling automatico
- Integrazione con Azure SignalR Service per scale-out

## Considerazioni per Scalabilità

### Scale-Out con Backplane
Per deployment multi-server, implementare un backplane:

**Opzioni:**
- **Redis**: `services.AddSignalR().AddStackExchangeRedis(connection)`
- **Azure SignalR Service**: Servizio managed per scale-out automatico
- **SQL Server**: Backplane basato su database

### Sticky Sessions
Configurare sticky sessions nel load balancer:
- Basato su IP client (`ip_hash` in NGINX)
- Basato su cookie di sessione
- Necessario se non si usa un backplane
