#  Talha's Cloud Server – ICT171 Project

**Student name:** Talha Arshad  
**Student number:** 35051172  
**Live server IP:** `15.135.223.199`  
**Live domain:** [https://talhatechub.online](https://talhatechub.online)  

---

##  Table of Contents

1. [Project Overview](#project-overview)
2. [Server Specifications](#server-specifications)
3. [Prerequisites](#prerequisites)
4. [Step-by-Step Build Guide](#step-by-step-build-guide)
   - [Step 1: Launch AWS EC2 Instance](#step-1-launch-aws-ec2-instance)
   - [Step 2: Connect to Your Server](#step-2-connect-to-your-server)
   - [Step 3: Install Nginx Web Server](#step-3-install-nginx-web-server)
   - [Step 4: Add Your Website Files](#step-4-add-your-website-files)
   - [Step 5: Set Up Domain DNS Records](#step-5-set-up-domain-dns-records)
   - [Step 6: Install SSL Certificate (HTTPS)](#step-6-install-ssl-certificate-https)
   - [Step 7: Create a Live Server Status Script](#step-7-create-a-live-server-status-script)
   - [Step 8: Automate Status Updates with Cron](#step-8-automate-status-updates-with-cron)
   - [Step 9: Link Status Page on Your Website](#step-9-link-status-page-on-your-website)
   - [Step 10: Final Verification](#step-10-final-verification)
5. [How to Rebuild from Scratch in Under 1 Hour](#how-to-rebuild-from-scratch-in-under-1-hour)
6. [Troubleshooting Common Issues](#troubleshooting-common-issues)
7. [References](#references)
8. [Video Explainer](#video-explainer) 

---

## Project Overview

This project builds a **secure, multi‑purpose cloud server** from scratch using **AWS EC2 (Ubuntu 24.04)**. It runs:

- A **static HTML/CSS website** accessible via a custom domain with **HTTPS**.
- A **live server status page** (`/status.html`) that shows real‑time system metrics (uptime, memory, disk, load) – automatically updated every 5 minutes.

All components are **manually installed and configured** (no pre‑packaged images). This documentation is written so that **anyone with basic Linux knowledge can rebuild the entire server** in less than one hour.

---

## Server Specifications

| Component                | Detail                                           |
|--------------------------|--------------------------------------------------|
| **Cloud Provider**       | Amazon Web Services (AWS)                        |
| **Service**              | EC2 (Elastic Compute Cloud)                      |
| **Region**               | ap‑southeast‑2 (Sydney)                          |
| **Operating System**     | Ubuntu 24.04 LTS (HVM)                           |
| **Instance Type**        | t2.micro (free tier eligible)                    |
| **Architecture**         | x86_64                                           |
| **Web Server**           | Nginx 1.24.0                                     |
| **SSL/TLS**              | Let's Encrypt (Certbot) – auto‑renewal enabled   |
| **Scripting**            | Bash + cron                                      |
| **Domain Registrar**     | GoDaddy                                          |

---

## Prerequisites

Before you start, make sure you have:

- An **AWS account** (free tier works)
- A **registered domain name** (e.g., from GoDaddy, Namecheap, Cloudflare)
- A **local computer** with SSH client (Windows PowerShell / Mac Terminal)
- Basic comfort with copy‑pasting commands

---

## Step-by-Step Build Guide

### Step 1: Launch AWS EC2 Instance

| Instruction | Action |
|-------------|--------|
| 1. Log in to AWS Console | Go to [https://console.aws.amazon.com](https://console.aws.amazon.com) |
| 2. Open EC2 Dashboard | Search for `EC2` and click it |
| 3. Click **Launch Instance** | |
| 4. Name your instance | `MyCloudServer` (or any name) |
| 5. Choose AMI | **Ubuntu 24.04 LTS (HVM)** – free tier eligible |
| 6. Instance type | `t2.micro` |
| 7. Key pair (login) | Click **Create new key pair** → name it `workshop-key` → download the `.pem` file → save it in `Downloads` |
| 8. Edit network settings | Click **Edit** |
| 9. Security group rules | Add these **inbound rules**: |

| Type | Port | Source |
|------|------|--------|
| SSH | 22 | My IP (or `0.0.0.0/0` for testing) |
| HTTP | 80 | `0.0.0.0/0` |
| HTTPS | 443 | `0.0.0.0/0` |

| 10. Storage | Leave as default (8 GB gp2) |
| 11. Click **Launch instance** | |

Wait 1–2 minutes for the instance to show `Running`.

---

### Step 2: Connect to Your Server

Open a terminal (PowerShell on Windows, Terminal on Mac/Linux).

```bash
# Go to the folder where your .pem key is
cd C:\Users\TALHA\Downloads   (adjust path for your computer)
```
# Connect using SSH
```bash
ssh -i "workshop-key.pem" ubuntu@15.135.223.199
```
 Success indicator: You see ubuntu@ip-172-31-17-231:~$

### Step 3: Install Nginx Web Server
Once connected, run these commands one by one:


# Update package lists
```bash
sudo apt update
```

# Install Nginx
```bash
sudo apt install nginx -y
```

# Enable Nginx to start on boot
```bash
sudo systemctl enable nginx
```
# Start Nginx now
```bash
sudo systemctl start nginx
```
Test your server: open a browser and go to http://15.135.223.199. You should see the default Nginx welcome page.

### Step 4: Add Your Website Files
Replace the default page with your own HTML/CSS.

```bash
# Edit the main index file
sudo nano /var/www/html/index.html
```
Delete everything inside and paste your HTML content (for example, your proposal blog).
Save with Ctrl+O, Enter, Ctrl+X.

Now refresh http://15.135.223.199 – your own website appears.

### Step 5: Set Up Domain DNS Records
Log in to your domain registrar (GoDaddy).

Record Type	Name/Host	Value (Points to)	TTL
A	@	15.135.223.199	600
A	www	15.135.223.199	600
Save changes. DNS propagation can take 5–30 minutes.

### Step 6: Install SSL Certificate (HTTPS)
bash
# Install Certbot for Nginx
sudo apt install certbot python3-certbot-nginx -y

# Obtain and install certificate for your domain
sudo certbot --nginx -d talhatechub.online -d www.talhatechub.online
Follow the prompts:

Enter your email address.

Agree to the terms (type A and Enter).

Choose whether to share email with EFF (type N).

When asked, select 2: Redirect – this forces all HTTP traffic to HTTPS.

After success, visit https://talhatechub.online. You’ll see a padlock in the address bar.

### Step 7: Create a Live Server Status Script
This script generates a webpage that shows real‑time server metrics.

bash
# Create a folder for scripts
mkdir -p ~/scripts

# Create the script file
nano ~/scripts/server_status.sh
Paste the exact content below (use right‑click or Ctrl+Shift+V):

bash
#!/bin/bash
OUTPUT_FILE="/var/www/html/status.html"

cat > $OUTPUT_FILE << EOF
<!DOCTYPE html>
<html>
<head><title>Server Status - Talha's Cloud</title>
<style>
body { font-family: monospace; background: #f0f0f0; }
pre { background: #ffffff; padding: 20px; border-radius: 8px; }
</style>
</head>
<body>
<h1>🖥️ Talha's Cloud Server – Live Status</h1>
<pre>
Uptime:        $(uptime)
Memory usage:  $(free -h | grep Mem)
Disk usage:    $(df -h / | tail -1)
Load average:  $(cat /proc/loadavg)
Open ports:    $(ss -tun | wc -l) connections
Last updated:  $(date)
</pre>
<p><a href="/">← Back to homepage</a></p>
</body>
</html>
EOF
Save: Ctrl+O, Enter, Ctrl+X.

Now make the script executable and run it once:

bash
chmod +x ~/scripts/server_status.sh
sudo ~/scripts/server_status.sh
Check that the status page exists:

bash
cat /var/www/html/status.html
You should see HTML output. Now open in a browser: https://talhatechub.online/status.html

### Step 8: Automate Status Updates with Cron
Cron will refresh the status page every 5 minutes automatically.

bash
# Add a cron job (this command adds it without opening an editor)
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/ubuntu/scripts/server_status.sh") | crontab -
Verify the cron job is added:

bash
crontab -l
You should see a line with */5 * * * * /home/ubuntu/scripts/server_status.sh.

### Step 9: Link Status Page on Your Website
Add a visible link to the status page from your main website.

bash
# Edit your homepage
sudo nano /var/www/html/index.html
Add this line somewhere inside the <body> section:

html
<a href="/status.html">📊 Live Server Status</a>
Save and exit. Then refresh https://talhatechub.online – you will see the link.

### Step 10: Final Verification
Test all three URLs in your browser:

Service	URL	Expected result
Main website	https://talhatechub.online	Your custom HTML page
Status page	https://talhatechub.online/status.html	Live server metrics
HTTPS	Check padlock in address bar	Secure connection
If all work, your server is complete.

## How to Rebuild from Scratch in Under 1 Hour
Launch a new EC2 instance with the same security group rules (Step 1).

SSH into it (Step 2).

Copy and paste the commands from Step 3 to Step 9 in order.

Replace talhatechub.online with your own domain.

Wait for DNS propagation, then run the Certbot command (Step 6).

Done. Your new server is identical.

💡 Tip: Save all commands in a text file. Then you can rebuild by simply copying and pasting.

## Troubleshooting Common Issues
Problem	Likely cause	Solution
ssh: Connection refused	Security group missing SSH rule	Add SSH (port 22) inbound rule with source 0.0.0.0/0
Permission denied (publickey)	Wrong key or username	Use the correct .pem file and username ubuntu
certbot command not found	Certbot not installed	Run sudo apt install certbot python3-certbot-nginx -y again
Status page shows old data	Cron not running	Run sudo ~/scripts/server_status.sh manually, then check crontab -l
Website loads without HTTPS	Certbot didn't redirect	Re‑run sudo certbot --nginx and select option 2
DNS not resolving	Records not saved or propagation delay	Wait 30 minutes, double‑check A records in registrar
References
Let's Encrypt / Certbot Documentation

AWS EC2 Documentation

Nginx Official Documentation

Ubuntu 24.04 Server Guide

Video Explainer
🎥 Link to my video walkthrough:
[Add your YouTube link here – unlisted is fine]

The video shows me building this server step‑by‑step, explaining each command, and demonstrating the final working website and status page.

End of documentation.
Built manually for ICT171 – Cloud Server Project.
