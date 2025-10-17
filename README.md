# BunkerWeb Docker Compose Setup

A complete Docker Compose configuration for deploying BunkerWeb as a security-focused reverse proxy with PostgreSQL backend, scheduler, and web UI.

## Overview

This setup includes:

- **BunkerWeb**: Main reverse proxy and security engine
- **BunkerWeb Scheduler**: Manages scheduled tasks and configuration updates
- **BunkerWeb UI**: Web interface for management and monitoring
- **PostgreSQL**: Database backend for storing configurations
- **Apache**: Example backend application server

## Architecture

```
Internet → BunkerWeb (ports 80/443) → Backend Applications
              ↓
         Scheduler ← → Database
              ↓
          Web UI
```

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Minimum 2GB RAM
- Port 80, 443, 7443, and 8081 available

## Network Architecture

The setup uses three Docker networks:

- **bunker** (`10.20.30.0/24`): Main network for BunkerWeb components
- **bunker-db**: Isolated network for database communication
- **bw-apps**: External network for backend applications

## Quick Start

1. **Clone or create the configuration file**

   Save the Docker Compose configuration as `docker-compose.yml`

2. **Create the external network**

   ```bash
   docker network create bw-apps
   ```

3. **Set your database password**

   Replace `***` in the configuration with a secure password in both:
   - `DATABASE_URI` in the environment anchor
   - `POSTGRES_PASSWORD` in the bw-db service

4. **Start the stack**

   ```bash
   docker-compose up -d
   ```

5. **Verify deployment**

   ```bash
   docker-compose ps
   ```

## Configuration

### Environment Variables

The configuration uses YAML anchors (`&bw-env`) to share common environment variables:

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `API_WHITELIST_IP` | Allowed IP ranges for API access | `127.0.0.0/8 10.20.30.0/24` |
| `DATABASE_URI` | PostgreSQL connection string | `postgresql://bunkerweb:***@bw-db:5432/db` |
| `MULTISITE` | Enable multi-site support | `yes` |
| `UI_HOST` | BunkerWeb UI endpoint | `http://bw-ui:7000` |

### Ports

| Service | Port | Protocol | Description |
|---------|------|----------|-------------|
| bunkerweb | 80 | TCP | HTTP traffic |
| bunkerweb | 443 | TCP/UDP | HTTPS/HTTP3 traffic |
| apache | 7443 | TCP | Apache HTTPS (example) |
| apache | 8081 | TCP | Apache HTTP (example) |

## Services

### BunkerWeb

The main reverse proxy and security engine. Handles incoming traffic and applies security rules.

**Exposed Ports**: 80, 443

### BunkerWeb Scheduler

Manages periodic tasks including:
- Configuration updates
- Security rule updates
- Certificate management
- Log rotation

### BunkerWeb UI

Web-based management interface accessible at `http://bw-ui:7000` (internal network only).

### PostgreSQL Database

Stores BunkerWeb configurations and persistent data. Uses volume `bw-dbdata` for persistence.

### Apache (Example Backend)

Example backend server demonstrating integration with BunkerWeb. Serves content from `/var/www/html`.

## Volumes

- **bw-data**: BunkerWeb configuration and cache data
- **bw-dbdata**: PostgreSQL database files

## Security Considerations

1. **Change default passwords**: Replace all `***` placeholders with strong, unique passwords
2. **API whitelist**: Adjust `API_WHITELIST_IP` to restrict API access
3. **Network isolation**: Database is isolated in `bunker-db` network
4. **Logging**: Configure log rotation to prevent disk space issues

## Logging

Both BunkerWeb and Apache services include log rotation:

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "10"  # 10 for bunkerweb, 3 for apache
```

## Adding Backend Applications

To add a new backend application:

1. Connect your service to the `bw-apps` network:

   ```yaml
   networks:
     - bw-apps
   ```

2. Configure BunkerWeb to route traffic to your service via the scheduler or UI

3. Restart BunkerWeb to apply changes

## Maintenance

### View logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f bunkerweb
```

### Restart services

```bash
# Restart all
docker-compose restart

# Restart specific service
docker-compose restart bunkerweb
```

### Update images

```bash
docker-compose pull
docker-compose up -d
```

### Backup database

```bash
docker-compose exec bw-db pg_dump -U bunkerweb db > backup.sql
```

### Restore database

```bash
docker-compose exec -T bw-db psql -U bunkerweb db < backup.sql
```

## Troubleshooting

### BunkerWeb won't start

Check logs for errors:
```bash
docker-compose logs bunkerweb
```

### Can't connect to database

Verify database is running and password matches:
```bash
docker-compose exec bw-db psql -U bunkerweb -d db
```

### External network not found

Create the `bw-apps` network:
```bash
docker network create bw-apps
```

## Resources

- [BunkerWeb Documentation](https://docs.bunkerweb.io/)
- [BunkerWeb GitHub](https://github.com/bunkerity/bunkerweb)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## License

This configuration is provided as-is. Refer to individual component licenses:
- BunkerWeb: AGPLv3
- PostgreSQL: PostgreSQL License
- Apache HTTP Server: Apache License 2.0

## Support

For issues related to:
- BunkerWeb configuration: [BunkerWeb GitHub Issues](https://github.com/bunkerity/bunkerweb/issues)
- Docker Compose setup: Open an issue in your repository
- Security vulnerabilities: Report privately to maintainers

---

**Note**: This is a production-ready configuration template. Always review and adjust security settings for your specific use case.
