# Personal SOC Lab

## Project Overview

This project sets up a **home cybersecurity lab (SOC - Security Operations Center)** using the **Elastic Stack (ELK)** for **SIEM (Security Information and Event Management)**.  
It includes two virtual machines: a **Kali Linux attacker VM** and a **vulnerable Windows 11 victim VM**.

The lab demonstrates:
- Log collection from Windows events
- Vulnerability configuration for testing
- Network isolation
- Kibana dashboards for visualization
- Basic attack simulation with SIEM notifications

This is a **beginner-friendly setup** to learn threat detection and response using free tools like **UTM** for virtualization and **Elastic** for monitoring.

### Key Goals
- Collect and analyze Windows logs in real-time  
- Simulate attacks from Kali to trigger SIEM alerts  
- Build dashboards for professional monitoring  



---

## Prerequisites

**Hardware:**
- MacBook Pro or similar (2016 or newer recommended)
- Minimum **16GB RAM** and **i5/i7 CPU** for smooth VM and Docker performance  
  *(Older 2016 MBP works but may be slow—allocate resources carefully.)*

**Software:**
- macOS 12+ (Monterey or later)
- [UTM app](https://utmapp.com) – Free virtualization app
- [Docker Desktop](https://www.docker.com)
- **CrystalFetch** (for fetching Windows 11 ISO)
- [Kali Linux ISO](https://www.kali.org)
- **Windows 11 Enterprise Evaluation ISO** (from CrystalFetch or [Microsoft Eval Center](https://www.microsoft.com/en-us/evalcenter/))
- [Elastic Winlogbeat ZIP](https://www.elastic.co/downloads/beats/winlogbeat)



> **Note:** Keep everything isolated—no real internet exposure to VMs to avoid risks.

---

## Project Details

### 1. Set Up Virtual Machines Using UTM

Use **UTM** to create two VMs: **Kali (attacker)** and **Windows 11 (victim)**.

#### Kali Linux VM
1. Download Kali ISO from [kali.org](https://www.kali.org).
2. In **UTM**:  
   `+ > New VM > Virtualize > Linux > Attach Kali ISO`
3. Configure:
   - CPU: 1–2 cores  
   - Memory: 2GB RAM  
   - Storage: 20GB  
   - Network: Shared Network (e1000) for setup, then switch to **Emulated VLAN** for isolation  
4. Start and install Kali (default settings, set password)

#### Windows 11 VM
1. Install **CrystalFetch** on Mac.  
2. In CrystalFetch:  
   `Windows 11 > Intel x86 > Latest build > Edition: Windows Enterprise`
3. Build and save ISO.  
4. In **UTM**:  
   `New VM > Virtualize > Windows > Attach ISO`
5. Configure:
   - CPU: 2 cores  
   - Memory: 4GB RAM  
   - Storage: 64GB  
   - Network: Shared Network for setup, then Emulated VLAN for isolation  
6. Start and install Windows (skip product key).  
   Set weak user: `labuser / password123`

---

### 2. Isolate the VMs on the Network

To prevent risks, use **UTM’s Emulated VLAN** for internal communication only:

1. Edit both VMs → Network tab → Change to **Emulated VLAN**  
2. Save and restart both  
3. Confirm isolation:
   - From Kali: `ping <Windows_IP>`  
   - Windows IP: Run `ipconfig` in CMD  
   - Should communicate internally, no external internet access

---

### 3. Set Up the Elastic Stack (ELK) on Mac

1. Install **Docker Desktop**  
2. In repo directory, create:
   - `docker-compose.yml`
   - `logstash.conf`  
   *(See `configs/` folder – replace `[REDACTED]` values)*
3. Run:
   ```bash
   docker compose up -d

4. Access Kibana:

Access Kibana at: [http://localhost:5601](http://localhost:5601)  

**Login:** `elastic / <your_password>`  

Generate passwords if needed (see `docs/setup.md`).

---

###  4. Install and Configure Winlogbeat on Windows VM

1. Download **Winlogbeat** from [elastic.co/downloads/beats/winlogbeat](https://www.elastic.co/downloads/beats/winlogbeat)  
2. Unzip to: `C:\Winlogbeat`  
3. Edit `winlogbeat.yml` (run Notepad as **Administrator**):


`winlogbeat.event_logs:
  - name: Application
  - name: System
  - name: Security

output.elasticsearch:
  hosts: ["<your_mac_ip>:9200"]
  username: "elastic"
  password: "<your_password>"
  ssl.verification_mode: none`
  
4. In PowerShell (Admin):

`cd C:\Winlogbeat
.\install-service-winlogbeat.ps1
Start-Service winlogbeat
.\winlogbeat.exe setup -e`

### 5. Make Windows Machine Vulnerable

To simulate attacks, make the VM intentionally weak (see docs/vulnerable-setup.md):

Disable Defender:
Settings → Update & Security → Windows Security → Virus & Threat Protection → Off

Disable Firewall:
Settings → Firewall → Off for all networks

Disable Windows Updates

Enable Remote Desktop (RDP) and SMBv1

Add weak user accounts (password123 or blank passwords)
