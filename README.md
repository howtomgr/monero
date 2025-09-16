# monero Installation Guide

monero is a free and open-source privacy coin node. Monero provides private, secure, untraceable cryptocurrency node

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
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 100GB+ for chain
  - Network: P2P protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 18080 (default monero port)
  - RPC on 18081
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

# Install monero
sudo dnf install -y monero

# Enable and start service
sudo systemctl enable --now monero

# Configure firewall
sudo firewall-cmd --permanent --add-port=18080/tcp
sudo firewall-cmd --reload

# Verify installation
monero --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install monero
sudo apt install -y monero

# Enable and start service
sudo systemctl enable --now monero

# Configure firewall
sudo ufw allow 18080

# Verify installation
monero --version
```

### Arch Linux

```bash
# Install monero
sudo pacman -S monero

# Enable and start service
sudo systemctl enable --now monero

# Verify installation
monero --version
```

### Alpine Linux

```bash
# Install monero
apk add --no-cache monero

# Enable and start service
rc-update add monero default
rc-service monero start

# Verify installation
monero --version
```

### openSUSE/SLES

```bash
# Install monero
sudo zypper install -y monero

# Enable and start service
sudo systemctl enable --now monero

# Configure firewall
sudo firewall-cmd --permanent --add-port=18080/tcp
sudo firewall-cmd --reload

# Verify installation
monero --version
```

### macOS

```bash
# Using Homebrew
brew install monero

# Start service
brew services start monero

# Verify installation
monero --version
```

### FreeBSD

```bash
# Using pkg
pkg install monero

# Enable in rc.conf
echo 'monero_enable="YES"' >> /etc/rc.conf

# Start service
service monero start

# Verify installation
monero --version
```

### Windows

```bash
# Using Chocolatey
choco install monero

# Or using Scoop
scoop install monero

# Verify installation
monero --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/monero

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
monero --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable monero

# Start service
sudo systemctl start monero

# Stop service
sudo systemctl stop monero

# Restart service
sudo systemctl restart monero

# Check status
sudo systemctl status monero

# View logs
sudo journalctl -u monero -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add monero default

# Start service
rc-service monero start

# Stop service
rc-service monero stop

# Restart service
rc-service monero restart

# Check status
rc-service monero status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'monero_enable="YES"' >> /etc/rc.conf

# Start service
service monero start

# Stop service
service monero stop

# Restart service
service monero restart

# Check status
service monero status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start monero
brew services stop monero
brew services restart monero

# Check status
brew services list | grep monero
```

### Windows Service Manager

```powershell
# Start service
net start monero

# Stop service
net stop monero

# Using PowerShell
Start-Service monero
Stop-Service monero
Restart-Service monero

# Check status
Get-Service monero
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream monero_backend {
    server 127.0.0.1:18080;
}

server {
    listen 80;
    server_name monero.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name monero.example.com;

    ssl_certificate /etc/ssl/certs/monero.example.com.crt;
    ssl_certificate_key /etc/ssl/private/monero.example.com.key;

    location / {
        proxy_pass http://monero_backend;
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
    ServerName monero.example.com
    Redirect permanent / https://monero.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName monero.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/monero.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/monero.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:18080/
    ProxyPassReverse / http://127.0.0.1:18080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend monero_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/monero.pem
    redirect scheme https if !{ ssl_fc }
    default_backend monero_backend

backend monero_backend
    balance roundrobin
    server monero1 127.0.0.1:18080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R monero:monero /etc/monero
sudo chmod 750 /etc/monero

# Configure firewall
sudo firewall-cmd --permanent --add-port=18080/tcp
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
sudo systemctl status monero

# View logs
sudo journalctl -u monero -f

# Monitor resource usage
top -p $(pgrep monero)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/monero"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/monero-backup-$DATE.tar.gz" /etc/monero /var/lib/monero

echo "Backup completed: $BACKUP_DIR/monero-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop monero

# Restore from backup
tar -xzf /backup/monero/monero-backup-*.tar.gz -C /

# Start service
sudo systemctl start monero
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u monero -n 100
sudo tail -f /var/log/monero/monero.log

# Check configuration
monero --version

# Check permissions
ls -la /etc/monero
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 18080

# Test connectivity
telnet localhost 18080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep monero)

# Check disk I/O
iotop -p $(pgrep monero)

# Check connections
ss -an | grep 18080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  monero:
    image: monero:latest
    ports:
      - "18080:18080"
    volumes:
      - ./config:/etc/monero
      - ./data:/var/lib/monero
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update monero

# Debian/Ubuntu
sudo apt update && sudo apt upgrade monero

# Arch Linux
sudo pacman -Syu monero

# Alpine Linux
apk update && apk upgrade monero

# openSUSE
sudo zypper update monero

# FreeBSD
pkg update && pkg upgrade monero

# Always backup before updates
tar -czf /backup/monero-pre-update-$(date +%Y%m%d).tar.gz /etc/monero

# Restart after updates
sudo systemctl restart monero
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/monero

# Clean old logs
find /var/log/monero -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/monero
```

## Additional Resources

- Official Documentation: https://docs.monero.org/
- GitHub Repository: https://github.com/monero/monero
- Community Forum: https://forum.monero.org/
- Best Practices Guide: https://docs.monero.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
