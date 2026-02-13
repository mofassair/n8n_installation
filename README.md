# Complete n8n Automation Setup Guide for elgenix.com

---
##Update: steps taken as of 20260213
- n8n at https://n8n.elgenix.com/ live.
- VPS is ready (Ubuntu, ~8 GB RAM, enough disk, running fine)
- Docker is installed and working
- Docker-Compose is installed and working
- n8n container starts correctly (no crash loop anymore)
- The permission problem is fixed
(/root/.n8n → owned by UID 1000 → no more EACCES error)
- n8n database migrations completed successfully
- n8n is reachable locally on the server
(http://127.0.0.1:5678 works)
- Sub-domain n8n.elgenix.com is created in DNS and points to your VPS
- Website n8n.elgenix.com exists in CyberPanel
- SSL certificate is installed for n8n.elgenix.com
- OpenLiteSpeed reverse proxy is configured to forward
/ → http://127.0.0.1:5678
- WebSocket support is enabled in the proxy
- n8n is configured for production (Path-B):
N8N_PROTOCOL=https
N8N_SECURE_COOKIE=true
N8N_PROXY_HOPS=1
correct public URLs set
- ⚠️ You are currently fixing the last header issue
(X-Forwarded-Proto) to remove the secure-cookie warning


---

## 📊 Path A vs Path B: Which Should You Choose?

| Factor | Path A: Direct Access | Path B: SSL + Domain |
|--------|----------------------|---------------------|
| **Setup Time** | ⏱️ 5 minutes | ⏱️ 30-60 minutes |
| **Access URL** | `http://109.123.254.58:5678` | `https://n8n.elgenix.com` |
| **Security** | ⚠️ HTTP (unencrypted) | ✅ HTTPS (encrypted) |
| **DNS Required** | ❌ No | ✅ Yes (Namecheap) |
| **SSL Certificate** | ❌ No | ✅ Yes (Let's Encrypt) |
| **Reverse Proxy** | ❌ No | ✅ Yes (CyberPanel) |
| **Firewall** | Open port 5678 | Closed port 5678 |
| **Professional Look** | ❌ IP:port format | ✅ Custom subdomain |
| **Webhook URLs** | Works but ugly | Clean & professional |
| **Production Ready** | ⚠️ Testing only | ✅ Yes |
| **Migration Later** | ✅ Easy (10 min) | N/A |
| **Best For** | Learning, testing | Production, business |

**💡 Recommendation:**
- **New to n8n?** → Start with Path A (learn fast, upgrade later)
- **Production from day 1?** → Go with Path B
- **Not sure?** → Path A first, then migrate when comfortable

---

## Table of Contents
1. [Understanding Ollama vs Paid AI APIs](#1-understanding-ollama-vs-paid-ai-apis)
2. [AI API Decision Guide](#2-ai-api-decision-guide)
3. [Prerequisites Check](#3-prerequisites-check)
4. [Setting Up n8n.elgenix.com Subdomain](#4-setting-up-n8nelgenixcom-subdomain)
5. [Installing Docker & Docker Compose](#5-installing-docker--docker-compose)
6. [Understanding Docker Compose Configuration](#6-understanding-docker-compose-configuration)
7. [Deploying n8n - Two Paths](#7-deploying-n8n)
   - Path A: Simple Direct Access (Quick Start)
   - Path B: Production Setup (SSL + Domain)
   - Migration Guide: A → B
8. [Configuring CyberPanel Reverse Proxy](#8-configuring-cyberpanel-reverse-proxy)
9. [SSL Certificate Setup](#9-ssl-certificate-setup)
10. [First Login & Configuration](#10-first-login--configuration)
11. [Setting Up Job Application Automation](#11-setting-up-job-application-automation)
12. [Setting Up WhatsApp Chatbot](#12-setting-up-whatsapp-chatbot)
13. [Cost Optimization Strategy](#13-cost-optimization-strategy)
14. [Troubleshooting](#14-troubleshooting)

---




**Check your VPS specs:**
```bash
# Check RAM
free -h
# Check CPU
lscpu | grep "CPU(s)"
# Check storage
df -h
```

---

## 2. AI API Decision Guide


### Cost Comparison (100 Job Applications/Month)

| Service | Model | Cost | Quality | When to Use |
|---------|-------|------|---------|-------------|
| **Gemini** | 1.5 Flash Free | $0 | ⭐⭐⭐⭐ | Start here, test workflows |
| **Ollama** | Mistral 7B | $0 | ⭐⭐⭐⭐ | After Gemini limits, privacy focus |
| **OpenAI** | GPT-4o-mini | $2-5 | ⭐⭐⭐⭐⭐ | Best quality, production ready |
| **Claude** | Haiku | $3-7 | ⭐⭐⭐⭐⭐ | Structured output, formal tone |
| **DeepSeek** | API | $1-2 | ⭐⭐⭐⭐ | Highest volume, budget focus |

---

## 5. Installing Docker & Docker Compose

### Installation Commands

## ✅ Check system basics
```bash
whoami
uname -a
df -h
free -h
```

Comment: If disk is nearly full or RAM < 2GB, Docker containers may crash.

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# 3. Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 4. Verify installation
docker --version
docker-compose --version
```

**Note:** If you do not add a user to the `docker` group, run Docker commands with `sudo` (recommended on a single-user VPS).

**Expected output:**
```
Docker version 25.x.x, build xxxxx
docker-compose version 1.29.x
```

---

## 6. Understanding Docker Compose Configuration

### What is `~/n8n-docker` Directory?

This directory is your **n8n project folder**. Think of it like this:

```
~/n8n-docker/               ← Your management center
├── docker-compose.yml      ← Configuration file (instructions for Docker)
├── .env (optional)         ← Environment variables (passwords, secrets)
└── logs/ (optional)        ← Log files

~/.n8n/                     ← Actual n8n data (workflows, credentials)
├── config/
├── database.sqlite
└── workflows/
```

**Analogy:**
- `~/n8n-docker/` = The control panel
- `~/.n8n/` = The actual workspace with your files



### Visual Flow Diagram

```
External User
    ↓
https://n8n.elgenix.com (Port 443)
    ↓
CyberPanel (Reverse Proxy)
    ↓ forwards to
http://127.0.0.1:5678 (localhost only)
    ↓
Docker Container (n8n)
    ↓ saves data to
~/.n8n (Persistent Storage on VPS)
```

---

## 7. Deploying n8n

### Deployment Strategy: Start Simple, Upgrade Later

**You have two paths:**

**Path A: Simple Start (Recommended for beginners)**
- Access via `http://YOUR_VPS_IP:5678` directly
- No reverse proxy needed
- Quick 5-minute setup
- ✅ Perfect for testing and learning
- ⚠️ HTTP only (no SSL), port must be opened

**Path B: Production Setup (SSL + Custom Domain)**
- Access via `https://n8n.elgenix.com`
- Full SSL encryption
- Professional setup
- ✅ Production-ready
- Requires reverse proxy configuration

**💡 Pro Tip: Start with Path A, migrate to Path B later!**

The migration is seamless - just stop the container, update the config, and restart. Your workflows and data remain intact.

---

### Path A: Simple Direct Access Setup

#### Step 1: Create Project Directory

```bash
# Create and enter directory
mkdir ~/n8n-docker
cd ~/n8n-docker
```

#### Step 2: Create docker-compose.yml (Simple Version)

```bash
# Create the configuration file
nano docker-compose.yml
```

**Paste this simple configuration:**

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=YOUR_VPS_IP
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://YOUR_VPS_IP:5678/
      - N8N_EDITOR_BASE_URL=http://YOUR_VPS_IP:5678/
      - N8N_SECURE_COOKIE=false
      - GENERIC_TIMEZONE=Europe/Berlin
    volumes:
      - /root/.n8n:/home/node/.n8n


```

**Save and exit:**
- Press `CTRL + X`
- Press `Y`
- Press `Enter`

#### Step 3: Open Firewall Port

**Method 1: CyberPanel GUI (Recommended - Easiest)**

1. Login to CyberPanel: https://109.123.254.58:8090/
2. Navigate to: Security -> Firewall
3. Add new rule:

  Rule Name:    n8n-port
  Protocol:     TCP
  Port:         5678
  IP Address:   0.0.0.0  (this opens it for all IPs)

4. Click: Add button
5. Click: Reload button (to apply changes)

Done! Port 5678 is now open.

**Method 2: Command Line**
CyberPanel uses firewalld. Here are the commands:

```bash
# Check if firewalld is running
sudo systemctl status firewalld

# Open port 5678
sudo firewall-cmd --zone=public --permanent --add-port=5678/tcp

# Reload firewall to apply changes
sudo firewall-cmd --reload

# Verify the port is open
sudo firewall-cmd --list-ports
```

Expected output:
```
5678/tcp 8090/tcp 80/tcp 443/tcp ...
```

#### Step 4: Start n8n

```bash
# Start in background
docker-compose up -d
```

#### Step 5: Access n8n

Open browser: `http://109.123.254.58:5678`

You should see the n8n setup wizard!

**⚠️ Limitations of this setup:**
- No SSL (data transmitted as plain text)
- Ugly URL with IP and port
- Webhooks work but URL looks unprofessional
- Not recommended for production use with sensitive data

**✅ Advantages:**
- Works immediately
- No DNS configuration needed
- No CyberPanel reverse proxy setup
- Perfect for testing and learning
- Easy to troubleshoot

---

### Path B: Production Setup (SSL + Custom Domain)

**Use this when you're ready for production or need SSL.**

#### Step 1: Create Project Directory

```bash
# Create and enter directory
mkdir ~/n8n-docker
cd ~/n8n-docker
```

#### Step 2: Create docker-compose.yml (Production Version)

```bash
# Create the configuration file
nano docker-compose.yml
```

**Paste this production configuration:**

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always

    # Only local access (CyberPanel will proxy)
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      # === PUBLIC URL SETTINGS (HTTPS) ===
      - N8N_HOST=n8n.elgenix.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.elgenix.com/
      - N8N_EDITOR_BASE_URL=https://n8n.elgenix.com/

      # CRITICAL behind reverse proxy
      - N8N_PROXY_HOPS=1

      # === SECURITY / COOKIES ===
      - N8N_SECURE_COOKIE=true

      - GENERIC_TIMEZONE=Europe/Berlin
    volumes:
      - /root/.n8n:/home/node/.n8n

```

**Save and exit:**
- Press `CTRL + X`
- Press `Y`
- Press `Enter`

**Note:** This requires completing Steps 4, 8, and 9 (DNS, Reverse Proxy, SSL)

---

### How to Migrate from Path A to Path B Later

When you're ready to upgrade from direct access to SSL + custom domain:

#### Step 1: Stop n8n

```bash
cd ~/n8n-docker
docker-compose down
```

#### Step 2: Update docker-compose.yml

```bash
nano docker-compose.yml
```

**Change these lines:**

```yaml
# BEFORE (Path A):
ports:
  - "5678:5678"
environment:
  - N8N_HOST=109.123.254.58
  - N8N_PROTOCOL=http
  - WEBHOOK_URL=http://109.123.254.58:5678/

# AFTER (Path B):
ports:
  - "127.0.0.1:5678:5678"
environment:
  - N8N_HOST=n8n.elgenix.com
  - N8N_PROTOCOL=https
  - WEBHOOK_URL=https://n8n.elgenix.com/
  - N8N_EDITOR_BASE_URL=https://n8n.elgenix.com/
```

## Setup DNS and Reverse Proxy

### Setting Up n8n.elgenix.com Subdomain
---

### Step 1: Add DNS Record in Namecheap

1. **Login to Namecheap**
2. **Go to Domain List** → Click `elgenix.com`
3. **Click "Advanced DNS"**
4. **Add New Record:**
   ```
   Type:     A Record
   Host:     n8n
   Value:    109.123.254.58
   TTL:      Automatic (or 300)
   ```
5. **Click "Save All Changes"**

**What this does:** Points `n8n.elgenix.com` to your VPS IP

### Step 2: Create Website in CyberPanel

1. **Login to CyberPanel** at `https://109.123.254.58:8090/`
2. **Go to**: Websites → Create Website
3. **Fill in:**
   ```
   Domain Name:     n8n.elgenix.com
   Email:           your@email.com
   Package:         Default
   PHP Version:     (doesn't matter)
   ```
4. **Click "Create Website"**

**What this does:** Creates a virtual host for `n8n.elgenix.com`

### Step 3: Wait for DNS Propagation

DNS changes take 5-60 minutes to propagate globally.

**Check if DNS is working:**
```bash
# On your local computer or VPS
ping n8n.elgenix.com

# Should return: 109.123.254.58
```

#### Step 4: Restart n8n

```bash
docker-compose up -d
```


#### Pre-check (must be true before proxy)

SSH into VPS and confirm n8n is running locally only:
```bash
docker-compose ps
curl -I http://127.0.0.1:5678
ss -lntp | grep 5678
```

Expected

curl returns headers (200/302)

ss shows 127.0.0.1:5678 (not 0.0.0.0)

If you see 0.0.0.0:5678, change compose to:
```bash
ports:
  - "127.0.0.1:5678:5678"
```

then restart.



#### Issue SSL certificate (Let’s Encrypt)

CyberPanel → SSL → Manage SSL
Select n8n.elgenix.com
Click Issue SSL
✅ Check:
curl -I https://n8n.elgenix.com

Expected: you get an HTTPS response (could still be default page, but must not show certificate error).


#### Add Reverse Proxy to n8n (OpenLiteSpeed / CyberPanel)
0) Confirm n8n is reachable locally (must pass)

On SSH:
```bash
curl -I http://127.0.0.1:5678
ss -lntp | grep 5678
```

Expected

- curl shows HTTP/1.1 200 or 302
- ss shows 127.0.0.1:5678 (ideal for Path B)

If it shows 0.0.0.0:5678, you can still proxy, but it’s better to bind to localhost in docker compose.

#### 1) Edit vHost Conf and add the proxy context

You are already looking at the vHost Conf. Scroll to the end (after vhssl { ... }) and append this block:
```bash
# ------------------------------------------------------------
# Reverse proxy n8n (Docker) behind OpenLiteSpeed (CyberPanel)
# Public:  https://n8n.elgenix.com
# Private: http://127.0.0.1:5678
# ------------------------------------------------------------

extprocessor n8nproxy {
  type                    proxy
  address                 http://127.0.0.1:5678
  maxConns                2000
  initTimeout             60
  retryTimeout            0
  respBuffer              0
}

# Proxy ALL requests to n8n
context / {
  type                    proxy
  handler                 n8nproxy
  enableWebSocket        1

  requestHeaders  {
    set X-Forwarded-Proto https
    set X-Forwarded-Host n8n.elgenix.com
  }

  rewrite  {
    enable                0
  }
}

# OPTIONAL: If you want to keep ACME challenge served locally (Let's Encrypt renewals)
# This is already present in your config:
# context /.well-known/acme-challenge { ... }
# So no change needed here.
```

Why this works

- extprocessor ... type proxy defines an upstream.
- context / ... type proxy makes / route to that upstream.
- .well-known/acme-challenge is already a special context in your file, so Let’s Encrypt renewal keeps working.

✅ Save changes.

#### 
Graceful restart OpenLiteSpeed

SSH:
```bash
systemctl restart lsws
```

Test immediately (from VPS)
```bash
curl -I https://n8n.elgenix.com
```

Expected
- HTTP/2 200 or HTTP/2 302
- NOT a CyberPanel default page

Now open in browser:
https://n8n.elgenix.com




#### Step 5: Update Workflows (if needed)

If you have workflows with hardcoded webhook URLs:
1. Go to each workflow
2. Update webhook URLs from `http://109.123.254.58:5678/` to `https://n8n.elgenix.com/`
3. Save workflows

**✅ Your data is safe!** The `~/.n8n` volume preserves all workflows, credentials, and settings during migration.






#### Common Migration Issues

**Issue: Workflows show old webhook URLs**

After migration, some workflows might still reference the old URL.

**Solution:**
```bash
# Find workflows with old URLs
grep -r "109.123.254.58:5678" ~/.n8n/

# Manual fix:
# 1. Open each affected workflow in n8n
# 2. Click on webhook nodes
# 3. URLs will auto-update to new domain
# 4. Save workflow
```

**Issue: "Cannot access n8n after migration"**

**Solution:**
```bash
# Check if both DNS and reverse proxy are working
curl -I https://n8n.elgenix.com

# Should return: HTTP/2 200

# If not, check:
# 1. DNS propagation (can take up to 1 hour)
# 2. CyberPanel proxy config
# 3. SSL certificate installed
```

**Issue: "Webhooks stopped working after migration"**

**Cause:** Webhook URLs changed from HTTP to HTTPS

**Solution:**
1. Go to external services (WhatsApp, job boards, etc.)
2. Update webhook URLs:
   - Old: `http://109.123.254.58:5678/webhook/xyz`
   - New: `https://n8n.elgenix.com/webhook/xyz`
3. Re-test webhooks in n8n

---

### Recommended Approach: Progressive Setup

**Week 1: Learn & Test (Path A)**
```bash
# Simple setup
ports: "5678:5678"
N8N_HOST: 109.123.254.58
N8N_PROTOCOL: http
```
- Access: `http://109.123.254.58:5678`
- Build your first workflows
- Test automations
- Learn n8n interface

**Week 2-3: Production (Path B)**
```bash
# Upgrade to SSL
ports: "127.0.0.1:5678:5678"
N8N_HOST: n8n.elgenix.com
N8N_PROTOCOL: https
```
- Setup DNS
- Configure reverse proxy
- Enable SSL
- Professional deployment

### Step 3: Start n8n

```bash
# Start in background
docker-compose up -d

# Expected output:
# Creating network "n8n-docker_default" with the default driver
# Pulling n8n (n8nio/n8n:)...
# Creating n8n-docker_n8n_1 ... done
```

### Step 4: Verify n8n is Running

```bash
# Check container status
docker-compose ps

# Should show:
# Name                Command         State           Ports
# n8n_n8n_1    tini -- /docker...   Up      127.0.0.1:5678->5678/tcp

# View logs (real-time)
docker-compose logs -f n8n

# Should see:
# n8n ready on port 5678
```

### Step 5: Test Local Access

```bash
# From your VPS, test if n8n responds
curl http://127.0.0.1:5678

# Should return HTML (n8n interface)
```

---

## 8. Configuring CyberPanel Reverse Proxy

**⚠️ Note: This section is ONLY needed for Path B (production setup with SSL + domain).**

**If you're using Path A (direct access), you can skip this section entirely.**

---

Now we connect CyberPanel to n8n so `https://n8n.elgenix.com` → `http://127.0.0.1:5678`

### Method 1: Using CyberPanel GUI (Recommended)

#### Step 1: Access Website Settings

1. **Login to CyberPanel**
2. **Go to**: Websites → List Websites
3. **Find**: `n8n.elgenix.com`
4. **Click**: "Manage"

#### Step 2: Configure Rewrite Rules

Find vHost Conf / Context / Proxy (wording varies slightly by version)

Create a Proxy Context:

- Type: Proxy
- URI: /
- Address: http://127.0.0.1:5678
- WebSocket: ON/Enabled

Save.

#### Restart OpenLiteSpeed:
CyberPanel → Server → LiteSpeed Status → Restart
(or SSH)

```bash
systemctl restart lsws
```


✅ Test after proxy
curl -I https://n8n.elgenix.com


Then open browser:
```bash
https://n8n.elgenix.com
```

## 4) Common failure points (with quick diagnosis)
### Problem: Site opens but n8n UI freezes / doesn’t update executions

Cause: websockets not enabled in proxy context
✅ Fix: Ensure proxy context has WebSocket ON

Check logs:
```bash
tail -n 80 /usr/local/lsws/logs/error.log
docker compose logs -n 80 n8n
```

### Problem: Login loop / keeps asking to login

Cause: missing secure cookie behind HTTPS proxy
✅ Fix: set:

- N8N_SECURE_COOKIE=true
- N8N_PROXY_HOPS=1

Restart container.

### Problem: Webhooks generate wrong URL

Cause: wrong WEBHOOK_URL and N8N_EDITOR_BASE_URL
✅ Fix: must match your public domain exactly.

Problem: Can’t access at all (timeout)

Checklist:
```bash
docker compose ps
curl -I http://127.0.0.1:5678
nslookup n8n.elgenix.com
curl -I https://n8n.elgenix.com
```


- If local curl fails → n8n container issue
- If DNS wrong → Namecheap record issue
- If HTTPS returns CyberPanel default → proxy not set / not saved / OLS not restarted


### WhatsApp webhook verification (correct approach in n8n)

Your old doc used a Function node returning JSON. Meta expects a raw challenge response.

✅ Correct n8n flow:

1) Webhook node (GET or POST depending on Meta setup)
2) IF verify token matches
3) Respond to Webhook node → respond with hub.challenge as plain text

Comment: This is the #1 reason WhatsApp verification fails.

## Backup & Restore (the part you’ll actually use)
Backup n8n data
```bash
tar -czf ~/n8n-backup-$(date +%Y%m%d-%H%M).tar.gz ~/.n8n
ls -lh ~/n8n-backup-*.tar.gz
```

Restore
```bash
docker compose down
rm -rf ~/.n8n
tar -xzf ~/n8n-backup-YYYYMMDD-HHMM.tar.gz -C ~/
docker compose up -d
```

### Minimal “health check” command set (copy/paste)
```bash
cd ~/n8n-docker
docker compose ps
docker compose logs -n 50 n8n
curl -I http://127.0.0.1:5678
curl -I https://n8n.elgenix.com
tail -n 50 /usr/local/lsws/logs/error.log
```

### factory resetting n8n 
If you previously created ~/.n8n with root, permission fix is enough.
If something is badly corrupted, you can reset (⚠️ deletes workflows):
```bash
docker-compose down
mv ~/.n8n ~/.n8n.bak-$(date +%Y%m%d-%H%M)
mkdir -p ~/.n8n
chown -R 1000:1000 ~/.n8n
docker-compose up -d
```
---

## 10. First Login & Configuration

### Step 1: Access n8n

**If using Path A (direct access):**
1. Open browser: `http://109.123.254.58:5678`
2. You'll see n8n setup wizard

**If using Path B (SSL + domain):**
1. Open browser: `https://n8n.elgenix.com`
2. You'll see n8n setup wizard (with green padlock 🔒)

### Step 2: Create Owner Account

```
Email:     your@email.com
First Name: Your Name
Last Name:  Your Last Name
Password:   (strong password)
```

**IMPORTANT:** Save these credentials securely!

### Step 3: Basic Settings

After login, go to **Settings** (gear icon):

1. **General**:
   - Timezone: Asia/Kolkata (or your timezone)
   - Date format: Preferred format
   
2. **Execution**:
   - Timeout: 300 (5 minutes for job applications)
   - Max Execution Time: 600 (10 minutes)
   
3. **Security**:
   - Enable: "Require authentication for webhook"
   - Disable: "Allow anonymous access"

---

## 11. Setting Up Job Application Automation

### Workflow Architecture

```
Job Board Scraper
    ↓
Filter & Match (AI)
    ↓
Customize CV (AI)
    ↓
Generate Cover Letter (AI)
    ↓
Create PDF
    ↓
Auto-apply OR Notify You
```

### Phase 1: Setup Google Sheets Tracking (Free)

1. **Create Google Sheet**: "Job Applications Tracker"
2. **Columns**:
   ```
   A: Date Applied
   B: Company
   C: Position
   D: Job URL
   E: Status
   F: CV Version Used
   G: Cover Letter Link
   H: Notes
   ```

3. **In n8n**: Add "Google Sheets" node
   - Authenticate with Google
   - Select your sheet
   - Operation: "Append Row"

### Phase 2: Setup AI Provider

#### Option A: Start with Gemini (Free)

1. **Get API Key**:
   - Visit: https://makersuite.google.com/app/apikey
   - Click "Create API Key"
   - Copy key

2. **In n8n**: 
   - Go to Credentials
   - Add "Google Gemini API"
   - Paste API key

#### Option B: Setup OpenAI (Paid)

1. **Get API Key**:
   - Visit: https://platform.openai.com/api-keys
   - Click "Create new secret key"
   - Copy key (starts with `sk-...`)

2. **In n8n**:
   - Go to Credentials
   - Add "OpenAI"
   - Paste API key

### Phase 3: Import Job Search Workflow Template

1. **In n8n**: Click "Workflows" → "Add Workflow"
2. **Import from URL**: 
   ```
   https://n8n.io/workflows/6391
   ```
   (AI-powered job search with resume rewriting)

3. **Or start from scratch** with these nodes:
   ```
   Trigger (Cron: Daily at 9 AM)
   ↓
   HTTP Request (Scrape job boards)
   ↓
   AI Agent (Match with your profile)
   ↓
   IF (Match score > 70%)
   ↓
   AI Agent (Customize CV)
   ↓
   AI Agent (Generate cover letter)
   ↓
   Convert to PDF
   ↓
   Google Drive (Upload)
   ↓
   Google Sheets (Log application)
   ↓
   Telegram/Email (Notify you)
   ```

### Phase 4: CV Customization Node

```javascript
// Example n8n Function node for CV customization

const jobDescription = $input.item.json.job_description;
const yourCV = $input.item.json.your_cv_text;

const prompt = `
You are an expert CV writer. Customize this CV for the following job:

JOB DESCRIPTION:
${jobDescription}

ORIGINAL CV:
${yourCV}

INSTRUCTIONS:
1. Keep the same experience, but highlight relevant skills
2. Adjust the summary to match job requirements
3. Reorder sections if needed
4. Keep it truthful, only emphasize relevant parts
5. Output in clean markdown format

CUSTOMIZED CV:
`;

return {
  json: {
    prompt: prompt,
    job_title: $input.item.json.job_title,
    company: $input.item.json.company
  }
};
```

### Phase 5: Cover Letter Generation

```javascript
// Function node for cover letter prompt

const jobDescription = $input.item.json.job_description;
const company = $input.item.json.company;
const position = $input.item.json.position;
const yourName = "Your Name";

const prompt = `
Write a professional cover letter for this job application:

POSITION: ${position}
COMPANY: ${company}
APPLICANT: ${yourName}

JOB DESCRIPTION:
${jobDescription}

REQUIREMENTS:
1. 3-4 paragraphs
2. Professional but personalized tone
3. Highlight 2-3 key qualifications from the job description
4. Show genuine interest in the company
5. End with a call to action

COVER LETTER:
`;

return {
  json: {
    prompt: prompt,
    company: company,
    position: position
  }
};
```

### Phase 6: PDF Generation (Free with Gotenberg)

Install Gotenberg alongside n8n:

```bash
cd ~/n8n-docker

# Edit docker-compose.yml
nano docker-compose.yml
```

**Add Gotenberg service:**

```yaml
version: '3.8'

services:
  n8n:
    # ... existing n8n config ...
  
  gotenberg:
    image: gotenberg/gotenberg:7
    restart: always
    ports:
      - "127.0.0.1:3000:3000"
    command:
      - "gotenberg"
      - "--api-port=3000"
```

**Restart:**
```bash
docker-compose up -d
```

**In n8n**: Use HTTP Request node
```
Method: POST
URL: http://gotenberg:3000/forms/chromium/convert/html
Body: Your HTML content (CV or cover letter)
Response Format: Binary
```

---

## 12. Setting Up WhatsApp Chatbot

### Architecture

```
WhatsApp Message Received
    ↓
n8n Webhook Triggered
    ↓
Extract Message & Sender
    ↓
Check Message Category (Job, Advice, General)
    ↓
Load Relevant Context (from Google Docs/Sheets)
    ↓
AI Agent Generates Response
    ↓
Send WhatsApp Reply
```

### Option A: Official WhatsApp Business API (Recommended)

#### Step 1: Create Meta Developer Account

1. Visit: https://developers.facebook.com/
2. Sign up / Login
3. Create new app → "Business" type
4. Add "WhatsApp" product

#### Step 2: Get WhatsApp Business API Access

1. In Meta App Dashboard → WhatsApp → "Getting Started"
2. Add Phone Number (need a real phone number, not used for personal WhatsApp)
3. Verify phone number
4. Get **Temporary Token** (for testing)
5. Get **Permanent Token** (for production):
   - Create "System User"
   - Generate token with `whatsapp_business_messaging` permission

#### Step 3: Configure Webhook in n8n

1. **In n8n**: Create new workflow
2. **Add "Webhook" node**:
   ```
   HTTP Method: POST
   Path: whatsapp-webhook
   ```
3. **Copy webhook URL**: 
   ```
   https://n8n.elgenix.com/webhook/whatsapp-webhook
   ```

4. **In Meta Dashboard**: WhatsApp → Configuration
   ```
   Callback URL: https://n8n.elgenix.com/webhook/whatsapp-webhook
   Verify Token: (make your own, like: "mySecretToken123")
   Webhook Fields: ✓ messages
   ```

#### Step 4: Verify Webhook

n8n must respond to Meta's verification request:

```javascript
// Add "Function" node after Webhook

const mode = $input.item.json['hub.mode'];
const token = $input.item.json['hub.verify_token'];
const challenge = $input.item.json['hub.challenge'];

if (mode === 'subscribe' && token === 'mySecretToken123') {
  return {
    json: {
      challenge: challenge
    }
  };
}
```

#### Step 5: Handle Incoming Messages

```javascript
// Function node: Extract message data

const entry = $input.item.json.entry[0];
const changes = entry.changes[0];
const value = changes.value;

if (!value.messages) {
  return { json: {} }; // Ignore non-message events
}

const message = value.messages[0];

return {
  json: {
    from: message.from, // Sender's phone number
    message_id: message.id,
    timestamp: message.timestamp,
    text: message.text.body,
    name: value.contacts[0].profile.name
  }
};
```

#### Step 6: Send Reply with AI

Workflow structure:
```
Webhook (Receive message)
    ↓
Function (Extract data)
    ↓
OpenAI/Gemini (Generate response)
    ↓
HTTP Request (Send WhatsApp message)
```

**HTTP Request node to send message:**
```
Method: POST
URL: https://graph.facebook.com/v18.0/YOUR_PHONE_NUMBER_ID/messages
Headers:
  Authorization: Bearer YOUR_ACCESS_TOKEN
  Content-Type: application/json
Body:
{
  "messaging_product": "whatsapp",
  "to": "{{$node["Extract Message"].json["from"]}}",
  "type": "text",
  "text": {
    "body": "{{$node["AI Agent"].json["response"]}}"
  }
}
```

### Option B: Unofficial WhatsApp (WAHA - Free)

**⚠️ Warning**: Violates WhatsApp TOS, use at your own risk. Good for personal testing.

#### Step 1: Install WAHA

```bash
cd ~/n8n-docker

# Add to docker-compose.yml
nano docker-compose.yml
```

**Add WAHA service:**
```yaml
  waha:
    image: devlikeapro/waha
    restart: always
    ports:
      - "127.0.0.1:3001:3000"
    environment:
      - WHATSAPP_HOOK_URL=http://n8n:5678/webhook/waha-whatsapp
    volumes:
      - ~/.waha:/app/.sessions
```

**Restart:**
```bash
docker-compose up -d
```

#### Step 2: Setup WhatsApp Session

1. Visit: `http://109.123.254.58:3001`
2. Start new session → Scan QR code with WhatsApp
3. Session ID: `default`

#### Step 3: n8n Webhook Setup

Same as official method, but simpler:

```javascript
// Incoming message structure
{
  "event": "message",
  "session": "default",
  "payload": {
    "from": "1234567890@c.us",
    "body": "User's message",
    "fromMe": false
  }
}
```

**Send message:**
```
Method: POST
URL: http://waha:3000/api/sendText
Body:
{
  "session": "default",
  "chatId": "{{$json["payload"]["from"]}}",
  "text": "Your AI response here"
}
```

### AI Advice Bot Logic

```javascript
// Function node: Categorize query

const message = $input.item.json.text.toLowerCase();

let category = "general";
let context = "";

if (message.includes("job") || message.includes("career")) {
  category = "job_advice";
  context = "You are a career counselor. Give practical job search advice.";
} else if (message.includes("code") || message.includes("programming")) {
  category = "tech_advice";
  context = "You are a senior developer. Give clear coding guidance.";
} else if (message.includes("health") || message.includes("fitness")) {
  category = "health_advice";
  context = "You are a health coach. Give general wellness tips. Remind them to consult professionals for serious issues.";
}

return {
  json: {
    category: category,
    system_prompt: context,
    user_message: $input.item.json.text
  }
};
```

**AI Agent Node:**
```
System Message: {{$json["system_prompt"]}}
User Message: {{$json["user_message"]}}
Max Tokens: 500
Temperature: 0.7
```

---

## 13. Cost Optimization Strategy

### The Zero-Dollar Month Challenge

**Objective**: Run 100 job applications + 500 chatbot messages with $0 cost

#### Free Resources to Use

1. **AI Models**:
   ```
   Gemini Free Tier: 1500 requests/day
   + Ollama (Mistral 7B): Unlimited
   = ~5000 AI operations/month FREE
   ```

2. **Storage**:
   ```
   Google Drive: 15GB free
   Google Sheets: Unlimited rows
   ```

3. **Notifications**:
   ```
   Telegram Bot: Free, unlimited
   (Better than email for automation alerts)
   ```

#### Hybrid Strategy (When You Outgrow Free)

**Budget: $5/month**

```
Simple tasks (70%):
  Gemini Free Tier + Ollama
  Cost: $0

Complex tasks (25%):
  OpenAI GPT-4o-mini
  Cost: $3-4/month

Critical tasks (5%):
  Claude Haiku (important cover letters)
  Cost: $1-2/month
```

**Routing logic in n8n:**

```javascript
// Function node: Choose AI model

const task_complexity = $input.item.json.complexity;
const is_important = $input.item.json.important;

let model = "gemini"; // default free

if (task_complexity > 7 || is_important) {
  model = "gpt-4o-mini"; // paid but better
}

if ($input.item.json.company_tier === "dream_job") {
  model = "claude-haiku"; // best quality
}

return {
  json: {
    selected_model: model,
    data: $input.item.json
  }
};
```

### Installing Ollama (Optional but Recommended)

**If you decide to add Ollama:**

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Start Ollama service
sudo systemctl start ollama

# Enable on boot
sudo systemctl enable ollama

# Download a model (e.g., Mistral 7B)
ollama pull mistral

# Test it
ollama run mistral
# Type: "Write a short cover letter for a software engineer position"
# Press Ctrl+D to exit

# Check running models
ollama list
```

**Add Ollama to n8n docker-compose.yml:**

```yaml
  ollama:
    image: ollama/ollama
    restart: always
    ports:
      - "127.0.0.1:11434:11434"
    volumes:
      - ~/.ollama:/root/.ollama
```

**In n8n**: Use HTTP Request node
```
Method: POST
URL: http://ollama:11434/api/generate
Body:
{
  "model": "mistral",
  "prompt": "Your prompt here",
  "stream": false
}
```

---

## 14. Troubleshooting

### Path-Specific Issues

#### Path A (Direct Access) Issues

**Issue: Can't access http://109.123.254.58:5678**

**Check 1: Is n8n running?**
```bash
docker ps

# Should show n8n container with status "Up"
```

**Check 2: Is firewall blocking port 5678?**
Use CyberPanel Firewall (GUI) or firewalld to allow port 5678.

CyberPanel GUI: Security -> Firewall -> add TCP 5678 and reload.

Command line (firewalld):
```bash
sudo firewall-cmd --zone=public --permanent --add-port=5678/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

**Check 3: Is VPS firewall/security group blocking?**
- Check your VPS provider's firewall settings
- Ensure port 5678 is allowed in cloud security groups
- Some providers (AWS, Google Cloud, Azure) have additional firewall layers

**Check 4: Test from VPS itself**
```bash
curl http://127.0.0.1:5678

# Should return HTML
```

**Check 5: Test port from outside**
```bash
# From your local computer
telnet 109.123.254.58 5678

# Should connect. If "Connection refused" → firewall issue
```

---

#### Path B (SSL + Domain) Issues

### Issue 1: Can't Access n8n.elgenix.com

**Check DNS:**
```bash
nslookup n8n.elgenix.com

# Should return: 109.123.254.58
```

**Check if n8n container is running:**
```bash
docker ps

# Should see: n8n-docker_n8n_1 with status "Up"
```

**Check if n8n responds locally:**
```bash
curl http://127.0.0.1:5678

# Should return HTML
```

**Check CyberPanel proxy:**
```bash
sudo tail -f /var/log/lsws/error.log

# Look for proxy-related errors
```

### Issue 2: SSL Certificate Error

**Reissue SSL:**
1. CyberPanel → SSL → Manage SSL
2. Select `n8n.elgenix.com`
3. Click "Delete SSL"
4. Click "Issue SSL" again

**Check certificate:**
```bash
sudo ls -la /etc/letsencrypt/live/n8n.elgenix.com/

# Should see: cert.pem, privkey.pem, chain.pem
```

### Issue 3: Webhooks Not Working

**Test webhook directly:**
```bash
curl -X POST https://n8n.elgenix.com/webhook/test \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

**Check webhook URL in n8n:**
1. Open workflow
2. Click Webhook node
3. Click "Copy Test URL"
4. Should be: `https://n8n.elgenix.com/webhook/your-path`

**Check n8n logs:**
```bash
docker-compose logs -f n8n | grep webhook
```

### Issue 4: AI API Errors

**Gemini "Quota Exceeded":**
- Switched to Ollama for some tasks
- Or upgrade to paid tier

**OpenAI "Rate Limited":**
- Add retry logic in n8n (built-in)
- Slow down execution (add "Wait" node)

**Check API key:**
```bash
# Test OpenAI key
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"

# Should return list of models
```

### Issue 5: Docker Issues

**Container keeps restarting:**
```bash
docker-compose logs n8n

# Check for errors
```

**Out of memory:**
```bash
free -h

# If low memory, upgrade VPS or reduce model size
```

**Reset everything:**
```bash
cd ~/n8n-docker

# Stop and remove containers
docker-compose down

# Remove volumes (⚠️ DELETES ALL DATA)
rm -rf ~/.n8n

# Start fresh
docker-compose up -d
```

### Useful Commands

```bash
# View all Docker containers
docker ps -a

# View n8n logs (live)
docker-compose logs -f n8n

# Restart n8n
docker-compose restart n8n

# Update n8n to latest version
docker-compose pull
docker-compose up -d

# Backup n8n data
tar -czf n8n-backup-$(date +%Y%m%d).tar.gz ~/.n8n

# Restore backup
tar -xzf n8n-backup-20260212.tar.gz -C ~/
```

---

## Next Steps

### Recommended Learning Path

**🎯 Path A Users (Direct Access):**

**Day 1: Get Started**
- [x] Install n8n with Quick Start guide
- [ ] Create first account
- [ ] Explore n8n interface
- [ ] Import a simple workflow template

**Week 1: Learn Basics**
- [ ] Setup Google Sheets tracker
- [ ] Configure Gemini API (free)
- [ ] Build your first automation
- [ ] Test webhooks

**Week 2: Job Automation**
- [ ] Import job search template
- [ ] Customize CV generation
- [ ] Setup PDF generation
- [ ] Test with 5 real applications

**Week 3: WhatsApp Bot**
- [ ] Setup WhatsApp (unofficial WAHA for testing)
- [ ] Create simple echo bot
- [ ] Add AI responses
- [ ] Test with friends

**Week 4: Go Production**
- [ ] Migrate to Path B (SSL + domain)
- [ ] Setup proper WhatsApp Business API
- [ ] Add monitoring & alerts
- [ ] Document your workflows

---

**🎯 Path B Users (Production from Day 1):**

**Week 1: Foundation**
- [x] Setup DNS (n8n.elgenix.com)
- [x] Install n8n with SSL
- [x] Configure reverse proxy
- [ ] Setup Google Sheets tracker
- [ ] Configure Gemini API
- [ ] Test simple workflow

**Week 2: Job Automation**
- [ ] Import job search template
- [ ] Customize CV generation
- [ ] Setup PDF generation
- [ ] Test with 5 real applications

**Week 3: WhatsApp Bot**
- [ ] Setup WhatsApp Business API (official)
- [ ] Create advice categories
- [ ] Build knowledge base in Google Docs
- [ ] Test with friends

**Week 4: Optimization**
- [ ] Add Ollama for cost reduction
- [ ] Setup monitoring & alerts
- [ ] Document your workflows
- [ ] Share learnings

---

**💡 Hybrid Approach (Recommended):**

Start with Path A, work through weeks 1-3, then migrate to Path B in week 4. This gives you:
- ✅ Immediate learning (no DNS wait)
- ✅ Safe testing environment
- ✅ Production migration when ready
- ✅ Understanding of both approaches

---

## Additional Resources

### Official Documentation
- n8n Docs: https://docs.n8n.io/
- Docker Docs: https://docs.docker.com/
- CyberPanel Wiki: https://cyberpanel.net/docs/

### Community & Templates
- n8n Community: https://community.n8n.io/
- Workflow Library: https://n8n.io/workflows/
- Reddit r/n8n: https://reddit.com/r/n8n

### AI API Documentation
- OpenAI: https://platform.openai.com/docs/
- Anthropic Claude: https://docs.anthropic.com/
- Google Gemini: https://ai.google.dev/docs
- Ollama: https://github.com/ollama/ollama

### Learning Resources
- n8n YouTube: https://youtube.com/@n8n_io
- Automation Examples: https://n8n.io/blog/

---

## Final Notes

**Remember**:
1. Start with **Gemini Free** → test everything
2. Add **Ollama** when you need more volume
3. Use **OpenAI** only for quality-critical tasks
4. Always backup `~/.n8n` folder weekly
5. Monitor costs in API dashboards

**Questions?**
- n8n Community Forum: https://community.n8n.io/
- CyberPanel Forum: https://community.cyberpanel.net/

**Good luck with your automation journey! 🚀**

---

*Last updated: February 2026*
*Author: Claude (Anthropic)*
*License: Free to use and modify*
