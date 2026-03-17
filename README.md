# Deploying Splunk on Ubuntu Server

![40Admin](Splunk/40Admin.png)

## Overview  
Splunk is a platform for ingesting, searching, and visualizing machine data. It delivers near real-time visibility, making it well-suited for network monitoring and security analysis.  
This walkthrough shows how to install and configure Splunk in a home lab, including forwarding logs from a Dream Machine router to review IPS events and authentication activity.

## Why Ubuntu Server?  
Ubuntu Server is optimized for server workloads and runs without a GUI, which keeps resource usage low—ideal for headless services like Splunk.

![1UbuntuSplunk](Splunk/1UbuntuSplunk.png)

---

## Step 1: Create a Virtual Machine (VirtualBox)

1. Open VirtualBox and select **New**.  

![2VM](Splunk/2VM.png)

2. Provide a name, choose a location, and attach the Ubuntu Server ISO.  

![3Name](Splunk/3Name.png)

3. Skip **Unattended Installation**.  

4. Allocate resources (adjust as needed):  
   - **RAM**: 6–8 GB  
   - **CPU**: 2–4 cores  
   - **Disk**: ~100 GB  

![specs](Splunk/4specs.png)

---

## Step 2: Install Ubuntu Server

1. Boot the VM and choose **Install Ubuntu Server** from GRUB.  

![GRUB](Splunk/5GRUB.png)

2. Create your user and password.  

![profile](Splunk/6profile.png)

3. Enable or skip SSH based on preference.  

![7openssh](Splunk/7openssh.png)

4. Finish installation and **Reboot Now**.  

![8none](Splunk/8none.png)
![9reboot](Splunk/9reboot.png)

5. If a failure notice appears, press **Enter** to continue.  

![10failed](Splunk/10failed.png)

---

## Step 3: Update the System

1. Log in, then run:  
```bash
sudo apt update && sudo apt upgrade -y
```

![11update](Splunk/11update.png)

---

## Step 4: Configure a Static IP

1. Check your interface and IP:  
```bash
ip a
```
Use **Bridged** for same-network access or **NAT** for isolation.

2. Edit Netplan:  
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

3. Example config (adjust IPs):  
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

![1staticip](Splunk/12.1staticip.png)

If your router (e.g., UniFi Gateway) handles static leases, you can configure it there instead.

![12staticip](Splunk/12staticip.png)

4. Apply changes:  
```bash
sudo netplan apply
```

![13netplan](Splunk/13netplan.png)

5. Verify again with `ip a`.

---

## Step 5: Install Splunk

1. Download Splunk (Linux .deb) from the official site.  

![14Splunkdownloading](Splunk/14Splunkdownloading.png)

2. Choose **Linux** and copy the `wget` link.  

![15download](Splunk/15download.png)

3. If SSH wasn’t installed, install it:  
```bash
sudo apt install openssh-server
```

4. Verify and enable SSH:  
```bash
sudo systemctl status ssh
sudo systemctl enable ssh
```

5. Configure firewall (UFW):  
```bash
sudo ufw status
sudo ufw allow ssh
sudo ufw allow 10154/udp
sudo ufw reload
```

![16ssh](Splunk/16ssh.png)

6. Connect via SSH:  
```bash
ssh <username>@<ubuntu-ip>
```

7. Download Splunk with your copied `wget` link.  

![17installsplunk](Splunk/17installsplunk.png)

8. Install the package:  
```bash
sudo dpkg -i <splunk-deb-file>
```

![18dpkg](Splunk/18dpkg.png)

9. Verify installation path:  
```bash
cd /opt/splunk
ls -la
```

10. Start Splunk as the splunk user:  
```bash
sudo -u splunk bash
cd bin
./splunk start
```

![19splunkstart](Splunk/19splunkstart.png)

Set admin credentials when prompted.

11. Enable startup on boot:  
```bash
cd /opt/splunk/bin/
sudo ./splunk enable boot-start -user splunk
```

![20splunkenable](Splunk/20splunkenable.png)

12. Access Splunk:  
```
http://<ip>:8000
```

![21splunk](Splunk/21splunk.png)

---

## Step 6: Send Data to Splunk

### Data Inputs Overview  
Splunk requires incoming data (logs/metrics) to index and analyze. Here we’ll ingest logs from a Dream Machine router, including IPS alerts and login events.

### Configure Router (Syslog Forwarding)
- Set Splunk server IP as the **SIEM/syslog destination**  
- Use a custom port like **10154**

![1udp](Splunk/26.1udp.png)

### Configure Splunk Input
- Go to **Settings → Data Inputs → UDP**  
- Add input on port `10154`  
- Set source type: `syslog`  

![24input](Splunk/24input.png)
![25addnew](Splunk/25addnew.png)

### Input Settings
- Port: `10154`  
- Source type: `syslog`  
- App context: e.g., `Unifi`  
- Index: create `ubnt`  

![26udp](Splunk/26udp.png)
![27settings](Splunk/27settings.png)

### Search Data
```spl
index=ubnt
```

![31splunk](Splunk/31splunk.png)

### Time Zone
Set under **Settings → Users** to align timestamps.

![29users](Splunk/29users.png)
![30time](Splunk/30time.png)

---

## Step 7: Field Extraction

1. Use **Extract New Field** from search results.  

![33Admin](Splunk/33Admin.png)

2. Define regex for parsing logs.  

![34Admin](Splunk/34Admin.png)

3. Example:
```regex
\|(?P<Admin_Activity>user[^|]+)\|
```

![36Admin](Splunk/36Admin.png)

4. Test:
```spl
index=ubnt Admin_Activity="user"
```

---

## Step 8: Build a Dashboard

1. Run query:
```spl
index=ubnt | dedup _time | stats count by Admin_Activity
```

![37aadmin](Splunk/37Aadmin.png)

2. Choose a visualization (e.g., Pie Chart).  

![38Aadmin](Splunk/38Aadmin.png)

3. Save to a new dashboard with a clear title and description.

4. Enable **Global Time Range** for dynamic filtering.

![Splunk/39Admin.png](Splunk/39Admin.png)

---

## Summary
You now have Splunk running on Ubuntu, ingesting syslog data, extracting fields, and visualizing activity. This lab can be expanded with more data sources, alerts, and dashboards for deeper security monitoring.
