# Incident Investigation Report — LAB-001

**Date:** 2026-11-17 (PST, UTC−08:00)  
**Classification:** Authorized simulated intrusion  
**Target:** 192.168.56.105 (Windows XP SP3)  
**Source:** 192.168.56.107 (Kali Linux)  
**Vulnerability / tool:** MS17-010 / Metasploit `ms17_010_psexec`

## Executive summary
Nmap reconnaissance identified SMB on TCP 139 and 445. Metasploit exploitation of the target's SMB service established Meterpreter sessions, with SYSTEM identity verified. Suricata recorded EternalBlue probe and payload-related signatures. Zeek captured SMB and RPC metadata. Subsequent host enumeration and a synthetic file download were confirmed from attacker-side screenshots, with limited network-level attribution for individual commands.

## Evidence timeline
| Time (PST) | Observed event | Source |
|---|---|---|
| 18:56 | Nmap SYN scan: TCP 139/445 open | [01](../screenshots/01-initial-nmap.png) |
| 18:57 | Service detection: Windows XP SMB | [01](../screenshots/01-initial-nmap.png) |
| 19:03 | Full TCP scan; 2 open, 2 closed, 65,531 filtered | [02](../screenshots/02-full-port-scan.png) |
| 19:20:32 | Suricata EternalBlue probe alert (MSF style) | [18](../screenshots/18-eternalblue-alert.png) |
| 19:23:02 | First Meterpreter session established | [04](../screenshots/04-first-meterpreter-session.png) |
| 19:29:08 / 19:30:36 | HTTP GETs to Kali server (first `/`, then `/test.txt`) | [05](../screenshots/05-http-server.png) |
| 19:34:43 | Suricata SMB executable-transfer, ADMIN$ and Metasploit payload-related events; Zeek SMB records | [12](../screenshots/12-suricata-alerts.png), [16](../screenshots/16-zeek-smb-events.png) |
| 19:34:45 | Second Meterpreter session established; SYSTEM identity | [07](../screenshots/07-second-meterpreter-session.png) |
| After compromise (exact time not shown) | Host/account discovery; synthetic file downloaded to Kali | [08](../screenshots/08-host-discovery.png), [09](../screenshots/09-account-discovery.png), [10](../screenshots/10-meterpreter-download.png) |

## Technical findings
### Finding 1: Exposed legacy SMB
Nmap detected TCP/139 and TCP/445 open on the Windows XP SP3 target. This establishes the accessible attack surface, not vulnerability status by itself.

### Finding 2: Confirmed remote compromise
Metasploit `exploit/windows/smb/ms17_010_psexec` established a reverse TCP Meterpreter session. `getuid` returned `NT AUTHORITY\SYSTEM`. The screenshots support successful remote code execution with high privileges.

### Finding 3: Network IDS detection
Suricata reported EternalBlue probing and payload-associated traffic. Zeek independently recorded SMB mapping, file-open and DCE/RPC-related activity. Correlation by source, destination, port and time strengthens attribution to the lab sequence.

### Finding 4: Post-exploitation and file movement
The host was enumerated using Windows commands. An HTTP server on Kali returned `test.txt` to the XP browser. Separately, Meterpreter successfully downloaded `C:\evidence-lab.txt` (19 bytes) to Kali. This demonstrates a synthetic outbound transfer from the target, but the uploaded screenshots do not prove that the file contents were decoded by Security Onion.

## Caveats
- Two successful sessions are visible; treat them as distinct executions, not a single continuous session.
- The `whoami` command failed on Windows XP; Meterpreter `getuid` is the successful privilege evidence.
- The HTTP GET is Kali-to-XP file delivery; the Meterpreter download is XP-to-Kali file retrieval.
- IDS detection does not imply automatic blocking. An alert detail shows `rule.action: allowed`.
- Some screenshots show a large total event count; this is not an incident-specific count.

## Containment and remediation recommendations
Patch or replace unsupported Windows XP, restrict TCP/445, disable SMBv1 where feasible, use network segmentation, and add endpoint monitoring to complement network evidence.
