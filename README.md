# Wazuh XDR/SIEM Home Lab — Automated Malware Detection & Response

A home lab built to simulate a real SOC environment using Wazuh as the central SIEM/XDR platform. The lab covers multi-OS agent deployment, file integrity monitoring, and automated malware removal through a VirusTotal + Active Response integration.

---

## Lab Architecture

| Machine | OS | Role |
|---|---|---|
| Wazuh Server | Ubuntu | SIEM manager + dashboard |
| Agent 1 | Ubuntu | Monitored Linux endpoint |
| Agent 2 | Kali Linux | Monitored attacker/test endpoint |
| Agent 3 | Windows | Monitored Windows endpoint |

All machines run as virtual machines. The Wazuh server hosts the manager, indexer, and dashboard components.

---

## What This Lab Covers

- Deploying and configuring a Wazuh server from scratch on Ubuntu
- Enrolling agents across three different operating systems (Ubuntu, Kali Linux, Windows)
- Configuring File Integrity Monitoring (FIM) to watch specific directories for changes
- Integrating the VirusTotal API to automatically scan file hashes against its threat database
- Setting up Active Response to automatically delete files flagged as malicious
- Writing custom Wazuh rules to surface active response results in the dashboard

---

## How the Malware Detection Pipeline Works

New file dropped in monitored directory
↓
FIM detects the change and captures the file hash
↓
Wazuh sends the hash to VirusTotal API
↓
VirusTotal returns a verdict (Rule ID 87105 fires if malicious)
↓
Active Response triggers remove-threat script on the agent
↓
File is automatically deleted + alert appears in Wazuh dashboard

## Key Configuration

**1. FIM — monitoring a directory on the agent**

In the agent's `ossec.conf`, configure which directories to watch:
```xml
<syscheck>
  <directories realtime="yes" checkall="yes">/root/snap/malware</directories>
</syscheck>
```

**2. VirusTotal integration — on the Wazuh manager**

Add the integration block to the manager's `ossec.conf` with your VirusTotal API key:
```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_VIRUSTOTAL_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>
```

**3. Active Response — auto-delete malicious files**

Configure the manager to run the removal script whenever Rule 87105 fires:
```xml
<command>
  <name>remove-threat</name>
  <executable>remove-threat.sh</executable>
  <timeout_allowed>no</timeout_allowed>
</command>

<active-response>
  <disabled>no</disabled>
  <command>remove-threat</command>
  <location>local</location>
  <rules_id>87105</rules_id>
</active-response>
```

**4. Custom rules — tracking active response results**

Added to `/var/ossec/rules/local_rules.xml` on the manager:
```xml
<group name="virustotal,">
  <rule id="100092" level="12">
    <if_sid>657</if_sid>
    <match>Successfully removed threat</match>
    <description>Active response: Malicious file successfully removed.</description>
  </rule>

  <rule id="100093" level="7">
    <if_sid>657</if_sid>
    <match>Error removing threat</match>
    <description>Active response: Failed to remove malicious file.</description>
  </rule>
</group>
```

---

## Screenshots

<img width="1913" height="1072" alt="Screenshot 2026-05-17 184404" src="https://github.com/user-attachments/assets/b485fdaa-b49b-4fdc-a6c5-9dba216f5706" />

<img width="1911" height="1063" alt="image" src="https://github.com/user-attachments/assets/fa0852aa-095b-47a9-8e7c-f757aad9987f" />



---

## Tools & Technologies

- **Wazuh** — SIEM/XDR platform (manager, indexer, dashboard)
- **VirusTotal API** — cloud-based threat intelligence
- **File Integrity Monitoring (FIM)** — built-in Wazuh capability
- **Active Response** — automated script execution on agents
- **VirtualBox / VMware** — virtualisation
- **Ubuntu, Kali Linux, Windows** — agent operating systems

---

## What I Learned

- How a SIEM ingests and correlates events from multiple endpoints across different OSes
- The difference between detection (FIM + VirusTotal) and response (Active Response scripts)
- How to write and tune custom Wazuh rules to reduce noise and surface meaningful alerts
- How automated active response reduces the time between detection and remediation — a core SOC concept

---

## References

- [Wazuh Official Documentation — VirusTotal Integration](https://documentation.wazuh.com/current/user-manual/capabilities/malware-detection/virus-total-integration.html)
- [Wazuh PoC — Detect and Remove Malware](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-remove-malware-virustotal.html)

