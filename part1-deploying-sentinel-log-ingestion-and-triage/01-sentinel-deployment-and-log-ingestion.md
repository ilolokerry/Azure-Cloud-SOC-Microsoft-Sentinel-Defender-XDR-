# Sentinel Deployment and Log Ingestion

## Overview

This report documents deploying Microsoft Sentinel as a cloud SIEM and configuring log ingestion. I created a Log Analytics Workspace (LAW), enabled Sentinel on it, configured data retention, then used the Content Hub to install and connect data connectors for Microsoft Entra ID and Threat Intelligence so that log data and indicators of compromise (IOCs) would start flowing into the workspace.

## Investigation

### Step 1: Create a Log Analytics Workspace

Sentinel runs as a service on top of a Log Analytics Workspace rather than as a standalone resource, so the first step was creating the workspace it would live on.

1. Logged into the Azure portal and searched for "Sentinel" in the top search bar.
2. Since no LAW with Sentinel enabled existed yet, selected Create Microsoft Sentinel.
3. On the Add Microsoft Sentinel to a workspace page, selected Create a new workspace.
4. Filled in the workspace details — Subscription, Resource Group, Name, and Region — noting that Region determines where ingested log data resides and cannot be changed after creation.
5. Selected Review + Create, then Create, and waited for the LAW deployment to finish.

[SCREENSHOT: Log Analytics Workspace creation form with subscription, resource group, name, and region filled in]

### Step 2: Add Sentinel to the Workspace

1. Returned to the Add Microsoft Sentinel to a workspace page and confirmed the new LAW appeared in the list.
2. Selected the workspace and clicked Add.
3. Waited for Sentinel to be added, then confirmed the "Microsoft Sentinel free trial activated" message.

[SCREENSHOT: Sentinel successfully added to the Log Analytics Workspace]

Browsing the Overview tiles — Incidents, Data, Automation, and Analytics — confirmed this was a brand-new deployment with no data connectors or incidents yet.

### Step 3: Set Data Retention

Sentinel billing is based on the volume of data ingested and retained, so I reviewed and adjusted the retention period rather than leaving it at default.

1. Navigated to Microsoft Sentinel > selected workspace > Configuration > Settings.
2. Since data is stored in the underlying LAW, went to Workspace Settings > Usage and estimated costs > Data Retention.
3. Reviewed the default retention of 31 days and adjusted the Data Retention (Days) slider to fit the lab scenario, noting that longer retention increases storage cost.

[SCREENSHOT: Data Retention configuration screen showing the selected retention period]

### Step 4: Install the Microsoft Entra ID Content Hub Solution and Connect It

1. Went to Content management > Content hub and confirmed no standalone contents or packaged solutions were installed yet.
2. Searched for Microsoft Entra ID, selected it, and clicked Install.
3. Reviewed the solution details after install: 63 Analytics Rules, 1 Data Connector, 11 Playbooks, and 2 Workbooks.

[SCREENSHOT: Microsoft Entra ID Content Hub solution showing Installed status]

4. Opened the Microsoft Entra ID connector page and reviewed its Description, Data Types (the log tables it would populate — SignInLogs, AuditLogs, and others), and Instructions sections.
5. Checked Prerequisites, which listed the permissions needed: workspace read/write, diagnostic settings read/write on Microsoft Entra ID, and Global Administrator or Security Administrator on the tenant.
6. Under Configuration, selected the Entra ID log types to connect.

[SCREENSHOT: Microsoft Entra ID connector configuration page with log types selected]

### Step 5: Install and Connect the Threat Intelligence Data Connector

1. Returned to Content Hub, searched for Threat Intelligence, and installed the solution — 5 Data Connectors, 1 Workbook, 52 Analytics Rules, and 5 Hunting Queries.

[SCREENSHOT: Threat Intelligence Content Hub solution installed]

2. Clicked Manage to review the deployed connectors, selected Microsoft Defender Threat Intelligence, and opened its connector page.
3. Checked Prerequisites — this connector only required workspace read/write permissions, which I already had.
4. Clicked Connect to begin importing IOCs (IPs, domains, URLs, and file hashes) from Microsoft Defender Threat Intelligence (MDTI) into Sentinel.

[SCREENSHOT: Microsoft Defender Threat Intelligence connector Configuration section before connecting]

5. Confirmed the connector status changed from Disconnected to Connected, and that the Last Log Received timestamp began updating.

[SCREENSHOT: Microsoft Defender Threat Intelligence connector showing Connected status]

6. Validated ingestion worked by opening the ThreatIntelligenceIndicator table in Log Analytics and confirming real, structured IOC data — fields like Action, ConfidenceScore, ThreatType, and Tags — was populated.

[SCREENSHOT: Log Analytics query results showing a populated ThreatIntelligenceIndicator event]

## Key Takeaways

- Sentinel deployment is really a two-part process: creating the underlying Log Analytics Workspace and then enabling Sentinel on it, which means some settings, like data retention, are managed at the workspace level rather than in Sentinel itself.
- Data retention is a direct cost/coverage tradeoff — longer retention supports deeper investigations but increases storage cost, so it needs to be set deliberately rather than left at the default.
- Installing a Content Hub solution deploys its data connector, but the connector still needs to be manually configured and connected before any logs populate the workspace.
- Permission requirements vary by connector, so checking the Prerequisites section on each connector page before attempting to connect it saves time and avoids permission errors.
- Sentinel access is role-based, and the right role depends on what the person needs to do in the workspace:

| Role | Typical User | Permissions |
|---|---|---|
| Microsoft Sentinel Reader | Stakeholders, SOC managers | View data, incidents, workbooks, and other Sentinel resources |
| Microsoft Sentinel Responder | Security analysts, incident responders (SOC L1) | Reader permissions, plus manage incidents |
| Microsoft Sentinel Contributor | Security engineers, Fusion Analytics team (SOC L2) | Responder permissions, plus install/update Content Hub solutions and create/edit workbooks and analytics rules |
| Microsoft Sentinel Playbook Operator | Automation team members (SOC L1) | List, view, and manually run playbooks |
| Microsoft Sentinel Automation Contributor | Not for user accounts | Allows Sentinel to add playbooks to automation rules |

## Conclusion

I deployed Microsoft Sentinel on a new Log Analytics Workspace, configured data retention, and installed and connected the Microsoft Entra ID and Threat Intelligence data connectors, validating that Entra ID and IOC log data was successfully ingested into the workspace.
