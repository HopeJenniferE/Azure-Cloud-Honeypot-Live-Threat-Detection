# Azure Honeypot Deployment Guide

Step-by-step instructions for deploying a cloud-native honeypot with Wazuh SIEM monitoring.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Phase 1: Azure Infrastructure](#phase-1-azure-infrastructure-setup)
3. [Phase 2: Wazuh Server Installation](#phase-2-wazuh-server-installation)
4. [Phase 3: Honeypot Configuration](#phase-3-honeypot-configuration)
5. [Phase 4: Verification](#phase-4-verification)
6. [Phase 5: Monitoring](#phase-5-monitoring-real-attacks)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

**Required:**
- Microsoft Azure account with active subscription
- Basic Linux command-line knowledge
- SSH client (Terminal on Mac/Linux, PuTTY on Windows)
- Understanding of networking concepts (ports, firewalls, IP addresses)

**Estimated Time:** 2-3 hours  
**Estimated Cost:** ~$20-50/month for 2 VMs (Standard_B2s)(To mininmize spend, deallocate VMs when not actively in use or testing)

---

## Phase 1: Azure Infrastructure Setup

### Step 1.1 Create Resource Group

1. Log in to **Azure Portal** (portal.azure.com)
2. Navigate to **Resource Groups** in the left menu
3. Click **+ Create**
4. Configure:
   - **Subscription:** Select your subscription
   - **Resource group name:** `SOC-LAB`
   - **Region:** Select closest region to you
5. Click **Review + Create** → **Create**

---

### Step 1.2 Deploy Wazuh Server VM

1. Navigate to **Virtual Machines** → **+ Create** → **Azure virtual machine**

2. **Basics Tab:**
   - **Resource Group:** SOC-LAB
   - **Virtual machine name:** `wazuh-server`
   - **Region:** Same as resource group
   - **Availability options:** No infrastructure redundancy required
   - **Image:** Ubuntu Server 24.04 LTS - x64 Gen2
   - **Size:** Standard_B2s (2 vCPUs, 4 GB RAM) - Click "See all sizes" if needed (I'll recommend you choose 8GB RAM for more size)
   - **Authentication type:** SSH public key
   - **Username:** `azureuser`
   - **SSH public key source:** Generate new key pair
   - **Key pair name:** `wazuh-server_key`

3. **Disks Tab:**
   - **OS disk type:** Standard SSD (locally-redundant storage)

4. **Networking Tab:**
   - **Virtual network:** Create new → Name: `honeypot-vnet`
   - **Subnet:** default (10.0.0.0/24)
   - **Public IP:** Create new
   - **NIC network security group:** Basic
   - **Public inbound ports:** Allow selected ports
   - **Select inbound ports:** SSH (22)

5. **Management Tab:**
   - **Enable auto-shutdown:** Optional (recommended to save costs)

6. Click **Review + Create** → **Create**

7. **IMPORTANT:** Download the SSH private key when prompted
   - Save as `wazuh-server_key.pem` in your Downloads folder

8. Wait for deployment to complete (2-3 minutes)

---

### Step 1.3 Deploy Honeypot VM

Repeat the same process with these changes:

1. Navigate to **Virtual Machines** → **+ Create**

2. **Basics Tab:**
   - **Virtual machine name:** `honeypot-vm`
   - **Use SAME region** as wazuh-server
   - **Size:** Standard_B1s (1 vCPU, 1 GB RAM) - Smaller is fine for honeypot
   - **Key pair name:** `honeypot-vm_key`

3. **Networking Tab:**
   - **Virtual network:** Select **existing** `honeypot-vnet` (same as Wazuh server)
   - **Subnet:** default

4. Download the SSH key: `honeypot-vm_key.pem`

---

## Phase 2: Wazuh Server Installation

### Step 2.1 Connect to Wazuh Server

**On your Mac/Linux terminal:**
```bash
# Navigate to Downloads
cd ~/Downloads

# Set correct permissions on SSH key
chmod 400 wazuh-server_key.pem

# Get the public IP from Azure Portal (wazuh-server → Overview → Public IP address)
# Replace <WAZUH_PUBLIC_IP> with your actual IP

# SSH into Wazuh server
ssh -i wazuh-server_key.pem azureuser@<WAZUH_PUBLIC_IP>


**First time connecting:** Type `yes` when asked about authenticity

---

### Step 2.2 Install Wazuh All-in-One

Once connected to the Wazuh server:
--In your Wazuh-server terminal

# Update system packages
sudo apt update && sudo apt upgrade -y

# Download Wazuh installation script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Run installation (takes 5-10 minutes)
sudo bash wazuh-install.sh -a


**CRITICAL:** At the end of installation, you'll see:

INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-ip>
    User: admin
    Password: <WAZUH GENERATED PASSWORD>
```

**COPY AND SAVE THIS PASSWORD!** You'll need it to access the dashboard.

---

### Step 2.3 Configure Azure Firewall for Wazuh

In **Azure Portal:**

1. Go to **wazuh-server** VM → **Networking** → **Network settings**
2. Click **Create port rule** → **Inbound port rule**

**Rule 1: HTTPS for Dashboard** (If not aready created automatically)
- **Source:** Any
- **Source port ranges:** *
- **Destination:** Any
- **Service:** Custom
- **Destination port ranges:** `443`
- **Protocol:** TCP
- **Action:** Allow
- **Priority:** 320
- **Name:** `HTTPS`
- Click **Add**

**Rule 2: Wazuh Agent Communication**
- **Source:** Any (or IP Addresses: `10.0.0.0/24` for more security) (If by chance both your VMs have same private IP you can use the public IP)
- **Source port ranges:** *
- **Destination:** Any
- **Service:** Custom
- **Destination port ranges:** `1514,1515`
- **Protocol:** TCP
- **Action:** Allow
- **Priority:** 310
- **Name:** `Wazuh-Agent-Communication`
- Click **Add**

---

### Step 2.4 Access Wazuh Dashboard

1. Open your browser
2. Go to: `https://<WAZUH_PUBLIC_IP>`
3. You'll see a security warning (self-signed certificate) - Click **Advanced** → **Proceed**
4. Login:
   - **Username:** `admin`
   - **Password:** (the password you saved from installation, in the wazuh terminal)

5. You should see the Wazuh dashboard!

---

## Phase 3: Honeypot Configuration

### Step 3.1 Connect to Honeypot VM

**Open a NEW terminal tab** (keep Wazuh server connection open):
 (--In ther honeypot-vm command terminal)
cd ~/Downloads

chmod 400 honeypot-vm_key.pem

ssh -i honeypot-vm_key.pem azureuser@<HONEYPOT_PUBLIC_IP>
```

---

### Step 3.2 Install Wazuh Agent
(Command Terminal)
# Add Wazuh repository GPG key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg

# Add Wazuh repository
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list

# Update package list
sudo apt-get update

# Install Wazuh agent
sudo apt-get install wazuh-agent -y
```

---

### Step 3.3 Configure Agent to Connect to Manager
(Command Terminal)
# Edit configuration file
sudo nano /var/ossec/etc/ossec.conf
```

**Find this section (near the top):**
```xml
<client>
  <server>
    <address>MANAGER_IP</address>
```

**Change `MANAGER_IP` to your Wazuh server's PUBLIC IP:**
```xml
<client>
  <server>
    <address>20.105.18.197</address>
```

(Replace with YOUR actual Wazuh server public IP)

**Save and exit:**
- Press `Ctrl + X`
- Press `Y`
- Press `Enter`

---

### Step 3.4 Start Wazuh Agent
```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable agent to start on boot
sudo systemctl enable wazuh-agent

# Start the agent
sudo systemctl start wazuh-agent

# Check status (should show "active (running)")
sudo systemctl status wazuh-agent
```

**Look for:** `Active: active (running)` in green

**Check connection logs:**
```bash
sudo tail -20 /var/ossec/logs/ossec.log
```

**You should see:** `INFO: Connected to the server ([IP]:1514/tcp).`

**If you see version mismatch errors, see Troubleshooting section below.**

---

### Step 3.5 Expose Honeypot to Internet

**In Azure Portal:**

1. Go to **honeypot-vm** → **Networking** → **Network settings**
2. The SSH rule (port 22) should already exist
3. **Verify it allows from "Any" source** - this makes your honeypot discoverable by attackers

If SSH rule doesn't exist or is restricted:
- Click **Create port rule** → **Inbound**
- **Source:** **Any** (this is intentional, as we want attackers to find it!)
- **Destination port ranges:** `22`
- **Protocol:** TCP
- **Action:** Allow
- **Priority:** 300
- **Name:** `SSH`

--IMPORTANT **This makes your honeypot vulnerable - that's the point!**

---

## Phase 4: Verification

### Step 4.1 Verify Agent in Wazuh Dashboard

1. Go to Wazuh Dashboard: `https://<WAZUH_PUBLIC_IP>`
2. You should see the list of agents and the click on the numbers if not, then click **Menu** (hamburger icon) → **Server management** → **Endpoints Summary**
3. You should see:
   - **Active agents: 1**
   - **honeypot-vm** listed with status **Active** (green dot)

**If agent is not showing:**
- Wait 2-3 minutes for registration
- Check agent logs: `sudo tail -f /var/ossec/logs/ossec.log`
- Verify firewall rules allow ports 1514-1515
- See Troubleshooting section

---

### Step 4.2 Test Attack Detection (Optional)

From **your local Mac terminal (If using a mac, if not just a fresh new terminal)** (not connected to any VM):
(Command Terminal)
# Try to SSH with a fake username
ssh fakeuser@<HONEYPOT_PUBLIC_IP>

# When it asks for password, type anything random
# Press Enter, repeat 3-5 times
```

**In Wazuh Dashboard:**
1. Click on **honeypot-vm** agent
2. Go to **Security events** tab
3. Within 1-2 minutes, you should see:
   - `"sshd: Attempt to login using a non-existent user"`

✅ **If you see this, your honeypot is working!**

---

## Phase 5: Monitoring Real Attacks

### Step 5.1 Wait for Attacks

- **Timeline:** Attacks typically begin within **few minutes-6 hours** of exposure
- **Why:** Automated bots continuously scan the entire internet for open SSH ports
- **No action needed:** Just wait and monitor the dashboard

### Step 5.2 What to Monitor

**In Wazuh Dashboard:**

1. **Modules** → **Security Events**
   - Filter by agent: honeypot-vm
   - Look for authentication failures

2. **Modules** → **MITRE ATT&CK**
   - See which attack techniques are being used
   - View threat actor groups with matching patterns

3. **Modules** → **PCI DSS**
   - View compliance requirement violations
   - See attack volume over time

### 5.3 Watch Live Logs (Optional)

**SSH into honeypot and run:**
(Command Terminal)
# Watch authentication attempts in real-time
sudo tail -f /var/log/auth.log
```

You'll see failed login attempts as they happen!

Press `Ctrl+C` to stop watching.

---

## Troubleshooting (Just a few blocks I ran into incase you do too)

### Issue 1: Agent Version Mismatch

**Error:** `"Agent version must be lower or equal to manager version"`

**Solution:**
(VM Command Terminal)
# On Wazuh SERVER (not honeypot):
sudo apt-get update
sudo apt-get install --only-upgrade wazuh-manager -y

# Check version
sudo /var/ossec/bin/wazuh-control info | grep VERSION

# Restart manager
sudo systemctl restart wazuh-manager
```

Then restart agent on honeypot:
(VM command Terminal)
sudo systemctl restart wazuh-agent
```

---

### Issue 2: Agent Won't Connect

**Error:** `"Unable to connect to enrollment service at [IP]:1515"`

**Check:**

1. **Firewall rules on Wazuh server:**
   - Azure Portal → wazuh-server → Networking
   - Verify ports 1514 and 1515 are open

2. **Agent configuration:**
(VM Command Terminal)
   sudo nano /var/ossec/etc/ossec.conf
```
   - Verify `<address>` has correct Wazuh server IP

3. **Network connectivity:**
(VM Command Terminal)
   # From honeypot, test connection
   telnet <WAZUH_SERVER_IP> 1514
```

---

### Issue 3: Dashboard Won't Load

**Check:**
1. Port 443 is open in Azure NSG
2. You're using `https://` not `http://`
3. Wazuh manager is running:
(In VM command Terminal)
   sudo systemctl status wazuh-manager
```

---

### Issue 4: No Attacks Showing

**Wait longer:** Attacks can take 2-6 hours to start

**Verify exposure:**
1. Azure Portal → honeypot-vm → Networking
2. SSH port 22 should be open from "Any"

**Test manually:** Try failed SSH logins from your computer (see Phase 4.2)

---

## Post-Project: Cleanup & Cost Management

### Stop VMs (Keep for Later)

# In Azure Portal:
# Go to each VM → Click "Stop"
# This stops billing for compute, but keeps storage
```

### Delete Everything (Permanent)

# In Azure Portal:
# Go to Resource Groups → SOC-LAB → Delete resource group
# Type the resource group name to confirm
```

VERY IMPORTANT **Deleting is permanent - make sure you've saved all screenshots and data first!**

---

## Summary

You've now successfully:
A. Deployed Azure cloud infrastructure  
B. Installed Wazuh SIEM server  
C. Configured honeypot with Wazuh agent  
D. Exposed system to capture real attacks  
E. Set up monitoring dashboard  

**Next:** Wait for attacks, document findings, analyze patterns.

---

## Additional Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Azure Virtual Machines Docs](https://docs.microsoft.com/en-us/azure/virtual-machines/)
- [SSH Best Practices](https://www.ssh.com/academy/ssh/best-practices)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

*This guide documents the actual deployment process used in this project, including real challenges encountered and their solutions.*
Screenshots of my actual setup are available in the `/screenshots` folder.*

**Note:** This deployment used pre-existing SSH keys and virtual networks from my previous setup. For first-time deployments, Azure will generate new keys automatically.
