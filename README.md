# prometheus-alertmanager Installation Guide

prometheus-alertmanager is a free and open-source alert handling component for Prometheus. Part of the Prometheus ecosystem, Alertmanager handles alerts sent by Prometheus server and routes them to receivers like email, Slack, or PagerDuty

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
  - RAM: 128MB minimum (512MB+ recommended)
  - Storage: 1GB for data
  - Network: HTTP API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9093 (default prometheus-alertmanager port)
  - Port 9094 for cluster peers
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

# Install prometheus-alertmanager
sudo dnf install -y alertmanager

# Enable and start service
sudo systemctl enable --now alertmanager

# Configure firewall
sudo firewall-cmd --permanent --add-port=9093/tcp
sudo firewall-cmd --reload

# Verify installation
alertmanager --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install prometheus-alertmanager
sudo apt install -y alertmanager

# Enable and start service
sudo systemctl enable --now alertmanager

# Configure firewall
sudo ufw allow 9093

# Verify installation
alertmanager --version
```

### Arch Linux

```bash
# Install prometheus-alertmanager
sudo pacman -S alertmanager

# Enable and start service
sudo systemctl enable --now alertmanager

# Verify installation
alertmanager --version
```

### Alpine Linux

```bash
# Install prometheus-alertmanager
apk add --no-cache alertmanager

# Enable and start service
rc-update add alertmanager default
rc-service alertmanager start

# Verify installation
alertmanager --version
```

### openSUSE/SLES

```bash
# Install prometheus-alertmanager
sudo zypper install -y alertmanager

# Enable and start service
sudo systemctl enable --now alertmanager

# Configure firewall
sudo firewall-cmd --permanent --add-port=9093/tcp
sudo firewall-cmd --reload

# Verify installation
alertmanager --version
```

### macOS

```bash
# Using Homebrew
brew install alertmanager

# Start service
brew services start alertmanager

# Verify installation
alertmanager --version
```

### FreeBSD

```bash
# Using pkg
pkg install alertmanager

# Enable in rc.conf
echo 'alertmanager_enable="YES"' >> /etc/rc.conf

# Start service
service alertmanager start

# Verify installation
alertmanager --version
```

### Windows

```bash
# Using Chocolatey
choco install alertmanager

# Or using Scoop
scoop install alertmanager

# Verify installation
alertmanager --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/alertmanager

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
alertmanager --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable alertmanager

# Start service
sudo systemctl start alertmanager

# Stop service
sudo systemctl stop alertmanager

# Restart service
sudo systemctl restart alertmanager

# Check status
sudo systemctl status alertmanager

# View logs
sudo journalctl -u alertmanager -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add alertmanager default

# Start service
rc-service alertmanager start

# Stop service
rc-service alertmanager stop

# Restart service
rc-service alertmanager restart

# Check status
rc-service alertmanager status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'alertmanager_enable="YES"' >> /etc/rc.conf

# Start service
service alertmanager start

# Stop service
service alertmanager stop

# Restart service
service alertmanager restart

# Check status
service alertmanager status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start alertmanager
brew services stop alertmanager
brew services restart alertmanager

# Check status
brew services list | grep alertmanager
```

### Windows Service Manager

```powershell
# Start service
net start alertmanager

# Stop service
net stop alertmanager

# Using PowerShell
Start-Service alertmanager
Stop-Service alertmanager
Restart-Service alertmanager

# Check status
Get-Service alertmanager
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream alertmanager_backend {
    server 127.0.0.1:9093;
}

server {
    listen 80;
    server_name alertmanager.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name alertmanager.example.com;

    ssl_certificate /etc/ssl/certs/alertmanager.example.com.crt;
    ssl_certificate_key /etc/ssl/private/alertmanager.example.com.key;

    location / {
        proxy_pass http://alertmanager_backend;
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
    ServerName alertmanager.example.com
    Redirect permanent / https://alertmanager.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName alertmanager.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/alertmanager.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/alertmanager.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9093/
    ProxyPassReverse / http://127.0.0.1:9093/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend alertmanager_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/alertmanager.pem
    redirect scheme https if !{ ssl_fc }
    default_backend alertmanager_backend

backend alertmanager_backend
    balance roundrobin
    server alertmanager1 127.0.0.1:9093 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chmod 750 /etc/alertmanager

# Configure firewall
sudo firewall-cmd --permanent --add-port=9093/tcp
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
sudo systemctl status alertmanager

# View logs
sudo journalctl -u alertmanager -f

# Monitor resource usage
top -p $(pgrep alertmanager)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/alertmanager"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/alertmanager-backup-$DATE.tar.gz" /etc/alertmanager /var/lib/alertmanager

echo "Backup completed: $BACKUP_DIR/alertmanager-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop alertmanager

# Restore from backup
tar -xzf /backup/alertmanager/alertmanager-backup-*.tar.gz -C /

# Start service
sudo systemctl start alertmanager
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u alertmanager -n 100
sudo tail -f /var/log/alertmanager/alertmanager.log

# Check configuration
alertmanager --version

# Check permissions
ls -la /etc/alertmanager
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9093

# Test connectivity
telnet localhost 9093

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep alertmanager)

# Check disk I/O
iotop -p $(pgrep alertmanager)

# Check connections
ss -an | grep 9093
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  alertmanager:
    image: alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./config:/etc/alertmanager
      - ./data:/var/lib/alertmanager
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update alertmanager

# Debian/Ubuntu
sudo apt update && sudo apt upgrade alertmanager

# Arch Linux
sudo pacman -Syu alertmanager

# Alpine Linux
apk update && apk upgrade alertmanager

# openSUSE
sudo zypper update alertmanager

# FreeBSD
pkg update && pkg upgrade alertmanager

# Always backup before updates
tar -czf /backup/alertmanager-pre-update-$(date +%Y%m%d).tar.gz /etc/alertmanager

# Restart after updates
sudo systemctl restart alertmanager
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/alertmanager

# Clean old logs
find /var/log/alertmanager -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/alertmanager
```

## Additional Resources

- Official Documentation: https://docs.alertmanager.org/
- GitHub Repository: https://github.com/alertmanager/alertmanager
- Community Forum: https://forum.alertmanager.org/
- Best Practices Guide: https://docs.alertmanager.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
