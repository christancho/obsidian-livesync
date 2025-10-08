# Obsidian LiveSync CouchDB Docker Setup

This repository contains a Docker Compose configuration to deploy CouchDB for use with Obsidian LiveSync. The configuration is compatible with both Docker Compose and Portainer.

## Quick Start

1. **Clone and setup environment:**
   ```bash
   git clone <your-repo-url>
   cd obsidian-live-sync
   cp env.example .env
   ```

2. **Configure your settings:**
   Edit the `.env` file and change the default passwords:
   ```bash
   nano .env
   ```
   
   **Important:** Change these default values:
   - `COUCHDB_PASSWORD` - Set a secure password
   - `COUCHDB_SECRET` - Generate a random secret key
   - `LIVESYNC_DIR` - Set the absolute path for data storage (e.g., `/opt/obsidian-livesync`)

3. **Start CouchDB:**
   ```bash
   docker-compose up -d
   ```

4. **Verify the installation:**
   ```bash
   curl http://localhost:5984
   ```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `COUCHDB_USER` | admin | CouchDB admin username |
| `COUCHDB_PASSWORD` | password | CouchDB admin password |
| `COUCHDB_SECRET` | mysecret | CouchDB secret key |
| `NODENAME` | couchdb@127.0.0.1 | CouchDB node name |
| `LIVESYNC_DIR` | /opt/obsidian-livesync | Absolute path for data storage |

### CouchDB Configuration

The CouchDB instance is configured with:
- CORS enabled for web access
- Single node setup (suitable for personal use)
- Optimized settings for Obsidian LiveSync
- Health checks enabled
- **Bind mounts** instead of Docker volumes for easier data management
- **Portainer compatible** environment variable format

## Obsidian LiveSync Setup

1. **Install Obsidian LiveSync plugin** in your Obsidian vault
2. **Configure the plugin:**
   - Database URL: `http://localhost:5984`
   - Username: `admin` (or your custom username)
   - Password: Your configured password
   - Database name: `obsidian-livesync` (or your preferred name)

3. **Create the database:**
   The plugin will automatically create the database when you first sync.

## Deployment Options

### Docker Compose
Use the standard Docker Compose commands as shown in the Management Commands section below.

### Portainer Stack
1. **Create a new Stack** in Portainer
2. **Copy the contents** of `docker-compose.yml` into the stack editor
3. **Set environment variables** in Portainer's environment section:
   - `COUCHDB_USER`: admin
   - `COUCHDB_PASSWORD`: your_secure_password
   - `COUCHDB_SECRET`: your_random_secret
   - `NODENAME`: couchdb@127.0.0.1
   - `LIVESYNC_DIR`: /opt/obsidian-livesync (or your preferred path)
4. **Deploy the stack**

## Management Commands

### Start services
```bash
docker-compose up -d
```

### Stop services
```bash
docker-compose down
```

### View logs
```bash
docker-compose logs -f couchdb
```

### Access CouchDB Fauxton (Web UI)
Open your browser and go to: http://localhost:5984/_utils

### Backup database
```bash
# Create backup directory
mkdir -p backups

# Backup the database
docker-compose exec couchdb couchdb-backup -H localhost:5984 -u admin -p your_password -d obsidian-livesync -f backups/obsidian-backup.json

# Alternative: Direct file backup (since data is in ./data/couchdb/)
cp -r data/couchdb backups/couchdb-backup-$(date +%Y%m%d)
```

### Restore database
```bash
docker-compose exec couchdb couchdb-restore -H localhost:5984 -u admin -p your_password -d obsidian-livesync -f backups/obsidian-backup.json

# Alternative: Direct file restore
# Stop the container first
docker-compose down
# Restore from backup
cp -r backups/couchdb-backup-YYYYMMDD/* data/couchdb/
# Start the container
docker-compose up -d
```

## Security Considerations

1. **Change default passwords** in the `.env` file
2. **Use HTTPS** in production environments
3. **Restrict network access** if running on a public server
4. **Regular backups** of your data

## Troubleshooting

### Connection Issues
- Ensure CouchDB is running: `docker-compose ps`
- Check logs: `docker-compose logs couchdb`
- Verify port 5984 is not blocked by firewall

### Permission Issues
- Ensure Docker has proper permissions
- Check bind mount directories are accessible
- If you get permission errors, try: `sudo chown -R 5984:5984 ${LIVESYNC_DIR}/data/`

### Performance Issues
- Monitor resource usage: `docker stats`
- Adjust CouchDB configuration in `couchdb/local.ini`

## File Structure

```
obsidian-live-sync/
├── docker-compose.yml      # Main Docker Compose configuration (Portainer compatible)
├── env.example            # Environment variables template
├── .env                   # Your environment variables (create from env.example)
├── couchdb/
│   └── local.ini         # CouchDB configuration
├── data/                  # Data directories (bind mounts)
│   ├── couchdb/          # CouchDB database files
│   └── couchdb-config/   # CouchDB configuration files
└── README.md             # This file
```

**Note:** The `data/` directory structure will be created at `${LIVESYNC_DIR}/data/` when using Portainer or custom paths.

## Support

For issues related to:
- **Docker setup**: Check Docker and Docker Compose documentation
- **CouchDB**: Visit [CouchDB documentation](https://docs.couchdb.org/)
- **Obsidian LiveSync**: Check the [plugin documentation](https://github.com/vrtmrz/obsidian-livesync)


