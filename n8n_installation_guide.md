# CyberPanel Configuration for n8n - SIMPLE STEPS

```bash
Internet
   │
   ▼
https://n8n.elgenix.com (443 SSL)
   │
   ▼
OpenLiteSpeed / CyberPanel (Reverse Proxy)
   │
   ▼
http://127.0.0.1:5678 (localhost only)
   │
   ▼
Docker Container: n8nio/n8n
   │
   ├── n8n_data        → database, credentials, workflows
   └── n8n_local_data  → files for automations
```

## Step 1: Add DNS on Namecheap

1. Login to Namecheap
2. Go to Domain List → Manage elgenix.com
3. Click "Advanced DNS"
4. Add A Record:
   - Type: A Record
   - Host: n8n
   - Value: 109.123.254.58
   - TTL: Automatic
5. Click Save

**Wait 5-10 minutes for DNS to propagate**

---

## Step 2: Create Website in CyberPanel

1. Login to CyberPanel: https://109.123.254.58:8090
2. Go to: Websites → Create Website
3. Fill in:
   - Domain Name: n8n.elgenix.com
   - Email: your email
   - Package: Default
   - SSL: ✓ CHECK THIS BOX
4. Click "Create Website"

---

## Step 3: Add External Processor in LiteSpeed

SSH into your server:
```bash
ssh root@109.123.254.58
```

Edit LiteSpeed config:
```bash
sudo nano /usr/local/lsws/conf/httpd_config.conf
```

Scroll to the BOTTOM (before the closing `</httpServerConfig>` tag).

Add this BEFORE `</httpServerConfig>`:
```xml
  <extProcessor>
    <type>proxy</type>
    <n>n8n_proxy</n>
    <address>http://127.0.0.1:5678</address>
    <maxConns>100</maxConns>
    <pcKeepAliveTimeout>60</pcKeepAliveTimeout>
    <initTimeout>60</initTimeout>
    <retryTimeout>0</retryTimeout>
    <respBuffer>0</respBuffer>
  </extProcessor>
</httpServerConfig>
```

Save: Ctrl+O, Enter, Ctrl+X

---

## Step 4: Add Rewrite Rules in CyberPanel (5 minutes)

1. In CyberPanel: Websites → List Websites
2. Click "Manage" next to n8n.elgenix.com
3. Click "vHost Conf"

### Add Rewrite Rules to This File:
You already have a rewrite section, but it's empty. Replace this section:
```apache
rewrite  {
 enable                  1
  autoLoadHtaccess        1
}
```

Add these lines replacing the above section:
```apache
rewrite  {
  enable                  1
  autoLoadHtaccess        1
  rules                   <<<END_rules
  RewriteCond %{REQUEST_URI} !^/.well-known/acme-challenge/
  RewriteRule ^(.*)$ http://n8n_proxy/$1 [P,L]
  END_rules
}
```
---

## Step 5: Restart LiteSpeed (1 minute)

In SSH:
```bash
sudo systemctl restart lsws
```

Or in CyberPanel: Dashboard → Restart LiteSpeed

---

## Step 6: Deploy n8n with Docker (5 minutes)

### On your server (SSH):

```bash
# Go to home directory
cd ~

# Create folder
mkdir n8n
cd n8n

# Upload the docker-compose.yml file here
nano docker-compose.yml
# Paste the docker-compose.yml content
#>>>> Docker compose file <<<<<<
# https://github.com/mofassair/n8n_installation/docker-compose.yml

# Save: Ctrl+O, Enter, Ctrl+X

# Start n8n
docker-compose up -d

# Check if running
docker ps


```

You should see n8n container running.

## Permissions Fix (Important)

```bash
# Stop the container first
docker-compose down
# Fix the permissions
sudo chown -R 1000:1000 n8n_data n8n_local_data
# Start again
docker-compose up -d
# Check logs
docker logs n8n -f
```
---

## Step 7: Access n8n (1 minute)

Open browser: https://n8n.elgenix.com

You should see:
- ✅ n8n welcome screen
- ✅ Green padlock (SSL working)
- ✅ NO "Connection Lost" error

Create your account and you're done! 🎉

---

## Troubleshooting

### "Connection Lost" error
```bash
cd ~/n8n
docker-compose restart
```

### Can't access n8n.elgenix.com
```bash
# Check DNS
nslookup n8n.elgenix.com
# Should show: 109.123.254.58

# Check if n8n is running
docker ps

# Check LiteSpeed
sudo systemctl status lsws
```

### 502 Bad Gateway
```bash
# Restart both
docker-compose restart
sudo systemctl restart lsws
```

---


## Backup & RESTORE

### Backup
```bash
tar -czvf n8n_backup_$(date +%Y%m%d).tar.gz n8n_data

```
### Restore

```bash
docker-compose down
rm -rf n8n_data
tar -xzvf backup.tar.gz
sudo chown -R 1000:1000 n8n_data
docker-compose up -d
```

---

## update n8n

```bash
docker-compose down
docker-compose pull
docker-compose up -d
```

---

## Health Checks

### Local
```bash
curl http://127.0.0.1:5678/healthz
```
### Public
```bash
curl https://n8n.elgenix.com/healthz
```

---

## Fast deply from github (for developer only)

Here the git will prepared such a way an developer may clone the git and run an sh file which shall do all the tasks. 


```bash



```

---

**That's it! Once it's working, we can add document processing and WhatsApp later.**
