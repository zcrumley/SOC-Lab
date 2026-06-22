# Lab 4: RDP Brute-Force Detection with Wazuh

## Overview

This lab focused on detecting repeated failed Remote Desktop Protocol authentication attempts against a Windows 11 endpoint using Wazuh. The goal was to simulate a basic brute-force-style login attempt from a Kali Linux machine and verify that the activity was collected and displayed in the Wazuh dashboard.

Remote Desktop Protocol is commonly used for remote administration, but it is also a frequent target for attackers. If RDP is exposed or poorly protected, attackers may attempt password guessing, credential stuffing, or brute-force login attempts. In this lab, the activity was performed in a controlled home lab environment for defensive security practice.

## Lab Objective

The objective of this lab was to:

* Generate failed RDP login attempts against a monitored Windows 11 endpoint.
* Verify that Wazuh collected Windows authentication failure events.
* Identify the source IP address responsible for the failed login activity.
* Review Windows Security Event IDs related to failed authentication and account lockout.
* Practice documenting the event from a SOC analyst perspective.

## Lab Environment

| Component             | Description          |
| --------------------- | -------------------- |
| SIEM                  | Wazuh                |
| Wazuh Manager         | Ubuntu Server        |
| Endpoint              | Windows 11 VM        |
| Endpoint Agent        | Wazuh Agent          |
| Attacker/Test Machine | Kali Linux VM        |
| Protocol Tested       | RDP                  |
| Target User           | `socla`              |
| Target IP             | `192.168.65.132`     |
| Source IP             | `192.168.65.133`     |
| Dashboard             | Wazuh Threat Hunting |

## Tools Used

* Wazuh
* Windows 11
* Kali Linux
* xFreeRDP
* Remote Desktop Protocol
* VMware Workstation

## Attack Simulation

The test was performed from Kali Linux using `xfreerdp` to attempt an RDP login against the Windows 11 endpoint.

The login attempt used the correct username but an intentionally incorrect password.

```bash
xfreerdp /v:192.168.65.132 /u:socla /p:Password123! /cert:ignore
```

### Command Breakdown

```bash
xfreerdp
```

Launches the FreeRDP client from Kali Linux.

```bash
/v:192.168.65.132
```

Specifies the target Windows 11 machine by IP address.

```bash
/u:socla
```

Specifies the username used for the login attempt.

```bash
/p:Password123!
```

Provides the password used during the login attempt. In this lab, the password was intentionally incorrect to generate failed authentication events.

```bash
/cert:ignore
```

Ignores certificate warnings from the RDP service. This is common in lab environments where self-signed certificates are used.

## Screenshot: RDP Login Attempt from Kali Linux

The screenshot below shows the `xfreerdp` command being executed from the Kali Linux VM against the Windows 11 endpoint.

![Kali xFreeRDP Attempt](screenshots/brute_force_kali.png)

## Wazuh Dashboard Results

After multiple failed RDP login attempts were generated, Wazuh displayed authentication failure activity for the Windows 11 endpoint.

The Wazuh Threat Hunting dashboard showed:

* `11` total alerts
* `11` authentication failure alerts
* `0` authentication success alerts
* Alerts associated with failed logon activity and account lockout behavior

This confirmed that the Windows 11 endpoint was forwarding authentication-related security events to Wazuh.

## Screenshot: Wazuh Authentication Failure Dashboard

![Wazuh Authentication Failure Dashboard](screenshots/brute_force_dashboard.png)

## Observed Security Events

During the investigation, Wazuh displayed Windows Security events related to the failed authentication activity.

Two important event types were observed:

| Event ID | Description                 |
| -------- | --------------------------- |
| `4625`   | Failed logon attempt        |
| `4740`   | User account was locked out |

## Event ID 4625: Failed Logon

Windows Security Event ID `4625` indicates that an account failed to log on. In this lab, the event showed that the user `socla` failed authentication from the Kali Linux source system.

Important fields observed included:

| Field                  | Value            |
| ---------------------- | ---------------- |
| Agent Name             | `Windows11PC`    |
| Agent IP               | `192.168.65.132` |
| Target Username        | `socla`          |
| Source IP Address      | `192.168.65.133` |
| Workstation Name       | `kali`           |
| Authentication Package | `NTLM`           |
| Logon Process          | `NtLmSsp`        |
| Logon Type             | `3`              |
| Status                 | `0xc000006d`     |
| Sub Status             | `0xc000006a`     |

The status and sub-status values are important because they provide additional context about the failed login. In this case, the failure was associated with a bad username or authentication failure, with the sub-status indicating an incorrect password.

## Screenshot: Failed Logon Event Details

![Failed Logon Event Details](screenshots/brute_force_event2.png)

## Event ID 4740: Account Lockout

Windows Security Event ID `4740` indicates that a user account was locked out. This occurred after repeated failed login attempts against the `socla` account.

Important fields observed included:

| Field           | Value                            |
| --------------- | -------------------------------- |
| Agent Name      | `Windows11PC`                    |
| Agent IP        | `192.168.65.132`                 |
| Target Username | `socla`                          |
| Target Domain   | `kali`                           |
| Event ID        | `4740`                           |
| Event Message   | `A user account was locked out.` |

The account lockout event confirmed that the failed authentication attempts triggered the Windows account lockout policy.

## Screenshot: Account Lockout Event Details

![Account Lockout Event Details](screenshots/brute_force_event.png)

## Key Indicators

| Indicator             | Value                                 |
| --------------------- | ------------------------------------- |
| Target Host           | `Windows11PC`                         |
| Target IP             | `192.168.65.132`                      |
| Target Username       | `socla`                               |
| Source Host           | `kali`                                |
| Source IP             | `192.168.65.133`                      |
| Protocol              | RDP                                   |
| Authentication Result | Failed login                          |
| Account Impact        | Account lockout                       |
| Suspicious Behavior   | Repeated failed remote login attempts |

## Why This Activity Matters

A single failed login attempt is not always malicious. Users mistype passwords frequently. However, repeated failed login attempts against RDP can indicate password guessing, brute-force activity, credential stuffing, or unauthorized access attempts.

For SOC analysts, failed authentication events become more important when they involve:

* Multiple failures in a short period of time.
* Attempts from an unusual source IP address.
* Attempts against administrative or privileged accounts.
* Failed logins followed by a successful login.
* Account lockout events.
* Remote access attempts from systems that normally do not connect to the endpoint.

In this lab, the repeated failed RDP login attempts were intentionally generated from Kali Linux to simulate suspicious remote authentication activity.

## SOC Analyst Review

From a SOC analyst perspective, this alert would be reviewed to determine whether the failed login activity was benign or suspicious.

The basic investigation process would include:

1. Identify the affected endpoint.
2. Review the username targeted by the login attempt.
3. Check the source IP address.
4. Determine whether the source IP is expected or unusual.
5. Review the number of failed attempts.
6. Check whether the account became locked out.
7. Determine whether any successful login occurred after the failures.
8. Review related events from the same source IP.
9. Escalate the event if the activity appears unauthorized.

In this case, the activity was expected because it was generated as part of a lab. In a real environment, repeated failed RDP login attempts from an unknown source would be treated as suspicious and investigated further.

## Result

The lab successfully demonstrated that Wazuh can detect failed RDP authentication activity from a monitored Windows endpoint.

The test confirmed that:

* The Windows 11 endpoint was reporting logs to Wazuh.
* Failed RDP authentication attempts were visible in the Wazuh dashboard.
* Wazuh displayed authentication failure alerts.
* The source IP address of the login attempt could be identified.
* The targeted username could be identified.
* Windows account lockout activity was visible in Wazuh.
* The event could be documented as a brute-force detection scenario.

## Defensive Recommendations

To reduce the risk of RDP brute-force attacks, organizations should:

* Disable RDP when it is not needed.
* Restrict RDP access to approved IP addresses.
* Require VPN access before allowing RDP connections.
* Enforce account lockout policies.
* Use strong passwords.
* Enable multi-factor authentication where possible.
* Monitor for repeated failed login attempts.
* Investigate failed logins followed by successful authentication.
* Avoid exposing RDP directly to the internet.
* Alert on repeated authentication failures from the same source IP.

## Conclusion

This lab demonstrated a basic RDP brute-force detection scenario using Kali Linux, Windows 11, and Wazuh. By intentionally attempting to authenticate with an incorrect password, failed login activity was generated and reviewed in the Wazuh dashboard.

The lab reinforced how SOC analysts investigate authentication failures, identify suspicious remote login behavior, and determine whether activity may indicate brute-force behavior.

Although the test activity was harmless and performed in a controlled lab environment, the same type of detection is important in real networks because RDP brute-force attempts are a common method attackers use to gain unauthorized access to Windows systems.
