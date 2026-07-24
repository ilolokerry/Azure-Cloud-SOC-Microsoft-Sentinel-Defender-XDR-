# XDR: Defense Evasion

## Overview

This report documents investigating a Defense Evasion incident in Microsoft Defender XDR — an attempt to disable Microsoft Defender Antivirus protection via the registry. I traced the incident from the alert queue through the process tree, alert timeline, and command-line evidence, identified the MITRE ATT&CK technique involved, and reviewed the response actions available to contain it.

## Investigation

### Step 1: Locate the Incident

1. Signed into the Microsoft Defender portal and navigated to Investigation & response > Incidents & Alerts > Incidents.
2. Set the Time Range to 6 months to widen the search window.
3. Searched for the incident name "Attempt to turn off Microsoft Defender Antivirus protection" in the search bar.

[SCREENSHOT: Incidents queue filtered to the Attempt to turn off Microsoft Defender Antivirus protection incident]

4. Scrolled down to locate the matching incident and selected it to open the full incident page.

[SCREENSHOT: Full incident page for Attempt to turn off Microsoft Defender Antivirus protection]

### Step 2: Review Incident Context

On the incident page, I reviewed the core context before digging into individual alerts:

- Device involved: vm-evil-xdr
- User account involved: evil-xdr
- Category: Defense evasion
- Severity: High

I noted this incident was flagged as part of a larger, multi-stage incident involving Execution and Lateral Movement leading toward ransomware on the same endpoint — a reminder that a single alert rarely tells the whole story on its own.

[SCREENSHOT: Incident page showing device, user, category, and severity details]

### Step 3: Walk the Alert Timeline

1. Switched to the Alert timeline tab to see the specific sequence of events tied to this alert, rather than just the summary.
2. Reviewed the event: `reg.exe attempted to turn off the Microsoft Defender Antivirus security feature`, flagged as Remote execution.

[SCREENSHOT: Alert timeline showing the reg.exe event flagged as Remote execution]

3. Expanded the event and reviewed the specific details:
   - Date and time of the event: 6/4/2026, 4:25:52 PM
   - Severity: High
   - Process name: powershell_ise.exe
   - Command-line tool used to attempt to disable Defender, initiated by explorer.exe
   - The registry value being changed, alongside its new and original state, for comparison
   - MITRE technique: T1562.001 — Disable or Modify Tools

[SCREENSHOT: Expanded event details showing process name, initiating process, and registry value change]

4. Noted the specific registry key targeted: `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender\DisableAntiSpyware` — a direct attempt to turn off Defender's antispyware protection via policy.

### Step 4: Investigate the Initiating Process

1. Selected the toggle icon on the initiating process (`reg.exe`) to expand its full execution details.
2. Reviewed the process details:
   - Process ID and image file path (`C:\Windows\System32\reg.exe`)
   - Execution details: Token elevation Full, Integrity level High
   - Image file SHA1 hash
   - Image file creation and last modification timestamps
   - Full command line used, targeting a different Defender setting (`Real-Time Protection`) in this instance

[SCREENSHOT: Expanded reg.exe process details with command line, hash, and execution details]

3. Confirmed the User and Initiated by fields to trace exactly which account and parent process launched the command.

[SCREENSHOT: User and Initiated by fields showing the account and parent process]

### Step 5: Review Available Response Actions

1. Reviewed the three-dot menu on the alert for management options, including Submit item to Microsoft for review if further explanation from Microsoft is needed.

[SCREENSHOT: Alert management menu showing Submit item to Microsoft for review]

2. Reviewed the device-level response actions available if the alert is confirmed malicious:
   - Collect Investigation Package for further analysis
   - Start Microsoft Defender XDR Automated Investigation
   - Initiate Live Response Session to review the registry changes directly
   - Isolate the device to prevent lateral movement or further access to other devices

[SCREENSHOT: Device action menu showing Collect Investigation Package, Initiate Automated Investigation, Initiate Live Response Session, and Isolate Device options]

3. Noted that this incident name recurred multiple times in the queue, each instance tied to a different registry setting or process — a reminder to check each recurrence individually rather than assuming they're duplicates of the same event.

## Key Takeaways

- Defense Evasion techniques like Impair Defense (T1562.001) target the security tooling itself, so registry changes to Defender-related keys are a high-value signal even before any other malicious activity is confirmed.
- The initiating process and its parent (explorer.exe → reg.exe here) matter as much as the command itself — tracing that chain shows whether the action was manually triggered, scripted, or launched remotely.
- Comparing the new and original registry values directly in the alert is what confirms intent — a registry key being set is only meaningful in the context of what it changed from and to.
- An incident that recurs multiple times in the queue isn't automatically duplicate noise — each occurrence can involve a different registry setting or process, so each is worth checking individually.
- Response actions in Defender XDR range from passive (collecting an investigation package) to active containment (isolating the device), so the right action depends on how confident the investigation is that the activity is malicious.

## Conclusion

I investigated a Defense Evasion incident involving an attempt to disable Microsoft Defender Antivirus via the registry, traced it to a `reg.exe` process launched under `explorer.exe`, confirmed the technique as T1562.001 (Disable or Modify Tools), and reviewed the available response actions for containment.
