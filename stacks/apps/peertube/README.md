# PeerTube Docker Swarm Setup

## Prerequisites

1. Traefik is running and configured
2. PostgreSQL is running (`postgresSQL` service)
3. The `homelab` network exists
4. DNS record for `videos.frostlabs.me` points to your server

## Setup Steps

### 1. Create PeerTube Database

Connect to your existing PostgreSQL instance and create the PeerTube database:

```bash
# Connect to PostgreSQL container
docker exec -it $(docker ps -q -f name=postgresSQL) psql -U admin -d postgres

# Create database (user 'admin' already exists with postgres-master secret)
CREATE DATABASE peertube;
GRANT ALL PRIVILEGES ON DATABASE peertube TO admin;
\q
```

### 2. Create Docker Secret for PeerTube

You already have the `postgres-master` secret for database access. You just need to create the PeerTube application secret:

```bash
# Generate and create the PeerTube secret
echo "$(openssl rand -hex 32)" | docker secret create peertube-secret -

# Verify the secret was created
docker secret ls | grep peertube
```

**Note:** The stack uses your existing `postgres-master` secret for database authentication with the `admin` user.

### 3. Verify Data Directory Permissions

```bash
# Check that the PeerTube appdata directory exists and has correct permissions
ls -la /home/doc/projects/unraid-appdata/PeerTube

# If needed, fix permissions (UID 999 is the PeerTube user)
sudo chown -R 999:999 /home/doc/projects/unraid-appdata/PeerTube
```

### 4. Deploy the Stack

```bash
docker stack deploy -c stack.yml peertube
```

### 5. Monitor Deployment

```bash
# Watch the services
docker service ls | grep peertube

# Check logs
docker service logs -f peertube_peertube

# Check if healthy
docker ps | grep peertube
```

### 6. Access PeerTube

Once deployed, access PeerTube at: https://videos.frostlabs.me

The first time you access it, you'll need to:
1. Complete the setup wizard
2. Create an admin account
3. Configure additional settings in the admin panel

## Configuration Notes

### Database Connection
- Host: `postgresSQL` (existing Postgres service)
- Port: 5432 (internal)
- Database: `peertube`
- User: `admin`
- Password: From `postgres-master` secret

### Redis Connection
- Host: `peertube-redis` (internal service)
- Port: 6379 (default)

### SMTP/Email
- Host: `peertube-postfix` (internal service)
- Port: 25
- From: noreply@videos.frostlabs.me

### Ports
- **9000**: PeerTube HTTP (internal, proxied by Traefik)
- **1935**: RTMP for live streaming (published on host)

### Traefik Integration
The stack is configured to use Traefik for:
- SSL/TLS certificates (Let's Encrypt)
- HTTPS on port 443
- HTTP to HTTPS redirect
- Domain: videos.frostlabs.me

## Storage Layout

All data is stored in `/home/doc/projects/unraid-appdata/PeerTube`:
- Videos and media files
- Thumbnails and previews
- User uploads
- Logs
- Configuration

## Troubleshooting

### Check service status
```bash
docker service ps peertube_peertube --no-trunc
```

### View logs
```bash
docker service logs peertube_peertube
docker service logs peertube_peertube-redis
docker service logs peertube_peertube-postfix
```

### Database connection issues
```bash
# Test connection from PeerTube container
docker exec -it $(docker ps -q -f name=peertube_peertube) sh
nc -zv postgresSQL 5432
```

### Restart services
```bash
docker service update --force peertube_peertube
```

### Remove and redeploy
```bash
docker stack rm peertube
# Wait for cleanup
docker stack deploy -c stack.yml peertube
```

## Updating PeerTube

```bash
# Update the image
docker service update --image chocobozzz/peertube:production-bookworm peertube_peertube

# Or redeploy the stack
docker stack deploy -c stack.yml peertube
```

## Security Considerations

1. Change the default admin password after first login
2. Keep PEERTUBE_SECRET secure and never commit it to version control
3. Regularly update the PeerTube image for security patches
4. Configure proper email settings for notifications
5. Review and configure user registration settings in admin panel

## Additional Resources

- [PeerTube Documentation](https://docs.joinpeertube.org/)
- [PeerTube Production Guide](https://docs.joinpeertube.org/install/docker)
- [PeerTube Admin Documentation](https://docs.joinpeertube.org/admin/following-instances)
