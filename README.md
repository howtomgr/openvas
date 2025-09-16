# openvas Installation Guide

openvas is a free and open-source vulnerability assessment scanner. OpenVAS provides comprehensive vulnerability scanning and management, serving as an open-source alternative to Nessus or Qualys

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
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 5GB for feeds
  - Network: HTTPS for interface
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9392 (default openvas port)
  - Scanner port 9391
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

# Install openvas
sudo dnf install -y openvas

# Enable and start service
sudo systemctl enable --now openvas-scanner

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
sudo firewall-cmd --reload

# Verify installation
openvas --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install openvas
sudo apt install -y openvas

# Enable and start service
sudo systemctl enable --now openvas-scanner

# Configure firewall
sudo ufw allow 9392

# Verify installation
openvas --version
```

### Arch Linux

```bash
# Install openvas
sudo pacman -S openvas

# Enable and start service
sudo systemctl enable --now openvas-scanner

# Verify installation
openvas --version
```

### Alpine Linux

```bash
# Install openvas
apk add --no-cache openvas

# Enable and start service
rc-update add openvas-scanner default
rc-service openvas-scanner start

# Verify installation
openvas --version
```

### openSUSE/SLES

```bash
# Install openvas
sudo zypper install -y openvas

# Enable and start service
sudo systemctl enable --now openvas-scanner

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
sudo firewall-cmd --reload

# Verify installation
openvas --version
```

### macOS

```bash
# Using Homebrew
brew install openvas

# Start service
brew services start openvas

# Verify installation
openvas --version
```

### FreeBSD

```bash
# Using pkg
pkg install openvas

# Enable in rc.conf
echo 'openvas-scanner_enable="YES"' >> /etc/rc.conf

# Start service
service openvas-scanner start

# Verify installation
openvas --version
```

### Windows

```bash
# Using Chocolatey
choco install openvas

# Or using Scoop
scoop install openvas

# Verify installation
openvas --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/openvas

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
openvas --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable openvas-scanner

# Start service
sudo systemctl start openvas-scanner

# Stop service
sudo systemctl stop openvas-scanner

# Restart service
sudo systemctl restart openvas-scanner

# Check status
sudo systemctl status openvas-scanner

# View logs
sudo journalctl -u openvas-scanner -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add openvas-scanner default

# Start service
rc-service openvas-scanner start

# Stop service
rc-service openvas-scanner stop

# Restart service
rc-service openvas-scanner restart

# Check status
rc-service openvas-scanner status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'openvas-scanner_enable="YES"' >> /etc/rc.conf

# Start service
service openvas-scanner start

# Stop service
service openvas-scanner stop

# Restart service
service openvas-scanner restart

# Check status
service openvas-scanner status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start openvas
brew services stop openvas
brew services restart openvas

# Check status
brew services list | grep openvas
```

### Windows Service Manager

```powershell
# Start service
net start openvas-scanner

# Stop service
net stop openvas-scanner

# Using PowerShell
Start-Service openvas-scanner
Stop-Service openvas-scanner
Restart-Service openvas-scanner

# Check status
Get-Service openvas-scanner
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream openvas_backend {
    server 127.0.0.1:9392;
}

server {
    listen 80;
    server_name openvas.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name openvas.example.com;

    ssl_certificate /etc/ssl/certs/openvas.example.com.crt;
    ssl_certificate_key /etc/ssl/private/openvas.example.com.key;

    location / {
        proxy_pass http://openvas_backend;
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
    ServerName openvas.example.com
    Redirect permanent / https://openvas.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName openvas.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/openvas.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/openvas.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9392/
    ProxyPassReverse / http://127.0.0.1:9392/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend openvas_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/openvas.pem
    redirect scheme https if !{ ssl_fc }
    default_backend openvas_backend

backend openvas_backend
    balance roundrobin
    server openvas1 127.0.0.1:9392 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R openvas:openvas /etc/openvas
sudo chmod 750 /etc/openvas

# Configure firewall
sudo firewall-cmd --permanent --add-port=9392/tcp
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
sudo systemctl status openvas-scanner

# View logs
sudo journalctl -u openvas-scanner -f

# Monitor resource usage
top -p $(pgrep openvas)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/openvas"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/openvas-backup-$DATE.tar.gz" /etc/openvas /var/lib/openvas

echo "Backup completed: $BACKUP_DIR/openvas-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop openvas-scanner

# Restore from backup
tar -xzf /backup/openvas/openvas-backup-*.tar.gz -C /

# Start service
sudo systemctl start openvas-scanner
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u openvas-scanner -n 100
sudo tail -f /var/log/openvas/openvas.log

# Check configuration
openvas --version

# Check permissions
ls -la /etc/openvas
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9392

# Test connectivity
telnet localhost 9392

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep openvas)

# Check disk I/O
iotop -p $(pgrep openvas)

# Check connections
ss -an | grep 9392
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  openvas:
    image: openvas:latest
    ports:
      - "9392:9392"
    volumes:
      - ./config:/etc/openvas
      - ./data:/var/lib/openvas
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update openvas

# Debian/Ubuntu
sudo apt update && sudo apt upgrade openvas

# Arch Linux
sudo pacman -Syu openvas

# Alpine Linux
apk update && apk upgrade openvas

# openSUSE
sudo zypper update openvas

# FreeBSD
pkg update && pkg upgrade openvas

# Always backup before updates
tar -czf /backup/openvas-pre-update-$(date +%Y%m%d).tar.gz /etc/openvas

# Restart after updates
sudo systemctl restart openvas-scanner
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/openvas

# Clean old logs
find /var/log/openvas -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/openvas
```

## Additional Resources

- Official Documentation: https://docs.openvas.org/
- GitHub Repository: https://github.com/openvas/openvas
- Community Forum: https://forum.openvas.org/
- Best Practices Guide: https://docs.openvas.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
