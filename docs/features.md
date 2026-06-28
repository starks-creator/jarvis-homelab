# Features

## Dashboard

- **Real-time infrastructure dashboard** — unified health score, container count, VM status, and AI model count at a glance
- Single-page status view that auto-refreshes every 5 seconds
- Per-host panels for compute, storage, virtualization, and AI services
- Visual alerts for unhealthy containers or unreachable hosts
- **Network topology diagram** — animated live diagram of the lab (Flint 2 Router → Dell Server + Mini-PC → Proxmox) with status-colored, animated packet links
- **Screenshot Mode** — one-click client-side toggle that redacts IPs, MAC addresses, hostnames, and usernames so the dashboard can be safely captured for documentation or sharing without a backend restart

## Monitoring

- **Automated health monitoring** — continuous SSH-based checks across all hosts with graceful degradation when a host is offline (one offline machine never breaks the full dashboard)
- **Docker container monitoring** — per-container health tracking with alerts for unhealthy or stopped services
- **Proxmox VM monitoring** — live VM inventory and status from the hypervisor
- **Jellyfin & ARR stack management** — service status visibility and one-click restarts for media services
- **AI integration (Ollama)** — local LLM model list, sizes, and Open WebUI container health
- Disk usage, memory utilization, and uptime per host
- Rolling health score across all monitored nodes

## Automation & Actions

- **One-click recovery actions** — restart services, trigger VM snapshots, send daily reports, and more from the Actions panel
- **Wake-on-LAN** — power on offline machines (Dell Server, workstations) directly from the dashboard
- **Approval queue** — higher-risk actions route through an approve/reject step before executing, adding a human checkpoint
- Token-gated action API for safe use from scripts and scheduled jobs
- Forced status refresh across all hosts on demand

## Notifications & Reporting

- **Discord notifications** — scheduled daily health summaries delivered to a private Discord channel via webhook
- JSON `/status` endpoint for programmatic consumption by external scripts or tooling

## Security

- **Tailscale remote access** — all inter-host communication runs over a private Tailscale overlay network; no ports exposed to the public internet
- All action endpoints require a private token passed via a custom request header
- **Rolling event log** — full audit trail of every action attempt, successful or not
- Higher-risk actions gated behind an approval step rather than executing immediately
- SSH calls use short connect timeouts and batch mode to prevent a single offline host from hanging the dashboard
