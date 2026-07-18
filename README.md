# Azure Cloud SOC — Microsoft Sentinel & Defender XDR

## Overview
This project documents my work through TryHackMe's Defending Azure path. I deployed Microsoft Sentinel as a cloud SIEM, ingested log sources, and worked through the full alert lifecycle from detection to triage to closure. I then used Microsoft Defender XDR to detect and investigate a set of common attack techniques, and closed out the project by working four end-of-course incidents end to end, documenting my investigation process and conclusions for each.

## What I Did
- Deployed Microsoft Sentinel and connected Azure log sources into a single workspace.
- Set up detection logic to surface security events from ingested logs.
- Worked through how Sentinel structures Events, Alerts, and Incidents, and how they relate to one another.
- Triaged incidents in Sentinel, assigning ownership and status and working assigned tasks.
- Investigated incidents using the timeline, raw logs, entity mapping, and visual investigation graph.
- Closed incidents out as true positive, benign positive, or false positive based on investigation findings.
- Used Defender XDR to detect and investigate defense evasion, privilege escalation, lateral movement, execution, and credential access techniques.
- Investigated four incidents end to end in Sentinel, documenting findings and conclusions for each.

## Tools and Data Sources
- Microsoft Sentinel (KQL analytics rules, incident triage, investigation graph)
- Microsoft Defender XDR
- MITRE ATT&CK framework for technique mapping

## Repository Structure
```
azure-cloud-soc-sentinel-defender-xdr/
├── README.md
├── part1-deploying-sentinel-log-ingestion-and-triage/
│   ├── sentinel-deployment-and-log-ingestion.md
│   ├── detection-rules-overview.md
│   ├── alert-triage-and-incident-investigation.md
│   └── screenshots/
├── part2-defender-xdr-detection-and-investigation/
│   ├── xdr-defense-evasion.md
│   ├── xdr-privilege-escalation.md
│   ├── xdr-lateral-movement.md
│   ├── xdr-execution.md
│   ├── xdr-credential-access.md
│   └── screenshots/
└── part3-incident-investigations/
    ├── incident-01-account-created-and-deleted.md
    ├── incident-02-signin-attempts-disabled-account.md
    ├── incident-03-explicit-mfa-deny.md
    ├── incident-04-privileged-role-assigned-outside-pim.md
    └── screenshots/
```

## Key Takeaways
- A working SIEM pipeline is only as good as the log sources feeding it — deployment and ingestion decisions upstream directly shape what can be detected downstream.
- The Events, Alerts, and Incidents hierarchy in Sentinel exists to cut down noise: not every event is an alert, and not every alert deserves an incident.
- Triage speed depends on knowing where to look first — ownership and status get an incident moving, but the timeline, logs, and entity map are what actually confirm or rule out a compromise.
- Defender XDR's cross-signal view (identity, endpoint, email) makes it possible to trace an attack technique across the parts of the environment a single log source would miss.
- Closing an incident correctly (true positive, benign positive, or false positive) is as much a part of the workflow as detecting it — it's what keeps the SOC's detection tuning honest over time.

## Note
This project does not cover the KQL skill-boost room from the same TryHackMe path, since that room teaches query syntax rather than an applied investigation scenario.

## Conclusion
I built and documented a complete Azure-based SOC workflow, from Sentinel deployment and log ingestion through alert triage, Defender XDR-based threat detection, and four full incident investigations.
