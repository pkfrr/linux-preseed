# 📦 Detailed Installation Guide

## 🖥️ Ubuntu 22.04 LTS Installation

### **Prerequisites**
- Ubuntu 22.04 LTS ISO (download from ubuntu.com)
- USB stick (8GB+) or CD/DVD
- Computer with network access
- HTTP server (optional but recommended)

### **Step 1: Prepare HTTP Server (Recommended Method)**

```bash
# On your server computer:
mkdir -p ~/preseed-server
cd ~/preseed-server

# Copy preseed.cfg here
cp preseed.cfg .

# Start Python HTTP server
python3 -m http.server 8000

# Find your server IP
hostname -I
# Example output: 192.168.1.100
```

### **Step 2: Create Bootable USB**

```bash
# On Linux/Mac:
# 1. Insert USB drive
# 2. Find device name
lsblk                    # or diskutil list (Mac)

# 3. Unmount it (if mounted)
sudo umount /dev/sdX*    # Replace X with your device

# 4. Write ISO to USB
sudo dd if=ubuntu-22.04-live-server-amd64.iso of=/dev/sdX bs=4M status=progress
sudo sync

# On Windows:
# Use Rufus: https://rufus.ie/
# Or Balena Etcher: https://www.balena.io/etcher/
```

### **Step 3: Boot from USB**

```
1. Insert USB into target computer
2. Power on and press: F12, Del, Esc, or F2 (depends on BIOS)
3. Select USB Boot Device
4. Wait for Ubuntu boot menu
```

### **Step 4: Start Installation with Preseed**

**At the boot menu, press `Tab` or `E` to edit boot parameters:**

```
linux /casper/vmlinuz ... preseed/url=http://192.168.1.100:8000/preseed.cfg
```

Replace `192.168.1.100` with your server's IP address.

**Press Enter to start automatic installation**

### **Step 5: Wait for Completion**

- Installation will run automatically
- System will reboot when finished
- Remove USB when prompted
- System will boot into Linux

### **Step 6: Verify Installation**

```bash
# Login as ubuntu user
ubuntu login: ubuntu
Password: ubuntu123

# Check system
uname -a
lsb_release -a
df -h
free -h
systemctl status
```

---

## 🔴 CentOS/RHEL 8+ Installation

### **Prerequisites**
- CentOS Stream 9 or RHEL 8+ ISO
- USB stick (8GB+)
- Network access
- HTTP server

### **Step 1: Prepare HTTP Server**

```bash
# Copy kickstart file
mkdir -p ~/kickstart-server
cp anaconda-ks.cfg ~/kickstart-server/

# Start server
cd ~/kickstart-server
python3 -m http.server 8000

# Get IP
hostname -I
```

### **Step 2: Create Bootable USB**

```bash
# Same as Ubuntu method
sudo dd if=CentOS-Stream-9-latest-x86_64-dvd1.iso of=/dev/sdX bs=4M
```

### **Step 3: Boot and Start Installation**

**At CentOS boot screen, press `Tab` to edit:**

```
ks=http://192.168.1.100:8000/anaconda-ks.cfg
```

Press Enter to start.

### **Step 4: Monitor Installation**

- CentOS will download and install automatically
- Watch the progress in the terminal
- Installation typically takes 10-30 minutes

### **Step 5: Post-Installation**

```bash
# Login as centos user
CentOS login: centos
Password: centos123

# Test sudo
sudo whoami           # Should output: root

# Check system
hostnamectl
df -h
free -h
dnf list installed | head -20
```

---

## 🔧 Troubleshooting

### **Ubuntu Installation Hangs**

**Problem:** Installation stops or doesn't proceed  
**Solution:**
```bash
# Try with additional parameters:
# At boot prompt:
linux /casper/vmlinuz preseed/url=http://192.168.1.100:8000/preseed.cfg quiet
```

### **Network Not Detected**

**Problem:** "No network devices found"  
**Solution:**
```bash
# Check cable connection
# Try manual network configuration:
# Press Ctrl+C during auto-detection
# Select "Configure the network manually"
# Enter IP, netmask, gateway details
```

### **HTTP Server Cannot Connect**

**Problem:** Preseed URL not accessible  
**Solution:**
```bash
# Verify server is running
ps aux | grep http.server

# Check firewall
sudo ufw allow 8000

# Test from another computer
curl http://192.168.1.100:8000/preseed.cfg

# Verify file exists
ls -la preseed.cfg
```

### **Disk Not Found (CentOS)**

**Problem:** "No disks found"  
**Solution:**
- Check SATA/NVMe controller in BIOS
- Try different SATA port
- Update BIOS to latest version
- Boot in Legacy mode instead of UEFI

### **Partition Fails**

**Problem:** "Cannot partition disk"  
**Solution:**
```bash
# Modify preseed/kickstart:
# Remove LVM setup if problematic
# Use standard partitioning instead

# In preseed.cfg change:
d-i partman-auto/method string regular  # instead of lvm

# In anaconda-ks.cfg:
autopart --type=plain  # instead of lvm
```

---

## 🚀 Advanced Configuration

### **Custom Network Configuration**

**Ubuntu (Preseed):**
```
d-i netcfg/choose_interface select eth0
d-i netcfg/disable_autoconfig boolean true
d-i netcfg/get_ipaddress string 192.168.1.50
d-i netcfg/get_netmask string 255.255.255.0
d-i netcfg/get_gateway string 192.168.1.1
d-i netcfg/get_nameservers string 8.8.8.8 8.8.4.4
```

**CentOS (Kickstart):**
```
network --device=eth0 --bootproto=static --ip=192.168.1.50 \
        --netmask=255.255.255.0 --gateway=192.168.1.1 \
        --nameserver=8.8.8.8,8.8.4.4
```

### **Add Custom Packages**

**Ubuntu:**
```
d-i pkgsel/include string openssh-server curl wget net-tools \
  htop vim nano git build-essential linux-headers-generic \
  python3 python3-pip docker.io nginx
```

**CentOS:**
```
%packages --nodefaults
@core
@standard
openssh-server
openssh-clients
python3
python3-pip
docker
nginx
%end
```

### **Add Post-Installation Scripts**

**Ubuntu:**
```
d-i preseed/late_command string \
  in-target bash -c 'echo "127.0.0.1 hostname" >> /etc/hosts'; \
  in-target systemctl enable ssh
```

**CentOS:**
```
%post
# Custom commands here
echo "127.0.0.1 myhost" >> /etc/hosts
systemctl enable sshd
%end
```

---

## 📊 Installation Times

| Component | Time |
|-----------|------|
| USB Boot | 1-2 min |
| Network Detection | 1-2 min |
| Download Packages | 5-10 min |
| Installation | 5-15 min |
| First Boot | 2-3 min |
| **Total** | **15-30 min** |

---

## ✅ Verification Checklist

After installation, verify:

- [ ] System boots successfully
- [ ] Network interface up (`ip a` or `ifconfig`)
- [ ] SSH server running (`systemctl status ssh/sshd`)
- [ ] Sudo works (`sudo whoami`)
- [ ] Disk usage reasonable (`df -h`)
- [ ] No GUI components installed
- [ ] Minimal package set (`dpkg -l` or `dnf list`)

---

## 📞 Getting Help

If you encounter issues:

1. Check boot logs: `dmesg`
2. Review installation logs:
   - Ubuntu: `/var/log/installer/`
   - CentOS: `/var/log/anaconda/`
3. Open GitHub issue with:
   - Error message
   - Your customizations
   - Hardware specs

---

**Version:** 1.0  
**Last Updated:** 2026-06-16
