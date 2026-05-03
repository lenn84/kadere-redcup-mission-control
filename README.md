# Kadere — Agentless Infrastructure Observer

A lightweight Flask application that monitors distributed infrastructure **without installing agents** on remote servers. Kadere uses native protocols (SSH, SQL, Unix Sockets) to observe system health from a single dashboard.

Built for small teams and home labs that need visibility across multiple nodes without the overhead of traditional monitoring stacks.

---

## Features

- **Agentless monitoring** — no software to install on target systems
- **Hub-and-spoke topology** — central observer queries remote nodes on demand
- **Real-time dashboard** with animated health indicators
- **Action Center** — one-click remediation (cache flush, container restart, report generation)
- **Immutable audit logging** — JSON-formatted, append-only log trail
- **Web terminal** — browser-based admin console via WebSocket

---

## Architecture

```mermaid
graph TB
    subgraph Observer["Kadere Observer"]
        Dashboard[Real-time Dashboard]
        ActionCenter[Action Center]
        AuditLog[Audit Logger]
    end

    subgraph Zones["Monitored Zones"]
        AppNode[Application Node]
        DBNode[Database Node]
        Edge[Edge Relay]
        Sentinel[Threat Intelligence]
    end

    Dashboard -->|SSH| AppNode
    Dashboard -->|SQL / SSH| DBNode
    Dashboard -->|SSH| Edge
    Dashboard -->|HTTP API| Sentinel

    ActionCenter --> AppNode
    ActionCenter --> DBNode
    AuditLog -.->|append-only| LogStore[(Log Storage)]

    style Observer fill:#1a1a2e,stroke:#e94560,color:#fff
    style Zones fill:#0f3460,stroke:#16213e,color:#fff
```

### Monitoring Zones

| Zone | Access Method | What's Monitored |
|------|---------------|------------------|
| Application | Unix Socket | Container health, resource usage |
| Edge | SSH (Ed25519) | TCP connections, latency |
| Database | SQL over SSH | Storage pool health, database sizes |
| Security | HTTP API | IP bans, threat intelligence feeds |

---

## Security Model

```mermaid
graph LR
    Internet((Internet)) -->|HTTPS| Proxy[Reverse Proxy + WAF]
    Proxy -->|Identity Check| Auth[Identity Provider]
    Auth -->|Validated Headers| Kadere[Kadere App]
    Kadere -->|Read-only| Docker[Container Runtime]

    style Internet fill:#e94560,stroke:#333,color:#fff
    style Proxy fill:#16213e,stroke:#e94560,color:#fff
    style Auth fill:#0f3460,stroke:#e94560,color:#fff
    style Kadere fill:#1a1a2e,stroke:#e94560,color:#fff
```

1. **Zero Trust Networking** — no public ports; internal service mesh only
2. **Identity Proxy** — reverse proxy validates identity before traffic reaches the app
3. **Read-Only Mounts** — container runtime socket mounted read-only; secrets injected at runtime

---

## Deployment

Kadere deploys through a GitOps pipeline:

```mermaid
graph LR
    Push[Code Push] --> Webhook[Webhook Trigger]
    Webhook --> Build[Container Build]
    Build --> Registry[Private Registry]
    Registry --> Deploy[Production Deploy]

    style Push fill:#1a1a2e,stroke:#e94560,color:#fff
    style Deploy fill:#16213e,stroke:#e94560,color:#fff
```

1. Push to source control
2. Webhook triggers an ephemeral build container
3. Image pushed to private registry
4. Admin triggers deployment to production

---

## Setup

### Prerequisites
- Python 3.10+
- Docker & Docker Compose
- SSH key pair (Ed25519 recommended)
- Access to target nodes

### Quick Start

```bash
# Clone the repository
git clone https://github.com/lenn84/kadere-redcup-mission-control.git
cd kadere-redcup-mission-control

# Copy environment template
cp .env.example .env
# Edit .env with your node addresses and credentials

# Build and run
docker compose up -d
```

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `KADERE_SECRET_KEY` | Flask session secret | `your-random-secret` |
| `SSH_KEY_PATH` | Path to SSH private key | `/secrets/id_ed25519` |
| `DB_CONNECTION` | Database connection string | `postgresql://user:pass@db-host/kadere` |
| `LOG_PATH` | Audit log output path | `/data/audit.log` |

---

## Usage

### Dashboard
Access the dashboard at your configured URL. Health indicators update in real time — green (healthy), amber (degraded), red (critical).

### Action Center
One-click operations:
- **Flush Cache** — clear application caches across nodes
- **Restart Container** — restart a specific service container
- **Generate Report** — export system health snapshot

### Web Terminal
Admin console available for authenticated users. Provides direct shell access to the observer node through WebSocket.

---

## Limitations

- Designed for small-to-medium infrastructure (tested with up to 10 nodes)
- SSH-based monitoring introduces latency compared to agent-based push models
- Web terminal requires WebSocket support in your reverse proxy

---

## License

MIT
