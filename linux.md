# Linux System Administration Quick Reference

## 1. Hard Links vs Soft Links

### Hard Links
- Direct reference to the same inode
- Same file data blocks
- Cannot link directories
- Must be on same filesystem
- Deleting original file doesn't affect hard link

```bash
# Create hard link
ln original.txt hardlink.txt
```

### Soft Links (Symbolic Links)
- Separate file with path reference
- Can link directories
- Can cross filesystems
- Breaks if original file is deleted

```bash
# Create soft link
ln -s original.txt softlink.txt
```

## 2. Services Management

### Service Commands (systemd)
```bash
# Start a service
systemctl start servicename

# Stop a service
systemctl stop servicename

# Restart a service
systemctl restart servicename

# Enable service at boot
systemctl enable servicename

# Disable service at boot
systemctl disable servicename

# Check service status
systemctl status servicename
```

### Common Services
- `sshd`: Secure Shell daemon
- `httpd`: Web server
- `mysqld`: MySQL database
- `firewalld`: Firewall service

## 3. Process Viewing

### Process Listing Commands
```bash
# List all processes
ps aux

# Real-time process monitor
top

# Detailed process view
htop

# Find process by name
pgrep processname

# Kill process
kill PID
killall processname
```

### Netstat Process Monitoring
```bash
# Show all network connections
netstat -tuln

# Show process with network connections
netstat -tulpn

# Active connections
netstat -a

# TCP connections
netstat -at

# Continuous monitoring
netstat -c
```

## 4. Networking Commands

### Network Configuration
```bash
# Show IP addresses
ip addr
ifconfig

# Network connectivity
ping google.com

# Trace network path
traceroute google.com

# DNS lookup
dig google.com
nslookup google.com
```

### SSH and Remote Access
```bash
# Basic SSH connection
ssh username@hostname

# SSH with specific port
ssh -p 22 username@hostname

# Generate SSH key
ssh-keygen -t rsa

# Copy SSH key to remote server
ssh-copy-id username@hostname
```

### Additional Network Tools
```bash
# Download files
curl -O http://example.com/file

# Network scanning
nmap localhost
nmap -sP 192.168.1.0/24

# Edit hosts file
sudo nano /etc/hosts
```

## 5. Volume Management

### Volume Attachment Methods
```bash
# List block devices
lsblk

# Check disk space
df -h

# Mount a volume
mount /dev/sdb1 /mnt/newvolume

# Unmount volume
umount /mnt/newvolume

# Persistent mounting (edit)
sudo nano /etc/fstab
```

### LVM (Logical Volume Management)
```bash
# Create physical volume
pvcreate /dev/sdb

# Create volume group
vgcreate vg_name /dev/sdb

# Create logical volume
lvcreate -L 10G -n lv_name vg_name
```

## 6. Important Ports Reference

| Port | Service      | Protocol | Description                |
|------|--------------|----------|----------------------------|
| 21   | FTP          | TCP      | File Transfer Protocol     |
| 22   | SSH          | TCP      | Secure Shell               |
| 80   | HTTP         | TCP      | Web Traffic                |
| 443  | HTTPS        | TCP      | Secure Web Traffic         |
| 3306 | MySQL        | TCP      | MySQL Database             |
| 3307 | MariaDB      | TCP      | MariaDB Database           |
| 5432 | PostgreSQL   | TCP      | PostgreSQL Database        |
| 27017| MongoDB      | TCP      | MongoDB Database           |
| 990  | SFTP         | TCP      | Secure File Transfer       |

### Port Checking
```bash
# Check if port is open
netstat -tuln | grep :PORT
nmap localhost -p PORT
```

### Tips
- Always use strong passwords
- Keep services updated
- Use firewall to restrict unnecessary ports
- Regularly monitor network connections
