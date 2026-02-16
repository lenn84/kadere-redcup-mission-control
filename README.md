# 🏗️ Architecture: The "Hub & Spoke" Model

- Kadere is a specialized Flask application acting as a Federated Observer.
- Unlike traditional monitoring tools that require agents installed on every server, Kadere is Agentless. It runs on the App Node (LXC 1002) but reaches out to monitor the entire infrastructure using native protocols (SSH, SQL, Unix Sockets).
---

```mermaid
graph TD
    %% The Hub
    subgraph "LXC 1002: App Node"
        KAD[🕹️ Kadere Dashboard]
        SOCK[🐳 Docker Socket]
        AUDIT[💾 Audit Log]
    end

    %% The Spokes
    VPS[☁️ VPS Relay]
    DB[🔒 LXC 1001: The Vault]
    SEC[🛡️ CrowdSec]

    %% Connections
    KAD -->|Unix Socket| SOCK
    KAD -->|SSH Encrypted Tunnel| VPS
    KAD -->|Postgres TCP 5432| DB
    KAD -->|SSH (ZFS Check)| DB
    KAD -->|HTTP API| SEC
    KAD -->|Write| AUDIT


```
---

## 🔭 Observability Scope

| Zone     | Component        | Target IP      | Method         | Metrics Probed                               |
| -------- | ---------------- | -------------- | -------------- | -------------------------------------------- |
| Factory  | LXC 1002 (Local) | `localhost`    | Unix Socket    | Container Health, RAM/CPU, Zombie Processes  |
| Edge     | VPS Relay        | `46.101.x.x`   | SSH (Ed25519)  | Active TCP Connections (DDoS check), Latency |
| Vault    | LXC 1001 (DB)    | `192.168.1.52` | Postgres / SSH | ZFS Pool Health, Database Schema Sizes       |
| Sentinel | CrowdSec         | `redcup_net`   | HTTP API       | Active IP Bans, Threat Intelligence          |

---

## 🛡️ Security Model
- Kadere is a high-privilege application secured via Three Layers of Defense:

**1️⃣ Zero Trust Networking**
- No public ports mapped
- Accessible only via internal Docker network

**2️⃣ Identity Proxy**
- Access proxied via Caddy + Authentik
- Requests must include valid `X-Authentik-User` headers

**3️⃣ Read-Only Mounts**

- Docker Socket mounted `:ro` (Read-Only)
- SSH keys injected at runtime via file mounts
- No secrets stored inside the image
---
## 🛠️ Operational Capabilities
**1️⃣ The Observer (Real-Time Monitoring)**
A “Kinetic” dashboard utilizing GSAP animations to visualize system load.
- Animated SVG gauges for CPU/RAM
- “Swiss Red” alert indicators for:
   - ZFS degradation
   - Container failures

 **2️⃣ The Action Center (SOAR)**
One-click remediation tools triggering background tasks:
- **Flush Cache** → Clears Redis keys
- **Restart Gateway** → Restarts Caddy container
- **Export Report** → Generates compliant `.docx` stakeholder report


**3️⃣ Teleportation (Console)**

⚠ Restricted to Admins
  - Secure ttyd embedded terminal
  - WebSocket-based access
  - Proxied securely through Caddy
  - Direct container access when authorized
---
## 🚀 Deployment (GitOps)
We use the RedCup Standard Integration Procedure (RCIP-01).

**Workflow**
- Push code to `main` → triggers Gitea Runner
- Docker image builds
- Image pushed to internal registry: `192.168.1.54:3000`
- Admin triggers deployment update on host

---

## 📜 Audit & Compliance
Every administrative action is logged to an immutable JSON ledger.
- Log Location (Host): `/data/production/kadere/audit.log`
- Log Format Example:

```json
{
  "timestamp": "2026-02-15T10:00:00",
  "user": "admin",
  "action": "RESTART",
  "target": "caddy",
  "status": "SUCCESS"
}
```
