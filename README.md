# 🛡️ Wazuh SIEM & XDR Security Lab

> A hands-on security monitoring lab built from scratch using **Wazuh** — an open-source SIEM and XDR platform — covering agent deployment, threat detection, active response, integrations, and real-time alerting.

---

## 📌 Project Overview

This project demonstrates the full deployment and configuration of a Wazuh-based Security Operations Center (SOC) lab environment built entirely from scratch.

The lab simulates real-world security scenarios including:

- Centralized log collection from **Linux** and **Windows** endpoints
- **File Integrity Monitoring (FIM)** with VirusTotal threat scanning
- **Active Response** for automated IP blocking
- **Custom Python scripts** for threat detection and removal
- Real-time alerting via **Email** and **Telegram**
- Custom **Security Dashboards** for event visualization

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Wazuh Server                      │
│  ┌─────────────┐ ┌──────────────┐ ┌─────────────┐  │
│  │Wazuh Manager│ │Wazuh Indexer │ │   Dashboard │  │
│  └─────────────┘ └──────────────┘ └─────────────┘  │
└──────────────────────┬──────────────────────────────┘
                       │ SSL/TLS
          ┌────────────┴────────────┐
          │                         │
  ┌───────▼──────┐         ┌────────▼─────┐
  │  Linux Agent │         │ Windows Agent│
  │  (Ubuntu)    │         │  (Windows)   │
  └──────────────┘         └──────────────┘
```

**Components:**
- **Wazuh Manager** — Central management and rule engine
- **Wazuh Indexer** — Data storage and search (OpenSearch-based)
- **Wazuh Dashboard** — Web UI for visualization and monitoring
- **Wazuh Agents** — Endpoint collectors (Linux & Windows)

---

## 📂 Project Structure

```
wazuh-siem-lab/
├── installation/
│   ├── server-install.md
│   ├── linux-agent.md
│   └── windows-agent.md
├── configuration/
│   ├── ossec.conf
│   └── active-response.md
├── integrations/
│   ├── virustotal.md
│   ├── email-alerts.md
│   └── telegram.md
├── scripts/
│   └── remove-threat.py
├── dashboards/
│   └── dashboard-setup.md
└── screenshots/
    └── (all lab screenshots)
```

---

## ⚙️ Key Features

| Feature | Description |
|---|---|
| ✅ Multi-Agent Support | Linux & Windows endpoint monitoring |
| ✅ File Integrity Monitoring | Real-time file change detection with FIM |
| ✅ VirusTotal Integration | Automatic malware scanning of suspicious files |
| ✅ Active Response | Auto IP blocking via firewall rules |
| ✅ Threat Removal Script | Custom Python agent to detect and delete threats |
| ✅ Email Alerts | Postfix + Gmail SMTP for alert notifications |
| ✅ Telegram Alerts | Real-time bot notifications via Webhook |
| ✅ Security Dashboard | Custom visualizations with multi-field buckets |

---

## 🚀 Installation & Setup

### 1. Wazuh Server Installation

A single-command automated install that deploys the Manager, Indexer, and Dashboard:

```bash
sudo curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && \
sudo bash ./wazuh-install.sh -a
```

The installer automatically provisions SSL certificates for secure communication between the server and all agents:

- Root Certificate
- Admin Certificate
- Wazuh Indexer Certificate
- Filebeat Certificate
- Dashboard Certificate

> 📁 See [Installation Guide](installation/server-install.md) for full details.

---

### 2. Admin Password Change

After installation, the admin password is changed using the built-in security tool:

```bash
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/
bash wazuh-passwords-tool.sh -u admin -p NEW_PASSWORD
```

---

### 3. Linux Agent Deployment

```bash
# Download the agent package
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.3-1_amd64.deb

# Install with manager configuration
WAZUH_MANAGER='192.168.197.144' \
WAZUH_AGENT_GROUP='Linux' \
WAZUH_AGENT_NAME='Linux_Agent' \
dpkg -i wazuh-agent_4.14.3-1_amd64.deb

# Enable and start the service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

### 4. Windows Agent Deployment

```powershell
# Download and install via PowerShell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.3-1.msi `
  -OutFile $env:tmp\wazuh-agent
msiexec.exe /i $env:tmp\wazuh-agent /q `
  WAZUH_MANAGER=192.168.197.144 `
  WAZUH_AGENT_GROUP=Windows `
  WAZUH_AGENT_NAME=Windows_Agent

# Start the agent service
NET START Wazuh
```

---

## 📊 Dashboard Setup

A custom **Security Events Dashboard** was built using the Wazuh Dashboard Visualize Library with `wazuh-alerts-*` as the index pattern.

**Configured Buckets (Split Rows):**

| Field | Purpose |
|---|---|
| `@timestamp` | Sort events by time (latest first) |
| `rule.level` | Group by threat severity |
| `data.srcip` | Identify attacking IP addresses |
| `data.dstuser` | Show targeted user accounts |
| `rule.description` | Display event description |

> 📁 See [Dashboard Setup Guide](dashboards/dashboard-setup.md)

---

## 🔥 Active Response — Automatic IP Blocking

Wazuh's Active Response module automatically blocks attacking IPs using firewall rules.

**How it works:**
1. Wazuh detects a suspicious event (e.g., brute-force attack)
2. The attacker's IP is identified
3. A firewall drop rule is applied automatically
4. Alert is generated: `Host Blocked by firewall-drop Active Response`

> 📁 See [Active Response Configuration](configuration/active-response.md)

---

## 🦠 VirusTotal Integration

Suspicious files detected by File Integrity Monitoring (FIM) are automatically submitted to VirusTotal for threat analysis.

**ossec.conf configuration:**

```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>
```

**FIM Monitoring on Windows Agent:**

```xml
<agent_config>
  <syscheck>
    <directories check_all="yes" realtime="yes">C:\Users\kenan\Downloads</directories>
  </syscheck>
</agent_config>
```

Events tracked: file **creation**, **deletion**, and **modification**.

> 📁 See [VirusTotal Integration Guide](integrations/virustotal.md)

---

## 🐍 Python Threat Removal Script

A custom **Active Response script** was developed for the Windows agent to automatically detect and remove malicious files.

**Script location:**
```
C:\Program Files (x86)\ossec-agent\active-response\bin\remove-threat.py
```

**Compiled executable:**
```
remove-threat.exe
```

The script is triggered by Wazuh when a VirusTotal alert fires, automatically deleting the flagged file from the endpoint.

> 📁 See [Scripts](scripts/remove-threat.py)

---

## 📧 Email Alert Integration

Security alerts are delivered via email using **Postfix** with Gmail SMTP.

**Setup summary:**
- Postfix configured at `/etc/postfix/main.cf`
- Gmail App Password used for SMTP authentication
- `email_maxperhour` parameter set to control notification rate

**Test command:**
```bash
echo "Test mail from postfix" | mail -s "Test Postfix" your@gmail.com
```

Alert emails include: alert level, event description, and rule details.

> 📁 See [Email Integration Guide](integrations/email-alerts.md)

---

## 📲 Telegram Alert Integration

Real-time security alerts are pushed to a Telegram bot using Wazuh's webhook notification system.

**Setup steps:**
1. Created a Telegram Bot via BotFather and obtained the bot token
2. Retrieved `chat_id` via the Telegram API:
   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```
3. Configured a **Custom Webhook** in Wazuh Notification Channels:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```
4. Created a **Per Query Monitor** on `wazuh-alerts-*` index with specific rule ID filters
5. Configured alert message format to include: Rule ID, Event Description, Agent Name

**Telegram alerts include:**
- 🔴 Alert Rule ID
- 📋 Event Description
- 🖥️ Agent Name
- 🔐 Security Context Details

> 📁 See [Telegram Integration Guide](integrations/telegram.md)

---

## 🧪 Testing & Validation

The lab was validated through the following tests:

- **Brute-Force Simulation** → Verified Active Response triggered IP block
- **Malware File Drop** → Confirmed VirusTotal scan + Python script removal
- **FIM Events** → Validated file create/edit/delete alerts on dashboard
- **Email Delivery** → Test message successfully received in Gmail
- **Telegram Alerts** → Real-time bot notifications confirmed on mobile

---

## 🎯 What I Learned

- Wazuh architecture and component interaction (Manager, Indexer, Dashboard, Agent)
- Agent deployment and group configuration across Linux and Windows
- File Integrity Monitoring (FIM) and real-time change detection
- Active Response rule design and automatic firewall blocking
- Third-party API integration (VirusTotal, Telegram)
- SMTP configuration with Postfix for alert delivery
- Custom Python scripting for endpoint threat removal
- Security dashboard creation with OpenSearch/Kibana-style visualizations
- Real-world SOC workflows and incident response fundamentals

---

## 🔮 Future Improvements

- [ ] Add MITRE ATT&CK framework rule mapping
- [ ] Integrate Shuffle SOAR for automated playbooks
- [ ] Deploy a honeypot endpoint for attacker simulation
- [ ] Add network traffic monitoring with Suricata
- [ ] Expand dashboards with GeoIP-based attack maps
- [ ] Implement vulnerability scanning with Wazuh SCA

---

## 🛠️ Tech Stack

![Wazuh](https://img.shields.io/badge/Wazuh-4.14-blue?style=flat-square&logo=wazuh)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Linux-orange?style=flat-square&logo=ubuntu)
![Windows](https://img.shields.io/badge/Windows-Agent-0078D6?style=flat-square&logo=windows)
![Python](https://img.shields.io/badge/Python-Script-3776AB?style=flat-square&logo=python)
![VirusTotal](https://img.shields.io/badge/VirusTotal-Integration-394EFF?style=flat-square)
![Telegram](https://img.shields.io/badge/Telegram-Bot-2CA5E0?style=flat-square&logo=telegram)
![Gmail](https://img.shields.io/badge/Gmail-SMTP-EA4335?style=flat-square&logo=gmail)

---

## 👤 Author

**Kenan Zeynalli**

> Also check out my [Splunk SOC Lab](https://github.com/KenanZeynalli/splunk-soc-lab) — a similar project built with Splunk SIEM.
