# 🧭 Heimdall Dashboard Setup on Proxmox PVE (Ubuntu/Debian CT)

This guide walks you through installing [Heimdall](https://heimdall.site/) — a web app dashboard — inside a Proxmox container using Docker and Docker Compose.

---

## 🏗️ Container Setup on Proxmox

1. In the Proxmox GUI, under your node (e.g., `PVE0`), click **"Create CT"**.
2. Go through each step and fill in the required information for:

   * Hostname
   * Password
   * OS template (use Ubuntu LTS)
   * Disk size
   * CPU & Memory
   * Network (you can switch to static IP after install)
3. Click **Finish** and start the container.

Log in to your CT via Console or SSH:

```bash
pct enter <CTID>
```

Default login (if not changed):

* **User:** `root`
* **Password:** `$nakEEater-100`

---

## 🧱 Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🐳 Step 2: Install Docker & Docker Compose

### Install Docker

```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Verify Docker:

```bash
sudo systemctl status docker
```

### Install Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

---

## 📂 Step 3: Download Heimdall with Docker Compose

```bash
mkdir -p ~/heimdall
cd ~/heimdall
nano docker-compose.yml
```

Paste this:

```yaml
version: '3'
services:
  heimdall:
    image: linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config:/config
    ports:
      - "80:80"
    restart: unless-stopped
```

---

## ▶️ Step 4: Start Heimdall

```bash
docker-compose up -d
```

Then go to:

```
http://<your-server-ip>
```

You should now see the Heimdall interface.

---

## 🔒 (Optional) Step 5: Set Up Nginx + SSL

### Install Nginx:

```bash
sudo apt install nginx
```

### Create config:

```bash
sudo nano /etc/nginx/sites-available/heimdall
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/heimdall /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

---

## 🧰 Troubleshooting

### 1. Check Docker container

```bash
docker ps
docker-compose logs
```

### 2. Port conflict?

```bash
sudo lsof -i :80
```

Change port in `docker-compose.yml` to something like:

```yaml
ports:
  - "8080:80"
```

Restart:

```bash
docker-compose down
docker-compose up -d
sudo ufw allow 8080/tcp
```

### 3. Check server IP:

```bash
hostname -I
```

Then access via:

```
http://<your-ip>:8080
```

### 4. Verify versions:

```bash
docker --version
docker-compose --version
```

### 5. Restart Docker:

```bash
sudo systemctl restart docker
```

---

## ✅ Final Verification

* Open your browser to `http://<your-server-ip>:<port>`
* Heimdall should be accessible
* Container should be running (`docker ps`)
* Logs should be clean (`docker-compose logs`)



Want me to add this into your existing GitHub repo as part of your combined documentation?
