# Linux System Administration Notes

Here's a comprehensive guide to basic Linux system administration tasks with examples:

## User Management

### Creating Users

```bash
# Create a new user
sudo useradd username

# Create user with home directory
sudo useradd -m username

# Create user with specific home directory
sudo useradd -m -d /home/custom_dir username

# Create user with specific shell
sudo useradd -m -s /bin/bash username
```

### Setting Up User Passwords

```bash
# Set password for a user
sudo passwd username

# Change your own password
passwd

# Force user to change password at next login
sudo passwd --expire username

# Set password policy (in /etc/login.defs)
# Example: Set maximum password age to 90 days
PASS_MAX_DAYS 90
```

## File Management

### Creating Files

```bash
# Create empty file
touch filename.txt

# Create file with content
echo "Hello World" > filename.txt

# Create file using a text editor
nano filename.txt
vim filename.txt
```

### Copying Files

```bash
# Copy a file
cp source.txt destination.txt

# Copy recursively (directories and contents)
cp -r source_dir destination_dir

# Copy with preserving attributes
cp -p source.txt destination.txt

# Copy with verbose output
cp -v source.txt destination.txt
```

### Moving Files

```bash
# Move/rename a file
mv old_name.txt new_name.txt

# Move a file to another directory
mv file.txt /path/to/destination/

# Move multiple files to a directory
mv file1.txt file2.txt /destination/
```

### Deleting Files

```bash
# Remove a file
rm filename.txt

# Remove a file with confirmation
rm -i filename.txt

# Remove directory and contents
rm -r directory/

# Force remove write-protected files
rm -f protected_file.txt
```

## Linux File Permissions

### Viewing Permissions

```bash
# View file permissions
ls -l filename.txt

# Example output:
# -rw-r--r-- 1 user group 1234 Apr 15 10:30 filename.txt
# (type)(user)(group)(others)
```

### Setting Permissions with Symbolic Mode

```bash
# Give owner execute permission
chmod u+x script.sh

# Remove write permission from group
chmod g-w filename.txt

# Give everyone read permission
chmod a+r filename.txt

# Set specific permissions for user, group, others
chmod u=rwx,g=rx,o=r filename.txt
```

### Setting Permissions with Numeric Mode

```bash
# Common permission sets:
chmod 755 script.sh    # rwxr-xr-x (executable script)
chmod 644 file.txt     # rw-r--r-- (readable file)
chmod 600 private.key  # rw------- (private file)
chmod 777 public_dir   # rwxrwxrwx (fully accessible dir)
```

### Changing Ownership

```bash
# Change file owner
sudo chown username filename.txt

# Change file group
sudo chgrp groupname filename.txt

# Change both owner and group
sudo chown username:groupname filename.txt

# Change recursively for directories
sudo chown -R username:groupname directory/
```

## Linux Package Management

### For Debian/Ubuntu (apt)

```bash
# Update package lists
sudo apt update

# Install a package
sudo apt install package_name

# Remove a package
sudo apt remove package_name

# Remove package and configuration files
sudo apt purge package_name

# Upgrade all packages
sudo apt upgrade

# Search for packages
apt search keyword
```

### For Red Hat/CentOS (yum/dnf)

```bash
# Install a package
sudo yum install package_name
sudo dnf install package_name

# Remove a package
sudo yum remove package_name
sudo dnf remove package_name

# Update package lists and upgrade
sudo yum update
sudo dnf update

# Search for packages
yum search keyword
dnf search keyword
```

## Linux File System

### Creating File Systems

```bash
# Format a partition with ext4
sudo mkfs.ext4 /dev/sdb1

# Format with XFS
sudo mkfs.xfs /dev/sdb2

# Label a filesystem
sudo e2label /dev/sdb1 mydisk

# Check filesystem for errors
sudo fsck /dev/sdb1
```

### Mounting File Systems

```bash
# Create a mount point
sudo mkdir /mnt/mydisk

# Mount manually
sudo mount /dev/sdb1 /mnt/mydisk

# Mount with specific options
sudo mount -o rw,noexec /dev/sdb1 /mnt/mydisk

# Show mounted filesystems
mount
df -h
```

### Unmounting File Systems

```bash
# Unmount a filesystem
sudo umount /mnt/mydisk
# Or
sudo umount /dev/sdb1

# Force unmount (if busy)
sudo umount -f /mnt/mydisk
```

### Automounting with /etc/fstab

```bash
# Add entry to /etc/fstab for permanent mounting
# Format: <device> <mount point> <filesystem> <options> <dump> <pass>

# Example entry:
/dev/sdb1 /mnt/mydisk ext4 defaults 0 2

# Mount all entries from fstab
sudo mount -a
```



# Linux Process Management

Here's a comprehensive overview of process management in Linux:

## Viewing Processes

### ps - Process Status
```bash
# Show all processes for the current user
ps

# Show all processes on the system
ps -e  # or ps -A

# Show detailed information
ps -ef  # or ps aux

# Format output for specific information
ps -eo pid,ppid,cmd,%cpu,%mem
```

### top - Real-time Process Viewer
```bash
# Basic usage
top

# Sort processes by memory usage
top -o %MEM

# Update every 2 seconds
top -d 2
```

### htop - Enhanced top (may need installation)
```bash
# Install htop
sudo apt install htop  # Debian/Ubuntu
sudo dnf install htop  # Fedora/RHEL

# Run htop
htop
```

## Process Control

### Starting Processes
```bash
# Run a program in foreground
command

# Run in background
command &

# Run with specific priority (nice value)
nice -n 10 command
```

### Killing Processes
```bash
# Kill by PID
kill PID

# Force kill (SIGKILL)
kill -9 PID
kill -KILL PID

# Kill by name
killall process_name

# Kill by pattern
pkill pattern
```


# Essential Linux Disk Management Commands

## Disk Information

```bash
# Check disk space usage
df -h

# Directory size
du -sh /path/to/directory

# Find largest directories
du -h /path | sort -rh | head -5

# List block devices
lsblk

# Detailed disk info
sudo fdisk -l
```

## Partitioning

```bash
# Create/modify partitions (MBR)
sudo fdisk /dev/sdb

# Create/modify partitions (GPT)
sudo gdisk /dev/sdb
# or
sudo parted /dev/sdb
```

## Filesystem Operations

```bash
# Format as ext4
sudo mkfs.ext4 /dev/sdb1

# Format as XFS
sudo mkfs.xfs /dev/sdb1

# Check filesystem
sudo fsck -f /dev/sdb1

# Mount filesystem
sudo mount /dev/sdb1 /mnt/data

# Unmount
sudo umount /mnt/data

# Add to fstab (persistent mount)
echo "UUID=$(sudo blkid -s UUID -o value /dev/sdb1) /mnt/data ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

## Logical Volume Management (LVM)

```bash
# Create physical volume
sudo pvcreate /dev/sdb

# Create volume group
sudo vgcreate vg_data /dev/sdb

# Create logical volume
sudo lvcreate -L 50G -n lv_data vg_data

# Extend logical volume
sudo lvextend -L +10G /dev/vg_data/lv_data

# Resize filesystem after extending
sudo resize2fs /dev/vg_data/lv_data  # for ext4
sudo xfs_growfs /mnt/data  # for XFS
```

## RAID Management

```bash
# Create RAID 1 (mirror)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1

# Check RAID status
cat /proc/mdstat
```

######################################################
# Adding an EBS Volume to an EC2 Instance

Here's a step-by-step guide on how to add an Amazon EBS (Elastic Block Store) volume to an EC2 instance:

## Method 1: Using AWS Management Console

### Step 1: Create a new EBS volume
1. Navigate to the AWS Management Console
2. Go to EC2 dashboard
3. In the navigation pane, select "Volumes" under "Elastic Block Store"
4. Click "Create Volume"
5. Configure volume settings:
   - Volume Type (gp3, gp2, io1, st1, or sc1)
   - Size (in GiB)
   - Availability Zone (must be the same as your EC2 instance)
   - Optional: Add encryption and tags
6. Click "Create Volume"

### Step 2: Attach the volume to your EC2 instance
1. Select the newly created volume
2. Click "Actions" → "Attach Volume"
3. Select your EC2 instance from the dropdown
4. Note the device name (e.g., /dev/sdf, /dev/xvdf)
5. Click "Attach"

### Step 3: Connect to your EC2 instance and format/mount the volume
1. SSH into your EC2 instance
2. Check if the volume is recognized:
   ```bash
   lsblk
   ```

3. Format the volume (if it's new):
   ```bash
   sudo mkfs -t ext4 /dev/xvdf
   ```

4. Create a mount point:
   ```bash
   sudo mkdir /data
   ```

5. Mount the volume:
   ```bash
   sudo mount /dev/xvdf /data
   ```

6. To make the mount persistent across reboots, add to /etc/fstab:
   ```bash
   echo '/dev/xvdf /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
   ```


   

