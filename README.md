# 🛡️ SOC Incident Analysis Report: Phishing Macro & C2 Beaconing

![Status](https://img.shields.io/badge/Status-Completed-green)
![Severity](https://img.shields.io/badge/Severity-Critical-red)
![Platform](https://img.shields.io/badge/Platform-Windows%207-blue)
![Tools](https://img.shields.io/badge/Tools-Wireshark%20%7C%20Zui%20%7C%20PCAP-orange)

## 📋 Report Information

| Item | Details |
| :--- | :--- |
| **Incident Title** | Accounting Workstation Compromise via Invoice Phishing & Malicious Macro |
| **Victim** | Sara (Accounting Department) |
| **Organization** | Umbrella Corporation |
| **Incident Date** | November 27, 2018 |
| **Analyst** | Abdulrezzak AL-Ahmad |
| **Tools Used** | Wireshark, Zui (Brim), PCAP Analysis, HTTP Stream Analysis |
| **Severity Level** | 🔴 Critical |

---

## 1. Executive Summary

The Security Operations Center (SOC) received a SIEM alert indicating connections from an accounting workstation (Sara) to a known malicious domain. Investigation revealed that Sara received an email containing a fake invoice with a malicious macro document. Upon opening the document, the macro downloaded malware from a compromised website, which then established a connection with a Command & Control (C2) server for remote command execution.

---

## 2. Timeline of Events

| Time (Approx.) | Event |
| :--- | :--- |
| 16:30:58 | Sara's host requests a disguised `.avi` file from `cochrimato.com` |
| 16:31:00 | First chunk of encrypted payload downloaded |
| 16:31:01 | Additional payload chunks downloaded |
| 16:31:03 | Final payload chunk downloaded |
| ~16:31:05 | Connection to C2 server `83.166.247.211` on port 443 begins |
| Ongoing | Periodic beaconing signals sent to C2 server |

---

## 3. Attack Chain

### 3.1 Reconnaissance & Delivery
- **Target:** Sara (Accountant receiving daily invoices) — ideal social engineering target.
- **Vector:** Email containing a fake invoice.
- **Technique:** Business Email Compromise (BEC) or compromised customer account.

### 3.2 Exploitation
- **File:** Office document (`.docm` or similar) containing a macro.
- **Behavior:** Upon opening, the macro executes and a "crash" is displayed as a decoy.
- **Result:** Malware downloaded from `cochrimato.com`.

### 3.3 Installation
- Use of **Payload Splitting** technique.
- Encrypted chunks disguised as `.avi` files downloaded.
- In-memory reassembly and execution without writing to disk.

### 3.4 Command & Control (C2)
- **Malicious Server:** `83.166.247.211`
- **Port:** 443 (HTTPS) for evasion.
- **Behavior:** Beaconing — small periodic signals (1-2 KB) requesting commands.
- **Persistence:** Background process (Backdoor) with persistence mechanism.

### 3.5 Actions on Objectives
- Host is now under attacker control.
- Potential for data exfiltration or ransomware deployment.

---

## 4. Indicators of Compromise (IOCs)

### 4.1 Network

| Type | Value | Description |
| :--- | :--- | :--- |
| Domain | `cochrimato.com` | Malicious payload delivery domain |
| IP | `83.166.247.211` | Command & Control (C2) server |
| IP | `95.181.198.231` | Suspicious IP — possible data exfiltration |
| Port | `443` | C2 communication port (HTTPS) |
| Port | `80` | Payload delivery port (HTTP) |
| Port | `53` | DNS queries — possible tunneling |
| URL Path | `/images/.../ojw.avi` | Disguised malicious file |
| URL Path | `/images/.../timxEQW.avi` | Disguised malicious file |
| URL Path | `/images/.../6.avi` | Disguised malicious file |
| User-Agent | `Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0)` | IE11 on Windows 7 |

### 4.2 Files

| Type | Value |
| :--- | :--- |
| Extension | `.avi` (used deceptively) |
| Content-Type | `text/html` (instead of `video/x-msvideo`) |
| Content | Encrypted strings (Base64/XOR) embedded in packets |

### 4.3 Host

| Type | Value |
| :--- | :--- |
| OS | Windows 7 (64-bit) |
| Browser | Internet Explorer 11 |
| Victim IP | `10.27.101.27` |

---

## 5. PCAP Analysis

### 5.1 C2 Connections (Beaconing)
- **Connection Count:** Dozens of connections (Streams 3-50+) to the same server.
- **Data Size:** 1-2 KB per connection (Heartbeat).
- **Interval:** Repeated and regular (automated).

### 5.2 DNS Protocol Usage
- Suspicious DNS queries via `208.67.222.222` (OpenDNS).
- Potential use of **DNS Tunneling** to bypass security controls.

### 5.3 Payload Splitting Technique
- **Reason:** Bypass firewalls and intrusion detection systems.
- **Method:** Split file into small, seemingly harmless chunks.
- **Result:** In-memory assembly without disk writes.

---

## 6. Technical Conclusion

The attack represents an initial **Advanced Persistent Threat (APT)** using advanced techniques:

1. **Social Engineering** (targeted phishing).
2. **Macro-based Execution** (execution via macro).
3. **Payload Splitting** (fragmentation to bypass defenses).
4. **In-Memory Execution** (execution without disk writes).
5. **HTTPS Beaconing** (encrypted C2 communication).
6. **DNS Tunneling** (potential).

This indicates an experienced Threat Actor, not a random attack.

---

## 7. Recommendations

### 7.1 Immediate Containment
- [ ] Isolate Sara's host (`10.27.101.27`) from the network immediately.
- [ ] Disable Sara's account and reset the password.
- [ ] Block domain `cochrimato.com` on firewall and DNS.
- [ ] Block IP `83.166.247.211` on firewall.

### 7.2 Forensics
- [ ] Perform memory forensics on Sara's host.
- [ ] Review EDR logs for suspicious processes (`powershell.exe`, `cmd.exe`).
- [ ] Analyze `Temp` and `Downloads` folders for payload remnants.
- [ ] Review Sara's email logs for similar messages.

### 7.3 Prevention
- [ ] Disable macros by default in Office for all users.
- [ ] Implement Security Awareness Training.
- [ ] Enable sandbox analysis on the email gateway.
- [ ] Apply the Principle of Least Privilege.
- [ ] Update browsers (IE11 is insecure — migrate to Chrome/Edge).
- [ ] Deploy EDR/XDR on all endpoints.

### 7.4 Long-Term Response
- [ ] Report malicious domain to hosting provider.
- [ ] Report C2 server `83.166.247.211` to CERT.
- [ ] Review network segmentation policies.

---
