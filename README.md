# deluge Installation Guide

deluge is a free and open-source BitTorrent client. Deluge provides a lightweight, cross-platform BitTorrent client with plugin support

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for downloads
  - Network: BitTorrent protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8112 (default deluge port)
  - Daemon on 58846
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

# Install deluge
sudo dnf install -y deluge

# Enable and start service
sudo systemctl enable --now deluge

# Configure firewall
sudo firewall-cmd --permanent --add-port=8112/tcp
sudo firewall-cmd --reload

# Verify installation
deluge --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install deluge
sudo apt install -y deluge

# Enable and start service
sudo systemctl enable --now deluge

# Configure firewall
sudo ufw allow 8112

# Verify installation
deluge --version
```

### Arch Linux

```bash
# Install deluge
sudo pacman -S deluge

# Enable and start service
sudo systemctl enable --now deluge

# Verify installation
deluge --version
```

### Alpine Linux

```bash
# Install deluge
apk add --no-cache deluge

# Enable and start service
rc-update add deluge default
rc-service deluge start

# Verify installation
deluge --version
```

### openSUSE/SLES

```bash
# Install deluge
sudo zypper install -y deluge

# Enable and start service
sudo systemctl enable --now deluge

# Configure firewall
sudo firewall-cmd --permanent --add-port=8112/tcp
sudo firewall-cmd --reload

# Verify installation
deluge --version
```

### macOS

```bash
# Using Homebrew
brew install deluge

# Start service
brew services start deluge

# Verify installation
deluge --version
```

### FreeBSD

```bash
# Using pkg
pkg install deluge

# Enable in rc.conf
echo 'deluge_enable="YES"' >> /etc/rc.conf

# Start service
service deluge start

# Verify installation
deluge --version
```

### Windows

```bash
# Using Chocolatey
choco install deluge

# Or using Scoop
scoop install deluge

# Verify installation
deluge --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/deluge

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
deluge --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable deluge

# Start service
sudo systemctl start deluge

# Stop service
sudo systemctl stop deluge

# Restart service
sudo systemctl restart deluge

# Check status
sudo systemctl status deluge

# View logs
sudo journalctl -u deluge -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add deluge default

# Start service
rc-service deluge start

# Stop service
rc-service deluge stop

# Restart service
rc-service deluge restart

# Check status
rc-service deluge status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'deluge_enable="YES"' >> /etc/rc.conf

# Start service
service deluge start

# Stop service
service deluge stop

# Restart service
service deluge restart

# Check status
service deluge status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start deluge
brew services stop deluge
brew services restart deluge

# Check status
brew services list | grep deluge
```

### Windows Service Manager

```powershell
# Start service
net start deluge

# Stop service
net stop deluge

# Using PowerShell
Start-Service deluge
Stop-Service deluge
Restart-Service deluge

# Check status
Get-Service deluge
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream deluge_backend {
    server 127.0.0.1:8112;
}

server {
    listen 80;
    server_name deluge.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name deluge.example.com;

    ssl_certificate /etc/ssl/certs/deluge.example.com.crt;
    ssl_certificate_key /etc/ssl/private/deluge.example.com.key;

    location / {
        proxy_pass http://deluge_backend;
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
    ServerName deluge.example.com
    Redirect permanent / https://deluge.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName deluge.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/deluge.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/deluge.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8112/
    ProxyPassReverse / http://127.0.0.1:8112/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend deluge_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/deluge.pem
    redirect scheme https if !{ ssl_fc }
    default_backend deluge_backend

backend deluge_backend
    balance roundrobin
    server deluge1 127.0.0.1:8112 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R deluge:deluge /etc/deluge
sudo chmod 750 /etc/deluge

# Configure firewall
sudo firewall-cmd --permanent --add-port=8112/tcp
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
sudo systemctl status deluge

# View logs
sudo journalctl -u deluge -f

# Monitor resource usage
top -p $(pgrep deluge)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/deluge"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/deluge-backup-$DATE.tar.gz" /etc/deluge /var/lib/deluge

echo "Backup completed: $BACKUP_DIR/deluge-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop deluge

# Restore from backup
tar -xzf /backup/deluge/deluge-backup-*.tar.gz -C /

# Start service
sudo systemctl start deluge
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u deluge -n 100
sudo tail -f /var/log/deluge/deluge.log

# Check configuration
deluge --version

# Check permissions
ls -la /etc/deluge
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8112

# Test connectivity
telnet localhost 8112

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep deluge)

# Check disk I/O
iotop -p $(pgrep deluge)

# Check connections
ss -an | grep 8112
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  deluge:
    image: deluge:latest
    ports:
      - "8112:8112"
    volumes:
      - ./config:/etc/deluge
      - ./data:/var/lib/deluge
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update deluge

# Debian/Ubuntu
sudo apt update && sudo apt upgrade deluge

# Arch Linux
sudo pacman -Syu deluge

# Alpine Linux
apk update && apk upgrade deluge

# openSUSE
sudo zypper update deluge

# FreeBSD
pkg update && pkg upgrade deluge

# Always backup before updates
tar -czf /backup/deluge-pre-update-$(date +%Y%m%d).tar.gz /etc/deluge

# Restart after updates
sudo systemctl restart deluge
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/deluge

# Clean old logs
find /var/log/deluge -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/deluge
```

## Additional Resources

- Official Documentation: https://docs.deluge.org/
- GitHub Repository: https://github.com/deluge/deluge
- Community Forum: https://forum.deluge.org/
- Best Practices Guide: https://docs.deluge.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
