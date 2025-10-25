# Nextcloud Homelab

A self-hosted Nextcloud instance with PostgreSQL database and Tailscale integration for secure remote access.

## Overview

This Docker Compose setup deploys:
- **Nextcloud** - Self-hosted file storage and collaboration platform
- **PostgreSQL** - Database backend for Nextcloud
- **Tailscale** - Secure VPN for remote access

## Prerequisites

- Docker & Docker Compose installed
- Tailscale account and auth key
- `/mnt/nextcloud/` directory structure with appropriate permissions

## Quick Start

### 1. Clone Repository
```bash
git clone <repository-url>
cd homelab-nextcloud
```

### 2. Configure Environment Variables

Edit `.env-config` and add your values:

```env
TS_AUTHKEY=tskey-auth-YOUR_KEY_HERE
POSTGRES_PASSWORD=your_secure_postgres_password
```

> ⚠️ **Important:** Never commit `.env-config` with real credentials

### 3. Create Directory Structure

```bash
mkdir -p /mnt/nextcloud/{nextcloud-config,nextcloud-storage,nextcloud-db,tailscale-state}
```

### 4. Start Services

```bash
docker-compose up -d
```

### 5. Access Nextcloud

- **Local:** https://localhost:443 (self-signed certificate)
- **Remote:** Access via Tailscale IP once connected

## Configuration

### PostgreSQL
- **Container:** nextcloud-postgres
- **Database:** nextcloud
- **User:** nextcloud
- **Password:** Set in `.env-config`

### Nextcloud Initial Setup
1. Open Nextcloud in browser
2. Create admin account
3. Choose PostgreSQL as database
   - Host: `postgres`
   - Database: `nextcloud`
   - User: `nextcloud`
   - Password: From `.env-config`

### Tailscale
Nextcloud will be accessible via your Tailscale network automatically. Check the container logs for the Tailscale IP:

```bash
docker logs nextcloud-ts
```

## Health Checks

All services include health checks:

```bash
# Check service status
docker-compose ps

# View specific service health
docker inspect nextcloud | grep -A 10 "Health"
```

## Management Commands

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f nextcloud
docker-compose logs -f postgres
```

### Stop Services
```bash
docker-compose down
```

### Stop & Remove Volumes
```bash
docker-compose down -v
```

### Restart Service
```bash
docker-compose restart nextcloud
```

## Backup

### Backup Database
```bash
docker exec nextcloud-postgres pg_dump -U nextcloud nextcloud > backup.sql
```

### Restore Database
```bash
docker exec -i nextcloud-postgres psql -U nextcloud nextcloud < backup.sql
```

## Troubleshooting

### Nextcloud won't start
- Check PostgreSQL is healthy: `docker-compose logs postgres`
- Verify volumes exist and have correct permissions
- Check `.env-config` has valid `POSTGRES_PASSWORD`

### Cannot connect via HTTPS
- Browser will show certificate warning (expected for self-signed)
- Click "Advanced" and proceed to localhost

### Tailscale connection issues
- Verify `TS_AUTHKEY` is valid and not expired
- Check container logs: `docker logs nextcloud-ts`
- Ensure `CAP_ADD` capabilities are present

## Security Notes

- Default self-signed certificate - consider Let's Encrypt integration for production
- Change default Nextcloud admin password immediately
- Keep `POSTGRES_PASSWORD` strong and secure
- Never commit `.env-config` to version control
- Consider adding reverse proxy (Nginx/Caddy) for production

## Architecture

```
┌─────────────────────────────┐
│   Tailscale (VPN)           │
│   nextcloud-ts              │
└──────────────┬──────────────┘
               │
┌──────────────┼──────────────┐
│              │              │
▼              ▼              ▼
Nextcloud  PostgreSQL    nextcloud-net
Port 443   Port 5432     (Docker Network)
```

## License

[Add your license here]

## Support

For issues or questions, open an issue in the repository.