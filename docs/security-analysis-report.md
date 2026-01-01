# Security Analysis Report
## Azure Cloud Honeypot: Live Threat Detection & Analysis

**Author:** Hope Jennifer  
**Date:** December 28-29, 2025  
**Project Duration:** 24 hours of active monitoring  
**Environment:** Microsoft Azure Cloud Infrastructure  

---

## Executive Summary

This report documents a security research project involving the deployment of an intentionally vulnerable honeypot system in Microsoft Azure to capture and analyze real-world cyber attacks. The honeypot successfully attracted **5,181 authentication-related attack attempts** within the first 24 hours of exposure to the internet.

### Key Findings:
- **Total Security Events:** 5,181+ failed authentication attempts
- **Peak Attack Period:** 18:00 UTC (165 events per 30-minute window)
- **Attack Vectors:** SSH brute-force attacks, credential stuffing
- **Threat Intelligence:** Attack patterns matched techniques used by nation-state APT groups including APT1 and Andariel
- **Geographic Distribution:** Attacks originated from multiple countries globally
- **Compliance Impact:** Multiple PCI DSS requirement violations detected (10.2.4, 10.2.5, 10.6.1)

**Risk Assessment:** The volume and sophistication of attacks demonstrates that exposed systems are continuously targeted by automated attack infrastructure within hours of deployment.

*Visual evidence and screenshots are available in the `/screenshots` directory of this repository.*

---

## 1. Project Objectives

### Primary Goals:
1. Deploy a cloud-native honeypot to capture real-world attack data.
2. Implement SIEM (Security Information and Event Management) monitoring using Wazuh.
3. Analyze attack patterns and map to industry frameworks (MITRE ATT&CK, PCI DSS)
4. Document threat intelligence findings
5. Demonstrate security operations capabilities

### Key achievements:
A. Functional honeypot exposed to internet  
B. Real-time security monitoring operational  
C. Capture and qunatify security events. 
D. Idnetify threats and mitigation strategies  
E. Document findings professionally

---

## 2. Infrastructure Architecture

### 2.1 Cloud Environment Overview

**Cloud Platform:** Microsoft Azure  
**Region:** Europe (North Europe for Wazuh, West Europe for Honeypot)  
**Virtual Network:** 10.0.0.0/24  

### 2.2 System Components

#### Honeypot VM
- **Hostname:** honeypot-vm
- **OS:** Ubuntu 24.04 LTS
- **VM Size:** Standard_B2ts_v2 (2 vCPUs, 1 GB RAM)
- **Private IP:** 10.0.0.4
- **Public IP:** 172.199.176.162
- **Exposed Services:** SSH (Port 22)
- **Purpose:** Intentionally vulnerable system to attract attacks

#### Wazuh SIEM Server
- **Hostname:** wazuh-server
- **OS:** Ubuntu 24.04 LTS
- **VM Size:** Standard_B2s_v2 (2 vCPUs, 8 GB RAM)
- **Private IP:** 10.0.0.4
- **Public IP:** 20.105.18.197
- **Services:** Wazuh Manager 4.14.1, OpenSearch Dashboard
- **Purpose:** Centralized security monitoring and analysis

#### Network Security
- **Azure Network Security Groups (NSG):** Configured to allow:
  - SSH (22/TCP) - Open to internet on honeypot
  - Wazuh agent communication (1514-1515/TCP)
  - HTTPS (443/TCP) - Dashboard access

*See architecture diagram in `/docs/architecture-diagram.md`*

---

## 3. Methodology

### 3.1 Deployment Process
1. Deployed two Ubuntu VMs in Azure with pre-existing SSH keys
2. Installed Wazuh Manager on monitoring server
3. Deployed Wazuh agent on honeypot
4. Configured NSG rules to expose honeypot SSH service
5. Verified agent connectivity and log forwarding
6. Monitored attack activity via Wazuh dashboard

### 3.2 Monitoring Period
- **Start:** December 28, 2025
- **End:** December 29, 2025
- **Duration:** 24+ hours continuous monitoring

### 3.3 Data Collection
- Real-time log analysis of authentication attempts
- Failed login event correlation
- IP address geolocation tracking
- Attack pattern identification
- MITRE ATT&CK technique mapping

---

## 4. Attack Analysis

### 4.1 Volume Statistics

| Metric | Count |
|--------|-------|
| Total Security Events | 5,181 |
| Failed Authentication Attempts | 5,181 |
| Peak Activity (30-min window) | 165 events |
| Average Events Per Hour | ~123 |

*Screenshots of attack volume dashboards available in `/screenshots` folder.*

### 4.2 Attack Timeline

**Initial Detection:** Within 5 minutes of honeypot deployment  
**Peak Activity:** 18:00 UTC  
**Attack Pattern:** Continuous bot activity with irregular spikes

The time-series analysis revealed consistent attack activity with a significant spike during evening hours (UTC), suggesting coordination with business hours in specific geographic regions.

### 4.3 Attack Techniques Observed

#### SSH Brute-Force Attacks
**Description:** Attackers attempted to gain unauthorized access by systematically trying multiple username/password combinations.

**Common Usernames Targeted:**
- `root` - System administrator account
- `admin` - Generic administrative account
- `user` - Default user account
- `test` - Test/development account
- `oracle` - Database service account
- `fakeuser` - Non-existent accounts (reconnaissance)

**Attack Pattern:**
1. Initial reconnaissance scan
2. Service identification (SSH on port 22)
3. Automated credential stuffing
4. Repeated login attempts with common passwords

**Example Log Entry:**
```
sshd: Attempt to login using a non-existent user fakeuser from 51.14.27.93 port 60115
```

*Screenshots of actual attack logs available in repository.*

### 4.4 MITRE ATT&CK Framework Mapping

Observed attack techniques were mapped to the MITRE ATT&CK framework:

| Technique ID | Technique Name | Description |
|--------------|----------------|-------------|
| T1110.001 | Brute Force: Password Guessing | Systematic password attempts |
| T1110.003 | Brute Force: Password Spraying | Same password across multiple accounts |
| T1021.004 | Remote Services: SSH | Exploitation of SSH service |
| T1078 | Valid Accounts | Attempts to use legitimate account names |
| T1595.002 | Active Scanning: Vulnerability Scanning | Port and service enumeration |

### 4.5 Threat Actor Attribution

Wazuh's threat intelligence module mapped observed techniques to known Advanced Persistent Threat (APT) groups:

**Groups with Matching Techniques:**

1. **APT1 (G0006)** - Chinese military cyber espionage unit (PLA Unit 61398)
   - Known for SSH brute-force campaigns
   - Targets: Government, defense, technology sectors

2. **Andariel (G0138)** - North Korean state-sponsored group
   - Conducts destructive attacks and financial theft
   - Uses credential access techniques matching our observations

3. **Ajax Security Team (G0130)** - Iranian cyber espionage group
   - Active since 2010
   - Targets US defense industrial base

4. **admin@338 (G0018)** - China-based threat group
   - Targets financial and economic organizations
   - Uses similar initial access techniques

**Important Note:** This attribution indicates that attack *techniques* match those used by these groups, not that these specific groups targeted this honeypot.

*MITRE ATT&CK mapping screenshots available in `/screenshots`.*

### 4.6 Geographic Analysis

Attacks originated from multiple countries, consistent with distributed botnet networks commonly used for brute-force attacks.

**Typical Attack Sources:**
- Asia-Pacific region
- Eastern Europe
- South America
- Distributed cloud infrastructure

---

## 5. Compliance & Security Standards

### 5.1 PCI DSS Requirements Violated

The Payment Card Industry Data Security Standard (PCI DSS) provides security benchmarks. The following requirements were triggered:

| Requirement | Description | Events |
|-------------|-------------|---------|
| 10.2.4 | Invalid logical access attempts | 2,953 |
| 10.2.5 | Use of identification and authentication mechanisms | 2,953 |
| 10.6.1 | Review of security events and logs | Continuous |

**Compliance Implications:**  
In a production environment processing payment card data, these violations would require:
- Immediate incident response
- Log review and analysis
- Implementation of access controls
- Multi-factor authentication
- Rate limiting and account lockout policies

*PCI DSS compliance dashboard screenshots available in repository.*

### 5.2 Security Control Gaps

The honeypot intentionally lacked standard security controls to observe attack behavior:

**Missing Controls:**
- ❌ Multi-factor authentication (MFA)
- ❌ Rate limiting on authentication attempts
- ❌ IP-based access restrictions
- ❌ Fail2ban or similar brute-force protection
- ❌ Strong password policies
- ❌ Account lockout after failed attempts

**Real-World Recommendation:** Production systems should implement ALL of these controls.

---

## 6. Security Event Details

### 6.1 Sample Attack Sequence

**Timestamp:** Dec 28 15:00:10  
**Source IP:** [Anonymized for security]  
**Attack Pattern:**
```
15:00:10 - Connection from [IP]
15:00:11 - Attempted user: root | Result: Failed
15:00:13 - Attempted user: admin | Result: Failed  
15:00:15 - Attempted user: user | Result: Failed
15:00:17 - Attempted user: test | Result: Failed
15:00:19 - Connection closed
```

This pattern repeated from the same source IP every 2-3 minutes, demonstrating automated attack tooling.

### 6.2 Attack Sophistication Analysis

**Level: Medium**

The attacks demonstrated:
-  Automated tooling (consistent timing, systematic approach)
-  Username enumeration (testing common account names)
-  Persistent attempts (repeated connections)
-  No advanced evasion techniques observed
-  No zero-day exploits or sophisticated payloads
-  No lateral movement attempts

**Assessment:** These are automated attacks targeting low-hanging and easy targets rather than sophisticated planned attacks.

---

## 7. Lessons Learned & Recommendations

### 7.1 Key Takeaways

1. **Speed of Attack:** Exposed systems are discovered and attacked within hours
2. **Attack Volume:** Even small systems face thousands of daily attack attempts
3. **Automation Prevalence:** 100% of observed attacks were automated
4. **Default Credentials:** Attackers systematically test common usernames
5. **Monitoring Value:** SIEM tools are essential for visibility into attack activity

### 7.2 Security Recommendations

For organizations operating similar infrastructure:

**1. Access Controls**
- Implement multi-factor authentication on all remote access services
- Use SSH key-based authentication instead of passwords
- Restrict SSH access to known IP addresses or VPN
- Disable root login via SSH

**2. Network Security**
- Deploy fail2ban or similar intrusion prevention tools
- Implement rate limiting on authentication services
- Use non-standard ports for SSH (security through obscurity as additional layer)
- Enable connection throttling (To limit connection attempts to servers)

**3. Monitoring & Detection**
- Deploy SIEM solutions for centralised log management
- Configure real-time alerting for failed authentication attempts
- Establish baseline behavior and alert on anomalies
- Integrate threat intelligence feeds

**4. Incident Response**
- Develop playbooks for brute-force attack response
- Automate IP blacklisting for repeat offenders
- Regularly review and analyze security logs
- Conduct periodic security assessments

**5. Compliance**
- Regularly audit systems against PCI DSS and relevant frameworks
- Document security controls and their effectiveness
- Maintain audit trails for compliance reporting
- Conduct regular security training for staff

---

## 8. Technical Challenges & Solutions

### 8.1 Challenges Encountered

**Challenge 1: Agent Version Mismatch**
- **Issue:** Wazuh agent (v4.14.1) was newer than manager (v4.7.5)
- **Error:** "Agent version must be lower or equal to manager version"
- **Solution:** Upgraded Wazuh manager to v4.14.1
- **Lesson:** Always verify version compatibility before deployment

**Challenge 2: Network Connectivity**
- **Issue:** Agent couldn't connect to manager due to firewall rules
- **Error:** "Unable to connect to enrollment service at [IP]:1515"
- **Solution:** Configured Azure NSG to allow ports 1514-1515
- **Lesson:** Document all required ports during planning phase

**Challenge 3: Private IP Configuration**
- **Issue:** Both VMs in different VNets with identical CIDR blocks
- **Resolution:** Configured agent to use manager's public IP for communication
- **Workaround:** Used public IP addressing for agent-manager communication
- **Lesson:** Plan network architecture carefully in cloud environments

---

## 9. Project Outcomes

### 9.1 Objectives Achieved

**Successfully deployed** cloud-native honeypot infrastructure  
**Captured real-world attack data** exceeding 2,900 events  
**Implemented enterprise SIEM** with Wazuh  
**Mapped attacks to MITRE ATT&CK** framework  
**Generated actionable threat intelligence**  
**Documented findings** in professional security report  

### 9.2 Skills Demonstrated

**Technical Skills:**
- Cloud infrastructure deployment (Azure)
- Linux system administration
- SIEM configuration and management
- Network security (firewalls, NSGs)
- Security log analysis
- Threat intelligence analysis

**Analytical Skills:**
- Attack pattern recognition
- Security event correlation
- Compliance framework mapping
- Risk assessment
- Incident documentation

**Professional Skills:**
- Technical documentation
- Security reporting
- Problem-solving and troubleshooting
- Project planning and execution

---

## 10. Conclusion

This project successfully demonstrated the deployment and operation of a cloud-based security monitoring system. The honeypot captured substantial real-world attack data, providing valuable insights into current threat landscapes and attack methodologies.

**Key Findings:**
- Internet-facing systems are under constant attack from automated infrastructure
- SSH brute-force attacks remain a prevalent threat vector
- Attack patterns match those used by sophisticated threat actors
- SIEM tools are essential for detecting and analyzing security events
- Proper security controls are critical for protecting production systems

**Project Value:**
This hands-on experience provided practical knowledge in:
- Cloud security architecture
- SIEM deployment and operation
- Threat detection and analysis
- Security operations center (SOC) workflows
- Compliance and security frameworks

**Future Enhancements:**
- Extended monitoring over longer periods
- Geographic visualization of attack sources
- Integration with additional threat intelligence feeds
- Automated response mechanisms
- Analysis of seasonal/temporal attack patterns

---

## 11. References & Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [PCI DSS Requirements](https://www.pcisecuritystandards.org/)
- [Azure Security Best Practices](https://docs.microsoft.com/en-us/azure/security/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

---

## Appendix: Technical Specifications

### System Requirements
- Wazuh Manager: 2 vCPUs, 4GB RAM (I used 8GB as using the 4GB gave me an error due to low size/storage)
- Honeypot: 2 vCPUs, 1GB RAM
- Storage: 128GB Premium SSD per VM
- Network: Azure VNet with NSG

### Software Versions
- OS: Ubuntu 24.04 LTS
- Wazuh Manager: 4.14.1
- Wazuh Agent: 4.14.1
- OpenSearch: Included with Wazuh installation

### Visual Evidence
All screenshots referenced in this report are available in the `/screenshots` directory:
- Wazuh dashboard overview
- Security events timeline
- MITRE ATT&CK threat group mappings
- PCI DSS compliance violations
- Attack pattern visualizations
- Azure network configuration
- Failed authentication logs

---

**Report Classification:** Public  
**Sensitive Data:** All IP addresses and credentials sanitized  
**Purpose:** Educational/Portfolio  

---

*This report was created as part of a cybersecurity training project. All systems were deployed in isolated environments for educational purposes only.*
