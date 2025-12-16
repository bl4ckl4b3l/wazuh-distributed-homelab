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
- Password synchronization across distributed components (indexer internal users vs. Manager API/JWT).
- Resolved recurrent API crashes (Error 900/BrokenProcessPool) via resource tuning and process cleanup.
- Enabled raw syslog ingestion from UDM-Pro (port 514 UDP) + archives indexing (`wazuh-archives-*`).
- Configured ISM policy for 180-day indexed alert retention.
- Automated NFS offload of compressed archives for multi-year forensic retention.
- Deployed endpoint agents with secure TCP communication (port 1514).

## Dashboard Screenshots

Final working Dashboard with agent monitoring, vulnerability detection, SCA, and MITRE ATT&CK mapping:

<img width="2538" height="1063" alt="image" src="https://github.com/user-attachments/assets/e8dda284-155e-429b-8ee7-a28703630603" />


ISM Retention Policy (`wazuh_retention-180d`):

<img width="1732" height="801" alt="image" src="https://github.com/user-attachments/assets/cf293973-b31e-4630-9b29-19cf06eee9c0" />


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
- Distributed SIEM architecture and troubleshooting (auth chains, multiprocessing errors).
- Log lifecycle management (ISM policies, NFS archival).
- Network/security integrations (syslog, agents, firewalls).
- Configuration debugging (YAML/XML, systemd, logs).

This project showcases initiative in building a scalable, secure homelab SIEM—perfect for SOC/SECOPS roles.
