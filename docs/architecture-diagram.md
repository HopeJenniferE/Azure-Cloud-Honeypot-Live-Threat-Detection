 Architecture Diagram

## Azure Cloud Honeypot Infrastructure

                                    INTERNET
                                       |
                                       | (Attacks from global threat actors)
                                       |
                                       ↓
                            ┌──────────────────────┐
                            │   Azure Cloud        │
                            │                      │
                            │  ┌────────────────┐  │
                            │  │  Honeypot-VM   │  │ ← Port 22 (SSH) OPEN
                            │  │  Ubuntu 24.04  │  │
                            │  │  10.0.0.4      │  │
                            │  │                │  │
                            │  │  Wazuh Agent   │  │
                            │  └────────┬───────┘  │
                            │           │          │
                            │           │ Sends logs
                            │           │          │
                            │           ↓          │
                            │  ┌────────────────┐  │
                            │  │  Wazuh Server  │  │
                            │  │  Ubuntu 24.04  │  │
                            │  │  10.0.0.4      │  │
                            │  │                │  │
                            │  │  Wazuh Manager │  │
                            │  │  Dashboard     │  │
                            │  └────────────────┘  │
                            │                      │
                            └──────────────────────┘
                                       │
                                       ↓
                            Security Analyst
                            (Monitors the incidents via Web Dashboard)
                            https://20.105.18.197

## Data Flow                           
                  
1. Attackers: Scans the iinternet for vulnerable systems
2. Honeypot-VM: Exposed with SSH port 22 open
3. Attack attempts: Captured in auhtenticatioh log (/var/log/auth.log)
4. Wazuh Agent: Forwards logs to Wazuh Manager
5. Wazuh-Server: Analyzes, correlates, alerts
6. Dashboard: Monitors, review aand analyzes incidents in real tie.

## Key Components

- Honeypot VM: Intentionally vulnerable Ubuntu server
- Wazuh Agent: Collects and forwards security events  
- Wazuh Manager: SIEM engine for analysis
- Network Security Groups: Firewall rules (port 22, 1514, 1515)
- Dashboard: Web interface for monitoring, reviewing and analysis.
