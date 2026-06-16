# Linux Preseed & Kickstart Configurations

🔧 **Minimal, Stable, High-Performance Linux System Configurations**

Automated installation configurations for **Ubuntu 22.04 LTS** and **CentOS/RHEL 8+** with all unnecessary components removed.

---

## 📋 Features

✅ **Minimal Installation** - Only essential packages included  
✅ **High Performance** - Optimized kernel parameters and I/O scheduler  
✅ **Server-Grade Stability** - Enterprise-ready configurations  
✅ **Security Hardened** - Automatic security updates enabled  
✅ **No GUI Bloat** - Server/CLI only, no desktop environment  
✅ **Fast Boot** - Disabled unnecessary daemons  
✅ **LVM Support** - Flexible disk partitioning  

---

## 📁 Files

### `preseed.cfg`
**Ubuntu 22.04 LTS automatic installation configuration**

**Specifications:**
- Language: English (US) | Keyboard: US
- Network: DHCP (auto-configured)
- Disk: LVM auto-partitioning
- Bootloader: GRUB2
- User: `ubuntu` (password: `ubuntu123`)
- Packages: OpenSSH, Git, Build-essentials, Development tools
- **Removed:** GNOME, KDE, CUPS, Avahi, Bluetooth, Apparmor logging
- Updates: Automatic security updates enabled

### `anaconda-ks.cfg`
**CentOS/RHEL 8+ automatic installation configuration**

**Specifications:**
- Language: English (US) | Keyboard: US
- Network: DHCP (auto-configured)
- Disk: LVM auto-partitioning
- Bootloader: GRUB2
- User: `centos` (password: `centos123`) with sudo access
- Root: Locked (use sudo instead)
- SELinux: Enforcing
- Firewall: Enabled (SSH allowed)
- **Removed:** WiFi drivers, Avahi, CUPS, Bluetooth, Firmware packages
- Updates: Automatic security updates via dnf-automatic

---

## 🚀 Usage

### **Ubuntu 22.04 LTS**

#### **Method 1: HTTP Server (Recommended)**
```bash
# 1. Copy preseed.cfg to your HTTP server
cp preseed.cfg /var/www/html/

# 2. Start HTTP server
python3 -m http.server 8000

# 3. Boot Ubuntu ISO with preseed parameter:
# linux /casper/vmlinuz preseed/url=http://YOUR_IP:8000/preseed.cfg
# Then press Enter
```

#### **Method 2: USB Stick Integration**
```bash
# 1. Mount Ubuntu ISO
mkdir -p /mnt/iso
sudo mount -o loop ubuntu-22.04-live-server-amd64.iso /mnt/iso

# 2. Create bootable USB
sudo dd if=ubuntu-22.04-live-server-amd64.iso of=/dev/sdX bs=4M status=progress

# 3. Add preseed.cfg to USB
sudo mkdir -p /mnt/usb/preseed
sudo cp preseed.cfg /mnt/usb/preseed/

# 4. Modify GRUB config to use local preseed
# Edit USB boot parameters with: preseed/file=/preseed/preseed.cfg
```

### **CentOS/RHEL 8+**

#### **Method 1: HTTP Server (Recommended)**
```bash
# 1. Copy anaconda-ks.cfg to HTTP server
cp anaconda-ks.cfg /var/www/html/

# 2. Boot with kickstart parameter:
# ks=http://YOUR_IP/anaconda-ks.cfg
```

#### **Method 2: Create Kickstart Boot Media**
```bash
# 1. Download CentOS ISO
wget https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/iso/CentOS-Stream-9-latest-x86_64-dvd1.iso

# 2. Create bootable USB
sudo dd if=CentOS-Stream-9-latest-x86_64-dvd1.iso of=/dev/sdX bs=4M status=progress

# 3. Configure boot loader with kickstart location
# ks=http://YOUR_IP/anaconda-ks.cfg
```

---

## 🔧 Customization

### **Change Hostname**
```bash
# Ubuntu (preseed.cfg)
d-i netcfg/get_hostname string your-hostname

# CentOS (anaconda-ks.cfg)
network  --hostname=your-hostname
```

### **Change Username & Password**
```bash
# Ubuntu (preseed.cfg)
d-i passwd/username string youruser
d-i passwd/user-password password yourpassword
d-i passwd/user-password-again password yourpassword

# CentOS (anaconda-ks.cfg)
user --name=youruser --plaintext --password=yourpassword --groups=wheel
```

### **Add Additional Packages**
```bash
# Ubuntu (preseed.cfg)
d-i pkgsel/include string openssh-server curl wget net-tools htop vim nano git build-essential linux-headers-generic YOUR_PACKAGES_HERE

# CentOS (anaconda-ks.cfg)
# Add packages in %packages section
%packages --nodefaults
@core
@standard
YOUR_PACKAGES_HERE
%end
```

### **Enable GUI (Desktop)**
**⚠️ Not recommended for servers, but if needed:**

```bash
# Ubuntu
# Remove from pkgsel/exclude line:
# xserver-xorg xserver-xorg-core ubuntu-desktop gnome-shell gdm3

# Add to pkgsel/include:
# ubuntu-desktop

# CentOS
# Add to %packages:
# @gnome-desktop
# gnome-shell
# gdm
```

---

## 📊 What Was Removed

| Component | Reason |
|-----------|--------|
| GNOME Desktop | Server doesn't need GUI |
| KDE/Wayland | Reduces RAM/disk usage |
| CUPS (Printing) | No printers on servers |
| Avahi (mDNS) | Not needed, creates noise |
| Bluetooth | Unused on servers |
| WiFi Firmware | Servers use Ethernet |
| ModemManager | Not applicable |
| Apparmor/AppArmor logging | Reduces overhead |
| NetworkManager GUI | CLI tools sufficient |

---

## 🎯 Performance Optimizations

### **Kernel Parameters** (CentOS/RHEL)
```bash
vm.swappiness = 10          # Reduce swap usage
net.core.somaxconn = 1024   # Connection backlog
```

### **I/O Scheduler**
```bash
elevator=deadline           # Better for VMs/SSDs
```

### **Boot Timeout**
```bash
GRUB_TIMEOUT=3              # Fast boot
```

---

## 📈 System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| **Disk** | 5 GB | 20 GB |
| **RAM** | 512 MB | 2 GB |
| **CPU** | 1 core | 2 cores |
| **Network** | 100 Mbps | 1 Gbps |

---

## ✅ Verification

### **After Ubuntu Installation**
```bash
# Check installed packages are minimal
dpkg -l | wc -l          # Should be ~300-400 packages

# Verify unnecessary services are disabled
systemctl list-unit-files | grep disabled

# Check disk usage
df -h /

# Monitor resource usage
free -h
top
```

### **After CentOS/RHEL Installation**
```bash
# List installed packages
dnf list installed | wc -l

# Verify SELinux status
getenforce

# Check enabled services
systemctl list-units --type=service --state=running
```

---

## 🛡️ Security Features

✅ **SSH Server** - Enabled and configured  
✅ **Firewall** - Active (SSH port open)  
✅ **SELinux** (CentOS/RHEL) - Enforcing mode  
✅ **Automatic Updates** - Enabled  
✅ **Root Login** - Disabled, use sudo  
✅ **Password Authentication** - Enabled  

---

## 📝 Notes

- Change default passwords immediately after installation
- Configure SSH keys for secure access
- Adjust network settings as needed for your environment
- Test configurations in VMs before production use
- Monitor system performance after deployment

---

## 📞 Support & Issues

For issues or improvements, please submit:
- GitHub Issues: https://github.com/pkfrr/linux-preseed/issues
- Pull Requests: https://github.com/pkfrr/linux-preseed/pulls

---

## 📄 License

These configurations are provided as-is for educational and production use.

---

**Created:** 2026-06-16  
**Last Updated:** 2026-06-16  
**Version:** 1.0
