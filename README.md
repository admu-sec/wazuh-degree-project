# Wazuh SIEM Configuration
**Project:** Optimizing Wazuh SIEM for Threat Detection  
**Author:** admu-sec  
**Date:** 2026-02-23/24  
**Version:** 1.0  

---

## 📋 Overview

This repository contains custom Wazuh SIEM rules and Active Response configurations designed to detect and respond to modern cyber threats in a Windows/Linux hybrid environment.

**Total Implementation:**
- **23 custom security rules**
- **12 MITRE ATT&CK techniques covered**
- **3 Active Response automations**
- **6 whitelisted critical systems**

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
```

---

## 📁 Repository Structure
```
wazuh-siem-config/
├── README.md                    # This file
├── ossec.conf/
│   └── ossec.conf              # Active Response configuration
└── local_rules.xml/
    └── local_rules.xml         # Custom detection rules (23 rules)
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

---

## 🎯 MITRE ATT&CK Coverage

### Tactics & Techniques (12 unique)

| Tactic | Technique | Rules |
|--------|-----------|-------|
| **Initial Access** | T1078 (Valid Accounts) | 100001, 100004, 100010 |
| **Execution** | T1059.001 (PowerShell) | 100011, 100019 |
| **Persistence** | T1543.003 (Windows Service) | 100014 |
| | T1547.001 (Registry Run Keys) | 100016, 100018, 100019 |
| **Defense Evasion** | T1027 (Obfuscation) | 100011 |
| | T1070.001 (Clear Logs) | 100020 |
| | T1562.001 (Disable AV) | 100026 |
| | T1562.004 (Disable Firewall) | 100021 |
| **Credential Access** | T1003.001 (LSASS/Mimikatz) | 100012, 100025 |
| | T1110 (Brute Force) | 100002, 100003, 100013 |
| **Lateral Movement** | T1021.002 (SMB Shares) | 100024 |
| | T1569.002 (PsExec) | 100023 |
| **Impact** | T1490 (Inhibit Recovery) | 100022 |

---

## 📦 Installation

### 1. Deploy Custom Rules
```bash
# On Wazuh Manager
sudo cp local_rules.xml /var/ossec/etc/rules/local_rules.xml
sudo chown root:wazuh /var/ossec/etc/rules/local_rules.xml
sudo chmod 640 /var/ossec/etc/rules/local_rules.xml
```

### 2. Configure Active Response
```bash
# Backup original configuration
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup

# Add Active Response section from ossec.conf to your configuration
sudo nano /var/ossec/etc/ossec.conf
# Paste the Active Response configuration
```

### 3. Validate & Restart
```bash
# Validate configuration
sudo /var/ossec/bin/wazuh-logtest -t

# Restart Wazuh Manager
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
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

## 📊 Dashboard Verification

1. Navigate to **Security events** → **Events**
2. Filter: `rule.id:(100001 OR 100002 OR 100012 OR 100022)`
3. View MITRE ATT&CK mapping: **Threats** → **MITRE ATT&CK**

---

## ⚠️ Important Notes

### Whitelist Considerations
- All infrastructure IPs are whitelisted to maintain visibility
- Active Response will NOT block whitelisted agents
- External attackers are still blocked automatically

### False Positive Management
- Rules 100005-100006 filter common Windows processes
- Rule 100017 whitelists legitimate software in Registry

### Defense in Depth
Even with whitelisted agents, multiple security layers remain active:
- ✅ Alert generation and logging
- ✅ Sysmon process/network monitoring
- ✅ MITRE ATT&CK mapping
- ✅ Historical forensics data
- ✅ Dashboard visualization

---

## 📝 Requirements

- **Wazuh Manager:** 4.x
- **Wazuh Agents:** Windows 10/11, Windows Server 2019/2022
- **Sysmon:** Installed on Windows agents (for rules 100011-100019, 100023, 100025)
- **Network:** Agents must communicate with Manager on port 1514/TCP

---

## 🔧 Future Enhancements

- [ ] File Integrity Monitoring (FIM) for Windows
- [ ] Vulnerability Detection (CVE scanning)
- [ ] Security Configuration Assessment (CIS benchmarks)
- [ ] Email/Slack alert integration
- [ ] Linux-specific rules (authorized_keys, cron, sudo)

---

## 📚 References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Wazuh Ruleset GitHub](https://github.com/wazuh/wazuh/tree/master/ruleset/rules)
- [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)

---

## 📄 License

This configuration is provided as-is for educational and security research purposes.

---

## 👤 Author

**admu-sec**  
Project: Wazuh SIEM Optimization  
Date: February 2026
