# Attack Simulation Commands — SOC Threat Detection Lab

This file documents the reconnaissance, authentication-attack, and
telemetry-generation commands used to simulate malicious activity against
the monitored Windows 11 endpoint from the Kali Linux attacker machine, and
the manual/PowerShell commands used to generate additional log telemetry.

> **Note:** IP addresses below are placeholders for the actual lab subnet
> (`192.168.56.0/24`) used during testing — replace with your own lab IPs if
> reproducing this setup. Real credentials/wordlists used in testing are not
> included here.

---

## 1. Reconnaissance — Nmap

Identifies the SMB service exposed on the Windows endpoint, simulating the
reconnaissance phase of an attack.

```bash
nmap -Pn -n -p 445 <Windows_IP>
```

**Purpose:** Confirm the SMB (port 445) service is reachable before
attempting authentication attacks against it.

**Expected Splunk result:** Network/connection telemetry indexed and
searchable; scan activity visible in Splunk (see Test Case T01 in the
testing plan).

---

## 2. Brute-Force Authentication — Hydra

Simulates a brute-force login attempt against the Windows SMB service to
generate repeated failed-authentication events.

```bash
hydra -l <username> -P <wordlist>.txt -m "local ntlmv2" smb://<Windows_IP>
```

**Purpose:** Generate repeated Event ID 4625 (failed logon) entries on the
Windows endpoint to validate the Multiple Failed Logon Detection alert.

**Expected Splunk result:** Repeated Event ID 4625 entries in
`index=main`; Failed Logon alert triggers (see Test Case T02).

---

## 3. Manual Failed Login

Performed directly on the Windows endpoint to generate an additional failed
logon event outside of the automated Hydra attempt.

**Steps:**
1. Lock the Windows 11 workstation.
2. Enter an incorrect password multiple times at the lock screen.

**Expected Splunk result:** Event ID 4625 generated and visible in the
Failed Logons dashboard panel (see Test Case T03).

---

## 4. Successful Login

Performed directly on the Windows endpoint to generate a successful logon
event for baseline comparison against failed-logon activity.

**Steps:**
1. Unlock the Windows 11 workstation using the correct credentials.

**Expected Splunk result:** Event ID 4624 generated and visible in the
Successful Logons dashboard panel (see Test Case T04).

---

## 5. DNS Query Generation

Generates outbound DNS resolution traffic on the Windows endpoint, captured
by Sysmon for network-activity visibility.

```powershell
nslookup openai.com
```

**Expected Splunk result:** Sysmon Event ID 22 generated; DNS Query
Activity alert triggers (see Test Case T05).

---

## 6. Process Creation Telemetry — Windows Commands

The following commands were executed on the Windows 11 endpoint to generate
Sysmon Event ID 1 (Process Creation) telemetry for baseline and dashboard
testing.

```powershell
whoami
hostname
tasklist
net user
Get-Process
Get-Service
Get-LocalUser
Get-NetIPConfiguration
Get-ChildItem C:\Users
```

**Expected Splunk result:** Sysmon Event ID 1 generated for each process;
events visible in the Process Creation dashboard panel (see Test Case T06).

---

## Summary of Test Coverage

| Command / Action | Attack Type | Detected Via | Test Case |
|---|---|---|---|
| `nmap -Pn -n -p 445 <Windows_IP>` | Reconnaissance | Network/connection events | T01 |
| `hydra -l <user> -P <wordlist> ... smb://<Windows_IP>` | Brute-force authentication | Event ID 4625 | T02 |
| Manual wrong password entry | Failed login | Event ID 4625 | T03 |
| Manual correct login | Successful login | Event ID 4624 | T04 |
| `nslookup openai.com` | DNS activity | Sysmon Event ID 22 | T05 |
| `whoami`, `tasklist`, etc. | Process execution | Sysmon Event ID 1 | T06 |
