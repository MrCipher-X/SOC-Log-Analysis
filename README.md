\# 🛡️ SOC Log Analysis: Brute Force Detection \& Threat Hunting with Splunk



\## 📖 Executive Summary

In this project, I simulated the role of a \*\*Level 1 SOC Analyst\*\* responding to a potential credential compromise. The objective was to deploy a SIEM solution (Splunk), ingest raw Windows Security logs, and develop high-fidelity detection rules to identify Brute Force attacks in real-time. This lab demonstrates proficiency in \*\*Log Analysis, SPL (Splunk Processing Language), and Incident Triage.\*\*



---



\## 🏗️ Technical Architecture

\* \*\*SIEM Platform:\*\* Splunk Enterprise 9.x

\* \*\*Log Source:\*\* Windows Event Logs (Security Channel) via Local Monitor.

\* \*\*Key Event IDs Monitored:\*\*

&nbsp;   \* `4625`: Logon Failure (Unknown user or bad password).

&nbsp;   \* `4624`: Successful Logon (Used to correlate successful entry after failures).

&nbsp;   \* `4688`: Process Creation (Used to inspect post-compromise activity).



---



\## 🔬 Investigation Methodology



\### Phase 1: Attack Simulation

To generate realistic data, I simulated a manual Brute Force attack:

1\.  Locked the target workstation.

2\.  Executed 15+ failed login attempts using the `Administrator` and `User` accounts within a 2-minute window.

3\.  Performed a successful login to simulate an "Account Takeover."



\### Phase 2: Log Ingestion \& Normalization

I configured Splunk to monitor the `WinEventLog:Security` channel. The raw XML data was parsed to extract critical fields:

\* `Account\_Name` (Target Identity)

\* `Workstation\_Name` (Source Device)

\* `Source\_Network\_Address` (Attacker IP - \*Simulated as Localhost\*)



\### Phase 3: Detection Engineering (SPL)

I wrote the following Splunk Query to detect the anomaly. I used `stats` to aggregate failures and `sort` to prioritize the highest threats.



```splunk

index=main sourcetype="WinEventLog:Security" EventCode=4625

| stats count by Account\_Name, Workstation\_Name, Source\_Network\_Address

| rename Account\_Name as "Target User", Workstation\_Name as "Machine", count as "Failed\_Attempts"

| where Failed\_Attempts > 5

| sort - Failed\_Attempts

