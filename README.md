# razor Installation Guide

razor is a free and open-source provisioning solution. Puppet Razor provides hardware provisioning solution

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
  - Storage: 50GB for repos
  - Network: HTTP/TFTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8150 (default razor port)
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

# Install razor
sudo dnf install -y razor

# Enable and start service
sudo systemctl enable --now razor

# Configure firewall
sudo firewall-cmd --permanent --add-port=8150/tcp
sudo firewall-cmd --reload

# Verify installation
razor --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install razor
sudo apt install -y razor

# Enable and start service
sudo systemctl enable --now razor

# Configure firewall
sudo ufw allow 8150

# Verify installation
razor --version
```

### Arch Linux

```bash
# Install razor
sudo pacman -S razor

# Enable and start service
sudo systemctl enable --now razor

# Verify installation
razor --version
```

### Alpine Linux

```bash
# Install razor
apk add --no-cache razor

# Enable and start service
rc-update add razor default
rc-service razor start

# Verify installation
razor --version
```

### openSUSE/SLES

```bash
# Install razor
sudo zypper install -y razor

# Enable and start service
sudo systemctl enable --now razor

# Configure firewall
sudo firewall-cmd --permanent --add-port=8150/tcp
sudo firewall-cmd --reload

# Verify installation
razor --version
```

### macOS

```bash
# Using Homebrew
brew install razor

# Start service
brew services start razor

# Verify installation
razor --version
```

### FreeBSD

```bash
# Using pkg
pkg install razor

# Enable in rc.conf
echo 'razor_enable="YES"' >> /etc/rc.conf

# Start service
service razor start

# Verify installation
razor --version
```

### Windows

```bash
# Using Chocolatey
choco install razor

# Or using Scoop
scoop install razor

# Verify installation
razor --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/razor

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
razor --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable razor

# Start service
sudo systemctl start razor

# Stop service
sudo systemctl stop razor

# Restart service
sudo systemctl restart razor

# Check status
sudo systemctl status razor

# View logs
sudo journalctl -u razor -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add razor default

# Start service
rc-service razor start

# Stop service
rc-service razor stop

# Restart service
rc-service razor restart

# Check status
rc-service razor status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'razor_enable="YES"' >> /etc/rc.conf

# Start service
service razor start

# Stop service
service razor stop

# Restart service
service razor restart

# Check status
service razor status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start razor
brew services stop razor
brew services restart razor

# Check status
brew services list | grep razor
```

### Windows Service Manager

```powershell
# Start service
net start razor

# Stop service
net stop razor

# Using PowerShell
Start-Service razor
Stop-Service razor
Restart-Service razor

# Check status
Get-Service razor
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream razor_backend {
    server 127.0.0.1:8150;
}

server {
    listen 80;
    server_name razor.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name razor.example.com;

    ssl_certificate /etc/ssl/certs/razor.example.com.crt;
    ssl_certificate_key /etc/ssl/private/razor.example.com.key;

    location / {
        proxy_pass http://razor_backend;
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
    ServerName razor.example.com
    Redirect permanent / https://razor.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName razor.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/razor.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/razor.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8150/
    ProxyPassReverse / http://127.0.0.1:8150/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend razor_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/razor.pem
    redirect scheme https if !{ ssl_fc }
    default_backend razor_backend

backend razor_backend
    balance roundrobin
    server razor1 127.0.0.1:8150 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R razor:razor /etc/razor
sudo chmod 750 /etc/razor

# Configure firewall
sudo firewall-cmd --permanent --add-port=8150/tcp
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
sudo systemctl status razor

# View logs
sudo journalctl -u razor -f

# Monitor resource usage
top -p $(pgrep razor)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/razor"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/razor-backup-$DATE.tar.gz" /etc/razor /var/lib/razor

echo "Backup completed: $BACKUP_DIR/razor-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop razor

# Restore from backup
tar -xzf /backup/razor/razor-backup-*.tar.gz -C /

# Start service
sudo systemctl start razor
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u razor -n 100
sudo tail -f /var/log/razor/razor.log

# Check configuration
razor --version

# Check permissions
ls -la /etc/razor
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8150

# Test connectivity
telnet localhost 8150

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep razor)

# Check disk I/O
iotop -p $(pgrep razor)

# Check connections
ss -an | grep 8150
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  razor:
    image: razor:latest
    ports:
      - "8150:8150"
    volumes:
      - ./config:/etc/razor
      - ./data:/var/lib/razor
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update razor

# Debian/Ubuntu
sudo apt update && sudo apt upgrade razor

# Arch Linux
sudo pacman -Syu razor

# Alpine Linux
apk update && apk upgrade razor

# openSUSE
sudo zypper update razor

# FreeBSD
pkg update && pkg upgrade razor

# Always backup before updates
tar -czf /backup/razor-pre-update-$(date +%Y%m%d).tar.gz /etc/razor

# Restart after updates
sudo systemctl restart razor
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/razor

# Clean old logs
find /var/log/razor -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/razor
```

## Additional Resources

- Official Documentation: https://docs.razor.org/
- GitHub Repository: https://github.com/razor/razor
- Community Forum: https://forum.razor.org/
- Best Practices Guide: https://docs.razor.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
