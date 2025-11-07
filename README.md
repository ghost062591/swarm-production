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

## Directory Structure

```
swarm/
├── conf/              # Traefik and service configurations
├── stacks/
│   ├── apps/         # Application services
│   │   ├── adminer/      # Database management
│   │   ├── n8n/          # Workflow automation
│   │   ├── outline/      # Documentation wiki
│   │   ├── paperless/    # Document management
│   │   └── uptime/       # Uptime monitoring
│   ├── core/         # Core infrastructure
│   │   ├── authentik/    # SSO/Authentication
│   │   ├── portainer/    # Container management
│   │   └── traefik/      # Reverse proxy
│   ├── data/         # Data services
│   │   └── rsync/        # Backup service
│   └── web/          # Web services
│       └── tracker/      # Tracker site
└── README.md
```

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
- outline
- paperless (webserver, redis)
- tracker-nginx
- uptime-kuma

## Network Configuration

All services are connected to the `homelab` external overlay network for inter-service communication.

### Local Deployment (2025-11-07)
- Services now use `.swarm.home` domains for local access
- TLS enabled without external certificate resolvers
- Simplified Traefik configuration for local development
- Removed Cloudflare DNS integration

## Recent Changes

### Local Configuration Update (2025-11-07)
- Migrated from external `.frostlabs.me` domains to local `.swarm.home` domains
- Updated Traefik labels across all services for local deployment
- Simplified `.gitignore` to exclude entire `conf/` directory
- Moved Authentik from `apps/` to `core/` directory structure
- Removed Traefik labels from n8n and paperless for direct access
- Updated Traefik stack configuration for simplified port bindings

### Swarm Rebalancing (2025-10-30)
- Promoted p1, p2, p3 from workers to managers
- Removed unnecessary hostname constraints from service configs
- Force-redeployed services to redistribute across all nodes
- Verified GlusterFS accessibility on all nodes
- Achieved balanced workload distribution across all 4 nodes
- Improved high availability with 4-node manager quorum
- Services now self-balance automatically when nodes fail/recover