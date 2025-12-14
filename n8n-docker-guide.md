# Host n8n with Docker: A Beginner's Guide

A comprehensive walkthrough for deploying n8n—a powerful workflow automation platform—using Docker. Whether you're new to containerization or automation workflows, this guide breaks down each step into manageable, understandable concepts.

**Reference:** [osher.com.au/blog/how-to-host-n8n-with-docker](https://osher.com.au/blog/how-to-host-n8n-with-docker/#configuring-n8n-for-production-use)

---

## Part 1: Setting Up Docker

### Installing Docker on macOS

Docker Desktop is your one-stop installation for Docker on macOS. It includes Docker Engine (the core runtime) and Docker Compose (for orchestrating multiple containers).

**Installation steps:**

1. Visit the [official Docker website](https://www.docker.com/products/docker-desktop)
2. Download Docker Desktop for your Mac (Intel or Apple Silicon)
3. Run the installer and follow the on-screen prompts
4. Launch Docker Desktop from your Applications folder

Once installed, Docker runs silently in the background—you interact with it via your terminal.

### Verifying Docker Installation

Confirm that Docker installed correctly by opening your terminal and running:

```bash
# Check Docker version
docker --version
```

You should see output similar to:

```
Docker version 29.1.2, build 890dcca
```

To verify the Docker engine is functional, run a test container:

```bash
# This downloads and runs a lightweight "hello-world" test image
# If successful, you'll see a confirmation message
docker run hello-world
```

If both commands succeed, Docker is ready to use.

### Understanding Docker Concepts: Images, Containers, and Volumes

Docker's power comes from three core concepts. Understanding these will make everything else click into place.

#### **Images**

An **image** is a lightweight, self-contained package containing your application and all its dependencies (runtime, libraries, configurations).

*   Think of it as a **blueprint** or **template** — read-only and unchanging.
*   The n8n Docker image contains the n8n application pre-configured and ready to run.
*   Images are stored on Docker Hub (a public registry) or your local machine.

**Example:** When you run `docker pull n8nio/n8n:latest`, you're downloading the official n8n image.

#### **Containers**

A **container** is a **running instance** of an image.

*   When you start an image, Docker creates an isolated, running container from it.
*   Containers are ephemeral—if you stop or delete one, any changes inside are lost (unless stored in a volume).
*   Multiple containers can run from the same image, each completely isolated from the others.

**Analogy:** If an image is a blueprint for a house, a container is an actual house built from that blueprint.

#### **Volumes**

A **volume** is external storage that persists **even if the container is deleted**.

*   Containers are meant to be temporary and replaceable. Your data should not be.
*   Volumes sit outside the container and survive container deletions or updates.
*   For n8n, volumes store your workflows, credentials, and execution history.

**Critical concept:** Always use volumes for production data. Never rely on container storage.

---

## Part 2: Preparing for n8n Deployment

### Selecting the Right n8n Image

The official n8n Docker image is available at `n8nio/n8n` (note the organization name is **n8nio**, not n8n).

*   **Image:** `n8nio/n8n`
*   **Recommended tag:** `latest` (points to the most recent stable release)
*   **Pull command:** `docker pull n8nio/n8n:latest`

### Creating a Docker Network for n8n

A dedicated Docker network is optional but highly recommended for production setups. It provides:

*   **Isolation:** n8n operates in its own network namespace, isolated from unrelated containers.
*   **Service discovery:** Containers on the same network can communicate using service names (e.g., `db` instead of an IP address).
*   **Security:** Control which containers can talk to each other.

**Create the network:**

```bash
# Create a custom bridge network named n8n-network
docker network create n8n-network
```

**Verify:**

```bash
# List all Docker networks
docker network ls

# You should see n8n-network in the output
```

### Setting Up Environment Variables

**Environment variables** configure n8n's behavior without requiring code changes. They control the hostname, port, database type, encryption, and more.

Instead of hardcoding values in your commands, create a `.env` file in your project directory:

```bash
# .env file (store this in your project root)

# Network Configuration
N8N_HOST=localhost              # The hostname or IP where n8n runs
N8N_PORT=5678                  # The port n8n listens on (default)
N8N_PROTOCOL=http              # Use https in production

# Security - CRITICAL: Generate a strong random key
# Example: openssl rand -base64 32
N8N_ENCRYPTION_KEY=<your-encryption-key>

# File Permissions & Features
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true    # Restrict config file access
N8N_RUNNERS_ENABLED=true                      # Enable task runners for parallel execution

# Database Configuration
DB_TYPE=sqlite                  # Options: sqlite, postgresdb, mysqldb
                               # SQLite is simple for testing; use PostgreSQL for production

# PostgreSQL Database (uncomment and configure when using postgresdb)
# DB_POSTGRESDB_HOST=db
# DB_POSTGRESDB_DATABASE=n8n
# DB_POSTGRESDB_USER=n8n
# DB_POSTGRESDB_PASSWORD=<your-password>
```

**Important security notes:**

- **Never commit `.env` to version control.** Add it to `.gitignore` immediately.
- Replace `<your-encryption-key>` with a strong, randomly generated key. Generate one with: `openssl rand -base64 32`
- Replace `<your-password>` with a strong, unique password different from your encryption key.
- Use different passwords for different services—never reuse database passwords across systems.
- Store real credentials only in your local `.env` or secure secret management tools, never in documentation or GitHub.

---

## Part 3: Deploying n8n with Docker

### Pulling the n8n Docker Image

Before running n8n, download the image from Docker Hub:

```bash
# Download the latest n8n image
docker pull n8nio/n8n:latest

# Verify the image exists locally
docker images
# You should see n8nio/n8n in the list
```

---

## Part 4: Configuring n8n for Production Use

### Setting Up Persistent Storage with Docker Volumes

**This is the most critical step.** Without a volume, restarting the container deletes all your data.

**Step 1: Create a named volume**

```bash
# Create a named volume that will hold n8n data
docker volume create n8n_data
```

**Step 2: Run the container with the volume**

Execute the following command to launch n8n with persistence:

```bash
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  --network n8n-network \
  --env-file .env \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:latest
```

**Command Explanation:**

| Flag | Purpose |
|------|---------|
| `-d` | **Detached mode** — runs the container in the background so you don't see logs spill across your terminal. |
| `--name n8n` | Assigns a static name so you can reference it easily (e.g., `docker stop n8n`). |
| `--restart unless-stopped` | Auto-restart if the container crashes or the host reboots. Stops only if you manually stop it. |
| `-p 5678:5678` | **Port mapping** — maps your machine's port 5678 to the container's port 5678 (format: `host:container`). |
| `--network n8n-network` | Connects the container to the `n8n-network` we created earlier. |
| `--env-file .env` | Reads all environment variables from your `.env` file. |
| `-v n8n_data:/home/node/.n8n` | **Volume mount** — stores n8n's internal data (`/home/node/.n8n`) in the Docker volume (`n8n_data`). This is where workflows and credentials live. |
| `n8nio/n8n:latest` | The image to run. |

**Step 3: Verify the container is running**

```bash
# List all running containers
docker ps

# You should see a container named "n8n" with status "Up"
```

To check if there were any startup errors:

```bash
# View container logs
docker logs n8n

# Follow logs in real-time (press Ctrl+C to exit)
docker logs -f n8n
```

### Accessing the n8n Dashboard

Once the container is running, access n8n in your browser:

1. Open `http://localhost:5678`
2. You'll see the n8n welcome screen
3. Create your admin account by following the on-screen prompts
4. Log in and begin building workflows

Congratulations! n8n is now running in a Docker container with persistent storage.

---

## Part 5: Managing and Maintaining Your n8n Instance

### Updating n8n to the Latest Version

Docker containers are immutable—you update by pulling a new image and replacing the old container.

**Update process:**

```bash
# Step 1: Pull the latest image
docker pull n8nio/n8n:latest

# Step 2: Stop the running container
docker stop n8n

# Step 3: Remove the old container
# (The volume n8n_data is NOT deleted, so your data is safe)
docker rm n8n

# Step 4: Start a new container with the updated image
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  --network n8n-network \
  --env-file .env \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:latest

# Step 5: Verify the update
docker logs n8n
# Look for the new version number in the startup logs
```

Since your data is stored in the volume (not inside the container), the new container starts with all your workflows and credentials intact.

### Backing Up and Restoring n8n Data

Regular backups protect you against accidental deletion or system failures.

#### **Backing Up:**

This command uses a temporary helper container to copy data out of your secure volume:

```bash
# Create a local backup folder
mkdir n8n_backup

# Copy data from the Docker volume to your local folder
# (Uses Alpine Linux as a lightweight helper to do the copy)
docker run --rm \
  -v n8n_data:/source \
  -v "$(pwd)/n8n_backup:/backup" \
  alpine cp -r /source/. /backup

# Compress the backup for safe storage
tar -czvf n8n_backup_$(date +%Y%m%d).tar.gz n8n_backup

# Optional: Delete the temporary folder
rm -rf n8n_backup
```

This creates a compressed file like `n8n_backup_20251213.tar.gz` that you can store offline.

#### **Restoring:**

If you need to restore from a backup:

```bash
# Step 1: Stop n8n
docker stop n8n

# Step 2: Extract the backup archive
tar -xzvf n8n_backup_20251213.tar.gz

# Step 3: Copy data back into the Docker volume
# (This overwrites the current data in the volume)
docker run --rm \
  -v n8n_data:/destination \
  -v $(pwd)/n8n_backup:/backup \
  alpine cp -r /backup/. /destination

# Step 4: Restart n8n
docker start n8n

# Verify the restore
docker logs -f n8n
```

---

## Part 6: Troubleshooting Common Issues

### Container Startup Problems

**Symptom:** Container crashes immediately or fails to start.

**Diagnosis steps:**

```bash
# Step 1: Check the logs for error messages
docker logs n8n

# Step 2: Inspect the container configuration
docker inspect n8n

# Step 3: Check if port 5678 is already in use
# (Run on macOS/Linux)
lsof -i :5678

# If something else is using the port, either:
# - Stop that service, OR
# - Change N8N_PORT in your .env file and re-run the container
```

**Common errors and solutions:**

| Error | Cause | Solution |
|-------|-------|----------|
| `Bind for 0.0.0.0:5678 failed` | Another service is using port 5678 | Change `N8N_PORT` in `.env` or stop the conflicting service |
| `failed to open database` | Database file is corrupted or inaccessible | Delete and recreate the volume: `docker volume rm n8n_data && docker volume create n8n_data` |
| `Permission denied` | Volume has incorrect permissions | Run: `docker exec n8n chown -R 1000:1000 /home/node/.n8n` |

### Network Connectivity Issues

**Symptom:** n8n can't connect to external services or the internet.

**Diagnostic steps:**

```bash
# Test DNS resolution (can n8n reach google.com?)
docker exec n8n ping -c 4 google.com

# Inspect the n8n-network to see connected containers
docker network inspect n8n-network

# Check which port the container is listening on
docker port n8n

# Test if the host firewall is blocking port 5678
# (On macOS, this is less common, but worth checking)
```

**Common causes:**

1. **DNS issues:** The container can't resolve domain names. Check `/etc/resolv.conf` inside the container.
2. **Firewall:** The host firewall blocks Docker's network interface.
3. **Network misconfiguration:** The container isn't connected to the right Docker network.

### Database Connection Errors

**Symptom:** "Cannot connect to database" or similar errors.

**If using SQLite (default):**

```bash
# Check if the SQLite database file exists and has correct permissions
docker exec n8n ls -la /home/node/.n8n/

# Typical output should show a file like "database.sqlite"
# If missing, n8n will auto-create it on next restart
```

**If using PostgreSQL or MySQL:**

Verify these environment variables in your `.env`:

```bash
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=db          # Service name (if using Docker Compose) or IP
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=<your-password>
```

Then test connectivity:

```bash
# Test if n8n can reach the database server
docker exec n8n ping -c 4 <DATABASE_HOST>

# Or use the appropriate database client
docker exec n8n psql -h <DATABASE_HOST> -U n8n -d n8n
```

---

## Part 7: Best Practices and Optimisation

### Optimising Docker Performance for n8n

#### **1. Set Resource Limits**

Prevent n8n from consuming all your system's CPU and memory:

```bash
docker run -d \
  --name n8n \
  --cpus 2 \
  --memory 2g \
  --memory-swap 2g \
  ... (other flags)
  n8nio/n8n:latest
```

**Flags explained:**

*   `--cpus 2`: Limit n8n to 2 CPU cores
*   `--memory 2g`: Limit n8n to 2 GB of RAM
*   `--memory-swap 2g`: Prevent swap usage (keep it equal to `--memory`)

**Check currently set limits:**

```bash
# View resource limits for your container
docker inspect n8n --format='CPU: {{.HostConfig.NanoCpus}}, Memory: {{.HostConfig.Memory}}'
```

#### **2. Verify Your Storage Driver**

Docker's storage driver affects performance. `overlay2` is the modern, efficient choice:

```bash
# Check your current storage driver
docker info | grep 'Storage Driver'

# Output should show: Storage Driver: overlay2
```

If you're using an older driver, consult Docker documentation to upgrade.

#### **3. Regular Cleanup**

Over time, unused images and stopped containers consume disk space.

```bash
# Safe cleanup (removes only stopped containers and dangling images)
docker system prune

# Aggressive cleanup (removes ALL unused images)
# ⚠️ Use with caution — this deletes images even if you might want them later
docker system prune -a

# See what will be removed without actually removing it
docker system prune --dry-run
```

| Command | What Gets Deleted |
|---------|-------------------|
| `docker system prune` | Stopped containers, unused networks, dangling images (untagged/unnamed), build cache |
| `docker system prune -a` | **All unused images**, even named ones with no running containers |

### Implementing Proper Security Measures

#### **1. Run as a Non-Root User**

By default, Docker containers run as root, which poses a security risk. Restrict privileges:

```bash
docker run -d \
  --name n8n \
  --user 1000:1000 \
  ... (other flags)
  n8nio/n8n:latest
```

#### **2. Use Environment Variables for Secrets**

Never hardcode API keys or passwords in your Dockerfile or commands. Use your `.env` file:

```bash
# ✓ GOOD: Reference variables from .env
--env-file .env

# ✗ BAD: Hardcoding secrets
docker run ... -e N8N_ENCRYPTION_KEY=<your-encryption-key> ...
```

#### **3. Enable Security Options**

Prevent containers from gaining additional privileges during runtime:

```bash
docker run -d \
  --name n8n \
  --security-opt no-new-privileges \
  ... (other flags)
  n8nio/n8n:latest
```

#### **4. Keep Everything Updated**

Regularly update n8n, Docker, and your operating system:

```bash
# Update n8n (see Part 5: Updating n8n)
docker pull n8nio/n8n:latest

# Update Docker Desktop on macOS
# → Check for updates in Docker Desktop menu → Settings
```

---

## Part 8: Scaling n8n with Docker Compose

As your n8n usage grows, managing containers via CLI becomes cumbersome. **Docker Compose** solves this by defining your entire infrastructure in a single YAML file.

### What is Docker Compose?

Docker Compose orchestrates multiple containers as a single application stack. Instead of running many `docker run` commands, you define everything once and manage it with simple commands like `docker-compose up` or `docker-compose down`.

### Creating a Docker Compose Setup

Create a file named `docker-compose.yml` in your project directory:

```yaml
version: '3.8'

services:
  # n8n service
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    environment:
      # Use environment variables from your .env file
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db          # "db" is the service name (Docker Compose DNS)
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=<your-password>
      - N8N_ENCRYPTION_KEY=<your-encryption-key>
    volumes:
      - n8n_data:/home/node/.n8n       # Persistent storage
    depends_on:
      - db                             # Wait for db service to start first
    networks:
      - n8n-network
    restart: unless-stopped

  # PostgreSQL database service (optional but recommended for production)
  db:
    image: postgres:13
    container_name: n8n_db
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=<your-password>
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n-network
    restart: unless-stopped

# Define persistent volumes
volumes:
  n8n_data:
  postgres_data:

# Define custom network
networks:
  n8n-network:
    driver: bridge
```

**Key concepts:**

*   **`services`:** Each service is a container. Here we have `n8n` and `db`.
*   **`environment`:** Configure the container's environment variables.
*   **`volumes`:** Define which volumes each service uses.
*   **`depends_on`:** Ensures the `db` service starts before `n8n` tries to connect to it.
*   **`networks`:** Services on the same network can communicate using service names (e.g., `n8n` can reach the database via hostname `db`).

**Security reminder:** When using Docker Compose in production, define sensitive values like `<your-password>` and `<your-encryption-key>` in your `.env` file and reference them in the YAML using `${VARIABLE_NAME}` syntax. Never hardcode real credentials in your `docker-compose.yml`.

### Managing Your Stack

**Start the entire stack:**

```bash
# Start all services in the background
docker-compose up -d

# View logs from all services
docker-compose logs -f

# View logs for a specific service
docker-compose logs -f n8n
```

**Stop the stack:**

```bash
# Stop all services (containers are not deleted, volumes are preserved)
docker-compose down

# Stop and remove all containers, but keep volumes
docker-compose down -v  # Remove volumes too (⚠️ use carefully)
```

**Scale n8n (advanced):**

If you want multiple n8n instances for load balancing:

```bash
# Scale the n8n service to 3 instances
docker-compose up -d --scale n8n=3

# Note: This requires a shared database and reverse proxy (nginx/Traefik)
```

---

## Summary: Key Takeaways

1. **Images, Containers, Volumes:** Master these three concepts and Docker becomes intuitive.
2. **Always Use Volumes:** Never rely on container storage for production data.
3. **Environment Variables:** Use `.env` files to keep configurations organized and secure.
4. **Start Simple, Scale Gradually:** Begin with a single Docker command, move to Docker Compose as complexity grows.
5. **Backup Regularly:** Automate backups of your `n8n_data` volume.
6. **Monitor and Update:** Keep n8n and Docker up-to-date for security and performance.
7. **Docker Compose is Production-Ready:** Use it from day one to make your setup reproducible and scalable.
8. **Protect Your Credentials:** Never commit `.env` files or real passwords to version control. Use placeholders in documentation and store real values securely.

---

## Next Steps

Now that n8n is running in Docker:

*   **Explore the Dashboard:** Build your first workflow
*   **Connect Integrations:** Link external services (Slack, Google Sheets, REST APIs, etc.)
*   **Set Up Webhooks:** Trigger workflows from external events
*   **Implement Error Handling:** Add fallback logic to your workflows
*   **Consider High Availability:** Use Docker Swarm or Kubernetes for production deployments across multiple servers

Happy automating!