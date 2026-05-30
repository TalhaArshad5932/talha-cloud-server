# Talha's Cloud Server – ICT171 Project

**Student name:** Talha Arshad  
**Student number:** 35051172  
**Live server IP:** `15.135.223.199`  
**Live domain:** [https://talhatechub.online](https://talhatechub.online)  


 This document contains every command needed to rebuild this cloud server from scratch on AWS EC2 (Ubuntu 24.04). Follow the steps in order, and you will have a working website with HTTPS and a live status page.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Server Specifications](#server-specifications)
3. [Prerequisites](#prerequisites)
4. [Step-by-Step Build Guide](#step-by-step-build-guide)
   - [Step 1: Launch AWS EC2 Instance](#step-1-launch-aws-ec2-instance)
   - [Step 2: Connect to Your Server (Windows & Mac/Linux)](#step-2-connect-to-your-server-windows--maclinux)
   - [Step 3: Install Nginx Web Server](#step-3-install-nginx-web-server)
   - [Step 4: Add Your Website Files](#step-4-add-your-website-files)
   - [Step 5: Set Up Domain DNS Records](#step-5-set-up-domain-dns-records)
   - [Step 6: Install SSL Certificate (HTTPS)](#step-6-install-ssl-certificate-https)
   - [Step 7: Create a Live Server Status Script](#step-7-create-a-live-server-status-script)
   - [Step 8: Automate Status Updates with Cron](#step-8-automate-status-updates-with-cron)
   - [Step 9: Link Status Page on Your Website](#step-9-link-status-page-on-your-website)
   - [Step 10: Final Verification](#step-10-final-verification)
5. [Script Explanation](#script-explanation)
6. [How to Rebuild from Scratch](#how-to-rebuild-from-scratch)
7. [Troubleshooting](#troubleshooting)
8. [References](#references)
9. [Video Explainer](#video-explainer)

---

## Project Overview

This project builds a personal cloud server for my ICT171 blog. It hosts:

- My own **static HTML/CSS website** showing my cloud computing proposal and blog ideas.  
- A **live server status page** (`/status.html`) that displays real‑time system metrics: uptime, memory usage, disk usage, load average, and active network connections. This page is automatically refreshed every 5 minutes.

The server is manually configured on AWS EC2 (Ubuntu 24.04) with Nginx, a custom domain (talhatechub.online), and a free Let's Encrypt SSL certificate (HTTPS). A simple bash script combined with cron generates the status page. All steps are documented so anyone can replicate it.

---

## Server Specifications

| Component                | Detail                                           |
|--------------------------|--------------------------------------------------|
| **Cloud Provider**       | Amazon Web Services (AWS)                        |
| **Service**              | EC2 (Elastic Compute Cloud)                      |
| **Region**               | ap-southeast-2 (Sydney)                          |
| **Operating System**     | Ubuntu 24.04 LTS (HVM)                           |
| **Instance Type**        | t3.micro (free tier eligible)                    |
| **Architecture**         | x86_64                                           |
| **Web Server**           | Nginx 1.24.0                                     |
| **SSL/TLS**              | Let's Encrypt (Certbot) – auto-renewal enabled   |
| **Scripting**            | Bash + cron                                      |
| **Domain Registrar**     | GoDaddy                                          |

---

## Prerequisites

Before you start, make sure you have:

- An AWS account (free tier works)
- A registered domain name (e.g., from GoDaddy, Namecheap, Cloudflare)
- A local computer with internet access
- Basic comfort with copying and pasting commands

---

## Step-by-Step Build Guide

### Step 1: Launch AWS EC2 Instance

| Instruction | Action |
|-------------|--------|
| 1. Log in to AWS Console | Go to [https://console.aws.amazon.com](https://console.aws.amazon.com) |
| 2. Open EC2 Dashboard | Search for `EC2` and click it |
| 3. Click **Launch Instance** | |
| 4. Name your instance | `MyCloudServer` |
| 5. Choose AMI | **Ubuntu 24.04 LTS (HVM)** – free tier eligible |
| 6. Instance type | `t3.micro` |
| 7. Key pair (login) | Click **Create new key pair** → name `workshop-key` → download `.pem` → save in `Downloads` |
| 8. Edit network settings | Click **Edit** |
| 9. Security group rules | Add these **inbound rules**: |

| Type | Port | Source |
|------|------|--------|
| SSH | 22 | My IP (or `0.0.0.0/0` for testing) |
| HTTP | 80 | `0.0.0.0/0` |
| HTTPS | 443 | `0.0.0.0/0` |

| 10. Storage | Default (8 GB gp2) |
| 11. Click **Launch instance** | |

Wait 1–2 minutes for the instance to show `Running`.

---

### Step 2: Connect to Your Server (Windows & Mac/Linux)

#### For Windows (PowerShell)

1. Open **PowerShell**.
2. Go to your Downloads folder:
   ```bash
   cd C:\Users\TALHA\Downloads
   ```
(Replace TALHA with your Windows username.)
<img width="959" height="473" alt="pic1" src="https://github.com/user-attachments/assets/7f27a111-019a-40ed-b55e-0dda38a57e06" />


3. Connect:

```bash
ssh -i "workshop-key.pem" ubuntu@15.135.223.199
```
<img width="959" height="506" alt="pic2" src="https://github.com/user-attachments/assets/2a20bf44-f0e3-4ff5-bcc5-b85c9004d458" />

For Mac / Linux (Terminal)
Open Terminal.

Go to Downloads:

```bash
cd ~/Downloads
```
Fix key permissions:

```bash
chmod 400 workshop-key.pem
```
Connect:

```bash
ssh -i "workshop-key.pem" ubuntu@15.135.223.199
```
Success indicator: You see ubuntu@ip-172-31-17-231:~$

### Step 3: Install Nginx Web Server
Run these commands one by one:

```bash
sudo apt update
```
<img width="944" height="474" alt="pic3" src="https://github.com/user-attachments/assets/0f4e94a4-5cad-4baa-a629-2d9bdcf98a8a" />

```bash
sudo apt install nginx -y
```


```bash
sudo systemctl enable nginx
```
```bash
sudo systemctl start nginx
```
Test: Open a browser to http://15.135.223.199 – you should see the Nginx welcome page.

### Step 4: Add Your Website Files
```bash
sudo nano /var/www/html/index.html
```
<img width="845" height="193" alt="pic4" src="https://github.com/user-attachments/assets/cba6e3f2-7ce1-4d29-a1fc-76c1c87f4521" />
then add HTML content (your proposal/blog).
<img width="959" height="473" alt="pic5" src="https://github.com/user-attachments/assets/b6dcbfc7-8083-4b4d-9b3e-1d703d1c33c3" />

Save with Ctrl+O, Enter, Ctrl+X.

Refresh http://15.135.223.199 – your own website appears.

### Step 5: Set Up Domain DNS Records
Log in to your domain registrar (GoDaddy).

Record Type	Name/Host	Value	TTL
A	@	15.135.223.199	600
A	www	15.135.223.199	600
Save changes. DNS propagation can take 5–30 minutes.

### Step 6: Install SSL Certificate (HTTPS)
```bash
sudo apt install certbot python3-certbot-nginx -y
```
```bash
sudo certbot --nginx -d talhatechub.online -d www.talhatechub.online
```
Follow prompts: enter your email, agree to terms, then choose 2: Redirect (forces HTTPS).

Test: https://talhatechub.online – padlock icon appears.

### Step 7: Create a Live Server Status Script
This script generates a clean, real-time status page.

```bash
mkdir -p ~/scripts
nano ~/scripts/server_status.sh
```
Paste the following:

```bash
cat > ~/scripts/server_status.sh << 'EOF'
#!/bin/bash
OUTPUT_FILE="/var/www/html/status.html"

cat > $OUTPUT_FILE << EOF2
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Server Status  Talha's Cloud</title>
<style>
body { font-family: monospace; background: #f0f0f0; }
pre { background: #ffffff; padding: 20px; border-radius: 8px; }
</style>
</head>
<body>
<h1>Talha's Cloud Server Live Status</h1>
<pre>
Uptime:        $(uptime)
Memory usage:  $(free -h | grep Mem)
Disk usage:    $(df -h / | tail -1)
Load average:  $(cat /proc/loadavg)
Open ports:    $(ss -tun | wc -l) connections
Last updated:  $(date)
</pre>
<p><a href="/">Back to homepage</a></p>
</body>
</html>
EOF2
EOF
```
Save (Ctrl+O, Enter, Ctrl+X). Then:

```bash
chmod +x ~/scripts/server_status.sh
sudo ~/scripts/server_status.sh
```
Verify: Open https://talhatechub.online/status.html – you see live server stats.

### Step 8: Automate Status Updates with Cron
```bash
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/ubuntu/scripts/server_status.sh") |
```
crontab -
Check with crontab -l – you should see the job.

### Step 9:  Link Status Page on Your Website
Edit your blog homepage to include a prominent link to the status page. This demonstrates integration between the blog and the monitoring component.

```bash
sudo nano /var/www/html/index.html
```
Add this line inside the <body> (e.g., in the navigation or a dedicated section):

html
```bash
<a href="/status.html" style="background:#28a745; color:white; padding:8px 15px; border-radius:30px;"> Live Server Status</a>
```
Save and refresh your website – the link appears.

### Step 10: Final Verification
Service	URL	Expected result
Main website	https://talhatechub.online	Your custom HTML page
Status page	https://talhatechub.online/status.html	Live server metrics
HTTPS	Padlock in address bar	Secure connection
If all work, your server is complete.

### Script Explanation
The script server_status.sh is creative and useful because:

It runs every 5 minutes via cron, providing verifiable output (the /status.html page).

It collects real‑time data: uptime, memory, disk, load average, open ports.

It outputs a valid HTML page that is directly linked from the blog’s navigation, showing clear integration between components.

The output is publicly accessible – any visitor can see the server’s health.

This satisfies the rubric’s requirements for scripting (creativity, documentation, verifiable output) and multi‑purpose server integration.

### How to Rebuild from Scratch
Launch a new EC2 instance with the same security group rules (Step 1).

Connect using your OS-specific method (Step 2).

Run commands from Step 3 to Step 9 in order.

Replace talhatechub.online with your own domain.

Wait for DNS propagation, then run Certbot (Step 6).

The entire rebuild takes less than one hour.

### Troubleshooting
Problem	Likely cause	Solution
ssh: Connection refused	Missing SSH rule	Add SSH (22) inbound rule with source 0.0.0.0/0
Permission denied (publickey)	Wrong key or username	Use correct .pem and ubuntu user; on Mac, chmod 400
certbot command not found	Certbot not installed	Run sudo apt install certbot python3-certbot-nginx -y
Status page shows old data	Cron not running	Run script manually: sudo ~/scripts/server_status.sh
No HTTPS	Certbot didn't redirect	Re-run sudo certbot --nginx and choose option 2
DNS not resolving	Propagation delay	Wait 30 minutes, double-check A records
### References
Let's Encrypt / Certbot Documentation

AWS EC2 Documentation

Nginx Official Documentation

Ubuntu 24.04 Server Guide

### Video Explainer
Video link to my walkthrough: https://www.youtube.com/watch?v=_WCqqNQPYo0


The video shows the entire build process, including connecting to AWS, installing Nginx, setting up SSL, creating the status script, and demonstrating the final live website.

Built manually for ICT171 – Cloud Server Project.
