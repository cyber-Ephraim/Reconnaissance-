# Network Reconnaissance & Open Port Analysis
**Using Nmap & Wireshark**  
Prepared by: Ephraim Chukwumerije  
Date: 15/11/2025

________________________________________

## 1. Executive Summary (Short)
This report documents the results of a local network reconnaissance performed using Nmap with optional packet analysis via Wireshark. The goal was to identify active hosts, detect open ports, discover running services and OS information, and highlight potential security risks with prioritized recommendations for remediation.

________________________________________

## 2. Tools Used
- Nmap v7.x  
- Wireshark (optional, for packet capture and deeper protocol inspection)  
- Kali Linux (VM) and Windows (host)

________________________________________

## 3. Network Information

Item | Value
--- | ---
Host Machine IP | 10.213.23.223
VM IP (if used) | 192.168.110.128
Network Range | 10.213.23.223/24
Default Gateway / Netmask | 255.255.255.0

________________________________________

## 4. Methodology
Steps performed during the reconnaissance:
1. Determined local addressing and network range using `ipconfig` / `ip a` in Windows and Kali to identify active NICs and IPs.  
2. Performed host discovery with Nmap: `nmap -sn 10.213.23.223/24` to enumerate live hosts.  
3. Performed TCP SYN (stealth) port scanning: `nmap -sS <target>` to find open ports.  
4. Performed OS and service detection: `nmap -A <target>` (or `-sV -O`) for service versions and OS fingerprints.  
5. Optionally captured traffic with Wireshark for any suspicious or plaintext protocols (e.g., POP3) for additional verification.  
6. Researched identified services and potential vulnerabilities (CVE research, vendor advisories).  
7. Compiled findings, risk analysis, and remediation recommendations.

________________________________________

## 5. Findings

### 5.1 Live Hosts Detected
- 10.213.23.223 — Host machine (Windows / Wi‑Fi connection)  
- 192.168.110.128 — Kali Linux VM (VMware)

> Note: These hosts reflect the environment used for the scan (host + VM). If additional hosts are present on the /24 range, run a full `nmap -sn 10.213.23.0/24` to enumerate them.

________________________________________

### 5.2 Open Ports & Services (Observed)
The following ports/services were observed during the scans (replace / augment with exact `nmap -sS -sV` output if available):

Port/proto | Service (common) | Purpose / Notes
--- | --- | ---
110/tcp | POP3 | Unencrypted email retrieval (plaintext credentials possible)
135/tcp | MS‑RPC (RPC Endpoint Mapper) | Windows RPC service used by many Microsoft services
139/tcp | NetBIOS-SSN | NetBIOS Session Service (file & printer sharing legacy)
445/tcp | Microsoft-DS (SMB) | SMB over TCP (file sharing); high-risk if exposed
902/tcp | VMware-auth / VMware remote management | VMware remote management (VMware Tools / ESXi related)

(If `nmap -sV` was run, include version strings here. If `nmap -A` returned further ports/services, list them similarly.)

________________________________________

### 5.3 OS Detection (From `nmap -A`)
IP | OS Detected / Notes
--- | ---
10.213.23.223 | Windows host — Wi‑Fi connection (fingerprint suggests Windows family)
192.168.110.128 | Kali Linux (VMware virtual machine)

> OS detection can be noisy; validate by combining service banners, TTL, and other fingerprints. If precise OS fingerprinting is required, run targeted scans with increased timing and privileges.

________________________________________

### 5.4 Wireshark Capture (Optional)
Summary of any notable captures or observations:
- If POP3 (port 110) was observed, Wireshark may show plaintext credentials or messages — high privacy risk.  
- NetBIOS/SMB traffic (139/445) may reveal hostnames, shared resource names, or unauthenticated information.  
- If management traffic to port 902 exists, check for unencrypted or unauthenticated management sessions.

(Attach pcap or include screenshots/packet snippets in appendices if available.)

________________________________________

## 6. Risk Analysis
Explanation of risks associated with the discovered open ports:

Port | Service | Purpose | Risk Level | Possible Attacks
--- | --- | --- | --- | ---
110/tcp | POP3 | Email inbox retrieval | High | Credential theft, MITM, brute force, plaintext password exposure
135/tcp | MS‑RPC | Windows RPC Endpoint Mapper | High | Remote code execution, DCOM attacks, malware propagation
139/tcp | NetBIOS | File & printer sharing | High | SMB/NetBIOS enumeration, credential leaks, session hijacking
445/tcp | SMB (Microsoft‑DS) | File sharing | Critical | Worms/exploits (e.g., EternalBlue), ransomware, lateral movement
902/tcp | VMware remote mgmt | VMware remote management | Medium–High | Unauthorized remote access, brute force, exploitation of management interfaces

Notes:
- SMB (445) is typically the highest immediate risk when exposed to untrusted networks or the internet.  
- POP3 (110) exposes credentials unless POP3S (995) or IMAPS is used.  
- MS‑RPC (135) facilitates many Windows remote operations and is commonly targeted by attackers for code execution or lateral movement.

________________________________________

## 7. Recommendations
Action | Description | Priority
--- | --- | ---
Disable SMB/NetBIOS if not required | Turn off ports 139 & 445 or block at network perimeter. Use SMB signing/SMBv3 where possible. | 🔥 High
Migrate POP3 → Encrypted email | Replace POP3 with POP3S (995) or IMAP over TLS (IMAPS) / use SMTP with STARTTLS. | 🔥 High
Restrict access using firewall / ACLs | Allow only trusted internal IPs to management/file sharing ports. Use host‑based firewall rules. | 🔥 High
Enforce strong passwords + account lockout | Prevent brute force and credential stuffing attacks. | High
Patch & update OS and services | Apply vendor security updates to mitigate known exploits (SMB/Windows patches). | High
Use VPN for remote management | Protect VMware/management interfaces (e.g., 902) behind VPN. | Medium
Enable TLS/SSL for management & email | Ensure all management and email services use TLS with strong cipher suites. | Medium
Enable IDS/IPS & logging | Monitor for suspicious activity and maintain centralized logs/alerts. | Medium

Additional actions:
- Run authenticated vulnerability scans and/or credentialed Nmap scripts (NSE) to assess missing patches and weak configurations.  
- If SMB is required, isolate file servers in segmented VLANs and restrict access to application-specific users.  
- Regularly rotate credentials and enable multi‑factor authentication where supported.

________________________________________

## 8. Conclusion
The network scan identified two primary hosts in the local test environment (host and Kali VM) and several services that, depending on exposure and configuration, could present material risks—most notably unencrypted email (POP3) and SMB services. The findings underscore the importance of minimizing exposed services, enforcing encryption for authentication and management, patching systems promptly, and applying network segmentation and access controls. Follow the recommendations above to reduce the attack surface and improve overall security posture.

________________________________________

Appendix / Next steps:
- Provide raw Nmap command outputs (scan .gnmap / .xml files) and Wireshark pcap(s) as attachments for auditing.  
- If you want, I can produce a checklist and prioritized remediation plan tailored to this environment, or help parse actual Nmap output (paste the raw scan) to produce a fully populated report with exact service versions and CVE mappings.