# Complete n8n Automation Setup Guide for elgenix.com
**VPS with CyberPanel | Job Applications + WhatsApp Chatbot**

---

## Table of Contents
1. [Understanding Ollama vs Paid AI APIs](#1-understanding-ollama-vs-paid-ai-apis)
2. [AI API Decision Guide](#2-ai-api-decision-guide)
3. [Prerequisites Check](#3-prerequisites-check)
4. [Setting Up n8n.elgenix.com Subdomain](#4-setting-up-n8nelgenixcom-subdomain)
5. [Installing Docker & Docker Compose](#5-installing-docker--docker-compose)
6. [Understanding Docker Compose Configuration](#6-understanding-docker-compose-configuration)
7. [Deploying n8n](#7-deploying-n8n)
8. [Configuring CyberPanel Reverse Proxy](#8-configuring-cyberpanel-reverse-proxy)
9. [SSL Certificate Setup](#9-ssl-certificate-setup)
10. [First Login & Configuration](#10-first-login--configuration)
11. [Setting Up Job Application Automation](#11-setting-up-job-application-automation)
12. [Setting Up WhatsApp Chatbot](#12-setting-up-whatsapp-chatbot)
13. [Cost Optimization Strategy](#13-cost-optimization-strategy)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. Understanding Ollama vs Paid AI APIs

### What is Ollama?

**Ollama** is a **FREE, self-hosted tool** that lets you run AI models (like Llama, Mistral, DeepSeek) **directly on your VPS** instead of paying for cloud APIs.

Think of it like this:
- **Cloud APIs (OpenAI, Claude, Gemini)**: Rent a brain in the cloud → pay per use
- **Ollama**: Buy a brain once → install on your server → use unlimited

### How Ollama Works

```
Your VPS → Ollama Software (FREE) → Downloads AI Model (FREE) → Runs Locally
                                                                    ↓
                                               Unlimited API calls (NO COST)
```

### The Trade-off Matrix

| Factor | Cloud APIs (OpenAI/Claude/Gemini) | Ollama (Self-Hosted) |
|--------|-----------------------------------|----------------------|
| **Cost** | $5-30/month for moderate use | $0/month (only electricity) |
| **Quality** | ⭐⭐⭐⭐⭐ Best available | ⭐⭐⭐⭐ Good (80-90% of cloud) |
| **Speed** | Fast (optimized servers) | Depends on your VPS specs |
| **Setup** | 5 minutes (just API key) | 30-60 minutes (installation) |
| **Privacy** | Data sent to cloud | 100% on your server |
| **Maintenance** | Zero | Medium (updates, monitoring) |
| **VPS Requirements** | Any (just n8n) | Minimum 4GB RAM, 2 vCPU |

### VPS Requirements for Ollama

Your current VPS needs to meet these specs:

**Minimum (Small models like Mistral 7B, Llama 3 8B):**
- 4GB RAM
- 2 vCPU cores
- 20GB storage

**Recommended (Better models like Llama 3 70B, DeepSeek):**
- 8GB+ RAM
- 4+ vCPU cores
- 40GB+ storage

**Check your VPS specs:**
```bash
# Check RAM
free -h

# Check CPU
lscpu | grep "CPU(s)"

# Check storage
df -h
```

### Popular Ollama Models for Your Use Case

| Model | Size | Quality | Best For | RAM Needed |
|-------|------|---------|----------|------------|
| **Llama 3.2 3B** | 2GB | ⭐⭐⭐ | Simple tasks, testing | 4GB |
| **Mistral 7B** | 4GB | ⭐⭐⭐⭐ | Cover letters, CV editing | 6GB |
| **Llama 3 8B** | 5GB | ⭐⭐⭐⭐ | Job matching, chatbot | 8GB |
| **DeepSeek Coder 6.7B** | 4GB | ⭐⭐⭐⭐ | Technical content | 6GB |
| **Qwen 2.5 7B** | 4GB | ⭐⭐⭐⭐ | Multilingual tasks | 6GB |

---

## 2. AI API Decision Guide

### Quick Decision Tree

```
START → Do you have 8GB+ RAM VPS?
           │
           ├─ YES → Start with Ollama (FREE) → Upgrade to cloud if quality issues
           │
           └─ NO → Start with Gemini Free Tier → Pay for GPT-4o-mini when hitting limits
```

### Cost Comparison (100 Job Applications/Month)

| Service | Model | Cost | Quality | When to Use |
|---------|-------|------|---------|-------------|
| **Gemini** | 1.5 Flash Free | $0 | ⭐⭐⭐⭐ | Start here, test workflows |
| **Ollama** | Mistral 7B | $0 | ⭐⭐⭐⭐ | After Gemini limits, privacy focus |
| **OpenAI** | GPT-4o-mini | $2-5 | ⭐⭐⭐⭐⭐ | Best quality, production ready |
| **Claude** | Haiku | $3-7 | ⭐⭐⭐⭐⭐ | Structured output, formal tone |
| **DeepSeek** | API | $1-2 | ⭐⭐⭐⭐ | Highest volume, budget focus |

### My Recommended Strategy

**Phase 1 (Week 1-2): FREE TESTING**
```
Gemini Free Tier (1500 requests/day)
↓
Build & test all workflows
↓
Measure actual usage
```

**Phase 2 (Week 3-4): COST OPTIMIZATION**
```
If usage < 50/day → Stay on Gemini Free
If usage > 50/day → Install Ollama
If quality issues → Add OpenAI GPT-4o-mini
```

**Phase 3 (Production): HYBRID APPROACH**
```
Simple tasks (notifications, formatting) → Ollama (FREE)
Complex tasks (cover letters, analysis) → OpenAI ($2-5/month)
High-stakes (important applications) → Claude Haiku ($3-7/month)
```

### If You Had to Buy ONE API

**🥇 Winner: OpenAI GPT-4o-mini**

**Why:**
- **Price**: $0.15 per 1M input tokens, $0.60 per 1M output tokens
- **Quality**: Best at understanding job requirements
- **Reliability**: 99.9% uptime, fastest response times
- **Integration**: Works seamlessly with n8n
- **JSON Mode**: Perfect for structured automation data

**Real Cost Example:**
- 100 job applications/month
- Each application: ~3,000 tokens input + 1,000 tokens output
- Total: 300,000 input + 100,000 output
- **Cost: ~$0.50-1.00/month**

Scaling:
- 500 applications/month = $2.50-5/month
- 1000 applications/month = $5-10/month

---

## 3. Prerequisites Check

Before starting, verify you have:

- [x] VPS with root/sudo access
- [x] Domain: `elgenix.com` (already owned)
- [x] CyberPanel installed and accessible at `https://109.123.254.58:8090/`
- [x] Namecheap account (for DNS management)
- [x] SSH access to your VPS

**Check SSH access:**
```bash
ssh root@109.123.254.58
# or
ssh yourusername@109.123.254.58
```

---

## 4. Setting Up n8n.elgenix.com Subdomain

### Simple Method: CyberPanel + Namecheap

You're right - there IS a simpler way than complex reverse proxies! Here's the clean approach:

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

Or use online tool: https://dnschecker.org/ → Enter `n8n.elgenix.com`

---

## 5. Installing Docker & Docker Compose

### Why Docker?

Docker packages n8n and all its dependencies into an isolated container, making it:
- Easy to install (one command)
- Easy to update (one command)
- Easy to backup (just copy data folder)
- Isolated from your other services

### Installation Commands

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 3. Add your user to docker group (optional, for non-root usage)
sudo usermod -aG docker $USER

# 4. Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 5. Verify installation
docker --version
docker-compose --version
```

**Expected output:**
```
Docker version 25.x.x, build xxxxx
docker-compose version 1.29.x
```

### Troubleshooting Docker Installation

If `docker-compose` command not found:
```bash
# Alternative installation via pip
sudo apt install python3-pip -y
sudo pip3 install docker-compose
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

### Docker Compose File Explained (Line by Line)

```yaml
version: '3.8'
# This tells Docker which version of compose syntax to use
# 3.8 is stable and widely supported

services:
  # "services" means "containers we want to run"
  
  n8n:
    # "n8n" is just a name for this container
    # You could call it "automation" or "workflow-engine"
    
    image: n8nio/n8n
    # This downloads the official n8n container from Docker Hub
    # Like downloading an app from the App Store
    
    restart: always
    # If n8n crashes OR your VPS reboots → Docker automatically restarts n8n
    # Ensures 24/7 availability
    
    ports:
      - "127.0.0.1:5678:5678"
    # Port mapping format: "HOST:CONTAINER"
    # 127.0.0.1:5678 = Only accessible from localhost (secure)
    # :5678 = n8n runs on port 5678 inside the container
    # 
    # WHY 127.0.0.1? 
    # - CyberPanel's reverse proxy will handle external access
    # - Direct external access on 5678 would bypass CyberPanel security
    
    environment:
      # These are like "settings" that n8n reads when it starts
      
      - N8N_HOST=n8n.elgenix.com
      # The domain name n8n will respond to
      # Must match your actual domain
      
      - N8N_PORT=5678
      # Internal port (inside container)
      
      - N8N_PROTOCOL=https
      # Whether to use http or https
      # Set to "https" because CyberPanel will provide SSL
      
      - WEBHOOK_URL=https://n8n.elgenix.com/
      # The public URL where external services send data
      # CRITICAL for: job board notifications, WhatsApp webhooks
      
      - N8N_EDITOR_BASE_URL=https://n8n.elgenix.com/
      # The URL for accessing the n8n editor interface
    
    volumes:
      - ~/.n8n:/home/node/.n8n
      # Volume mapping format: "HOST_PATH:CONTAINER_PATH"
      # 
      # ~/.n8n = Folder on your VPS (persistent storage)
      # /home/node/.n8n = Folder inside container (where n8n expects data)
      # 
      # WHY THIS MATTERS:
      # - If you delete the container, your workflows are SAFE in ~/.n8n
      # - Easy backups: just copy ~/.n8n folder
      # - Upgrades are safe: data persists across container versions
```

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

### Step 1: Create Project Directory

```bash
# Create and enter directory
mkdir ~/n8n-docker
cd ~/n8n-docker
```

### Step 2: Create docker-compose.yml

```bash
# Create the configuration file
nano docker-compose.yml
```

**Paste this configuration:**

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - N8N_HOST=n8n.elgenix.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.elgenix.com/
      - N8N_EDITOR_BASE_URL=https://n8n.elgenix.com/
      - GENERIC_TIMEZONE=Asia/Kolkata
    volumes:
      - ~/.n8n:/home/node/.n8n
```

**Save and exit:**
- Press `CTRL + X`
- Press `Y`
- Press `Enter`

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

Now we connect CyberPanel to n8n so `https://n8n.elgenix.com` → `http://127.0.0.1:5678`

### Method 1: Using CyberPanel GUI (Recommended)

#### Step 1: Access Website Settings

1. **Login to CyberPanel**
2. **Go to**: Websites → List Websites
3. **Find**: `n8n.elgenix.com`
4. **Click**: "Manage"

#### Step 2: Configure Rewrite Rules

1. **Go to**: "Rewrite Rules" (in left sidebar)
2. **Click**: "Save & Edit"
3. **Replace** the entire content with:

```apache
<VirtualHost *:443>
    ServerName n8n.elgenix.com
    ServerAdmin admin@elgenix.com
    
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/n8n.elgenix.com/cert.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/n8n.elgenix.com/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/n8n.elgenix.com/chain.pem
    
    ProxyPreserveHost On
    ProxyRequests Off
    
    # WebSocket support (critical for n8n)
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule .* ws://127.0.0.1:5678%{REQUEST_URI} [P]
    
    # Regular HTTP proxy
    ProxyPass / http://127.0.0.1:5678/
    ProxyPassReverse / http://127.0.0.1:5678/
    
    # Headers for proper forwarding
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
</VirtualHost>

<VirtualHost *:80>
    ServerName n8n.elgenix.com
    
    # Redirect all HTTP to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
</VirtualHost>
```

4. **Click "Save"**
5. **Restart OpenLiteSpeed**:
   ```bash
   sudo systemctl restart lsws
   ```

### Method 2: Direct Configuration File Edit

If CyberPanel GUI doesn't work:

```bash
# Find the config file
cd /usr/local/lsws/conf/vhosts/n8n.elgenix.com

# Edit vhost config
sudo nano vhost.conf

# Add the proxy configuration (same as above)

# Restart web server
sudo systemctl restart lsws
```

### Enable Apache Modules (if needed)

```bash
# Enable proxy modules
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo a2enmod rewrite
sudo a2enmod headers

# Restart Apache (if using Apache instead of LiteSpeed)
sudo systemctl restart apache2
```

---

## 9. SSL Certificate Setup

### Using CyberPanel's Built-in SSL (Easiest)

1. **In CyberPanel**: Go to SSL → Issue SSL
2. **Select Website**: `n8n.elgenix.com`
3. **Click**: "Issue SSL"
4. **Wait**: 30-60 seconds

CyberPanel will automatically:
- Generate Let's Encrypt certificate
- Configure SSL in web server
- Set up auto-renewal

### Verify SSL is Working

```bash
# Test from command line
curl -I https://n8n.elgenix.com

# Should return:
# HTTP/2 200
# server: LiteSpeed
```

Visit in browser: `https://n8n.elgenix.com`

**Expected:**
- Green padlock in browser
- n8n login screen

---

## 10. First Login & Configuration

### Step 1: Access n8n

1. Open browser: `https://n8n.elgenix.com`
2. You'll see n8n setup wizard

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

### Week 1: Foundation
- [x] Install n8n
- [ ] Setup Google Sheets tracker
- [ ] Configure Gemini API
- [ ] Test simple workflow

### Week 2: Job Automation
- [ ] Import job search template
- [ ] Customize CV generation
- [ ] Setup PDF generation
- [ ] Test with 5 real applications

### Week 3: WhatsApp Bot
- [ ] Setup WhatsApp Business API
- [ ] Create advice categories
- [ ] Build knowledge base in Google Docs
- [ ] Test with friends

### Week 4: Optimization
- [ ] Add Ollama for cost reduction
- [ ] Setup monitoring & alerts
- [ ] Document your workflows
- [ ] Share learnings

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
