```mermaid
flowchart TD
    %% Clients
    Admin["Admin web app<br><small>Owner / pengelola</small>"]
    Client["Browser / mobile<br><small>Akses dashboard</small>"]

    %% Gateway & Security
    Traefik["Traefik API Gateway<br><small>Routing • load-balance • TLS</small>"]
    AuthMid["Auth middleware<br><small>Validates JWT on all routes</small>"]

    %% Core Microservices & Databases
    subgraph Services ["Core Microservices"]
        direction TB
        AuthSvc["Auth service<br><small>Login • token</small>"] --> AuthDB[("Auth DB")]
        RoomSvc["Room service<br><small>CRUD kamar</small>"] --> RoomDB[("Room DB")]
        TenantSvc["Tenant service<br><small>CRUD penghuni</small>"] --> TenantDB[("Tenant DB")]
        PaymentSvc["Payment service<br><small>Tagihan</small>"] --> PaymentDB[("Payment DB")]
        MaintSvc["Maintenance<br><small>Keluhan</small>"] --> MaintDB[("Maint. DB")]
    end

    %% Async & Real-time Layer
    subgraph Async ["Real-time & Async Layer"]
        Broker["RabbitMQ / MQTT<br><small>Message broker</small>"]
        Queue["Event queue<br><small>Message buffer</small>"]
        NotifSvc["Notification service<br><small>Alerts • overdue • status</small>"]
        WS["WebSocket<br><small>Live push</small>"]

        Broker --> Queue
        Queue --> NotifSvc
        NotifSvc --- WS
    end

    %% Monitoring Stack
    subgraph Monitoring ["Monitoring Stack"]
        Prom["Prometheus<br><small>Scrapes metrics</small>"]
        Grafana["Grafana<br><small>Dashboard visual</small>"]
        cAdv["cAdvisor<br><small>Container metrics</small>"]
        NodeExp["Node Exporter<br><small>Host metrics</small>"]
        
        cAdv --> Prom
        NodeExp --> Prom
        Prom --> Grafana
    end

    %% Connections: Client to Gateway
    Admin -->|REST API| Traefik
    Client -->|REST API| Traefik
    Traefik --> AuthMid

    %% Connections: Gateway to Services
    AuthMid --> AuthSvc
    AuthMid --> RoomSvc
    AuthMid --> TenantSvc
    AuthMid --> PaymentSvc
    AuthMid --> MaintSvc

    %% Connections: DBs to Async Layer
    RoomDB -.-> Broker
    TenantDB -.-> Broker
    PaymentDB -.-> Broker
    MaintDB -.-> Broker

    %% Connections: Live Push Back to Client
    WS -.->|WebSocket / Broker| Client

    %% Custom Styling to match your image color scheme
    classDef client fill:#ffffff,stroke:#333333,stroke-width:1px;
    classDef gateway fill:#f3f0ff,stroke:#7451eb,stroke-width:2px;
    classDef service fill:#e6f7ff,stroke:#1890ff,stroke-width:1.5px;
    classDef db fill:#ffffff,stroke:#bfbfbf,stroke-width:1.5px;
    classDef async fill:#f6ffed,stroke:#52c41a,stroke-width:1.5px;
    classDef monitor fill:#fffbe6,stroke:#faad14,stroke-width:1.5px;
    classDef monitorWhite fill:#ffffff,stroke:#bfbfbf,stroke-width:1.5px;

    class Admin,Client client;
    class Traefik,AuthMid gateway;
    class AuthSvc,RoomSvc,TenantSvc,PaymentSvc,MaintSvc service;
    class AuthDB,RoomDB,TenantDB,PaymentDB,MaintDB db;
    class Broker,Queue,NotifSvc,WS async;
    class Prom,Grafana monitor;
    class cAdv,NodeExp monitorWhite;
