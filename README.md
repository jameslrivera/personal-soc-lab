# Personal SIEM & Threat Monitoring Lab

A hands-on security operations center (SOC) lab that demonstrates end-to-end log and network data collection, visualization, and alerting using the Elastic Stack.

##  Overview

- **Elasticsearch & Kibana**: central storage & dashboards  
- **Filebeat**: ships system logs (macOS/Linux/Windows)  
- **Packetbeat**: captures and summarizes network traffic (HTTP, DNS, TLS)  
- **Next Steps**: integrate Suricata IDS, threat-intel enrichment, and automated response playbooks

## Features

- Live, searchable log index in Elasticsearch  
- Pre-built dashboards for host health & network activity  
- Simple YAML configs and Docker Compose for rapid deployment  
- Easily extensible to add more Beats, IDS data, and alert rules

## Prerequisites

- Docker & Docker Compose installed  
- Homebrew (**macOS**) or Chocolatey (**Windows**) to install Filebeat & Packetbeat  
- Git

## 🚀 Quick Start

1. **Clone this repo**  
   ```bash
   git clone git@github.com:jameslrivera/personal-soc-lab.git
   cd personal-soc-lab
