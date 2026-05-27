# Talha's Cloud Server – ICT171 Assignment

**Student name:** Talha Arshad  
**Student number:** 35051172  
**Live server IP:** 15.135.223.199  
**Domain:** https://talhatechub.online

---

## How to rebuild this server from scratch

Follow these steps in order. Each step has one instruction and one command.

---

### 1. Launch an AWS EC2 instance

- AMI: Ubuntu 24.04 LTS  
- Instance type: t2.micro  
- Key pair: download `your-key.pem`  
- Security group inbound rules:  
  - SSH (port 22) from your IP  
  - HTTP (port 80) from 0.0.0.0/0  
  - HTTPS (port 443) from 0.0.0.0/0

---

### 2. Connect to your server

Instruction: Use SSH with your key file.

```bash
ssh -i "your-key.pem" ubuntu@15.135.223.199
```
### 3. Update the server
Instruction: Run system update.

```bash
sudo apt update && sudo apt upgrade -y
```
### 4. Install Nginx web server
Instruction: Install and enable Nginx.

```bash
sudo apt install nginx -y && sudo systemctl enable nginx
```
### 5. Place your website files
Instruction: Copy your HTML/CSS files to the web directory.
(Example: create a simple index.html)

```bash
echo '<h1>My Cloud Server</h1>' | sudo tee /var/www/html/index.html
```
### 6. Buy a domain and set DNS records
Instruction: At your domain registrar (e.g., GoDaddy), add two A records pointing to your server IP.

Name: @ → Value: 15.135.223.199

Name: www → Value: 15.135.223.199

### 7. Install SSL certificate (Let's Encrypt)
Instruction: Install Certbot and get a free HTTPS certificate.

```bash
sudo apt install certbot python3-certbot-nginx -y
```
```bash
sudo certbot --nginx -d talhatechub.online -d www.talhatechub.online
```
When prompted: enter your email, agree to terms, and choose option 2 (redirect to HTTPS).

### 8. Create a status script
Instruction: Create a script that generates a live server status page.

```bash
mkdir -p ~/scripts
```
```bash
nano ~/scripts/server_status.sh
```
Inside nano, paste this:

```bash
#!/bin/bash
OUTPUT_FILE="/var/www/html/status.html"
cat > $OUTPUT_FILE << EOF
<!DOCTYPE html>
<html>
<head><title>Server Status</title>
<style>body{font-family:monospace;} pre{background:#f4f4f4;padding:15px;}</style>
</head>
<body>
<h1>Live Server Status</h1>
<pre>
Uptime:        $(uptime)
Memory usage:  $(free -h | grep Mem)
Disk usage:    $(df -h / | tail -1)
Load average:  $(cat /proc/loadavg)
Last updated:  $(date)
</pre>
<p><a href="/">Back to Home</a></p>
</body>
</html>
EOF
```
Save with Ctrl+O, Enter, then Ctrl+X.

### 9. Make the script executable and run it
Instruction: Allow execution and generate the status page for the first time.

```bash
chmod +x ~/scripts/server_status.sh && sudo ~/scripts/server_status.sh
```
### 10. Automate updates every 5 minutes (cron)
Instruction: Add a cron job to refresh the status page.

```bash
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/ubuntu/scripts/server_status.sh") | crontab -
```
### 11. Link the status page on your website
Instruction: Add a link to /status.html in your index.html.

```bash
echo '<a href="/status.html"> Live Server Status</a>' | sudo tee -a /var/www/html/index.html
```
### 12. Test everything
Instruction: Open these URLs in a browser.

Main website: https://talhatechub.online

Status page: https://talhatechub.online/status.html

Both should load without errors.

How to rebuild from scratch (summary)
Launch EC2 instance with the security group rules above.

SSH into it.

Run the commands from steps 3 to 11 in order.

Replace talhatechub.online with your own domain.

Wait for DNS propagation (up to 30 minutes), then run the Certbot command.

References
Let's Encrypt / Certbot

AWS EC2 Documentation
