# Distributed Wazuh SIEM/XDR Homelab in Proxmox

**Project Goal**: Migrate from a single all-in-one Wazuh installation to a fully distributed 3-node cluster (Wazuh Indexer, Manager, Dashboard on separate VMs) for better scalability, security isolation, and real-world cybersecurity practice.

**Why This Project?**
- Demonstrates production-like SIEM deployment in a resource-constrained homelab.
- Focuses on troubleshooting distributed systems (password sync, API crashes, YAML errors).
- Implements hybrid storage: Fast indexed alerts (180-day retention via ISM) + long-term raw archives on NFS.
- Integrates network device syslog (Ubiquiti UDM-Pro) and endpoint agents.

**Tech Stack**
- Proxmox VE cluster (3 nodes: "gloom" primary + others)
- Wazuh 4.14 distributed (separate VMs for Indexer ~100GB disk, Manager, Dashboard)
- NFS on TrueNAS for archives
- OpenSearch ISM for retention
- Agents on Ubuntu VM + Linux VMs + Windows + syslog from UDM-Pro


## Key Achievements
- Deployed distributed Wazuh cluster in Proxmox VE (separate VMs for Indexer ~100GB disk, Manager, Dashboard).
- Resolved complex distributed connectivity issues:
  - Password synchronization across Indexer internal users, Manager API (JWT), and Dashboard (manual propagation + keystore updates).
  - API daemon crashes (Error 900/BrokenProcessPool) via resource tuning, orphaned process cleanup, and process_pool_size reduction.
  - YAML syntax errors in Dashboard config causing "server not ready" state.
- Enabled raw syslog ingestion from Ubiquiti UDM-Pro (port 514 UDP) with archives indexing (`wazuh-archives-*` pattern in OpenSearch Dashboards).
- Implemented hybrid log retention:
  - Indexed alerts: 180-day hot retention with automated deletion via Index State Management (ISM) policy.
  - Raw archives: Long-term (1-2+ years) offload to TrueNAS NFS share via daily cron job moving compressed files.
- Migrated host Docker runtime from confined snap to official Docker CE:
  - Full backup/restore of 12 named volumes and images using rsync and tar via temp containers.
  - Recreated custom networks (Traefik proxy bridge, macvlan for direct LAN IPs).
  - Enabled comprehensive container monitoring in Wazuh (json.log application logs + Docker wodle listener for runtime events) using isolated Python virtual environment to resolve dependency conflicts.

## Dashboard Screenshots

Final working Dashboard with agent monitoring, vulnerability detection, SCA, and MITRE ATT&CK mapping:

<img width="2538" height="1063" alt="image" src="https://github.com/user-attachments/assets/e8dda284-155e-429b-8ee7-a28703630603" />


ISM Retention Policy (`wazuh_retention-180d`):

<img width="1732" height="801" alt="image" src="https://github.com/user-attachments/assets/cf293973-b31e-4630-9b29-19cf06eee9c0" />

**Dashboard Highlights**
- Endpoint monitoring (Ubuntu VM agent) with FIM, SCA, vulnerability detection, MITRE ATT&CK mapping.
- Network visibility via UDM-Pro syslog.
- Container security (runtime events + application logs).


## Setup Steps Summary
1. **Proxmox Prep**: Create 3 VMs (Indexer: 100GB disk, Manager/Dashboard: modest resources).
2. **Install Distributed Wazuh**: Follow official docs for separate components.
3. **Fix Connectivity**: Password tool sync, API enablement in `api.yaml`, JWT testing.
4. **Enable Archives**: `<logall_json>yes</logall_json>` + Filebeat archives enabled.
5. **Syslog Integration**: `<remote><connection>syslog</connection></remote>` block.
6. **Retention**: ISM policy (hot → delete after 180d) + NFS cron offload.
7. **Agent Deployment**: Clean reinstall pointing to Manager IP.

See `lessonslearned.md` for detailed troubleshooting.

## Skills Demonstrated 
- Distributed SIEM architecture and troubleshooting (auth chains, multiprocessing errors, API/JWT).
- Log lifecycle management (ISM policies, NFS archival, cron automation).
- Container runtime migration and monitoring (snap → official Docker, wodle configuration, virtual environments).
- Network/security integrations (syslog, agents, macvlan, firewalls, reverse proxy).
- Configuration debugging (XML/YAML, systemd, APT repo management, GPG conflicts).

This project showcases initiative in building a scalable, secure homelab SIEM—perfect for SOC/SECOPS roles.
