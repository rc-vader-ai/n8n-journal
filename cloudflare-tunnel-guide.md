# Secure n8n with Cloudflare Zero Trust Tunnel: A Beginner's Guide

A step-by-step guide to securely expose your locally hosted n8n Docker instance to the internet using Cloudflare's Zero Trust Tunnel. This approach requires **no open firewall ports**, **no static IP address**, and maintains **end-to-end encryption**—making it ideal for remote access and webhook integrations.

---

## Understanding the Big Picture

Before diving into the setup, let's understand what you're building:

**The Problem:** You have n8n running locally on your computer (e.g., `http://localhost:5678`), but you need:
- External services (Slack, webhooks, APIs) to reach your n8n workflows
- Secure access from anywhere on the internet
- No open firewall ports (safer and simpler than port forwarding)

**The Solution:** Cloudflare Tunnel creates an **encrypted, outbound-only connection** from your local machine to Cloudflare's servers. When someone visits your public domain, Cloudflare safely routes the traffic back through the tunnel to your local n8n.

**Why This Works:**
- Your machine initiates the connection to Cloudflare (not the other way around)
- No incoming traffic directly hits your network
- Cloudflare handles DDoS protection and HTTPS automatically
- Works with dynamic IPs and residential internet connections

---

## Prerequisites

Ensure you have the following in place before starting:

**Account & Domain:**
- A **Cloudflare account** (free tier is sufficient). Sign up at [cloudflare.com](https://cloudflare.com)
- A **registered domain** added to Cloudflare. You can:
  - Purchase a domain from Cloudflare (~$10/year for a `.com`)
  - Transfer an existing domain to Cloudflare
  - Use a domain you already own and point its nameservers to Cloudflare

**Technical Setup:**
- **Docker and Docker Compose** installed on your machine
- **n8n running in a Docker container** (see the [n8n Docker Guide](https://github.com/YourUsername/n8n-docker-setup-guide) if you need help)
- **Terminal/Command Prompt** access on your computer
- Basic familiarity with editing text files (`.env`)

---

## Step 1: Create a Cloudflare Tunnel

A tunnel is your secure connection gateway. You'll set it up entirely in the Cloudflare web interface.

**In your browser:**

1. Log in to your Cloudflare account at [dash.cloudflare.com](https://dash.cloudflare.com)
2. Select your domain from the list
3. In the left sidebar, click **Zero Trust**
4. Click **Networks** in the left menu
5. Click the **Tunnels** tab
6. Click the blue **Create a tunnel** button
7. Select **Cloudflared** as the connector type (this is the Docker-friendly option)
8. Name your tunnel something memorable, like `n8n-tunnel`
9. Click **Save tunnel**

**Why Cloudflared?** Cloudflared is Cloudflare's official connector. It's lightweight, runs in Docker, and handles all the encryption automatically.

---

## Step 2: Obtain Your Tunnel Token

After creating the tunnel, Cloudflare provides installation instructions. You'll need the **Docker command** that contains your tunnel token.

**In the Cloudflare dashboard:**

1. After saving your tunnel, you'll see installation instructions for different operating systems
2. Look for the section labeled **Docker**
3. You'll see a command that looks like this:

```bash
# Example (do NOT use this—copy yours from Cloudflare)
docker run -d --network n8n-network cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <your-cloudflare-tunnel-token>
```

The part after `--token` is your **unique tunnel token**. This is a long string of characters that authenticates your Docker container with Cloudflare.

**Security Note:**
- **Never share this token.** It authenticates your tunnel.
- **Never commit it to GitHub** or any public place.
- We'll store it securely in your local `.env` file.

**Copy just the token part** (the long string after `--token`) and save it temporarily. You'll use it in the next step.

---

## Step 3: Update Your n8n Environment Configuration

Now you'll configure n8n to work with the Cloudflare Tunnel. Update your `.env` file to include your public domain and the tunnel token.

**Edit or create your `.env` file:**

```bash
# .env file (store this in your project root)

# === NETWORK CONFIGURATION ===
N8N_HOST=n8n.yourdomain.com        # Replace with your actual domain
N8N_PORT=5678
N8N_PROTOCOL=https                  # Cloudflare provides HTTPS automatically

# === WEBHOOK & PUBLIC ACCESS ===
# Critical for external services to reach n8n
WEBHOOK_URL=https://n8n.yourdomain.com/
N8N_EDITOR_BASE_URL=https://n8n.yourdomain.com/

# === SECURITY ===
# Generate a strong encryption key: openssl rand -base64 32
N8N_ENCRYPTION_KEY=<your-encryption-key>

# === TUNNEL TOKEN ===
# Paste the token you copied from Cloudflare
TUNNEL_TOKEN=<your-cloudflare-tunnel-token>

# === ADVANCED SETTINGS (Optional) ===
NODE_ENV=production
GENERIC_TIMEZONE=UTC                # Change to your timezone if desired
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
N8N_RUNNERS_ENABLED=true

# === DATABASE ===
DB_TYPE=sqlite                      # For production, use postgresdb
```

**Important replacements:**
- `yourdomain.com` → Your actual Cloudflare domain (e.g., `example.com`)
- `n8n.yourdomain.com` → Your chosen subdomain (e.g., `n8n.example.com`)
- `<your-encryption-key>` → Generate with: `openssl rand -base64 32`
- `<your-cloudflare-tunnel-token>` → Paste the token from Cloudflare

**Security reminders:**
- Add `.env` to your `.gitignore` to prevent accidental commits
- Never store real tokens in your `docker-compose.yml`; always use `.env` files
- Keep this file private and backed up safely

---

## Step 4: Create Your Docker Compose File

Instead of running multiple `docker run` commands, use Docker Compose to manage both n8n and cloudflared together.

**Create `docker-compose.yml` in your project directory:**

```yaml
services:
  # n8n Service
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      # Network settings
      - N8N_HOST=${N8N_HOST}              # From .env
      - N8N_PORT=5678
      - N8N_PROTOCOL=${N8N_PROTOCOL}      # https
      
      # Public URLs for webhooks and external access
      - WEBHOOK_URL=${WEBHOOK_URL}
      - N8N_EDITOR_BASE_URL=${N8N_EDITOR_BASE_URL}
      
      # Security
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      
      # Performance
      - NODE_ENV=production
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - N8N_RUNNERS_ENABLED=true
    
    volumes:
      - n8n_data:/home/node/.n8n          # Persistent data storage
    
    networks:
      - n8n-network
    
    # Optional: Health check to monitor container status
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5678/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3
    
    depends_on:
      - cloudflared

  # Cloudflare Tunnel Service
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    
    # The tunnel command connects to Cloudflare's infrastructure
    command: tunnel run
    
    environment:
      # Cloudflared automatically reads TUNNEL_TOKEN from .env
      - TUNNEL_TOKEN=${TUNNEL_TOKEN}
    
    networks:
      - n8n-network

# Persistent volume for n8n data (workflows, credentials, execution history)
volumes:
  n8n_data:
    driver: local

# Custom network allows n8n and cloudflared to communicate
networks:
  n8n-network:
    driver: bridge
```

**Key concepts explained:**

| Component | Purpose |
|-----------|---------|
| `${VARIABLE}` | References values from your `.env` file |
| `n8n_data` volume | Stores all n8n data persistently |
| `n8n-network` | Private network so containers can talk to each other |
| `depends_on` | Ensures n8n starts before cloudflared tries to connect |
| `healthcheck` | Monitors n8n's health and auto-restarts if needed |

---

## Step 5: Start Your Containers

Now you're ready to bring everything online.

**In your terminal:**

```bash
# Navigate to your project directory (where docker-compose.yml is located)
cd /path/to/your/n8n/docker

# Start all services in the background
docker-compose up -d

# View real-time logs (press Ctrl+C to exit)
docker-compose logs -f

# Check container status
docker-compose ps
```

**What to expect:**

You should see both containers listed as **"Up"**:

```
NAME          STATUS
n8n           Up 2 seconds (healthy)
cloudflared   Up 1 seconds
```

**If cloudflared shows as "exited":**

The tunnel token is likely incorrect. Check:
```bash
# View error logs
docker-compose logs cloudflared

# Re-verify your token in .env
cat .env | grep TUNNEL_TOKEN
```

---

## Step 6: Configure Your Public Hostname in Cloudflare

You've created a tunnel and started the containers. Now tell Cloudflare where to route incoming traffic.

**In the Cloudflare dashboard:**

1. Return to **Zero Trust** → **Networks** → **Tunnels**
2. Click on your tunnel name (e.g., `n8n-tunnel`)
3. Click the **Public Hostnames** tab
4. Click **Add a public hostname**
5. Fill in the routing details:

| Field | Value | Example |
|-------|-------|---------|
| **Subdomain** | Your chosen subdomain | `n8n` |
| **Domain** | Your Cloudflare domain | `yourdomain.com` |
| **Service Type** | `HTTP` | (select from dropdown) |
| **URL** | `localhost:5678` or `n8n:5678` | `n8n:5678` |

6. Click **Save hostname**

**Why these values matter:**
- **Subdomain + Domain** = your public URL (e.g., `n8n.yourdomain.com`)
- **Service Type:** HTTP is correct because n8n runs over HTTP internally (Cloudflare adds HTTPS on top)
- **URL:** Use `n8n:5678` because Docker Compose resolves the container name automatically through the internal network

---

## Step 7: Verify Tunnel Status

Before testing, confirm that your tunnel is properly connected.

**In the Cloudflare dashboard:**

1. Go to **Zero Trust** → **Networks** → **Tunnels**
2. Click on your tunnel
3. Look at the **Status** indicator:
   - **Healthy** or **Connected** ✓ (Good to proceed)
   - **Disconnected** ✗ (Check cloudflared logs with `docker-compose logs cloudflared`)

4. Verify your hostname is listed under **Public Hostnames**

**If you see "Bad Gateway" or connection errors:**

Common causes and fixes:

| Issue | Cause | Fix |
|-------|-------|-----|
| Service URL incorrect | You used `localhost:5678` instead of `n8n:5678` | Update in Cloudflare dashboard; Docker containers can't resolve `localhost` |
| n8n container not running | The n8n service crashed | Run `docker-compose ps` and `docker-compose logs n8n` |
| Network misconfiguration | Containers on different networks | Verify both use `n8n-network` in docker-compose.yml |
| Old token | Token expired or incorrect | Re-copy from Cloudflare and update `.env` |

---

## Step 8: Verify n8n Configuration for Webhooks

This is the **most critical step for webhook functionality**. When external services (Slack, APIs, webhooks) send requests to n8n, they need the correct public URLs.

**In your terminal, verify your environment variables:**

```bash
# Check that n8n has the correct configuration
docker-compose exec n8n env | grep -E "N8N_HOST|N8N_PROTOCOL|WEBHOOK_URL"
```

Expected output:
```
N8N_HOST=n8n.yourdomain.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.yourdomain.com/
```

**If values are incorrect:**

1. Update your `.env` file
2. Restart the containers:
   ```bash
   docker-compose restart n8n
   docker-compose logs -f n8n
   ```
3. Wait 10-15 seconds for n8n to reinitialize

**Why this matters:**

When you create a webhook trigger in n8n, it automatically generates a webhook URL based on these environment variables:

- ✓ Correct: `https://n8n.yourdomain.com/webhook/my-trigger`
- ✗ Wrong: `http://localhost:5678/webhook/my-trigger` (external services can't reach localhost)

---

## Step 9: Test Your Setup

Now test that everything works end-to-end.

### Test 1: Access n8n via Your Public URL

1. Open your browser and navigate to `https://n8n.yourdomain.com`
2. You should see the n8n login screen
3. Check for a **green lock icon** (indicates HTTPS is working)
4. Log in with your credentials

### Test 2: Verify Webhook URLs

1. In n8n, create a **new workflow**
2. Add a **Webhook trigger** node
3. Click on the trigger to open its settings
4. Look at the **Webhook URL** field
5. It should display your public HTTPS domain, e.g.: `https://n8n.yourdomain.com/webhook/abc123...`

✓ **If the URL shows your domain:** Webhook configuration is correct.
✗ **If it shows localhost:** There's a configuration issue. Re-check Step 8.

### Test 3: Send a Test Webhook

1. Copy the webhook URL from Step 9.2
2. Open a terminal and use `curl` to send a test request:

```bash
# Replace with your actual webhook URL
curl -X POST https://n8n.yourdomain.com/webhook/abc123... \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

Or use **Postman** or **Insomnia** to send a POST request to your webhook URL.

3. Check if the workflow executed:
   - Go to **Executions** in n8n
   - You should see your test webhook listed

✓ **Execution appears:** Your webhook is working!
✗ **No execution:** Check logs with `docker-compose logs n8n`

---

## Step 10: Common Configuration Mistakes and Fixes

### Mistake 1: Trailing Slash in WEBHOOK_URL

**Wrong:**
```bash
WEBHOOK_URL=https://n8n.yourdomain.com
```

**Correct:**
```bash
WEBHOOK_URL=https://n8n.yourdomain.com/
# Note the trailing slash
```

### Mistake 2: Protocol Mismatch

**Wrong:**
```bash
N8N_PROTOCOL=http
# External services expect https through Cloudflare
```

**Correct:**
```bash
N8N_PROTOCOL=https
```

### Mistake 3: Hardcoded Token in Docker Compose

**Wrong:**
```yaml
environment:
  - TUNNEL_TOKEN=<your-cloudflare-tunnel-token>
```

**Correct:**
```yaml
environment:
  - TUNNEL_TOKEN=${TUNNEL_TOKEN}
# Token is read from .env file
```

### Mistake 4: Using Localhost Instead of Container Name

**Wrong (in Cloudflare Public Hostname):**
```
URL: localhost:5678
```

**Correct:**
```
URL: n8n:5678
# Docker resolves the container name automatically
```

---

## Updating and Maintaining Your Setup

### Update n8n to the Latest Version

```bash
# Pull the latest image
docker-compose pull n8n

# Restart n8n with the new image
docker-compose up -d n8n

# Check logs for successful startup
docker-compose logs -f n8n
```

Your data in `n8n_data` volume persists automatically.

### Update Cloudflared

```bash
# Pull the latest tunnel image
docker-compose pull cloudflared

# Restart cloudflared
docker-compose up -d cloudflared

# Verify tunnel is connected
docker-compose logs -f cloudflared
```

### Monitor Tunnel Health

```bash
# View real-time tunnel status
docker-compose logs -f cloudflared

# Expected healthy output includes:
# "tunnel registered successfully"
# No error messages about tokens or connections
```

---

## Security Best Practices

### 1. Protect Your Environment Variables

```bash
# Add to .gitignore
echo ".env" >> .gitignore

# Never commit .env to version control
git status  # Verify .env is not listed
```

### 2. Use Strong Encryption Keys

```bash
# Generate a new encryption key
openssl rand -base64 32

# Update in .env
N8N_ENCRYPTION_KEY=<paste-generated-key>

# Restart n8n
docker-compose restart n8n
```

### 3. Enable Cloudflare Access (Additional Layer)

For sensitive n8n instances, add Cloudflare Access for email-based authentication:

1. Go to **Zero Trust** → **Access** → **Applications**
2. Create a new application
3. Configure authentication (email, SSO, etc.)
4. Assign to your tunnel's hostname

This adds a login screen **before** reaching n8n.

### 4. Keep Images Updated

```bash
# Check for updates
docker-compose pull --dry-run

# Apply updates
docker-compose pull
docker-compose up -d
```

### 5. Monitor Logs for Suspicious Activity

```bash
# Review cloudflared logs for connection issues
docker-compose logs cloudflared | tail -50

# Review n8n logs for unauthorized access attempts
docker-compose logs n8n | grep -i error
```

---

## Troubleshooting Reference

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| `cloudflared` container exits immediately | Invalid tunnel token | Re-copy token from Cloudflare; verify in `.env` |
| "Bad Gateway" error | n8n container not running or wrong service URL | Check `docker-compose ps`; verify URL in Cloudflare (use `n8n:5678`, not `localhost:5678`) |
| Webhook shows `localhost` URL | n8n didn't read correct environment variables | Restart n8n with `docker-compose restart n8n`; wait 15 seconds |
| HTTPS connection error | Cloudflare certificate issue | Clear browser cache; try incognito mode |
| Can't reach n8n from internet | Tunnel disconnected | Check `docker-compose logs cloudflared` for errors |
| n8n won't start | Database or permission issues | Check logs: `docker-compose logs n8n` |

---

## Summary: What You've Built

You now have:

✓ **n8n running locally** in a Docker container  
✓ **Secure Cloudflare Tunnel** connecting your local instance to the internet  
✓ **Public HTTPS domain** (e.g., `https://n8n.yourdomain.com`)  
✓ **Working webhook support** for external services  
✓ **No open firewall ports** (safer than port forwarding)  
✓ **No static IP requirement** (works with dynamic/residential networks)  
✓ **End-to-end encryption** for all traffic  
✓ **Persistent data** stored safely in Docker volume  

---

## Next Steps

With your n8n instance now publicly accessible:

1. **Create workflows** that integrate external services (Slack, Google Sheets, APIs)
2. **Set up webhook triggers** for automated workflows
3. **Enable authentication** with Cloudflare Access for additional security
4. **Monitor tunnel health** regularly with `docker-compose logs`
5. **Back up your data** periodically (see n8n Docker Guide)
6. **Scale up** with PostgreSQL instead of SQLite when handling many workflows

---

## Additional Resources

- **Cloudflare Zero Trust Documentation:** [developers.cloudflare.com/zero-trust](https://developers.cloudflare.com/zero-trust)
- **n8n Webhook Documentation:** [docs.n8n.io](https://docs.n8n.io)
- **Docker Compose Reference:** [docs.docker.com/compose](https://docs.docker.com/compose)

Happy automating!
