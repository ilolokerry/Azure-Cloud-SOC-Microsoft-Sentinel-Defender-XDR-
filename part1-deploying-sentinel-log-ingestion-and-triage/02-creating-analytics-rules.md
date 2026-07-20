# Creating and Enabling Analytics Rules

## Overview

This report documents creating a custom analytics rule in Microsoft Sentinel to detect rare, sensitive subscription-level operations in Azure. I enabled the rule from a built-in template, reviewed its detection logic and MITRE ATT&CK mapping, configured its schedule and incident settings, and added an automation rule to tag incidents it generates.

## Investigation

### Step 1: Enable an Analytics Rule from a Template

1. Under Configuration, selected Analytics, then switched to the Rule templates tab.
2. Searched for the "rare subscription-level operations" rule.
3. Selected the rule template and clicked Create rule.

![step1.1](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step1.1.png)

4. On the General tab, reviewed the rule description — the rule looks for sensitive subscription-level events based on Azure Activity Logs, such as the "Create or Update Snapshot" operation, which is legitimately used for backups but could be misused by attackers to dump hashes or extract sensitive information from disk.
5. Expanded the MITRE ATT&CK dropdown and confirmed the rule maps to two tactics: Persistence and Credential Access.
![step1.2](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step1.2.png)

### Step 2: Review the Rule Logic

1. On the Set rule logic tab, read through the rule query and noted the `SensitiveOperationList` array, which defines the specific Azure operations the rule watches for.
2. Noted that the rule query can be customized by editing the values in the `SensitiveOperationList` array directly.
3. Used the Test with current data option to view a results simulation — a chart of the last 50 rule evaluations, with the option to click a point and view the raw events behind it.
4. Updated the Query scheduling to run every 1 hour, looking up data from the last 7 days.

![step2](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step2.png)

### Step 3: Configure Incident Settings and Automated Response

1. On the Incident settings tab, left the default enabled so that incidents are created from alerts triggered by this rule, noting that alert grouping can also be configured here to reduce noise from repeated single alerts.

![step3.1](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/st6ep3.1.png)
![step3.15](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step3.15.png)

2. Skipped the Automated response tab at this stage, since it can throw an error before the rule is initially saved.
3. Selected Review + create to save the rule.
4. Went back to the Active rules tab, searched for the newly created rule, selected it, and clicked Edit on the sidebar to return to the wizard.
5. On the Automated response tab, added a new automation rule:
   - Rule Name: Tag Rule
   - Trigger: When incident is created
   - Actions: Add tags — Tag: subscription
   - Rule expiration: left as default
6. Clicked Apply, then Review + create to finalize the rule.

![step3.2](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step3.2.png)

### Step 4: Review the Enabled Analytics Rule

1. Returned to the Analytics page and switched to the Active rules tab.
2. Selected the saved rule, Rare subscription-level operations in Azure, and reviewed its details on the right pane, confirming:
   - Severity: Low
   - Status: Enabled
   - Rule frequency: runs every 1 hour
   - Rule period: looks back over the last 7 days
![step4.1](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step4.1.png)

3. Clicked Compare with template to view the differences between my configured settings and the original rule template in YAML, confirming the query frequency and query period had been changed from the template's defaults.

![step4.2](https://github.com/ilolokerry/Azure-Cloud-SOC-Microsoft-Sentinel-Defender-XDR-/blob/c791fb16e473924307560fab157d46702a5f2f14/part1-deploying-sentinel-log-ingestion-and-triage/Media/Analytics%20Rules%20media/step4.2.png)

## Key Takeaways

- Enabling a rule template doesn't just turn detection logic on — the query frequency, lookup period, and incident settings all need to be reviewed and tuned to fit the environment rather than left at template defaults.
- Detection logic in Sentinel is customizable at the query level — arrays like `SensitiveOperationList` can be edited directly to expand or narrow what a rule watches for.
- Automation rules attached to an analytics rule (like auto-tagging incidents) are configured separately from the detection logic itself, and are best added after the rule is initially saved to avoid wizard errors.
- The Compare with template feature is a fast way to confirm exactly which settings were customized away from the built-in template, which is useful both for documentation and for troubleshooting rule behavior later.

## Conclusion

I enabled and customized the "Rare subscription-level operations in Azure" analytics rule, adjusting its schedule, confirming its MITRE ATT&CK mapping to Persistence and Credential Access, and adding an automation rule to tag incidents it generates.
