# Splunk Installation and Configuration on Ubuntu Server

## Overview
Splunk is a leading platform for collecting, analyzing, and visualizing machine data. It enables real-time monitoring and is widely used for security analysis and operational insights.

This guide walks through setting up Splunk in a home lab environment. The setup includes forwarding logs from a network device (e.g., Dream Machine router) to monitor IPS events and authentication activity.

---

## Why Use Ubuntu Server?
Ubuntu Server is a lightweight operating system optimized for server workloads. It runs without a graphical interface, reducing resource usage and making it ideal for running services like Splunk.

---

## Step 1: Create a Virtual Machine

1. Open VirtualBox and create a new VM.
2. Assign a name, choose a storage location, and attach the Ubuntu Server ISO.
3. Skip unattended installation.
4. Allocate resources:
   - RAM: 6–8 GB
   - CPU: 2–4 cores
   - Storage: ~100 GB

---

## Step 2: Install Ubuntu Server

1. Start the VM and select “Install Ubuntu Server”.
2. Configure user credentials.
3. Optionally enable SSH during setup.
4. Complete installation and reboot.

---

## Step 3: Update the System

After logging in, run:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 4: Configure a Static IP

1. Identify your network interface:

```bash
ip a
```

2. Edit Netplan configuration:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

3. Example configuration:

```yaml
network:
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.x.x/24
      gateway4: 192.168.x.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
  version: 2
```

4. Apply changes:

```bash
sudo netplan apply
```

---

## Step 5: Install Splunk

1. Download Splunk from the official site (Linux .deb package).
2. (Optional) Install SSH:

```bash
sudo apt install openssh-server
```

3. Enable SSH:

```bash
sudo systemctl enable ssh
```

4. Configure firewall:

```bash
sudo ufw allow ssh
sudo ufw allow 10154/udp
```

5. Install Splunk:

```bash
sudo dpkg -i splunk.deb
```

6. Start Splunk:

```bash
sudo -u splunk /opt/splunk/bin/splunk start
```

7. Enable auto-start:

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
```

8. Access Splunk via browser:

```
http://<server-ip>:8000
```

---

## Step 6: Send Logs to Splunk

### Configure Router
- Set Splunk server IP as syslog destination
- Use UDP port (e.g., 10154)

### Configure Splunk Input
- Go to Settings > Data Inputs
- Add UDP input on port 10154
- Set source type: syslog
- Create a custom index (e.g., ubnt)

### Search Logs

```spl
index=ubnt
```

### Set Time Zone
Adjust user time zone under Settings > Users to ensure accurate timestamps.

---

## Step 7: Field Extraction

1. Navigate to search results and select “Extract New Field”.
2. Use regex to capture data. Example:

```regex
\|(?P<Admin_Activity>user[^|]+)\|
```

3. Test extraction:

```spl
index=ubnt Admin_Activity="user"
```

---

## Step 8: Build a Dashboard

1. Run query:

```spl
index=ubnt | stats count by Admin_Activity
```

2. Visualize results (e.g., Pie Chart).
3. Save to a new dashboard.
4. Enable global time range for dynamic filtering.

---

## Conclusion
You now have a working Splunk setup capable of ingesting and analyzing network logs. This environment can be extended with additional data sources, dashboards, and detection use cases for deeper security insights.
