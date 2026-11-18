# Detection Analysis

## Suricata findings
| Signature shown | Interpretation | Evidentiary limit |
|---|---|---|
| ET EXPLOIT Possible ETERNALBLUE Probe MS17-010 (MSF style) | Metasploit-style probe pattern | Does not independently prove compromise |
| ET EXPLOIT ETERNALBLUE Probe Vulnerable System Response MS17-010 | Vulnerable-response indicator | Correlate with exploitation result |
| ET EXPLOIT Possible ETERNALBLUE Probe MS17-010 (Generic Flags) | Additional suspicious SMB probe | Pattern-based alert |
| ET INFO SMB Executable File Transfer | SMB executable transfer pattern | Does not by itself establish execution |
| GPL NETBIOS SMB-DS ADMIN$ share access | Administrative share activity | Can also occur in legitimate administration |
| ET HUNTING PE EXE Download over raw TCP | Windows PE transfer indicator | Examine flow and context |
| ET MALWARE Possible Metasploit Payload Common Construct Bind_API | Payload-associated content pattern | Signature match, not complete attribution |
| ET INFO Python SimpleHTTP ServerBanner | Kali-hosted HTTP service response | Informational rather than malicious by itself |

See [Suricata event list](../screenshots/12-suricata-alerts.png) and [EternalBlue alert](../screenshots/18-eternalblue-alert.png).

## Zeek findings
Observed datasets: `zeek.smb_mapping`, `zeek.smb_files`, `zeek.dce_rpc`, `zeek.ntlm`, `zeek.file`. Four `zeek.smb_files` events in the shown result use `SMB::FILE_OPEN`. These provide SMB metadata, not independent proof of payload execution. See [Zeek timeline](../screenshots/16-zeek-smb-events.png) and [file events](../screenshots/17-zeek-file-events.png).

## Correlation example
At approximately 19:34:43 PST, Suricata reported SMB share access and executable-transfer activity between 192.168.56.107 and 192.168.56.105 on TCP/445. Zeek recorded SMB file and DCE/RPC events in the same interval. Suricata also showed a raw-TCP executable transfer and a Metasploit-related signature on the subsequent reverse session. At 19:34:45, Metasploit reported a successful Meterpreter session. This sequence supports a coherent intrusion reconstruction.

## Detection gaps and follow-up
1. Export individual alert JSON including `rule.name`, signature ID, flow ID, source/destination, and exact timestamps.
2. Retrieve matching Zeek `conn` events and correlate their UIDs with SMB/DCE-RPC events.
3. Inspect a bounded PCAP around 19:34:43 to verify the SMB exchange and payload connection.
4. Determine whether Security Onion can distinguish the 19:23 and 19:34 attempts as separate incidents.
5. Verify any endpoint command activity with Windows-side logs or attacker-side evidence; network sensors alone cannot reliably recover all commands.
6. Avoid calling the HTTP test an exfiltration event: it was a download **to XP**. The 19-byte Meterpreter download is the lab's XP-to-Kali transfer.
