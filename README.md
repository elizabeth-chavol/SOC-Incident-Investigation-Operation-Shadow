# 🛡️ Threat Hunting & SOC Incident Investigation: Operation Shadow

## 📌 Incident Summary
An end-to-end security incident was investigated within the corporate domain `techcorp.local`. A malicious actor executed a brute-force attack against an Active Directory account, performed lateral movement via Remote Desktop Protocol (RDP), established persistence, and exfiltrated sensitive data using DNS Tunneling.

---

## 🗺️ Cyber Kill Chain Timeline

| Phase | Technique / Event | Indicators of Compromise (IoCs) & Details |
| :--- | :--- | :--- |
| **1. Initial Access** | Brute Force Attack | **Source IP:** `10.0.2.88`<br>**Target User:** `a_garcia`<br>**Events:** `4625` (Failures) $\rightarrow$ `4624` (Success) |
| **2. Privilege Escalation**| Admin Rights Assigned | **Event ID:** `4672`<br>**Privileges Assigned:** `SeDebugPrivilege`, `SeSecurityPrivilege` |
| **3. Lateral Movement** | Remote Desktop Abuse | **Target Server:** `10.0.0.20` (`FS-01`)<br>**Protocol:** RDP (Port 3389)<br>**Logon Type:** `10` |
| **4. Persistence** | Rogue Account Creation | **Event ID:** `4720`<br>**Account Created:** `svc_support_test` |
| **5. Exfiltration** | Covert DNS Tunneling | **Query Type:** `TXT`<br>**Payload (Base64):** `Q29uZmlkZW50aWFsX0RhdGE=`<br>**Decoded Data:** `Confidential_Data` |

---

## 🔍 Investigation Findings & Technical Analysis

### 1. Intrusion & Privilege Escalation
A rapid succession of failed logon attempts (`Event ID 4625`) was observed within an 11-second window from IP `10.0.2.88`. Upon successful authentication (`Event ID 4624`), high-risk privileges (`SeDebugPrivilege`) were granted, allowing process injection capabilities.

### 2. Lateral Movement & Persistence
The threat actor established an interactive remote session (**Logon Type 10**) targeting the File Server (`10.0.0.20`). To maintain access without relying on the compromised credentials, a local backdoor account named `svc_support_test` was created (**Event ID 4720**).

### 3. Data Exfiltration
Data exfiltration was achieved via DNS queries using `TXT` records directed to an external attacker-controlled domain. Decodification of the encoded subdomains revealed exfiltrated sensitive strings (`Confidential_Data`).

---

## 🛠️ Mitigation & Recommendations

1. **Account Isolation:** Disable `a_garcia` and delete the rogue account `svc_support_test` immediately.
2. **Credential Reset:** Force a domain-wide password reset for affected administrative users.
3. **Network & RDP Hardening:** Restrict RDP access (Port 3389) using Network Level Authentication (NLA) and MFA.
4. **SIEM Detections:** Implement correlation rules for bursts of `Event ID 4625` followed by `4624` and `4720` within 30 minutes.
5. **DNS Inspection:** Monitor and limit outbound DNS `TXT` record query volume.

