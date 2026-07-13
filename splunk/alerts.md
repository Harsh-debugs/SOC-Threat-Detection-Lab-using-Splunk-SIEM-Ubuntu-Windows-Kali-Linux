# Splunk Alerts — SOC Threat Detection Lab

This file documents the three correlation-based alerts configured in Splunk
to automatically detect suspicious activity from the monitored Windows
endpoint. All alerts were scheduled searches, run hourly, and verified to
trigger correctly during testing (see `docs/testing-plan.md`).

---

## 1. Multiple Failed Logon Detection

Detects repeated Windows failed logon attempts (Event ID 4625), indicating a
possible brute-force or unauthorized access attempt.

**Search**
```spl
index=main EventCode=4625
```

| Setting | Value |
|---|---|
| Trigger Condition | Number of Results is > 0 |
| Alert Type | Scheduled — Hourly, at 15 minutes past the hour |
| Severity | Medium |
| Action | Add to Triggered Alerts |
| Permissions | Private, owned by admin |

---

## 2. Successful Logon Detection

Detects successful Windows logons (Event ID 4624), used to correlate against
failed-logon activity and confirm whether an authentication attack ultimately
succeeded.

**Search**
```spl
index=main EventCode=4624
```

| Setting | Value |
|---|---|
| Trigger Condition | Number of Results is > 0 |
| Alert Type | Scheduled — Hourly, at 15 minutes past the hour |
| Action | Add to Triggered Alerts |
| Permissions | Private, owned by admin |

---

## 3. DNS Query Activity

Detects DNS resolution requests captured via Sysmon (Event ID 22) on the
monitored endpoint, providing visibility into outbound name-resolution
activity.

**Search**
```spl
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" "EventID>22</EventID>"
```

| Setting | Value |
|---|---|
| Trigger Condition | Number of Results is > 0 |
| Alert Type | Scheduled — Hourly, at 15 minutes past the hour |
| Action | Add to Triggered Alerts |
| Permissions | Private, owned by admin |

---

## Verification

All three alerts were confirmed to have triggered correctly in the
**Triggered Alerts** view within Splunk during testing:

- Multiple Failed Logon Detection — triggered on scheduled runs following simulated failed logons.
- Successful Logon Detection — triggered twice during the test window, confirming successful authentication events were captured.
- DNS Query Activity — triggered twice during the test window, confirming DNS telemetry from Sysmon was correctly indexed.

See `screenshots/` for the triggered-alert views (Multiple Failed Logon,
Successful Logon, DNS Query Activity).
