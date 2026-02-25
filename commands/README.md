# commands

A collection of useful Wazuh commands.

## Contents

- [Manager](#manager)
- [Agent](#agent)
- [Rules & Decoders](#rules--decoders)
- [Logs](#logs)
- [ossec.conf](#ossecconf)
- [local_rules.xml](#local_rulesxml)

## Manager

| Description | Command |
|-------------|---------|
| Start Wazuh Manager | `sudo systemctl start wazuh-manager` |
| Stop Wazuh Manager | `sudo systemctl stop wazuh-manager` |
| Restart Wazuh Manager | `sudo systemctl restart wazuh-manager` |
| Check Manager status | `sudo systemctl status wazuh-manager` |
| Check Manager version | `sudo /var/ossec/bin/wazuh-control info` |

## Agent

| Description | Command |
|-------------|---------|
| Start Wazuh Agent | `sudo systemctl start wazuh-agent` |
| Stop Wazuh Agent | `sudo systemctl stop wazuh-agent` |
| Restart Wazuh Agent | `sudo systemctl restart wazuh-agent` |
| Check Agent status | `sudo systemctl status wazuh-agent` |
| List all agents | `sudo /var/ossec/bin/manage_agents -l` |
| Check agent connection | `sudo /var/ossec/bin/agent_control -l` |

## Rules & Decoders

| Description | Command |
|-------------|---------|
| Test rules and decoders | `sudo /var/ossec/bin/wazuh-logtest` |
| Reload rules | `sudo /var/ossec/bin/wazuh-control reload` |
| List loaded rules | `sudo /var/ossec/bin/ruleset_test` |

## Logs

| Description | Command |
|-------------|---------|
| View Manager logs | `sudo tail -f /var/ossec/logs/ossec.log` |
| View Agent logs | `sudo tail -f /var/ossec/logs/ossec.log` |
| View alerts | `sudo tail -f /var/ossec/logs/alerts/alerts.log` |
| Search for specific alert | `sudo grep "sshd" /var/ossec/logs/alerts/alerts.log` |
| View JSON alerts | `sudo tail -f /var/ossec/logs/alerts/alerts.json` |

## ossec.conf

| Description | Command |
|-------------|---------|
| Edit ossec.conf | `sudo nano /var/ossec/etc/ossec.conf` |
| Validate ossec.conf | `sudo /var/ossec/bin/wazuh-control check` |
| View ossec.conf | `sudo cat /var/ossec/etc/ossec.conf` |
| Backup ossec.conf | `sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak` |

## local_rules.xml

| Description | Command |
|-------------|---------|
| Edit local_rules.xml | `sudo nano /var/ossec/etc/rules/local_rules.xml` |
| Validate local_rules.xml | `sudo /var/ossec/bin/wazuh-logtest` |
| View local_rules.xml | `sudo cat /var/ossec/etc/rules/local_rules.xml` |
| Backup local_rules.xml | `sudo cp /var/ossec/etc/rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml.bak` |
