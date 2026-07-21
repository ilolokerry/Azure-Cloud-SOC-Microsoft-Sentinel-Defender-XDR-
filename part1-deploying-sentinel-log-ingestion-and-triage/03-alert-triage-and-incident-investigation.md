# Alert Triage and Incident Investigation

## Overview

This report documents triaging and investigating an incident in Microsoft Sentinel's Incidents queue, working through ownership, evidence review, entity investigation, and escalation the way an analyst would handle it on a live queue rather than just walking through the interface.

## Incident Summary

| Field | Detail |
|---|---|
| Incident Name | Soloriate Network Beacon |
| Incident Number | 3 |
| Severity | High |
| Alert Provider | Azure Sentinel |
| Description | Identifies a match across various data feeds for domain IOCs related to the Soloriate incident |
| Evidence | 6 Events, 5 Alerts, 0 Bookmarks |

## Triage

Three new, open incidents were sitting in the queue — two Medium severity (sign-ins from atypical IPs, a malicious inbox rule) and one High severity (Soloriate Network Beacon). With limited time, severity is what decides where attention goes first, so I prioritized the High severity incident over the two Medium ones.

[SCREENSHOT: Incidents queue showing 3 open incidents by severity]

Before touching the evidence, I took ownership of the incident by assigning it to myself and moving its status from New to In Progress. This matters on a real queue — it tells the rest of the team the incident is being worked and stops duplicate effort on the same alert.

[SCREENSHOT: Incident details pane showing owner assignment and status change to In Progress]

Note: Soloriate (SolarWinds) is a real, complex supply-chain compromise. A full investigation of the actual campaign is out of scope here — this exercise is about practicing Sentinel's incident handling and triage workflow, not reproducing the SolarWinds investigation itself.

## Investigation

### Reviewing the Incident Timeline

I opened View full details to move from the summary card into the full Incident details page, then worked through the Incident timeline — five alerts, all tied to the same Soloriate Network Beacon detection. Rather than treating five alerts as five separate problems, I read them as one incident with five pieces of supporting evidence, which is the point of Sentinel's alert-to-incident correlation.

[SCREENSHOT: Incident details Overview tab with the incident timeline listing five alerts]

I opened one alert from the timeline to review its detail flyout — severity, status, the tactic it mapped to (Command and Control), and its two associated entities: an IP address and a domain. This confirmed the alert wasn't a one-off signature match; it had real network entities attached that could be pivoted on.

[SCREENSHOT: Alert flyout showing severity, tactics, and entities]

### Validating the Raw Event Data

An alert is a conclusion, not evidence, so I clicked into the alert's linked events to see the actual log data behind it rather than taking the detection at face value. This opened the Logs pane in the context of the incident, showing a KQL query matching DNS lookups against a list of known-malicious domains.

[SCREENSHOT: Logs pane showing the underlying KQL query and query results]

The query results confirmed a DNS lookup event for `avsvmcloud.com`, resolved via Cisco Umbrella DNS logging, tied to a source IP entity. This is the point where a suspected alert becomes a confirmed technical finding — the domain match wasn't just a detection rule firing, it was a real DNS query that occurred on the network.

### Investigating the Entities

Closing the Logs pane surfaced the Entities panel next to the timeline, listing the IP address and the DNS domain as the two entities tied to this incident. I selected the IP entity and reviewed its Info tab, which returned geolocation and ownership data: registered to Apple Inc. (Manufacturing), geolocated to Dongguan, Guangdong, China.

[SCREENSHOT: Entity Info panel showing geolocation for the IP address]

An IP registered to a legitimate manufacturer but geolocated abroad and tied to a known C2 domain is exactly the kind of mismatch that warrants deeper scrutiny — it doesn't confirm malicious infrastructure on its own, but it's inconsistent enough with normal traffic that it can't be dismissed as noise either.

### Decision Point

At this stage I had: a High severity incident, five correlated alerts mapped to Command and Control, a confirmed DNS query to a known-malicious domain, and an IP entity with geolocation that didn't cleanly explain the activity. That combination — confirmed technical evidence plus unresolved attribution — is the threshold where a Tier-1 analyst hands off rather than continuing to dig solo. Deep C2 attribution and infrastructure analysis is Tier-2 threat hunting territory, so I made the call to escalate rather than attempt to close this out myself.

## Escalation

Before reassigning the incident, I logged a task so the next analyst wasn't starting cold:

- Task: Threat Hunting
- Note: Tier-2 threat hunting required for IP entity

[SCREENSHOT: Incident tasks pane with the Threat Hunting task added]

I then reassigned ownership from myself to the SOC LVL 2 Analyst group, formally handing the incident off with the evidence, timeline, and task already documented on the incident.

[SCREENSHOT: Owner reassignment dropdown showing SOC LVL 2 Analyst group selected]

## Key Takeaways

- Severity is the first triage filter on a live queue — with three open incidents, working the High severity one first before the two Medium ones is a basic but essential discipline.
- Taking ownership and updating status isn't just bookkeeping — it communicates work-in-progress to the rest of the team and prevents duplicate investigation.
- An alert is a conclusion; the underlying log data is the evidence. Pivoting from the alert into the raw query results is what turns a detection into a confirmed finding.
- Entity investigation (geolocation, ownership) adds context but doesn't replace it — a mismatch between an IP's registered owner and its geolocation is a flag to escalate, not proof of compromise on its own.
- Knowing when to escalate is as much a Tier-1 skill as running the investigation itself — leaving a clear task and evidence trail for Tier-2 is what makes the handoff actually useful.

## Conclusion

I triaged and investigated the Soloriate Network Beacon incident from initial ownership through evidence review and entity analysis, and escalated it to SOC LVL 2 for threat hunting once the evidence pointed toward infrastructure attribution beyond Tier-1 scope.
