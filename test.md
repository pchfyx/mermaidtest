# Desain Arsitektur Microservice - Sistem Manajemen Kos

```mermaid
flowchart TD
    %% ── CLIENT LAYER ──
    A1["🖥️ Admin Web App\nOwner / Pengelola"]
    A2["📱 Browser / Mobile\nAkses Dashboard"]

    %% ── GATEWAY & SECURITY ──
    GW["⚡ Traefik API Gateway\nRouting · Load Balance · TLS"]
    MW["🔐 Auth Middleware\nValidates JWT on all routes"]

    %% ── CORE SERVICES ──
    S1["🔑 Auth Service\nLogin · Token"]
    S2["🏠 Room Service\nCRUD Kamar"]
    S3["👤 Tenant Service\nCRUD Penghuni"]
    S4["💳 Payment Service\nTagihan · Transaksi"]
    S5["🔧 Maintenance Service\nKeluhan · Tiket"]

    %% ── DATABASES ──
    DB1[("Auth DB")]
    DB2[("Room DB")]
    DB3[("Tenant DB")]
    DB4[("Payment DB")]
    DB5[("Maintenance DB")]

    %% ── REAL-TIME LAYER ──
    MQ["🐇 RabbitMQ / MQTT\nMessage Broker"]
    EQ["📬 Event Queue\nMessage Buffer"]
    NS["🔔 Notification Service\nAlerts · Status"]
    WS["🔌 WebSocket Service\nLive Push"]

    %% ── MONITORING ──
    CA["📦 cAdvisor\nContainer Metrics"]
    NE["🖧 Node Exporter\nHost Metrics"]
    PR["📊 Prometheus\nScrapes Metrics"]
    GR["📈 Grafana\nDashboard Visual"]

    %% ── FLOWS ──
    A1 -->|HTTPS| GW
    A2 -->|HTTPS| GW
    GW -->|REST| MW

    MW -->|REST| S1
    MW -->|REST| S2
    MW -->|REST| S3
    MW -->|REST| S4
    MW -->|REST| S5

    S1 --- DB1
    S2 --- DB2
    S3 --- DB3
    S4 --- DB4
    S5 --- DB5

    S4 -.->|publish event| MQ
    S5 -.->|publish event| MQ
    MQ -.->|buffer| EQ
    EQ -.->|consume| NS
    NS -.->|push| WS
    WS -.->|real-time| A1
    WS -.->|real-time| A2

    CA -->|metrics| PR
    NE -->|metrics| PR
    PR -->|visualize| GR
    CA -..->|scrapes| S1
    CA -..->|scrapes| S2
    CA -..->|scrapes| S3
    CA -..->|scrapes| S4
    CA -..->|scrapes| S5

    %% ── STYLING ──
    classDef client fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    classDef gateway fill:#534AB7,stroke:#3C3489,color:#fff
    classDef service fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    classDef database fill:#F1EFE8,stroke:#5F5E5A,color:#2C2C2A
    classDef realtime fill:#E1F5EE,stroke:#0F6E56,color:#085041
    classDef monitor fill:#FAEEDA,stroke:#854F0B,color:#633806

    class A1,A2 client
    class GW,MW gateway
    class S1,S2,S3,S4,S5 service
    class DB1,DB2,DB3,DB4,DB5 database
    class MQ,EQ,NS,WS realtime
    class CA,NE,PR,GR monitor
```

## Legend

| Style | Layer |
|---|---|
| 🟣 Purple | Client & Gateway / Security |
| 🔵 Blue | Core Business Services |
| ⬜ Gray | Isolated Databases |
| 🟢 Green | Real-time & Async Layer |
| 🟡 Amber | Monitoring Stack |

Solid arrows `-->` = REST API communication  
Dashed arrows `-.->` = WebSocket / Message Broker (async/real-time)

## Container Count: 22 Total

| Layer | Containers |
|---|---|
| Client | Admin Web App, Browser/Mobile |
| Gateway & Security | Traefik, Auth Middleware |
| Core Services | Auth, Room, Tenant, Payment, Maintenance |
| Databases | Auth DB, Room DB, Tenant DB, Payment DB, Maint. DB |
| Real-time | RabbitMQ, Event Queue, Notification, WebSocket |
| Monitoring | Prometheus, Grafana, cAdvisor, Node Exporter |
