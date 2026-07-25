# XDR: Privilege Escalation

## Overview

This report documents investigating a Privilege Escalation incident in Microsoft Defender XDR — a UAC bypass carried out via a registry hijack of the default file association for `.mscfile`, combined with launching an auto-elevating Windows binary. I traced the incident from the alert queue through the MITRE ATT&CK mapping, evidence, and process chain that confirmed the technique.

## Investigation

### Step 1: Locate the Incident

1. Signed into the Microsoft Defender portal and navigated to Incidents & alerts > Incidents.
2. Changed the Time Range filter to 6 months.
3. Located and selected the incident "Multi-stage incident involving Privilege escalation on one endpoint."

[SCREENSHOT: Incidents queue showing the multi-stage privilege escalation incident]

### Step 2: Review Incident Context

On the Attack story tab, I reviewed the incident's core details before diving into individual alerts:

- Severity: High
- Status: Active
- Incident ID: 88
- Categories: Execution, Persistence, Privilege escalation, Defense evasion, Exfiltration
- 17 correlated alerts, 2 assets involved

The incident graph showed a single endpoint at the center, connected to multiple processes, IPs, files, a registry value, and URLs — confirming this was one coordinated attack chain spanning several tactics rather than isolated alerts.

[SCREENSHOT: Attack story tab showing the incident graph and incident details pane]

### Step 3: Review Alerts by Category

1. Switched to the Alerts tab (17 alerts) and reviewed each one alongside its assigned category — Exfiltration, Execution, Privilege escalation, and Persistence were all represented.
2. Identified "UAC bypass was detected" as the alert most relevant to the Privilege escalation category and selected it to open its full alert details.

[SCREENSHOT: Alerts tab listing all 17 alerts with their categories]

### Step 4: Review the UAC Bypass Alert

1. On the alert details page, reviewed the severity (Medium) and status (Detected, New).
2. Checked the MITRE ATT&CK Techniques listed in the right-hand pane:
   - T1112 — Modify Registry
   - Bypass User Account Control

[SCREENSHOT: UAC bypass alert details pane showing MITRE ATT&CK techniques]

3. Reviewed the Evidence section, which listed three suspicious entities:
   - `registry-value` — Suspicious
   - `reg.exe` (PID: 3924) — Suspicious
   - A link-local IPv6 address (`fe80::...`) — Suspicious

[SCREENSHOT: Evidence section listing the registry value, reg.exe process, and IPv6 address]

### Step 5: Confirm the Technique in the Process Tree

I cross-checked the alert against the Process tree to confirm the detection was accurate rather than taking the alert summary at face value.

The confirmed process chain was: `ntoskrnl.exe` started a `powershell.exe` process, which set a registry key, which in turn led to `cmd.exe` being launched through the hijacked association.

The full command executed was:
```powershell
powershell.exe & {New-Item "HKCU:\software\classes\mscfile\shell\open\command" -Force
Set-ItemProperty "HKCU:\software\classes\mscfile\shell\open\command" -Name "(default)" -Value "C:\Windows\System32\cmd.exe" -Force
Start-Process "C:\Windows\System32\eventvwr.msc"}
```

[SCREENSHOT: Process tree showing the registry key set event with MITRE technique T1546.001]

This is a fileless UAC bypass: the script hijacks the default open command for `.mscfile` files under `HKCU:\software\classes\mscfile\shell\open\command`, pointing it at `cmd.exe` instead of its normal handler. It then launches `eventvwr.msc` — a Microsoft binary that auto-elevates without a UAC prompt — which, due to how Windows resolves `.mscfile` associations, ends up executing the hijacked command instead, spawning an elevated `cmd.exe` without ever triggering the UAC consent dialog.

## Key Takeaways

- A UAC bypass alert is really two techniques working together: a registry-based default file association hijack (T1546.001 / T1112) paired with abuse of an auto-elevating trusted binary — neither technique alone explains the full attack.
- The Evidence section on an alert (suspicious entities) is a useful shortlist, but confirming the actual process chain in the Process tree is what verifies the alert is accurate rather than a false positive.
- Registry values under `HKCU:\software\classes\` controlling file-type command handlers are a common UAC bypass target, since modifying them doesn't require elevated privileges but can result in elevated execution once the associated trusted binary is launched.
- A single incident with 17 alerts spanning five categories (Execution, Persistence, Privilege escalation, Defense evasion, Exfiltration) is a reminder that privilege escalation rarely happens in isolation — it's usually one stage in a longer attack chain.

## Conclusion

I investigated a UAC bypass incident that hijacked the `.mscfile` default file association to launch an elevated `cmd.exe` via `eventvwr.msc`, confirmed the technique through the process tree and command line, and mapped it to MITRE ATT&CK techniques T1112 (Modify Registry) and Bypass User Account Control.
