# kibana Installation Guide

kibana is a free and open-source Elasticsearch visualization. Kibana provides visualization for Elasticsearch data

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
  - RAM: 2GB minimum
  - Storage: 2GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5601 (default kibana port)
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

# Install kibana
sudo dnf install -y kibana

# Enable and start service
sudo systemctl enable --now kibana

# Configure firewall
sudo firewall-cmd --permanent --add-port=5601/tcp
sudo firewall-cmd --reload

# Verify installation
kibana --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kibana
sudo apt install -y kibana

# Enable and start service
sudo systemctl enable --now kibana

# Configure firewall
sudo ufw allow 5601

# Verify installation
kibana --version
```

### Arch Linux

```bash
# Install kibana
sudo pacman -S kibana

# Enable and start service
sudo systemctl enable --now kibana

# Verify installation
kibana --version
```

### Alpine Linux

```bash
# Install kibana
apk add --no-cache kibana

# Enable and start service
rc-update add kibana default
rc-service kibana start

# Verify installation
kibana --version
```

### openSUSE/SLES

```bash
# Install kibana
sudo zypper install -y kibana

# Enable and start service
sudo systemctl enable --now kibana

# Configure firewall
sudo firewall-cmd --permanent --add-port=5601/tcp
sudo firewall-cmd --reload

# Verify installation
kibana --version
```

### macOS

```bash
# Using Homebrew
brew install kibana

# Start service
brew services start kibana

# Verify installation
kibana --version
```

### FreeBSD

```bash
# Using pkg
pkg install kibana

# Enable in rc.conf
echo 'kibana_enable="YES"' >> /etc/rc.conf

# Start service
service kibana start

# Verify installation
kibana --version
```

### Windows

```bash
# Using Chocolatey
choco install kibana

# Or using Scoop
scoop install kibana

# Verify installation
kibana --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kibana

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kibana --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kibana

# Start service
sudo systemctl start kibana

# Stop service
sudo systemctl stop kibana

# Restart service
sudo systemctl restart kibana

# Check status
sudo systemctl status kibana

# View logs
sudo journalctl -u kibana -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kibana default

# Start service
rc-service kibana start

# Stop service
rc-service kibana stop

# Restart service
rc-service kibana restart

# Check status
rc-service kibana status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kibana_enable="YES"' >> /etc/rc.conf

# Start service
service kibana start

# Stop service
service kibana stop

# Restart service
service kibana restart

# Check status
service kibana status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kibana
brew services stop kibana
brew services restart kibana

# Check status
brew services list | grep kibana
```

### Windows Service Manager

```powershell
# Start service
net start kibana

# Stop service
net stop kibana

# Using PowerShell
Start-Service kibana
Stop-Service kibana
Restart-Service kibana

# Check status
Get-Service kibana
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kibana_backend {
    server 127.0.0.1:5601;
}

server {
    listen 80;
    server_name kibana.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kibana.example.com;

    ssl_certificate /etc/ssl/certs/kibana.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kibana.example.com.key;

    location / {
        proxy_pass http://kibana_backend;
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
    ServerName kibana.example.com
    Redirect permanent / https://kibana.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kibana.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kibana.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kibana.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5601/
    ProxyPassReverse / http://127.0.0.1:5601/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kibana_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kibana.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kibana_backend

backend kibana_backend
    balance roundrobin
    server kibana1 127.0.0.1:5601 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kibana:kibana /etc/kibana
sudo chmod 750 /etc/kibana

# Configure firewall
sudo firewall-cmd --permanent --add-port=5601/tcp
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
sudo systemctl status kibana

# View logs
sudo journalctl -u kibana -f

# Monitor resource usage
top -p $(pgrep kibana)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kibana"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kibana-backup-$DATE.tar.gz" /etc/kibana /var/lib/kibana

echo "Backup completed: $BACKUP_DIR/kibana-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kibana

# Restore from backup
tar -xzf /backup/kibana/kibana-backup-*.tar.gz -C /

# Start service
sudo systemctl start kibana
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kibana -n 100
sudo tail -f /var/log/kibana/kibana.log

# Check configuration
kibana --version

# Check permissions
ls -la /etc/kibana
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5601

# Test connectivity
telnet localhost 5601

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kibana)

# Check disk I/O
iotop -p $(pgrep kibana)

# Check connections
ss -an | grep 5601
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kibana:
    image: kibana:latest
    ports:
      - "5601:5601"
    volumes:
      - ./config:/etc/kibana
      - ./data:/var/lib/kibana
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kibana

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kibana

# Arch Linux
sudo pacman -Syu kibana

# Alpine Linux
apk update && apk upgrade kibana

# openSUSE
sudo zypper update kibana

# FreeBSD
pkg update && pkg upgrade kibana

# Always backup before updates
tar -czf /backup/kibana-pre-update-$(date +%Y%m%d).tar.gz /etc/kibana

# Restart after updates
sudo systemctl restart kibana
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kibana

# Clean old logs
find /var/log/kibana -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kibana
```

## Additional Resources

- Official Documentation: https://docs.kibana.org/
- GitHub Repository: https://github.com/kibana/kibana
- Community Forum: https://forum.kibana.org/
- Best Practices Guide: https://docs.kibana.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
