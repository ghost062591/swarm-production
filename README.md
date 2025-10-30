# swarm-production

Production Docker Swarm Infrastructure

## Cluster Overview

### Nodes
- **p0** (Manager/Leader) - Infrastructure services
- **p1** (Manager) - Application services
- **p2** (Manager) - Application services
- **p3** (Manager) - Application services

All nodes are managers providing a 4-node quorum (can tolerate 2 node failures while maintaining quorum).

### Storage
- **GlusterFS** mounted at `/home/doc/swarm-data/` on all nodes
- Shared storage enables services to run on any node without storage constraints

## Service Distribution Strategy

### Pinned Services
Services that must run on specific nodes:

- **traefik** (p0) - Published ports 80/443, needs stable IP for DNS
- **portainer** (p0) - Management UI, stays with leader for convenience
- **rsync** (manager constraint) - Backup service, needs manager access

### Floating Services
Services that can run on any node (swarm auto-balances):

- adminer
- authentik (server, worker, redis)
- n8n
- paperless (webserver, redis)
- tracker-nginx
- uptime-kuma

## Recent Changes (2025-10-30)

### Swarm Rebalancing
- Promoted p1, p2, p3 from workers to managers
- Removed unnecessary hostname constraints from service configs
- Force-redeployed services to redistribute across all nodes
- Verified GlusterFS accessibility on all nodes

### Results
- Achieved balanced workload distribution across all 4 nodes
- Improved high availability with 4-node manager quorum
- Services now self-balance automatically when nodes fail/recover
- Fixed Portainer agent connectivity by restarting agents after manager promotion