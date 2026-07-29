# Sentrix Architecture and Flow Control

## Architecture Overview

```mermaid
flowchart LR
    subgraph Agent [Sentrix Agent]
        direction TB
        A1[cleint/main.py]
        A2[cleint/registration.py]
        A3[cleint/storage.py]
        A4[cleint/container_runner.py]
        A5[cleint/pipeline/streamer.py]
        A6[cleint/logger.py]
        A7[cleint/commands.py]
        A8[cleint/systeminfo.py]
        A9[cleint/watcher.py]
    end

    subgraph SessionManager [Session Manager]
        direction TB
        S1[docker/session-manager/app.py]
        S2[docker/session-manager/session.py]
        S3[docker/session-manager/auth.py]
        S4[Redis]
        S5[Keycloak]
    end

    subgraph Deployment [Docker Compose Stack]
        direction TB
        D1[suricata]
        D2[wazuh.manager]
        D3[elasticsearch]
        D4[kibana]
        D5[filebeat]
        D6[thehive]
        D7[session-manager]
        D8[threat-engine]
    end

    subgraph ThreatEngine [Threat Engine]
        direction TB
        T1[engines/threat_engine/v8_engine/main.py]
        T2[sentrix_core API Routers]
    end

    A1 -->|POST /api/register| S1
    A1 -->|POST /api/verify-session| S1
    A1 -->|POST /api/session/services| S1
    A1 -->|POST /api/session/containers| S1
    A1 -->|POST /api/heartbeat| S1
    A1 -->|POST /api/command| S1
    A1 -->|POST /api/command/result| S1
    A1 -->|POST /api/logs| S1

    S1 --> S2
    S1 --> S3
    S1 --> S4
    S3 --> S5

    S1 -->|container specs| A4
    S2 -->|session hash, provisioning| A1

    A4 -->|starts local containers| D1
    A4 -->|starts local containers| D2
    A4 -->|starts local containers| D8

    D1 -->|logs to| D5
    D2 -->|alerts to| D5
    D5 -->|writes to| D3
    D3 -->|serves dashboard| D4

    A5 -->|ingest events| T1
    T1 --> T2

    classDef agent fill:#f9f,stroke:#333,stroke-width:2px;
    classDef server fill:#bbf,stroke:#333,stroke-width:2px;
    classDef deployment fill:#bfb,stroke:#333,stroke-width:2px;
    classDef threat fill:#fbb,stroke:#333,stroke-width:2px;

    class Agent agent;
    class SessionManager server;
    class Deployment deployment;
    class ThreatEngine threat;
```

## Detailed Flow Control

```mermaid
flowchart TB
    subgraph AgentInit [Agent Initialization & Boot]
        direction TB
        Start[Start: cleint/main.py]
        LoadSession[Load session from APP_DATA_DIR/session.json]
        HasSession?{Session file exists?}
        Register[Register with server via cleint/registration.py]
        Verify[Verify session hash via cleint/storage.py]
        TamperCheck{Verification status}
        FetchServices[POST /api/session/services]
        FetchContainers[POST /api/session/containers]
        SetupContainers[cleint/container_runner.py -> ensure_container()]
        LaunchThreads[Start heartbeat, command poll, stream, watcher threads]
    end

    Start --> LoadSession
    LoadSession --> HasSession?
    HasSession? -->|No| Register
    Register --> FetchServices
    Register --> FetchContainers
    FetchContainers --> SetupContainers
    HasSession? -->|Yes| Verify
    Verify --> TamperCheck
    TamperCheck -->|valid| FetchServices
    TamperCheck -->|unreachable| FetchServices
    TamperCheck -->|unknown| Register
    TamperCheck -->|tampered| AlertAdmin[Report tamper and poll approval]
    AlertAdmin -->|approved| Register
    AlertAdmin -->|rejected| EndFail[Exit agent]
    FetchServices --> FetchContainers
    FetchContainers --> SetupContainers
    SetupContainers --> LaunchThreads

    subgraph Runtime [Runtime Loops]
        direction TB
        Heartbeat[Heartbeat thread]
        CommandPoll[Command poll thread]
        Streamer[Streamer loop]
        Watcher[File watcher]
    end

    LaunchThreads --> Heartbeat
    LaunchThreads --> CommandPoll
    LaunchThreads --> Streamer
    LaunchThreads --> Watcher

    Heartbeat -->|POST /api/heartbeat| S1[Session Manager]
    CommandPoll -->|POST /api/command| S1
    Streamer --> Collect[collect events from Suricata/Wazuh]
    Watcher --> Tail[tail logs from logs/suricata/eve.json]
    Collect --> Process[process(event) in cleint/pipeline/compute.py]
    Process --> Write[write(event) in cleint/pipeline/writer.py]
    Write --> ThreatIngest[send_to_Threat_engine in cleint/pipeline/threat_engine_client.py]
    Write --> DashboardIngest[send_to_dashboard]
    ThreatIngest -->|POST /api/v1/threat/events/ingest| T1[Threat Engine]
    DashboardIngest -->|POST /api/events| Dashboard[Dashboard service]
    CommandPoll -->|received command| Execute[cleint/commands.py handle_command()]
    Execute -->|POST result| S1

    subgraph Services [Service Routing & Normalization]
        direction LR
        ELK[Elasticsearch/Kibana]
        Wazuh[Wazuh Manager]
        Suricata[Suricata]
        Filebeat[Filebeat]
    end

    Suricata -->|eve.json logs| ELK
    Wazuh -->|alerts| ELK
    Filebeat -->|ship logs| ELK
    ELK -->|used by| Dashboard
    ELK -.->|optional API/alert stream| Streamer

    classDef agent fill:#fdf6e3,stroke:#333,stroke-width:1px;
    classDef runtime fill:#e7f5ff,stroke:#333,stroke-width:1px;
    classDef services fill:#e8ffe8,stroke:#333,stroke-width:1px;
    class AgentInit, Runtime agent;
    class Services services;
```
