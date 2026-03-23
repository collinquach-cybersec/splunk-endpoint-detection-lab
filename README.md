# Cybersecurity Home Lab: SIEM Deployment & Endpoint Brute Force Detection

An end-to-end virtual SOC lab featuring Windows 10, Kali Linux, and Splunk to simulate, detect, and investigate an SMB brute-force attack using Sysmon endpoint telemetry.

## Objective
The objective of this project was to build a functional Security Operations Center (SOC) environment to simulate and detect an active network attack. This involved configuring a Windows endpoint with advanced telemetry (Sysmon), forwarding those logs to a central SIEM (Splunk), and utilizing a Kali Linux attacker machine to conduct reconnaissance and execute an SMB brute-force attack.

## Tools & Technologies
* **Hypervisor**: Oracle VirtualBox
* **Operating Systems**: Windows 10 (Target), Kali Linux (Attacker)
* **SIEM**: Splunk Enterprise
* **Telemetry**: Windows Event Logs, Sysinternals Sysmon
* **Offensive Tools**: Nmap (Reconnaissance), NetExec / CrackMapExec (Exploitation)

---

## The Process

### Phase 1: Infrastructure Setup
To ensure a safe and isolated testing environment, two virtual machines were deployed on a custom NAT Network. The victim machine was provisioned with Windows 10, and the attacker machine was built using a clean installation of Kali Linux.

<img width="1520" height="1080" alt="1 Infrastructure" src="https://github.com/user-attachments/assets/5b0d8338-6665-4e3a-b06a-13ae032009f7" />

### Phase 2: Telemetry & SIEM Configuration
To gain deep visibility into the endpoint's activity, Sysmon was installed on the Windows target using a custom XML configuration file. Splunk Enterprise was installed locally. Direct edits were made to Splunk's backend `inputs.conf` file to force the ingestion of Sysmon operational logs and standard Windows Security logs, bypassing GUI limitations to establish a robust telemetry pipeline.

<img width="1707" height="1377" alt="8 Telemetry pipeline" src="https://github.com/user-attachments/assets/f9b369d6-99bc-415c-ad0d-796c2cf20a6b" />

### Phase 3: Reconnaissance & Lowering Defenses
Switching to the Kali Linux machine, reconnaissance was performed using Nmap to identify open ports on the target (10.0.2.15). To simulate a vulnerable legacy system, the Windows Defender Firewall was disabled across all profiles, exposing high-value ports like 445 (SMB) and 139 (NetBIOS) to the network.

<img width="2562" height="1449" alt="11 After Firewalls Down" src="https://github.com/user-attachments/assets/f04e0179-a339-46d9-b8d5-d5fc2e9790cd" />

### Phase 4: Adversary Emulation
With Port 445 exposed, a targeted dictionary brute-force attack was launched against the SMB service. A custom password list was generated, and NetExec was utilized to hammer the endpoint with authentication requests, ultimately cracking the target's credentials.

<img width="1277" height="1377" alt="13 Successful Breach" src="https://github.com/user-attachments/assets/aca15c39-8074-4280-8e87-318233e9533e" />

### Phase 5: Detection & Investigation
Acting as the Blue Team, the final phase involved querying the Splunk SIEM to detect the breach. Using Splunk Processing Language (SPL), the main index was filtered for Windows Security Event Code 4624 (Successful Logon). The resulting telemetry confirmed the exact time of the breach, the Logon Type (3 - Network), and explicitly identified the attacker's source IP address.

<img width="2565" height="1449" alt="14 Splunk Security Log" src="https://github.com/user-attachments/assets/2707346d-5b11-414a-a90f-796a46c9b6c1" />

### Phase 6: Infrastructure Teardown
Following the successful detection, the environment was gracefully spun down. System snapshots were taken at key milestones (Clean Install, SIEM Configured) to allow for rapid redeployment and future testing without rebuilding the infrastructure.

<img width="1517" height="1077" alt="15 Lab Complete VM's Saved" src="https://github.com/user-attachments/assets/0f97696b-8deb-428b-8a6c-cb9e1cbb1500" />

---

## Outcomes & Skills Applied
* **Log Ingestion & Configuration**: Navigated back-end `.conf` files to manually route endpoint telemetry and Sysmon data to a SIEM.
* **Offensive Reconnaissance & Exploitation**: Conducted network service enumeration with Nmap and executed network-based password cracking using NetExec.
* **Threat Detection & SPL**: Developed precision queries in Splunk to isolate malicious logon events (Event ID 4624/4625) and trace attacker origins.
* **Infrastructure Management**: Managed virtual networks, modified security controls for testing, and safely preserved configurations using VM snapshots.

---

## Troubleshooting & Lessons Learned

### 1. Splunk Service Stability and Resource Provisioning
* **Issue**: The `splunkd` service would initialize and then immediately terminate.
* **Resolution**: Identified a resource bottleneck within the Virtual Machine. By increasing the allocated RAM from **4GB to 6GB** and ensuring **2 CPU cores** were provisioned, the Splunk indexing engine achieved a stable state.

### 2. Manual Data Ingestion and UI Workarounds
* **Issue**: The Splunk Web interface encountered "Page Not Found" errors and failed to populate the Sysmon log path in the standard data input menus.
* **Resolution**: Bypassed GUI limitations by manually configuring the **`inputs.conf`** file in `C:\Program Files\Splunk\etc\system\local\`. By explicitly defining the Sysmon operational log stanza and restarting the service via the CLI, I successfully established the telemetry pipeline.

### Key Lessons
* **Configuration over GUI**: Relying on backend configuration files is often more reliable than web interfaces when managing enterprise security tools.
* **Infrastructure Dependencies**: A SIEM's effectiveness is directly dependent on the underlying hardware allocation. Monitoring system resources is a critical first step in troubleshooting ingestion failures.
