# Wazuh SIEM Configuration

**Project:** Optimizing Wazuh SIEM for Threat Detection  
**Author:** admu-sec  
**Date:** April 2026  
**Version:** 1.2

---

## 📋 Overview

This repository contains custom Wazuh SIEM rules and Active Response configurations designed to detect and respond to modern cyber threats in a Windows/Linux hybrid environment.

**Total Implementation:**

- **36 custom security rules** (4 SSH + 23 Windows + 4 Linux + 5 other)
- **33 unique MITRE ATT&CK techniques covered**
- **3 Active Response automations**
- **6 whitelisted critical systems**
- **File Integrity Monitoring (FIM) enabled**

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
wazuh-degree-project/
├── README.md                    # Project overview and documentation
├── commands/
│   └── README.md                # Command reference & testing guide
├── rules/
│   └── local_rules.xml          # Custom detection rules (36 rules)
├── config/
│   ├── ossec.conf               # Active Response + FIM configuration
│   └── local_decoder.xml        # Custom decoders
└── sysmon/
    ├── sysmonconfig-DC.xml      # Sysmon config for AD/DC
    └── sysmonconfig-W1.xml      # Sysmon config for W1 (incl. LSASS entry)
```

---

# 🛡️ Implemented Rules

## SSH Security (4 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100001 | SSH Authentication Failure | 10 | T1110.001, T1078 | ✅ |
| 100002 | SSH Brute Force (5+ attempts) | 10 | T1110.001 | ✅ |
| 100003 | SSH Password Spray Attack | 12 | T1110.003 | ✅ |
| 100004 | Successful Login After Brute Force | 14 | T1078 | ✅ |

---

## Windows Security (23 rules)

### Noise Reduction (2 rules)

| Rule ID | Description | Level | Verified |
|---------|-------------|-------|----------|
| 100005 | Ignore Disk Cleanup | 0 | ✅ |
| 100006 | Ignore Windows Update | 0 | ✅ |

### DC Security (3 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100010 | Administrator Login on DC | 12 | T1078 | ✅ |
| 100055 | Domain Account Network Logon | 8 | T1078.002 | ✅ |
| 100056 | Kerberoasting – RC4 Encrypted Ticket | 15 | T1558.003 | ❌ |

### PowerShell & Credential Dumping (3 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100011 | PowerShell Encoded Command | 12 | T1059.001, T1027 | ✅ |
| 100012 | Mimikatz Detection | 15 | T1003.001 | ✅ |
| 100025 | LSASS Memory Access | 15 | T1003.001 | ✅ |

### RDP Security (1 rule)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100013 | RDP Brute Force | 10 | T1110 | ✅ |

### Persistence Detection (6 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100014 | New Windows Service | 10 | T1543.003 | ✅ |
| 100015 | OneDrivePatcher.exe | 12 | T1036 | ✅ |
| 100016 | Registry Run Key Modified | 11 | T1547.001 | ✅ |
| 100017 | Registry Whitelist (Legitimate) | 0 | N/A | ✅ |
| 100018 | Registry Suspicious Path | 14 | T1547.001 | ✅ |
| 100019 | PowerShell in Registry | 15 | T1547.001, T1059.001 | ✅ |

### Critical Security Events (4 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100020 | Security Logs Cleared | 15 | T1070.001 | ✅ |
| 100021 | Windows Firewall Disabled | 12 | T1562.004 | ✅ |
| 100022 | Ransomware Shadow Copy Deletion | 14 | T1490 | ✅ |
| 100054 | New User Account Created | 12 | T1136 | ✅ |

### Lateral Movement (3 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100023 | PsExec Detection | 11 | T1569.002 | ✅ |
| 100053 | Lateral Tool Transfer (xcopy/robocopy) | 13 | T1570 | ✅ |
| ~~100024~~ | ~~SMB Admin Share Access~~ | - | T1021.002 | Deprecated – covered by built-in rule 67017 |

### Defense Evasion (1 rule)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100026 | Windows Defender Disabled | 13 | T1562.001 | ✅ |

### Ingress Tool Transfer (1 rule)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100052 | File Download Tool Executed | 13 | T1105 | ✅ |

---

## Linux Security (4 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100040 | Critical System File Modified | 12 | T1222, T1098 | ✅ |
| 100041 | SSH Authorized Keys Modified | 10 | T1098.004 | ✅ |
| 100042 | Cron Job Created/Modified | 10 | T1053.003 | ✅ |
| 100043 | Suspicious Sudo Usage | 8 | T1548.003 | ✅ |

**What they detect:**
- **100040**: Changes to `/etc/passwd`, `/etc/shadow`, `/etc/sudoers` (privilege escalation, backdoor users)
- **100041**: SSH backdoors via `authorized_keys` modifications (persistence mechanism)
- **100042**: Malware persistence via cron jobs in `/etc/cron*` and `/var/spool/cron`
- **100043**: Sudo commands from non-privileged users (compromised accounts)

---

## Adversary Emulation – Caldera (8 rules)

| Rule ID | Description | Level | MITRE ATT&CK | Verified |
|---------|-------------|-------|--------------|----------|
| 100044 | Signed Binary Proxy Execution (Msiexec) | 11 | T1218.007 | ✅ |
| 100045 | Scheduled Task Created | 11 | T1053.005 | ✅ |
| 100046 | LSASS Access / Mimikatz | 15 | T1003.001 | ✅ |
| 100047 | Process Injection | 12 | T1055.001 | ✅ |
| 100048 | WMI Event Subscription | 13 | T1546.003 | ✅ |
| 100049 | Screen Capture | 10 | T1113 | ✅ |
| 100050 | Process Discovery | 8 | T1057 | ✅ |
| 100051 | Data Compressed/Archived | 11 | T1560 | ✅ |

---

## 📂 File Integrity Monitoring (FIM)

### Monitored Directories

| Directory | Monitoring Type | Purpose |
|-----------|-----------------|---------|
| `/etc` | Periodic | System configuration files |
| `/usr/bin`, `/usr/sbin` | Periodic | System binaries |
| `/bin`, `/sbin`, `/boot` | Periodic | Critical boot files |
| `/root/.ssh` | Realtime | SSH backdoor detection |
| `/var/spool/cron` | Realtime | Cron persistence detection |
| `/var/ossec/etc` | Realtime | Wazuh config tampering |

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

### Tactics & Techniques (33 unique techniques)

| Tactic | Technique | Rules |
|--------|-----------|-------|
| **Initial Access** | T1078 (Valid Accounts) | 100001, 100004, 100010 |
| **Execution** | T1059.001 (PowerShell) | 100011, 100019 |
| | T1569.002 (PsExec) | 100023 |
| **Persistence** | T1543.003 (Windows Service) | 100014 |
| | T1547.001 (Registry Run Keys) | 100016, 100018, 100019 |
| | T1098.004 (SSH Authorized Keys) | 100041 |
| | T1053.003 (Scheduled Task: Cron) | 100042 |
| | T1053.005 (Scheduled Task: Windows) | 100045 |
| | T1546.003 (WMI Event Subscription) | 100048 |
| | T1136 (Create Account) | 100054 |
| **Privilege Escalation** | T1548.003 (Sudo) | 100043 |
| | T1098 (Account Manipulation) | 100040 |
| | T1055.001 (Process Injection) | 100047 |
| **Defense Evasion** | T1027 (Obfuscation) | 100011 |
| | T1036 (Masquerading) | 100015 |
| | T1070.001 (Clear Logs) | 100020 |
| | T1218.007 (Msiexec) | 100044 |
| | T1562.001 (Disable AV) | 100026 |
| | T1562.004 (Disable Firewall) | 100021 |
| | T1222 (File Permissions Modification) | 100040 |
| **Credential Access** | T1003.001 (LSASS/Mimikatz) | 100012, 100025, 100046 |
| | T1110.001 (Password Guessing) | 100001, 100002 |
| | T1110.003 (Password Spraying) | 100003 |
| | T1110 (Brute Force) | 100013 |
| | T1558.003 (Kerberoasting) | 100056 |
| **Discovery** | T1057 (Process Discovery) | 100050 |
| | T1078.002 (Domain Accounts) | 100055 |
| **Lateral Movement** | T1021.002 (SMB Shares) | Built-in rule 67017 |
| | T1570 (Lateral Tool Transfer) | 100053 |
| **Collection** | T1113 (Screen Capture) | 100049 |
| | T1560 (Archive Collected Data) | 100051 |
| **Command & Control** | T1105 (Ingress Tool Transfer) | 100052 |
| **Impact** | T1490 (Inhibit Recovery) | 100022 |

---

## 📦 Installation

### 1. Deploy Custom Rules

```bash
sudo cp rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml
sudo chown root:wazuh /var/ossec/etc/rules/local_rules.xml
sudo chmod 640 /var/ossec/etc/rules/local_rules.xml
```

### 2. Deploy Custom Decoder

```bash
sudo cp config/local_decoder.xml /var/ossec/etc/decoders/local_decoder.xml
sudo chown root:wazuh /var/ossec/etc/decoders/local_decoder.xml
sudo chmod 640 /var/ossec/etc/decoders/local_decoder.xml
```

### 3. Configure Active Response & FIM

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup
sudo cp config/ossec.conf /var/ossec/etc/ossec.conf
```

### 4. Validate & Restart

```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
sudo tail -100 /var/ossec/logs/ossec.log | grep -i "monitoring directory"
```

### 5. Deploy Sysmon on Windows Agents

```powershell
# Use sysmonconfig-W1.xml for W1, sysmonconfig-DC.xml for DC
sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml
```

---

## 🧪 Testing

### Test SSH Brute Force Detection

```bash
for i in {1..6}; do ssh root@WAZUH_MANAGER_IP; sleep 2; done
```

**Expected Result:** Rule 100002 triggers, IP blocked for 10 minutes.

---

### Test Mimikatz Detection

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
.\mimikatz.exe "sekurlsa::logonpasswords" "exit"
```

**Expected Result:** Rule 100012 triggers (Level 15), IP blocked for 1 hour.

---

### Test Linux FIM – SSH Backdoor

```bash
sudo sh -c 'echo "# test" >> /root/.ssh/authorized_keys'
sleep 10
sudo grep '"id":"100041"' /var/ossec/logs/alerts/alerts.json | tail -1
```

**Expected Result:** Rule 100041 triggers (Level 10), MITRE T1098.004.

---

### Test PsExec Detection

```cmd
C:\PsExec.exe /accepteula cmd.exe /c hostname
```

**Expected Result:** Rule 100023 triggers (Level 11), MITRE T1569.002.

---

## 📊 Dashboard Verification

1. Navigate to **Security events** → **Events**
2. Filter: `rule.id:(100002 OR 100012 OR 100041 OR 100023)`
3. View MITRE ATT&CK mapping: **Threats** → **MITRE ATT&CK**
4. Check FIM events: **File Integrity** → **Files changed**

---

## ⚠️ Important Notes

### Whitelist Considerations
- All infrastructure IPs are whitelisted to maintain visibility
- Active Response will NOT block whitelisted agents
- External attackers are still blocked automatically

### False Positive Management
- Rules 100005–100006 filter common Windows processes
- Rule 100017 whitelists legitimate software in Registry
- FIM ignores: `/etc/mtab`, `/etc/resolv.conf`, `/etc/adjtime`

---

## 📝 Requirements

- **Wazuh Manager:** 4.14.1
- **Wazuh Agents:** Windows 10/11, Windows Server 2019/2022/2025
- **Sysmon:** v15.15 on Windows agents
- **Network:** Agents must communicate with Manager on port 1514/TCP
- **Linux:** Amazon Linux 2023 (Wazuh Manager)

---

## 🔧 Completed Enhancements

- [x] File Integrity Monitoring (FIM) for Linux critical files ✅
- [x] Linux-specific rules (authorized_keys, cron, sudo, /etc modifications) ✅
- [x] Caldera adversary emulation rules (100044–100051) ✅
- [x] PsExec detection (100023) ✅
- [x] Fixed if_sid references for all rules ✅
- [ ] Vulnerability Detection (CVE scanning)
- [ ] Security Configuration Assessment (CIS benchmarks)
- [ ] Email/Slack alert integration
- [ ] FIM for Windows directories

---

## 🚀 Real-World Attack Scenarios Detected

### Scenario 1: SSH Backdoor
```bash
echo "ssh-rsa AAAA... attacker@evil.com" >> /root/.ssh/authorized_keys
```
**Wazuh:** ✅ Rule 100041 → Alert (Level 10) → MITRE T1098.004

### Scenario 2: Privilege Escalation via sudoers
```bash
echo "attacker ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```
**Wazuh:** ✅ Rule 100040 → Alert (Level 12) → MITRE T1222, T1098

### Scenario 3: Malware Persistence via Cron
```bash
echo "0 * * * * /tmp/.hidden/backdoor.sh" >> /var/spool/cron/root
```
**Wazuh:** ✅ Rule 100042 → Alert (Level 10) → MITRE T1053.003

### Scenario 4: Mimikatz Credential Theft
```powershell
mimikatz.exe sekurlsa::logonpasswords
```
**Wazuh:** ✅ Rule 100012 → Alert (Level 15) + Active Response → MITRE T1003.001

### Scenario 5: PsExec Lateral Movement
```cmd
C:\PsExec.exe /accepteula \\target cmd.exe /c hostname
```
**Wazuh:** ✅ Rule 100023 → Alert (Level 11) → MITRE T1569.002

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
Date: April 2026  
Version: 1.2 (Rule fixes, verification & PsExec detection)

---

## 📋 Changelog

### Version 1.2 (2026-04-24)
- ✅ Fixed `if_sid` references for rules 100041, 100042, 100043, 100026, 100012, 100023
- ✅ Fixed rule 100003 (Password Spray) – replaced `same_user` with `same_source_ip`
- ✅ Added PsExec detection rule 100023 (T1569.002) with correct `if_sid` 61603
- ✅ Rule 100024 (SMB Admin Shares) deprecated – covered by built-in rule 67017 (T1021.002)
- ✅ Added rules 100052–100056 (Ingress Tool Transfer, Lateral Tool Transfer, Create Account, Domain Logon, Kerberoasting)
- ✅ Verified all active rules in production against live events
- ✅ Total rules: 27 → 36

### Version 1.1 (2026-02-27)
- ✅ Added File Integrity Monitoring (FIM) for Linux
- ✅ Added 4 Linux security rules (100040–100043)
- ✅ Increased MITRE ATT&CK coverage to 16 techniques (+4)
- ✅ Verified rule 100041 (SSH backdoor detection) in production
- ✅ Total rules: 23 → 27

### Version 1.0 (2026-02-26)
- Initial implementation with 23 rules (SSH + Windows)
- Active Response for SSH, Mimikatz, Ransomware
- Sysmon integration
- MITRE ATT&CK mapping (12 techniques)
