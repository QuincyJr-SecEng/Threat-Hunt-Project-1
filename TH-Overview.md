# 🔵 Threat Hunting & Incident Response — Akira Ransomware (Cyber Range)

A hands-on incident response and threat hunting exercise simulating a real-world Akira ransomware intrusion. Conducted across Microsoft Defender for Endpoint and Microsoft Sentinel using KQL-based Advanced Hunting to track a full attack chain from initial access through to ransomware deployment.

---

## 🎯 What This Project Is

This is a two-part Cyber Range investigation built by **SancLogic** — *The Broker* (initial compromise) and *The Buyer* (ransomware deployment). The attacker returns to an environment using pre-staged access and deploys **Akira ransomware**, exfiltrating data before encryption.

The challenge required working **backwards from impact** — starting with an alert, pivoting across multiple hosts, and correlating infrastructure across both intrusions without guided steps.

> **Difficulty:** Advanced  
> **Platform:** Microsoft Defender for Endpoint + Microsoft Sentinel  
> **Focus:** Threat Hunting, Incident Response, KQL, Windows Forensics

---

## 🧠 Skills Demonstrated

### Threat Hunting with KQL
- Wrote Advanced Hunting queries across `DeviceProcessEvents`, `DeviceFileEvents`, `DeviceNetworkEvents`, `DeviceRegistryEvents`, and `DeviceEvents`
- Used `summarize`, `extend`, `parse_json()`, and `make_set()` to correlate and group telemetry
- Identified beaconing patterns from file write intervals and named pipe access timing
- Detected obfuscated PowerShell via script block logging analysis

### Incident Response
- Reconstructed a full attack timeline across two hosts (AS-PC2, AS-SRV) from raw endpoint telemetry
- Identified initial access, C2 establishment, credential theft, lateral movement, exfiltration, and ransomware deployment
- Documented the complete kill chain with timestamps, process trees, and MITRE ATT&CK mappings

### Windows Endpoint Forensics
- Analysed named pipe access (`\Device\NamedPipe\lsass`, `\Device\NamedPipe\srvsvc`) to identify credential theft and share enumeration
- Detected LSASS memory reads from `powershell.exe` and `MsMpEng.exe`
- Identified process hollowing indicators — lsass pipe access with no recorded initiating process image
- Traced a defence evasion chain through `kill.bat` → `Set-MpPreference` → registry modification → service stop

### Malware & Attacker Tooling Analysis
- Identified attacker-deployed tools disguised as legitimate binaries (`updater.exe`, `wsync.exe`, `scan.exe`, `st.exe`)
- Recognised LOLBINs used for download (`bitsadmin`, `Invoke-WebRequest`) and compression (`makecab.exe`)
- Tracked beacon deployment failures and versioned replacements across hosts
- Extracted C2 infrastructure from network telemetry and process command lines

### OSINT & Infrastructure Analysis
- Attributed attacker infrastructure across payload hosting, C2, and RAT relay domains
- Correlated C2 IPs across two separate intrusions
- Identified AnyDesk relay domain used to proxy remote access sessions

---

## 🗺️ Attack Chain Summary

```
Initial Access          →  AnyDesk (pre-staged, C:\Users\Public)
↓
C2 Establishment        →  wsync.exe beacon dropped via PowerShell IWR
↓
Defence Evasion         →  kill.bat disables Defender, firewall, VSS
↓
Credential Access       →  LSASS named pipe + memory read
↓
Reconnaissance          →  Advanced IP Scanner (scan.exe) via bitsadmin → IWR
↓
Lateral Movement        →  AS.SRV.Administrator → AS-SRV
↓
Exfiltration            →  st.exe compresses data → exfil_data.zip
↓
Ransomware Deployment   →  updater.exe (Akira) encrypts files → .akira extension
↓
Anti-Forensics          →  clean.bat deletes ransomware binary
```

---

## 🔍 Key Investigative Findings

| Finding | Detail |
|---|---|
| **Threat Actor** | Akira ransomware group |
| **Initial Access Vector** | Pre-staged AnyDesk RAT from prior intrusion |
| **Attacker IP** | `88.97.164.155` |
| **C2 Domain** | `sync.cloud-endpoint.net` |
| **Beacon** | `wsync.exe` — custom C2 implant, not a known binary |
| **Credential Theft** | LSASS named pipe access + direct memory reads |
| **Lateral Movement Account** | `AS.SRV.Administrator` |
| **Exfiltration Archive** | `exfil_data.zip` (~1.1 MB) |
| **Ransomware Binary** | `updater.exe` (18 KB, disguised as update process) |
| **Encryption Start** | `22:18:33 UTC` |
| **Affected Hosts** | AS-PC2, AS-SRV |

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| **Microsoft Defender for Endpoint** | Primary telemetry source — process, file, network, registry events |
| **Microsoft Sentinel** | Log aggregation and cross-host correlation |
| **KQL (Kusto Query Language)** | All threat hunting queries |
| **MITRE ATT&CK** | Technique mapping across the full kill chain |

---

## 📂 Repository Contents

```
├── README.md                        ← This file — project overview
├── IR_Report_Akira_AshfordSterling  ← Full incident response report
│   ├── README.md                    ← Detailed findings with screenshots
│   └── IR_Report.docx               ← Formal IR report document
```

---

## 📚 What I Learned

**Working backwards from ransomware impact** is a different muscle than hunting from first access forward. You have to resist the urge to start at the beginning — the encrypted files and ransom note are your anchors, and you trace the process tree and file events backwards to rebuild the chain.

The most interesting technical finding was the **anonymous lsass pipe access** — events where the initiating process image was completely absent. That's a strong indicator of process injection or handle duplication rather than a direct tool execution, and it wouldn't have been caught by hunting on process names alone.

The **beacon versioning** was also notable — wsync.exe was deployed, failed (likely caught by Defender before kill.bat ran), killed via `Stop-Process`, then re-downloaded and re-deployed after defences were disabled. That retry loop is visible in the file events and is a good pattern to hunt for.

---

## 🔗 Related

- [SancLogic Cyber Range](https://sanclogic.com/sanclogic-ops) — where this investigation was conducted
- [MITRE ATT&CK](https://attack.mitre.org) — technique reference
- [Akira Ransomware — CISA Advisory](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-109a)

---

<div align="center">
<b>Quincy Jones Jr</b><br>
Threat Hunter & Incident Responder
</div>
