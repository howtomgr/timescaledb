# timescaledb Installation Guide

timescaledb is a free and open-source time-series database built on PostgreSQL. TimescaleDB extends PostgreSQL with time-series superpowers, providing SQL interface for time-series data as an alternative to InfluxDB or proprietary solutions

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (8GB+ recommended)
  - Storage: 10GB+ for data
  - Network: PostgreSQL protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5432 (default timescaledb port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install timescaledb
sudo dnf install -y timescaledb

# Enable and start service
sudo systemctl enable --now postgresql

# Configure firewall
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --reload

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install timescaledb
sudo apt install -y timescaledb

# Enable and start service
sudo systemctl enable --now postgresql

# Configure firewall
sudo ufw allow 5432

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### Arch Linux

```bash
# Install timescaledb
sudo pacman -S timescaledb

# Enable and start service
sudo systemctl enable --now postgresql

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### Alpine Linux

```bash
# Install timescaledb
apk add --no-cache timescaledb

# Enable and start service
rc-update add postgresql default
rc-service postgresql start

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### openSUSE/SLES

```bash
# Install timescaledb
sudo zypper install -y timescaledb

# Enable and start service
sudo systemctl enable --now postgresql

# Configure firewall
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --reload

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### macOS

```bash
# Using Homebrew
brew install timescaledb

# Start service
brew services start timescaledb

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### FreeBSD

```bash
# Using pkg
pkg install timescaledb

# Enable in rc.conf
echo 'postgresql_enable="YES"' >> /etc/rc.conf

# Start service
service postgresql start

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

### Windows

```bash
# Using Chocolatey
choco install timescaledb

# Or using Scoop
scoop install timescaledb

# Verify installation
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/timescaledb

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable postgresql

# Start service
sudo systemctl start postgresql

# Stop service
sudo systemctl stop postgresql

# Restart service
sudo systemctl restart postgresql

# Check status
sudo systemctl status postgresql

# View logs
sudo journalctl -u postgresql -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add postgresql default

# Start service
rc-service postgresql start

# Stop service
rc-service postgresql stop

# Restart service
rc-service postgresql restart

# Check status
rc-service postgresql status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'postgresql_enable="YES"' >> /etc/rc.conf

# Start service
service postgresql start

# Stop service
service postgresql stop

# Restart service
service postgresql restart

# Check status
service postgresql status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start timescaledb
brew services stop timescaledb
brew services restart timescaledb

# Check status
brew services list | grep timescaledb
```

### Windows Service Manager

```powershell
# Start service
net start postgresql

# Stop service
net stop postgresql

# Using PowerShell
Start-Service postgresql
Stop-Service postgresql
Restart-Service postgresql

# Check status
Get-Service postgresql
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream timescaledb_backend {
    server 127.0.0.1:5432;
}

server {
    listen 80;
    server_name timescaledb.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name timescaledb.example.com;

    ssl_certificate /etc/ssl/certs/timescaledb.example.com.crt;
    ssl_certificate_key /etc/ssl/private/timescaledb.example.com.key;

    location / {
        proxy_pass http://timescaledb_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName timescaledb.example.com
    Redirect permanent / https://timescaledb.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName timescaledb.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/timescaledb.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/timescaledb.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5432/
    ProxyPassReverse / http://127.0.0.1:5432/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend timescaledb_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/timescaledb.pem
    redirect scheme https if !{ ssl_fc }
    default_backend timescaledb_backend

backend timescaledb_backend
    balance roundrobin
    server timescaledb1 127.0.0.1:5432 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R timescaledb:timescaledb /etc/timescaledb
sudo chmod 750 /etc/timescaledb

# Configure firewall
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status postgresql

# View logs
sudo journalctl -u postgresql -f

# Monitor resource usage
top -p $(pgrep timescaledb)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/timescaledb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/timescaledb-backup-$DATE.tar.gz" /etc/timescaledb /var/lib/timescaledb

echo "Backup completed: $BACKUP_DIR/timescaledb-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop postgresql

# Restore from backup
tar -xzf /backup/timescaledb/timescaledb-backup-*.tar.gz -C /

# Start service
sudo systemctl start postgresql
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u postgresql -n 100
sudo tail -f /var/log/timescaledb/timescaledb.log

# Check configuration
psql -c "SELECT extversion FROM pg_extension WHERE extname='timescaledb';"

# Check permissions
ls -la /etc/timescaledb
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5432

# Test connectivity
telnet localhost 5432

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep timescaledb)

# Check disk I/O
iotop -p $(pgrep timescaledb)

# Check connections
ss -an | grep 5432
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  timescaledb:
    image: timescaledb:latest
    ports:
      - "5432:5432"
    volumes:
      - ./config:/etc/timescaledb
      - ./data:/var/lib/timescaledb
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update timescaledb

# Debian/Ubuntu
sudo apt update && sudo apt upgrade timescaledb

# Arch Linux
sudo pacman -Syu timescaledb

# Alpine Linux
apk update && apk upgrade timescaledb

# openSUSE
sudo zypper update timescaledb

# FreeBSD
pkg update && pkg upgrade timescaledb

# Always backup before updates
tar -czf /backup/timescaledb-pre-update-$(date +%Y%m%d).tar.gz /etc/timescaledb

# Restart after updates
sudo systemctl restart postgresql
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/timescaledb

# Clean old logs
find /var/log/timescaledb -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/timescaledb
```

## Additional Resources

- Official Documentation: https://docs.timescaledb.org/
- GitHub Repository: https://github.com/timescaledb/timescaledb
- Community Forum: https://forum.timescaledb.org/
- Best Practices Guide: https://docs.timescaledb.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
