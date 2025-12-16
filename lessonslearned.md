# Lessons Learned & Key Issues/Fixes

This migration was full of real-world pitfalls—great for learning distributed systems in cybersecurity.

### 1. Password Mismatches
- **Issue**: Running `--change-all` on Indexer broke Manager → Indexer (Filebeat) and Dashboard connections.
- **Fix**: Separate tools for indexer internal users vs. Manager API users; manual propagation to Filebeat keystore and Dashboard `wazuh.yml`.
- **Lesson**: Distributed auth is layered (OpenSearch security + Wazuh RBAC + JWT)—never assume global sync.

### 2. API Not Starting/Crashing (wazuh-apid, Error 900)
- **Issue**: Daemon "running" but BrokenProcessPool/orphaned processes holding port 55000.
- **Fix**: Enable in `api.yaml`, kill zombies (`pkill -9 wazuh-apid`), increase VM RAM/CPU, reduce `process_pool_size`.
- **Lesson**: Resource constraints cause multiprocessing crashes in homelabs—monitor with `htop`.

### 3. Dashboard "Not Ready" / YAML Errors
- **Issue**: Stuck initialization or fatal YAML parse (multiline comments from docs).
- **Fix**: Clean `wazuh.yml` (proper comments/indentation), update Indexer creds in keystore.
- **Lesson**: YAML is picky—validate configs before restarts.

### 4. Syslog/Archives Visibility
- **Issue**: UDM-Pro logs arriving (tcpdump) but not in Dashboard.
- **Fix**: Enable archives (`logall_json`), Filebeat archives, create `wazuh-archives-*` pattern via /app/opensearch-dashboards.
- **Lesson**: Indexed alerts ≠ raw logs—archives for forensics.

### 5. Retention & Storage
- **Issue**: Indexer disk growth.
- **Fix**: ISM policy (hot → delete 180d) + NFS cron for archives.
- **Lesson**: Hybrid strategy = best homelab balance (fast local search + cheap long-term NAS).

Overall: Persistence pays off in cyber—logs, curl tests, and patience solved everything. This setup now monitors endpoints + network perimeter reliably.
