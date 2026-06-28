# Architecture

## Overview

Jarvis is a centralized monitoring and automation platform for a multi-node homelab. A single Flask application serves as the hub: it collects health and status data from every node in the environment, presents that data through a unified dashboard, and exposes a controlled set of operational actions that can be triggered manually or by scheduled automation.

## Network Topology

```
Internet
     │
AT&T Fiber
     │
Flint 2 Router
     │
───────────────
│             │
Dell Server   Mini-PC (Jarvis)
│             │
Jellyfin      Ollama
ARR           Sentinel
Docker        Dashboard
│
Proxmox Host
├── Ubuntu VM
├── Windows VM
└── pfSense
```

## Machine Roles

| Machine | Role | Services |
|---|---|---|
| **Mini-PC** | AI + Automation Hub | Jarvis dashboard, Ollama (local LLMs), Sentinel, automation scripts |
| **Dell Server** | Media + Storage | Docker, Jellyfin, ARR stack (qBittorrent, Prowlarr, Flaresolverr), primary storage |
| **Lenovo (Proxmox Host)** | Hypervisor | Proxmox VE, Ubuntu Lab VM, Windows Lab VM, pfSense VM |
| **HP** | Windows Lab | Windows 11, Active Directory, Microsoft service testing |

All inter-host communication is tunneled through **Tailscale**, so no ports are exposed to the public internet.

## Application Design

Jarvis runs as a single Flask application with three responsibilities:

1. **Status collection** — a Bash script gathers local system metrics, Docker container state, and AI service status, then pulls equivalent data from remote hosts over SSH with short timeouts so an offline machine degrades gracefully instead of breaking the dashboard.
2. **Presentation** — the Flask app parses and caches that status, renders it as a single-page dashboard, and exposes a `/status` JSON endpoint for programmatic consumption.
3. **Action execution** — token-gated endpoints perform maintenance operations (service restarts, VM snapshots, Wake-on-LAN). Higher-risk actions route through an approve/reject queue, and every attempt is recorded in an event log.

```
            ┌──────────────────────────────┐
            │     Dashboard (Browser)       │
            └──────────────┬───────────────┘
                           │ HTTP
                           ▼
            ┌──────────────────────────────┐
            │   Jarvis (Mini-PC)            │
            │   · Status aggregation        │
            │   · Action dispatch           │
            │   · Event logging             │
            └───┬──────────────────────┬───┘
                │                      │
     Local data │                      │ SSH over Tailscale
                ▼                      ▼
   ┌────────────────────┐   ┌──────────────────────────┐
   │ AI / Automation     │   │ Dell Server (Media)       │
   │ · Ollama            │   │ · Docker containers       │
   │ · Open WebUI        │   │ · Jellyfin / ARR stack    │
   └────────────────────┘   │ Proxmox Host (Lenovo)     │
                             │ · VM inventory             │
                             │ HP / Windows endpoints     │
                             └──────────────────────────┘
```

## Data Flow

1. **Collection** — A status-collection script runs on the Mini-PC. It gathers local metrics directly (memory usage, Docker container state, AI service status) and connects to the Dell Server, the Proxmox host, and the Windows endpoints over the Tailscale network using SSH, with short connection timeouts.
2. **Aggregation** — The Flask application parses the output of the collection script into a structured, per-host representation and caches it briefly to avoid issuing redundant SSH calls on every dashboard load.
3. **Presentation** — The aggregated status is rendered as a single dashboard view and is also available as a JSON endpoint for programmatic consumption.
4. **Degradation handling** — If any individual host is unreachable, that host is reported as offline rather than causing a failure elsewhere in the system, so the dashboard always reflects the best available picture of the environment.

## Monitoring

Jarvis continuously evaluates the health of each component:

- **Mini-PC** — memory utilization, running containers, AI service availability
- **Dell Server** — disk usage, key service status, container health
- **Proxmox Host** — virtual machine inventory and state
- **Windows Endpoints** — uptime and basic resource availability

Container and service health are evaluated against expected state, and any deviation (a stopped or unhealthy container, an unreachable host) is surfaced as an alert on the dashboard rather than requiring manual inspection.

## Automation

A defined set of operational actions can be triggered from the dashboard or invoked programmatically:

- Service restarts on the Dell Server (Jellyfin, full ARR media stack)
- Virtual machine snapshots on the Proxmox Host
- Wake-on-LAN to power on offline endpoints
- Forced status refresh across all hosts
- Scheduled daily health report posted to Discord

Actions are implemented as discrete, auditable operations rather than open-ended command execution. Higher-risk actions can be routed through an approval step before they are executed, separating the request from the action itself. Every action attempt, successful or not, is recorded in an event log.

## Reporting

A scheduled job periodically generates a summary of overall environment health and delivers it to a private Discord channel via webhook. The webhook URL is stored outside of source control and loaded at runtime.

## Security Boundary

The public version of this project excludes all credentials, network identifiers, private configuration, and operational secrets. Security-sensitive configuration is managed outside of source control. See [security.md](security.md) for details.
