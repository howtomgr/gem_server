# gem-server Installation Guide

gem-server is a free and open-source Ruby gem repository. Gem Server provides private Ruby gem hosting

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
  - Storage: 10GB for gems
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9292 (default gem-server port)
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

# Install gem-server
sudo dnf install -y gem_server

# Enable and start service
sudo systemctl enable --now gem-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9292/tcp
sudo firewall-cmd --reload

# Verify installation
gem-server --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gem-server
sudo apt install -y gem_server

# Enable and start service
sudo systemctl enable --now gem-server

# Configure firewall
sudo ufw allow 9292

# Verify installation
gem-server --version
```

### Arch Linux

```bash
# Install gem-server
sudo pacman -S gem_server

# Enable and start service
sudo systemctl enable --now gem-server

# Verify installation
gem-server --version
```

### Alpine Linux

```bash
# Install gem-server
apk add --no-cache gem_server

# Enable and start service
rc-update add gem-server default
rc-service gem-server start

# Verify installation
gem-server --version
```

### openSUSE/SLES

```bash
# Install gem-server
sudo zypper install -y gem_server

# Enable and start service
sudo systemctl enable --now gem-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9292/tcp
sudo firewall-cmd --reload

# Verify installation
gem-server --version
```

### macOS

```bash
# Using Homebrew
brew install gem_server

# Start service
brew services start gem_server

# Verify installation
gem-server --version
```

### FreeBSD

```bash
# Using pkg
pkg install gem_server

# Enable in rc.conf
echo 'gem-server_enable="YES"' >> /etc/rc.conf

# Start service
service gem-server start

# Verify installation
gem-server --version
```

### Windows

```bash
# Using Chocolatey
choco install gem_server

# Or using Scoop
scoop install gem_server

# Verify installation
gem-server --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gem_server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gem-server --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gem-server

# Start service
sudo systemctl start gem-server

# Stop service
sudo systemctl stop gem-server

# Restart service
sudo systemctl restart gem-server

# Check status
sudo systemctl status gem-server

# View logs
sudo journalctl -u gem-server -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gem-server default

# Start service
rc-service gem-server start

# Stop service
rc-service gem-server stop

# Restart service
rc-service gem-server restart

# Check status
rc-service gem-server status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gem-server_enable="YES"' >> /etc/rc.conf

# Start service
service gem-server start

# Stop service
service gem-server stop

# Restart service
service gem-server restart

# Check status
service gem-server status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gem_server
brew services stop gem_server
brew services restart gem_server

# Check status
brew services list | grep gem_server
```

### Windows Service Manager

```powershell
# Start service
net start gem-server

# Stop service
net stop gem-server

# Using PowerShell
Start-Service gem-server
Stop-Service gem-server
Restart-Service gem-server

# Check status
Get-Service gem-server
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gem_server_backend {
    server 127.0.0.1:9292;
}

server {
    listen 80;
    server_name gem_server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gem_server.example.com;

    ssl_certificate /etc/ssl/certs/gem_server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gem_server.example.com.key;

    location / {
        proxy_pass http://gem_server_backend;
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
    ServerName gem_server.example.com
    Redirect permanent / https://gem_server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gem_server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gem_server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gem_server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9292/
    ProxyPassReverse / http://127.0.0.1:9292/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gem_server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gem_server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gem_server_backend

backend gem_server_backend
    balance roundrobin
    server gem_server1 127.0.0.1:9292 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gem_server:gem_server /etc/gem_server
sudo chmod 750 /etc/gem_server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9292/tcp
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
sudo systemctl status gem-server

# View logs
sudo journalctl -u gem-server -f

# Monitor resource usage
top -p $(pgrep gem_server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gem_server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gem_server-backup-$DATE.tar.gz" /etc/gem_server /var/lib/gem_server

echo "Backup completed: $BACKUP_DIR/gem_server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gem-server

# Restore from backup
tar -xzf /backup/gem_server/gem_server-backup-*.tar.gz -C /

# Start service
sudo systemctl start gem-server
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gem-server -n 100
sudo tail -f /var/log/gem_server/gem_server.log

# Check configuration
gem-server --version

# Check permissions
ls -la /etc/gem_server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9292

# Test connectivity
telnet localhost 9292

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gem_server)

# Check disk I/O
iotop -p $(pgrep gem_server)

# Check connections
ss -an | grep 9292
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gem_server:
    image: gem_server:latest
    ports:
      - "9292:9292"
    volumes:
      - ./config:/etc/gem_server
      - ./data:/var/lib/gem_server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gem_server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gem_server

# Arch Linux
sudo pacman -Syu gem_server

# Alpine Linux
apk update && apk upgrade gem_server

# openSUSE
sudo zypper update gem_server

# FreeBSD
pkg update && pkg upgrade gem_server

# Always backup before updates
tar -czf /backup/gem_server-pre-update-$(date +%Y%m%d).tar.gz /etc/gem_server

# Restart after updates
sudo systemctl restart gem-server
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gem_server

# Clean old logs
find /var/log/gem_server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gem_server
```

## Additional Resources

- Official Documentation: https://docs.gem_server.org/
- GitHub Repository: https://github.com/gem_server/gem_server
- Community Forum: https://forum.gem_server.org/
- Best Practices Guide: https://docs.gem_server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
