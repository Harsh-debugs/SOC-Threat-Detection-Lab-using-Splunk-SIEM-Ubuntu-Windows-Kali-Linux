# SPL Queries — SOC Threat Detection Lab

This file documents the Splunk Search Processing Language (SPL) queries used
throughout this project to detect and analyse authentication events, process
creation, and DNS activity collected from the monitored Windows endpoint.

---

## Authentication Events

### Failed Logons (Detailed)
Lists failed logon attempts with account, failure reason, source IP, hostname, and logon type.
```spl
index=main EventCode=4625
| table _time Account_Name Failure_Reason Source_Network_Address ComputerName Logon_Type
| sort - _time
```

### Failed Logons (Summary View)
Simplified failed-logon table without logon type — used for a compact dashboard panel.
```spl
index=main EventCode=4625
| table _time Account_Name Failure_Reason Source_Network_Address ComputerName
| sort - _time
```

### Successful Logons
Lists successful logons with account, logon type, source IP, and hostname.
```spl
index=main EventCode=4624
| table _time Account_Name Logon_Type Source_Network_Address ComputerName
| sort - _time
```

---

## Process Creation (Sysmon Event ID 1)

### Process Creation Details
Extracts the executed image path and command line for every process creation event.
```spl
host=Client source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>1</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]+)"
| table _time Image CommandLine
| sort - _time
```

### Top Executed Processes
Ranks the 10 most frequently executed processes, excluding the Splunk forwarder itself.
```spl
host=Client source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>1</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)"
| search NOT Image="*SplunkUniversalForwarder*"
| top limit=10 Image
```

---

## DNS Queries (Sysmon Event ID 22)

### DNS Query Details
Extracts DNS query names and their resolved results.
```spl
host=Client source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>22</EventID>"
| rex field=_raw "<Data Name='QueryName'>(?<DNS_Query>[^<]+)"
| rex field=_raw "<Data Name='QueryResults'>(?<QueryResults>[^<]+)"
| table _time DNS_Query QueryResults
| sort - _time
```

---

## Dashboard Panel Queries (Event Counts)
Single-value count queries used to power the summary panels on the SOC Threat Detection Dashboard.

```spl
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>1</EventID>"
| stats count
```
```spl
index=main EventCode=4625
| stats count
```
```spl
index=main EventCode=4624
| stats count
```
```spl
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>22</EventID>"
| stats count
```

---

## Alert Base Searches
Raw searches configured behind the three scheduled alerts in Splunk. These run
on a schedule and trigger the alert when results are returned.

**Multiple Failed Logon Detection**
```spl
index=main EventCode=4625
```

**Successful Logon Detection**
```spl
index=main EventCode=4624
```

**DNS Query Activity**
```spl
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>22</EventID>"
```

---

## General / Health-Check

### Events by Source Type
Breaks down all indexed events by their source — useful as a log-ingestion health-check panel.
```spl
index=main
| stats count by source
```
