# Wazuh SIEM Configuration

**Project:** Optimizing Wazuh SIEM for Threat Detection.  
**Author:** admu-sec  
**Date:** 2026-02-27  
**Version:** 1.1

---

## 📋 Overview

This repository contains custom Wazuh SIEM rules and Active Response configurations designed to detect and respond to modern cyber threats in a Windows/Linux hybrid environment.

**Total Implementation:**
- **27 custom security rules** (4 SSH + 19 Windows + 4 Linux)
- **16 MITRE ATT&CK techniques covered**
- **3 Active Response automations**
- **6 whitelisted critical systems**
- **File Integrity Monitoring (FIM)** enabled

---

## 🏗️ Architecture
```
Wazuh Manager (10.46.21.108) + Dashboard
    ↓
Wazuh Indexer (10.46.21.60) - Elasticsearch
    ↑
Monitored Agents:
  - Windows DC (192.168.5.97)
  - Windows W1 (192.168.5.99)
  - Windows W2 (192.168.5.100)
  - Wazuh Manager (self-monitoring)
```

---

## 📁 Repository Structure
```
wazuh-siem-config/
│
├── 📄 README.md                    # Project overview and documentation
│
├── 📂 commands/
│   └── 📄 README.md               # Command reference & testing guide
│
├── 📂 rules/
│   └── 📄 local_rules.xml         # Custom detection rules (27 rules)
│
└── 📂 config/
    └── 📄 ossec.conf              # Active Response + FIM configuration
```

---

## 🛡️ Implemented Rules

### SSH Security (4 rules)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100001 | SSH Authentication Failure | 10 | T1110.001, T1078 |
| 100002 | SSH Brute Force (5+ attempts) | 10 | T1110.001 |
| 100003 | SSH Password Spray Attack | 12 | T1110.003 |
| 100004 | Successful Login After Brute Force | 14 | T1078 |

---

### Windows Security (19 rules)

#### Noise Reduction (2 rules)
| Rule ID | Description | Level |
|---------|-------------|-------|
| 100005 | Ignore Disk Cleanup | 0 |
| 100006 | Ignore Windows Update | 0 |

#### DC Security (1 rule)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100010 | Administrator Login on DC | 12 | T1078 |

#### PowerShell & Credential Dumping (3 rules)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100011 | PowerShell Encoded Command | 12 | T1059.001, T1027 |
| 100012 | Mimikatz Detection | **15** | T1003.001 |
| 100025 | LSASS Memory Access | **15** | T1003.001 |

#### RDP Security (1 rule)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100013 | RDP Brute Force | 10 | T1110 |

#### Persistence Detection (6 rules)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100014 | New Windows Service | 10 | T1543.003 |
| 100015 | OneDrivePatcher.exe | 12 | T1036 |
| 100016 | Registry Run Key Modified | 11 | T1547.001 |
| 100017 | Registry Whitelist (Legitimate) | 0 | N/A |
| 100018 | Registry Suspicious Path | **14** | T1547.001 |
| 100019 | PowerShell in Registry | **15** | T1547.001, T1059.001 |

#### Critical Security Events (3 rules)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100020 | Security Logs Cleared | **15** | T1070.001 |
| 100021 | Windows Firewall Disabled | 12 | T1562.004 |
| 100022 | Ransomware Shadow Copy Deletion | **14** | T1490 |

#### Lateral Movement (2 rules)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100023 | PsExec Detection | 11 | T1569.002 |
| 100024 | SMB Admin Share Access | 8 | T1021.002 |

#### Defense Evasion (1 rule)
| Rule ID | Description | Level | MITRE ATT&CK |
|---------|-------------|-------|--------------|
| 100026 | Windows Defender Disabled | 13 | T1562.001 |

---

### Linux Security (4 rules) **NEW!**

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100040 | Critical System File Modified | **12** | T1222, T1098 | ✅ |
| 100041 | SSH Authorized Keys Modified | 10 | T1098.004 | ✅ **TESTED** |
| 100042 | Cron Job Created/Modified | 10 | T1053.003 | ✅ |
| 100043 | Suspicious Sudo Usage | 8 | T1548.003 | ✅ |

**What they detect:**
- **100040:** Changes to `/etc/passwd`, `/etc/shadow`, `/etc/sudoers` (privilege escalation, backdoor users)
- **100041:** SSH backdoors via `authorized_keys` modifications (persistence mechanism) ✅ **Verified in production**
- **100042:** Malware persistence via cron jobs in `/etc/cron*` and `/var/spool/cron`
- **100043:** Sudo commands from non-privileged users (compromised accounts)

---

## 📂 File Integrity Monitoring (FIM) **NEW!**

### Monitored Directories

| Directory | Monitoring Type | Purpose |
|-----------|-----------------|---------|
| `/etc` | Periodic | System configuration files |
| `/usr/bin`, `/usr/sbin` | Periodic | System binaries |
| `/bin`, `/sbin`, `/boot` | Periodic | Critical boot files |
| **`/root/.ssh`** | **Realtime** | **SSH backdoor detection** |
| **`/var/spool/cron`** | **Realtime** | **Cron persistence detection** |
| **`/var/ossec/etc`** | **Realtime** | **Wazuh config tampering** |

**Features:**
- ✅ **Realtime monitoring** with inotify
- ✅ **Checksums:** MD5, SHA1, SHA256
- ✅ **Permissions & ownership** tracking
- ✅ **Ignore list** for false positives (mtab, resolv.conf)

**Configuration:** See `config/ossec.conf` → `<syscheck>` section

---

## ⚡ Active Response

### Automated IP Blocking

| Trigger Rule | Threat | Timeout | Scope |
|--------------|--------|---------|-------|
| 100002 | SSH Brute Force | 10 minutes | Local |
| 100012 | Mimikatz | 1 hour | All agents |
| 100022 | Ransomware | 1 hour | All agents |

### Whitelisted Systems
- `127.0.0.1` - Localhost
- `10.46.21.108` - Wazuh Manager + Dashboard
- `10.46.21.60` - Wazuh Indexer (Elasticsearch)
- `192.168.5.97` - Windows DC
- `192.168.5.99` - Windows W1
- `192.168.5.100` - Windows W2

**Why whitelist?**
- Maintains visibility even during attacks
- Prevents agent-manager communication disruption
- External threats still blocked automatically

---

## 🎯 MITRE ATT&CK Coverage

### Tactics & Techniques (16 unique)

| Tactic | Technique | Rules |
|--------|-----------|-------|
| **Initial Access** | T1078 (Valid Accounts) | 100001, 100004, 100010 |
| **Execution** | T1059.001 (PowerShell) | 100011, 100019 |
| **Persistence** | T1543.003 (Windows Service) | 100014 |
| | T1547.001 (Registry Run Keys) | 100016, 100018, 100019 |
| | **T1098.004 (SSH Authorized Keys)** | **100041** ✅ |
| | **T1053.003 (Scheduled Task: Cron)** | **100042** ✅ |
| **Defense Evasion** | T1027 (Obfuscation) | 100011 |
| | T1070.001 (Clear Logs) | 100020 |
| | T1562.001 (Disable AV) | 100026 |
| | T1562.004 (Disable Firewall) | 100021 |
| | **T1222 (File Permissions Modification)** | **100040** ✅ |
| **Privilege Escalation** | **T1548.003 (Sudo and Sudo Caching)** | **100043** ✅ |
| | **T1098 (Account Manipulation)** | **100040** ✅ |
| **Credential Access** | T1003.001 (LSASS/Mimikatz) | 100012, 100025 |
| | T1110 (Brute Force) | 100002, 100003, 100013 |
| **Lateral Movement** | T1021.002 (SMB Shares) | 100024 |
| | T1569.002 (PsExec) | 100023 |
| **Impact** | T1490 (Inhibit Recovery) | 100022 |

**New techniques added (4):**
- ✅ T1098.004 - SSH Authorized Keys
- ✅ T1053.003 - Scheduled Task: Cron
- ✅ T1222 - File Permissions Modification
- ✅ T1548.003 - Sudo and Sudo Caching

---

## 📦 Installation

### 1. Deploy Custom Rules
```bash
# On Wazuh Manager
sudo cp rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml
sudo chown root:wazuh /var/ossec/etc/rules/local_rules.xml
sudo chmod 640 /var/ossec/etc/rules/local_rules.xml
```

### 2. Configure Active Response & FIM
```bash
# Backup original configuration
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup

# Merge sections from config/ossec.conf
sudo nano /var/ossec/etc/ossec.conf
# Add: <global> (whitelist), <command>, <active-response>, <syscheck>
```

### 3. Validate & Restart
```bash
# Validate configuration
sudo /var/ossec/bin/wazuh-control info

# Restart Wazuh Manager
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager

# Verify FIM loaded
sudo tail -100 /var/ossec/logs/ossec.log | grep -i "monitoring directory"
```

---

## 🧪 Testing

### Test SSH Brute Force Detection
```bash
# From external machine (not whitelisted)
for i in {1..6}; do
  ssh root@WAZUH_MANAGER_IP
  sleep 2
done
# Enter wrong password each time
```

**Expected Result:**
- Rule 100002 triggers after 5th attempt
- Active Response blocks IP for 10 minutes
- Alert visible in Wazuh Dashboard

---

### Test Mimikatz Detection (Simulation)
```powershell
# On Windows DC (PowerShell as Admin)
powershell.exe -Command "Write-Host 'Testing sekurlsa::logonpasswords detection'"
```

**Expected Result:**
- Rule 100012 triggers (Level 15 - Critical)
- Alert shows in Dashboard
- DC remains whitelisted (no blocking)

---

### Test Linux FIM - SSH Backdoor Detection **NEW!**
```bash
# On Wazuh Manager
sudo mkdir -p /root/.ssh
echo "ssh-rsa TEST_$(date +%s) test@wazuh" | sudo tee -a /root/.ssh/authorized_keys

# Wait 10 seconds
sleep 10

# Verify alert
sudo grep '"id":"100041"' /var/ossec/logs/alerts/alerts.json | tail -1
```

**Expected Result:** ✅ **VERIFIED**
- Rule 100041 triggers (Level 10)
- Alert: "SSH authorized_keys file modified - potential backdoor"
- MITRE T1098.004 mapping
- Realtime detection (<10 seconds)

---

### Test Critical File Modification **NEW!**
```bash
# Modify /etc/passwd
echo "# Wazuh test - $(date)" | sudo tee -a /etc/passwd

# Wait and verify
sleep 10
sudo grep '"id":"100040"' /var/ossec/logs/alerts/alerts.json
```

**Expected Result:**
- Rule 100040 triggers (Level 12 - CRITICAL)
- Alert: "System authentication file modified"

---

## 📊 Dashboard Verification

1. Navigate to **Security events** → **Events**
2. Filter: `rule.id:(100002 OR 100012 OR 100041 OR 100042)`
3. View MITRE ATT&CK mapping: **Threats** → **MITRE ATT&CK**
4. Check FIM events: **File Integrity** → **Files changed**

---

## ⚠️ Important Notes

### Whitelist Considerations
- All infrastructure IPs are whitelisted to maintain visibility
- Active Response will NOT block whitelisted agents
- External attackers are still blocked automatically
- Agents can always communicate with Manager even during attacks

### False Positive Management
- Rules 100005-100006 filter common Windows processes
- Rule 100017 whitelists legitimate software in Registry
- FIM ignores: `/etc/mtab`, `/etc/resolv.conf`, `/etc/adjtime`

### Defense in Depth
Even with whitelisted agents, multiple security layers remain active:
- ✅ Alert generation and logging
- ✅ Sysmon process/network monitoring
- ✅ File Integrity Monitoring (realtime)
- ✅ MITRE ATT&CK mapping
- ✅ Historical forensics data
- ✅ Dashboard visualization

---

## 📝 Requirements

- **Wazuh Manager:** 4.9.x
- **Wazuh Agents:** Windows 10/11, Windows Server 2019/2022/2025
- **Sysmon:** Installed on Windows agents (for rules 100011-100019, 100023, 100025)
- **Network:** Agents must communicate with Manager on port 1514/TCP
- **Linux:** Ubuntu 24.04 or compatible for Wazuh Manager

---

## 🔧 Completed Enhancements

- [x] **File Integrity Monitoring (FIM)** for Linux critical files ✅
- [x] **Linux-specific rules** (authorized_keys, cron, sudo, /etc modifications) ✅
- [ ] Vulnerability Detection (CVE scanning)
- [ ] Security Configuration Assessment (CIS benchmarks)
- [ ] Email/Slack alert integration
- [ ] FIM for Windows directories

---

## 🚀 Real-World Attack Scenarios Detected

### Scenario 1: SSH Backdoor
```bash
# Attacker adds their SSH key
echo "ssh-rsa AAAA... attacker@evil.com" >> /root/.ssh/authorized_keys
```
**Wazuh:** ✅ Rule 100041 → Alert (Level 10) → MITRE T1098.004

---

### Scenario 2: Privilege Escalation
```bash
# Attacker adds themselves to sudoers
echo "attacker ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```
**Wazuh:** ✅ Rule 100040 → Alert (Level 12 - CRITICAL) → MITRE T1222, T1098

---

### Scenario 3: Malware Persistence via Cron
```bash
# Malware creates scheduled backdoor
echo "0 * * * * /tmp/.hidden/backdoor.sh" >> /var/spool/cron/root
```
**Wazuh:** ✅ Rule 100042 → Alert (Level 10) → MITRE T1053.003

---

### Scenario 4: Mimikatz Credential Theft
```powershell
# Attacker runs Mimikatz on Windows DC
mimikatz.exe sekurlsa::logonpasswords
```
**Wazuh:** ✅ Rule 100012 → Alert (Level 15) + Active Response (1 hour block) → MITRE T1003.001

---

## 📚 References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Wazuh Ruleset GitHub](https://github.com/wazuh/wazuh/tree/master/ruleset/rules)
- [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)
- [File Integrity Monitoring Guide](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/)

---

## 📄 License

This configuration is provided as-is for educational and security research purposes.

---

## 👤 Author

**admu-sec**  
Project: Wazuh SIEM Optimization  
Date: February 2026  
Version: 1.1 (Updated with Linux FIM & Security Rules)

---

## 📋 Changelog

### Version 1.1 (2026-02-27)
- ✅ Added File Integrity Monitoring (FIM) for Linux
- ✅ Added 4 Linux security rules (100040-100043)
- ✅ Increased MITRE ATT&CK coverage to 16 techniques (+4)
- ✅ Verified rule 100041 (SSH backdoor detection) in production
- ✅ Total rules: 23 → 27

### Version 1.0 (2026-02-26)
- Initial implementation with 23 rules (SSH + Windows)
- Active Response for SSH, Mimikatz, Ransomware
- Sysmon integration
- MITRE ATT&CK mapping (12 techniques)
