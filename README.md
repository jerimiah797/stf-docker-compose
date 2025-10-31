# STF Docker Compose Template

A production-ready template for deploying [Smartphone Test Farm (STF)](https://github.com/DeviceFarmer/stf) using Docker Compose with a distributed architecture.

## What is STF?

Smartphone Test Farm is an open-source web application for debugging and controlling Android devices remotely. It provides:

- Remote device control through web browser
- Real-time device screenshots and screen streaming
- ADB shell access through web interface
- File upload/download to devices
- Device information and resource monitoring
- Multi-user device sharing and reservations

## Architecture

This template supports a **distributed deployment** with:

- **Central Server**: Hosts the STF web UI, API, database, and core services
- **Remote Providers**: Run on machines with physical Android devices connected via USB
- **SSL/TLS**: Production-ready HTTPS configuration with nginx reverse proxy
- **Custom Device Database**: Support for device metadata and images

## Features

- Production-ready Docker Compose configuration
- Distributed architecture for scaling across multiple locations
- SSL/TLS termination with nginx
- Custom Docker images for TLS validation bypass
- Environment-based configuration
- Example configurations for remote providers
- Comprehensive documentation

## Prerequisites

### Central Server

- Docker and Docker Compose
- Public hostname/IP address
- SSL certificate (Let's Encrypt recommended)
- Open ports: 80, 443, 7100-7700

### Remote Providers

- Docker and Docker Compose
- Physical Android devices connected via USB
- Network access to central server
- Open ports: 7400-7700

## Quick Start

### 1. Central Server Setup

```bash
# Clone this repository
git clone <your-repo-url> stf-deployment
cd stf-deployment

# Copy and configure environment
cp .env.example .env
nano .env  # Update PUBLIC_IP and SECRET

# Set up SSL certificates
cd nginx/certs
# Follow instructions in nginx/certs/README.md
# to generate or copy certificates as server.crt and server.key

# Start services
docker-compose up -d

# Check logs
docker-compose logs -f
```

### 2. Remote Provider Setup

```bash
# On remote machine with USB devices
cd remote/

# Copy and configure environment
cp .env.example .env
nano .env  # Update configuration (see below)

# Start provider
docker-compose up -d
```

### 3. Configure nginx for Remote Providers

Edit `nginx/nginx.conf` and add location blocks for each remote provider. See the commented examples in the file at lines 79-97.

After editing, restart nginx:

```bash
docker-compose restart nginx
```

## Configuration

### Central Server (.env)

```bash
# Your STF server's public hostname
PUBLIC_IP=stf.yourdomain.com

# Shared secret for authentication (generate a strong random string)
SECRET=change-this-to-a-random-secret-key

# RethinkDB connection (internal Docker network)
RETHINKDB_PORT_28015_TCP=tcp://rethinkdb:28015
```

### Remote Provider (remote/.env)

```bash
# Central server's public hostname (must match central server)
PUBLIC_IP=stf.yourdomain.com

# Shared secret (MUST MATCH central server)
SECRET=same-secret-as-central-server

# Unique name for this remote station (appears in STF UI)
STATION_NAME=REMOTE1

# This remote machine's public hostname/IP
# Clients connect directly to this IP for device interactions
STATION_IP=remote1.yourdomain.com

# RethinkDB connection (not used by remote, but required by STF)
RETHINKDB_PORT_28015_TCP=tcp://rethinkdb:28015
```

## SSL Certificate Setup

For production deployments, you need SSL certificates. See `nginx/certs/README.md` for detailed instructions on:

- Generating self-signed certificates (testing only)
- Using Let's Encrypt (recommended for production)
- Using commercial certificates

**Quick Let's Encrypt setup:**

```bash
# Install certbot
sudo apt-get install certbot

# Get certificates
sudo certbot certonly --standalone -d stf.yourdomain.com

# Copy to nginx/certs
sudo cp /etc/letsencrypt/live/stf.yourdomain.com/fullchain.pem nginx/certs/server.crt
sudo cp /etc/letsencrypt/live/stf.yourdomain.com/privkey.pem nginx/certs/server.key
sudo chmod 644 nginx/certs/server.crt
sudo chmod 600 nginx/certs/server.key
```

## Adding Remote Providers

For each remote machine with USB devices:

1. **Deploy remote provider** on the machine with devices:
   ```bash
   cd remote/
   cp .env.example .env
   # Edit .env with unique STATION_NAME and STATION_IP
   docker-compose up -d
   ```

2. **Update nginx configuration** on central server:

   Edit `nginx/nginx.conf` and add a location block:
   ```nginx
   location ~ "^/d/REMOTE1/([^/]+)/(?<port>[0-9]{3,5})/$" {
     proxy_pass http://remote1.yourdomain.com:$port/;
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection $connection_upgrade;
     proxy_set_header X-Forwarded-For $remote_addr;
     proxy_set_header X-Real-IP $remote_addr;
   }
   ```

3. **Restart nginx**:
   ```bash
   docker-compose restart nginx
   ```

## Custom Device Database

To use custom device metadata and images:

1. **Clone the device database**:
   ```bash
   git clone https://github.com/DeviceFarmer/stf-device-db.git stf-devices
   ```

2. **Mount as submodule** in docker-compose.yml:
   ```yaml
   provider:
     volumes:
       - ./stf-devices:/app/node_modules/@devicefarmer/stf-device-db/dist
   ```

3. **Add your devices** following the [device addition workflow](https://github.com/DeviceFarmer/stf-device-db#adding-new-devices)

## Accessing STF

After deployment:

1. Open your browser to `https://stf.yourdomain.com`
2. Log in with the mock authentication (default in template)
3. You should see connected devices from all remote providers

## Architecture Details

### Central Server Components

- **nginx**: Reverse proxy with SSL/TLS termination
- **rethinkdb**: Database for device state and user management
- **app**: Web UI frontend
- **auth**: Authentication service (mock auth by default)
- **processor**: Handles device events and state changes
- **triproxy**: ZeroMQ message router for device communication
- **storage-temp**: Temporary file storage
- **storage-plugin-apk**: APK file management
- **storage-plugin-image**: Device screenshot storage
- **websocket**: WebSocket server for real-time updates
- **api**: REST API endpoints

### Remote Provider Components

- **local-adb**: ADB server with USB device access
- **local-provider**: STF provider connecting devices to central server

## Troubleshooting

### Devices not appearing

1. Check remote provider logs:
   ```bash
   docker-compose logs -f local-provider
   ```

2. Verify SECRET matches between central and remote

3. Check firewall rules allow ports 7250, 7270, 7400-7700

4. Verify nginx location blocks match STATION_NAME

### Cannot connect to devices

1. Check that STATION_IP is publicly accessible
2. Verify ports 7400-7700 are open on remote provider
3. Check nginx reverse proxy configuration
4. Test websocket connection in browser console

### SSL certificate errors

1. Ensure certificates are valid and not expired
2. Check certificate permissions (644 for .crt, 600 for .key)
3. Verify certificate matches your PUBLIC_IP hostname
4. For self-signed certs, provider containers have `NODE_TLS_REJECT_UNAUTHORIZED=0`

### Database connection issues

1. Check RethinkDB is running:
   ```bash
   docker-compose logs rethinkdb
   ```

2. Verify RETHINKDB_PORT_28015_TCP in .env

3. Check internal Docker network connectivity

## Advanced Topics

For comprehensive documentation on advanced topics, see the [Distributed STF Deployment Guide](https://github.com/DeviceFarmer/stf-device-db/blob/master/DISTRIBUTED_SETUP_GUIDE.md):

- Network architecture and port mapping
- Security considerations
- Performance tuning
- Monitoring and maintenance
- Multi-site deployments
- Custom authentication providers

## Production Considerations

Before deploying to production:

1. **Generate a strong SECRET**: Use a cryptographically secure random string
2. **Use proper SSL certificates**: Let's Encrypt or commercial CA
3. **Configure firewalls**: Restrict access to necessary ports only
4. **Set up monitoring**: Monitor Docker containers and device connectivity
5. **Plan for backups**: Regular RethinkDB backups
6. **Use authentication**: Replace mock auth with OAuth or LDAP
7. **Review security**: Follow security best practices for your environment

## Updating STF

To update to a newer STF version:

1. Edit Dockerfiles to change the STF version
2. Rebuild custom images:
   ```bash
   docker-compose build
   ```
3. Restart services:
   ```bash
   docker-compose down
   docker-compose up -d
   ```

## Contributing

Contributions are welcome! Please:

1. Fork this repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request with clear description

## Resources

- [STF Official Repository](https://github.com/DeviceFarmer/stf)
- [STF Documentation](https://github.com/DeviceFarmer/stf/blob/master/doc/DEPLOYMENT.md)
- [Device Database Repository](https://github.com/DeviceFarmer/stf-device-db)
- [Distributed Deployment Guide](https://github.com/DeviceFarmer/stf-device-db/blob/master/DISTRIBUTED_SETUP_GUIDE.md)

## License

This template is provided as-is for use with STF. STF itself is licensed under Apache 2.0.

## Support

For issues specific to this template, please open an issue in this repository.

For STF-related questions, see the [official STF documentation](https://github.com/DeviceFarmer/stf) or community forums.
