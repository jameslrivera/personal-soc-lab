# Personal SOC Lab

I created this project to learn about and get hands on expierence with SIEM systems and their tools. This project was built from the ideas of industry standard SOC(Security Operations Center) labs, and modified to include some tools that I was interested in learning about. This project is made up of a MacOS machine (hosts Elastic Search, Kibana, and Fleet Server), a Windows 11 machine (the monitored endpoint device), and a Kali Linux machine (simulated threat actor). The lab is equipt with agents to monitor, alert, and respond to events and threat on the endpoint device using the following: **Windows Log Events**, **System metrics**, **Network Packet Capture**, and **Elastic Defend**.


This project was inspired by Evermight's Elastic Stack project: `https://github.com/evermight/elastic-stack-docker-part-two`

### Key Goals
- Set up a SIEM lab using Docker, Elastic Search, Kibana, Fleet Sever and Elastic Agent.
- Depoly agents to collect, and prevent events and threats in real-time using a Endpoint Detection and Response system (Elastic Defend_.  
- Simulate attacks from a malicious Kali Linux machine and learn to prevent them. 
- Build visualizations and dashboards for endpoint device monitoring and management

## Setup

The setup establishes the foundation for the SOC lab, including virtualization for isolated environments and containerization for the Elastic Stack core components. This ensures a controlled, reproducible environment for testing without risking production systems.

### Crystal Fetch
- Used Crystal Fetch to securely download the ISO files for Kali Linux and Windows 11.
- This tool was chosen for its efficiency and integrity checks, ensuring tamper-free media.

- This step is necessary to obtain bootable images for VM creation.

### UTM
- Set up two virtual machines in UTM (UTM is a free macOS virtualization tool supporting UEFI for modern OSes).
- **For Kali Linux:**
  - Create a new VM, attach the Kali ISO, configure bridged networking for external access (to communicate with the host and Windows VM), allocate 4GB RAM/2 cores, and boot from ISO to install.
- **For Windows 11:**
  - Similar setup, attach Windows ISO, enable TPM 2.0/Secure Boot in UTM for compatibility, bridged network, 8GB RAM/4 cores, and install.
- Bridged networking allows the VMs to reach the host MacBook (IP 10.0.0.106) and each other, essential for attack simulation and agent enrollment.

### Docker

- Configure the `docker-compose.yml` with approriate IP address for certificate generation
- Configured `.env` for passwords, ports (ES:9200, Kibana:5601, Fleet:8220), and license (trial for full features).
- In the 'personal-soc-lab' directory run `docker compose up -d` to deploy Elastic Search, Kibana and Fleet Server.
 - The certifcates will be generated and the CA.crt and CA fingerprint will need to be retrivead and saved to complete the fleet server registration and secure communication channel.
 

<img width="956" height="412" alt="Screen Shot 2025-11-22 at 5 44 00 PM" src="https://github.com/user-attachments/assets/fad32f3b-78fd-48bb-9212-d6e01e959017" />



---

## Installation of Agents

Agents collect and send data to the SIEM, enabling monitoring and response. Configuration includes policies for integrations and certs for secure tunnels, ensuring encrypted communication between VMs and the host stack.

### Fleet Server 
- Enrolled the Fleet Server (already in Docker) using Kibana-generated token and re-enrolled if needed.
- On Windows VM, downloaded Elastic Agent 8.8.2 ZIP, extracted, and installed as admin with:
  - `.\elastic-agent.exe install --url=https://10.0.0.106:8220 --enrollment-token=[token] --certificate-authorities=ca.crt`
- Imported `ca.crt` to Local Computer Trusted Root store for trust.
- This creates a secure SSL/TLS tunnel for data transfer.

### Configuring Policies and Integrations
- In Kibana > Fleet > Agent policies, created **"Windows-lab"** policy.
- Added integrations:
  - Windows for log events (security, system, application)
  - Packetbeat for network packet capture (protocols like HTTP/DNS)
  - System for metrics (CPU/memory/disk)
  - Elastic Defend for EDR (malware detection/response)
- Configured Defend in "Detect/Prevent" mode (trial license), with event collection (processes/files/network).
- Assigned policy to Windows agent, upgraded for application.
- Debugged health issues by creating exception lists (e.g., trusted apps with file hashes) to sync artifacts.

<img width="1216" height="472" alt="Screen Shot 2025-11-24 at 3 28 08 PM" src="https://github.com/user-attachments/assets/35216778-a5ca-4e48-97d5-2f7610ae07aa" />

---

## Testing and Demonstration

This phase validates the SOC by simulating attacks from Kali, capturing logs/metrics, and visualizing/responding in Kibana. It demonstrates the lab's ability to detect and mitigate threats.

### Data Views and Visualizations in Kibana
- Created data view "Packetbeat Network Traffic" on `logs-network_traffic.*-default`.
- Imported pre-built Packetbeat dashboards for flows/DNS.
- Built custom viz:
  - Line chart for traffic volume (sum network.bytes over time)
  - Donut for top protocols
  - Bar for IP conversations
  - Heatmap for port activity
- Assembled into **"Network Packet Capture Dashboard"** with filters.
- Similar for system metrics (line for CPU usage) and Windows logs (pie for event IDs).
- This enables quick anomaly spotting.

### Attack Simulation from Kali
- Attempted RDP login to Windows (failed, captured in Windows security logs as event 4625).
- Performed Nmap SYN scan:
  - `nmap -sS [Windows IP]`
- Packetbeat logged flows in `logs-network_traffic.flow-default`, showing port probes in heatmap/viz spikes.

### EDR Demonstration
- Downloaded EICAR test file on Windows—Defend detected malware, alerted in `logs-endpoint.events.*`, and (in Prevent mode) blocked execution.
- Visualized alerts in Discover/dashboard, showing rule triggers/response actions like quarantine.


---

## Conclusion and What I Learned

This project reinforced key cybersecurity concepts through hands-on building.

- Docker proved invaluable for containerizing Elastic Stack, allowing easy deployment, isolation, and scalability on my MacBook. I learned its role in reproducible environments, troubleshooting compose files, and managing volumes for persistence—essential for SOC reliability.
- Certificates taught me generation (via elasticsearch-certutil), SAN/IP inclusion for mismatch fixes, and SSL/TLS importance for secure tunnels. Debugging trust issues highlighted CA imports and fingerprints, emphasizing encryption for protecting data in transit.
- Elasticsearch impressed with its all-encompassing nature as a SIEM foundation, handling massive logs/metrics via integrations like Packetbeat for network capture and System for metrics. I learned its versatility for search, alerting, and applications beyond security (e.g., app monitoring).
- The EDR (Elastic Defend) showed why it's crucial for proactive defense, detecting/preventing threats like malware at the endpoint. In this project, it handled responses (alerts/quarantine), teaching policy configs, artifact sync, and integration with Fleet for centralized management—vital for modern SOCs.# Personal SOC Lab


