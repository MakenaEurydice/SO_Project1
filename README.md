# EternalBlue Intrusion Investigation with Security Onion

## Overview
A controlled lab attack against Windows XP SP3 was investigated using Security Onion Evaluation mode, Suricata and Zeek. Kali Linux performed Nmap reconnaissance, Metasploit MS17-010 exploitation, host discovery and a synthetic file transfer. The objective was to compare attacker-side confirmation with network detection evidence.

**Outcome:** Two Meterpreter sessions were observed in the screenshots, with `NT AUTHORITY\SYSTEM` confirmed. Security Onion recorded EternalBlue probe signatures, SMB executable-transfer alerts and a possible Metasploit payload signature. This is a lab demonstration, not evidence of a production incident.

## Lab inventory
| Role | System | IP |
|---|---|---|
| Attacker | Kali Linux | `192.168.56.107` |
| Target | Windows XP SP3, x86, hostname MESOPOTAMIA | `192.168.56.105` |
| Monitor | Security Onion Evaluation mode | SOC at `10.0.2.16` (management UI) |


## Attack chain
1. **Discovery:** Nmap SYN scan of TCP ports 1–1000 found 139/tcp and 445/tcp open; full scan found the same two open ports, 2869 and 3389 closed, and 65,531 filtered. [Initial scan](screenshots/01-initial-nmap.png) · [Full scan](screenshots/02-full-port-scan.png)
2. **Initial access:** Metasploit `exploit/windows/smb/ms17_010_psexec` against TCP/445, using `windows/meterpreter/reverse_tcp`. A session opened at 19:23:02 PST and another at 19:34:45 PST. [Successful session](screenshots/07-second-meterpreter-session.png)
3. **Privilege confirmation:** Meterpreter `getuid` reported `NT AUTHORITY\SYSTEM`; `sysinfo` identified Windows XP SP3 x86. [Evidence](screenshots/07-second-meterpreter-session.png)
4. **Discovery:** Windows commands `hostname`, `ipconfig /all`, `net user`, and `net localgroup administrators` were executed. `whoami` was unavailable on the XP host. [Host](screenshots/08-host-discovery.png) · [Accounts](screenshots/09-account-discovery.png)
5. **Synthetic transfer:** An XP browser retrieved `test.txt` from Kali's HTTP server (GET 200); later, Meterpreter downloaded a 19-byte lab file from XP to Kali. These are **separate transfers** and should not be conflated. [HTTP](screenshots/06-xp-http-test.png) · [Meterpreter download](screenshots/10-meterpreter-download.png)

## Detection highlights
- Suricata: `ET EXPLOIT Possible ETERNALBLUE Probe MS17-010 (MSF style)`, `ET EXPLOIT ETERNALBLUE Probe Vulnerable System Response MS17-010`, and generic-flags probe signatures. [Alert](screenshots/18-eternalblue-alert.png)
- Suricata: `ET INFO SMB Executable File Transfer`, `GPL NETBIOS SMB-DS ADMIN$ share access`, `ET HUNTING PE EXE Download over raw TCP`, and `ET MALWARE Possible Metasploit Payload Common Construct Bind_API`. [Events](screenshots/12-suricata-alerts.png)
- Zeek: `zeek.smb_mapping`, `zeek.smb_files`, `zeek.dce_rpc`, `zeek.ntlm`, and `zeek.file` records, including `SMB::FILE_OPEN` activity. [Zeek](screenshots/16-zeek-smb-events.png) · [File events](screenshots/17-zeek-file-events.png)

## Interpretation
A Suricata signature is evidence of matching traffic, not proof of compromise. Successful exploitation is independently confirmed by the Meterpreter session and SYSTEM identity. Zeek adds protocol context, but the screenshots do not establish all commands executed within the remote session or the content of the Meterpreter file transfer.

## Reports
- [Incident investigation and timeline](docs/incident-report.md)
- [Detection analysis and limitations](docs/detection-analysis.md)

## MITRE ATT&CK
- T1046 Network Service Discovery
- T1210 Exploitation of Remote Services
- T1082 System Information Discovery
- T1087 Account Discovery
- T1005 Data from Local System (synthetic lab file)

## Defensive recommendations
Remove unsupported Windows versions, patch MS17-010, disable SMBv1 where feasible, restrict SMB exposure, segment legacy systems, and correlate network alerts with endpoint telemetry.

## Scope
Authorized, isolated virtual lab. All addresses are private lab addresses. No real-world victim or sensitive data was involved.
