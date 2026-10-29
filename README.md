# SNMP Enumeration Using Metasploit

## Project Overview
A hands-on cybersecurity lab focused on understanding Simple Network Management Protocol (SNMP) enumeration using Kali Linux, SNMP utilities, and the Metasploit Framework.

## Objectives
- Understand SNMP and UDP port 161.
- Check SNMP service reachability using Nmap.
- Retrieve system information using `snmp-check`, `snmpget`, and `snmpwalk`.
- Practice SNMP enumeration with Metasploit.
- Analyze information disclosure risks and document verified findings.

## Lab Environment
- **Operating System:** Kali Linux
- **Target Operating System:** Windows 7 SP1
- **Target Hostname:** `Manav-PC`
- **Target IP:** `192.168.1.46`
- **Protocol:** SNMP over UDP/161
- **SNMP Version Tested:** SNMPv2c
- **Community String:** `labpublic` (lab-only)
- **Tools:** Nmap, snmp-check, snmpget, snmpwalk, Metasploit Framework

## Methodology
1. Check UDP port 161 on the authorized target.
2. Verify SNMP responses using `snmp-check`.
3. Retrieve individual system objects using `snmpget`.
4. Enumerate system and network-interface information using `snmpwalk`.
5. Run the Metasploit SNMP enumeration module and record the actual output.
6. Analyze security implications and document evidence.

## Results
SNMP queries successfully returned system and network-interface information from the Windows 7 lab target. Detailed observations are documented in the practical sections below.

## Security Recommendations
- Restrict SNMP access to trusted management hosts.
- Prefer SNMPv3 with authentication and privacy where supported.
- Avoid guessable or reused SNMP community strings.
- Disable SNMP if it is not required.
- Review the system and network information exposed through SNMP.

## Disclaimer
All testing was performed in an isolated lab environment on an authorized target. The results apply only to the lab configuration.

---

## Practical 1: SNMP Service Reachability Check with Nmap

### Objective
Check whether SNMP is reachable on UDP port 161 on the authorized Windows 7 target.

### Lab Details
- **Tool:** Nmap
- **Target:** `192.168.1.46`
- **Port:** `161/UDP`

### Command
```bash
sudo nmap -sU -sV -p 161 192.168.1.46
```

### Observed Results
- Nmap reported `161/udp open|filtered snmp no-response`.
- The UDP scan result alone did not confirm whether the port was open.
- Subsequent successful SNMP client responses confirmed that the target responded to valid SNMP queries.

### Security Analysis
UDP scans may report `open|filtered` when no response is received. A successful SNMP query provides stronger evidence of service reachability.

### Evidence
Screenshot: `screenshots/nmap-snmp-udp161.png`

### Learning Outcome
Practiced checking UDP service reachability and interpreting the limitations of UDP scan results.

---

## Practical 2: SNMP Enumeration Using snmp-check

### Objective
Retrieve system and network information exposed by the SNMP agent.

### Lab Details
- **Tool:** `snmp-check`
- **Target:** `192.168.1.46`
- **Community String:** `labpublic`

### Command
```bash
snmp-check 192.168.1.46 -c labpublic
```

### Observed Results
- Hostname: `Manav-PC`
- Operating system reported: Windows 7 SP1, Build 7601
- Target IP: `192.168.1.46`
- Gateway reported: `192.168.1.1`
- User account information was returned.
- Additional system, network, and service information was exposed.

### Security Analysis
The SNMP agent returned system and network metadata to a client using the configured community string. Depending on the information exposed, this can assist an attacker with target profiling.

### Recommendations
- Restrict SNMP queries to trusted hosts.
- Review the information exposed to SNMP clients.
- Use SNMPv3 with authentication and privacy where supported.

### Evidence
Screenshot: `screenshots/snmp-check-enumeration.png`

### Learning Outcome
Practiced collecting and interpreting SNMP system information with `snmp-check`.

---

## Practical 3: System Description Retrieval Using snmpget

### Objective
Retrieve the system description from a specific SNMP object identifier (OID).

### Command
```bash
snmpget -v2c -c labpublic 192.168.1.46 1.3.6.1.2.1.1.1.0
```

### Observed Results
- The query returned a system description string.
- The response reported Windows Version 6.1, Build 7601.
- Hardware platform information was also returned.

### Analysis
The `sysDescr.0` OID provides a textual system description. This information can help identify the target platform during authorized enumeration.

### Evidence
Screenshot: `screenshots/snmpget-system-description.png`

### Learning Outcome
Learned how to retrieve a specific SNMP object using its OID.

---

## Practical 4: Hostname Retrieval Using snmpget

### Objective
Retrieve the target hostname using the SNMP system-name OID.

### Command
```bash
snmpget -v2c -c labpublic 192.168.1.46 1.3.6.1.2.1.1.5.0
```

### Observed Results
```text
STRING: "Manav-PC"
```

- The returned hostname was `Manav-PC`.
- The query successfully retrieved the `sysName.0` object.

### Security Analysis
Hostnames can help identify systems and understand a network's naming conventions. Hostname disclosure should be assessed alongside the other information available to an SNMP client.

### Evidence
Screenshot: `screenshots/snmpget-hostname.png`

### Learning Outcome
Practiced retrieving a specific system attribute using its OID.

---

## Practical 5: System Information Enumeration Using snmpwalk

### Objective
Enumerate multiple objects under the standard SNMP system group.

### Command
```bash
snmpwalk -v2c -c labpublic 192.168.1.46 1.3.6.1.2.1.1
```

### Observed Results
- System description: Windows 7, Build 7601
- System object ID: `1.3.6.1.4.1.311.1.1.3.1.1`
- Hostname: `Manav-PC`
- System uptime: approximately 32 minutes in the captured output
- System contact and location values were empty
- System services value: `76`

### Analysis
The `snmpwalk` command queried multiple objects within the system subtree. The returned information provided an overview of the target system's identity and SNMP management data.

### Evidence
Screenshot: `screenshots/snmpwalk-system-group.png`

### Learning Outcome
Practiced enumerating a standard SNMP MIB subtree instead of querying each OID individually.

---

## Practical 6: Network Interface Enumeration Using snmpwalk

### Objective
Retrieve network-interface descriptions from the target's SNMP agent.

### Command
```bash
snmpwalk -v2c -c labpublic 192.168.1.46 1.3.6.1.2.1.2.2.1.2
```

### Observed Results
The response included descriptions for:
- Software Loopback Interface
- WAN Miniport components
- RAS Async Adapter
- Intel PRO/1000 MT Network Connection
- Microsoft ISATAP adapters
- Network-related QoS components

Some interface descriptions were returned as hexadecimal strings.

### Analysis
Interface enumeration can help identify physical and virtual network components. Interface descriptions alone do not confirm which adapters are active or which IP addresses are assigned to them.

### Recommendations
- Review exposed interface information.
- Restrict SNMP access to authorized management hosts.
- Avoid exposing unnecessary network metadata.

### Evidence
Screenshot: `screenshots/snmpwalk-interfaces.png`

### Learning Outcome
Practiced enumerating interface descriptions and interpreting hexadecimal SNMP output.

---

## Practical 7: SNMP Enumeration with Metasploit

### Objective
Use a Metasploit auxiliary module to enumerate SNMP information from the authorized Windows 7 lab target.

### Lab Details
- **Tool:** Metasploit Framework
- **Module:** `auxiliary/scanner/snmp/snmp_enum`
- **Target:** `192.168.1.46`
- **Community String:** `labpublic`

### Methodology
1. Search for the SNMP enumeration module.
2. Select `auxiliary/scanner/snmp/snmp_enum`.
3. Configure the authorized target and community string.
4. Run the module.
5. Record the actual output and verify the information returned.

### Observed Results
Add the verified Metasploit output and findings from the completed run here.

### Security Analysis
The impact depends on the information returned by the module and the target's SNMP configuration. Findings should be documented from observed output rather than assumed from the module's expected behavior.

### Evidence
Screenshot: `screenshots/metasploit-snmp-enumeration.png`

### Learning Outcome
Practiced using a Metasploit auxiliary scanner for SNMP enumeration and interpreting the returned information.

---

## Overall Learning Outcomes
- Understood SNMP reachability checks and UDP scan limitations.
- Practiced using `snmp-check`, `snmpget`, and `snmpwalk`.
- Learned how OIDs identify specific system and interface objects.
- Identified system and network metadata exposed by the lab's SNMP configuration.
- Reviewed defensive measures for reducing unnecessary SNMP information disclosure.

## References
- Nmap documentation: https://nmap.org/book/man.html
- Net-SNMP documentation: https://www.net-snmp.org/docs/

## Scope and Disclaimer
This project documents authorized testing in an isolated Windows 7 lab. The community string is included only to describe the lab configuration; do not reuse it in production environments.
