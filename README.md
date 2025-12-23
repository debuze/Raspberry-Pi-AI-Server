# DIY Privacy-First AI Infrastructure on Raspberry Pi 4

Build your own secure, open-source AI assistant that runs entirely on your hardware. Perfect for privacy-conscious developers who want full control over their AI stack.

## 📋 Table of Contents

- [What You're Building](#what-youre-building)
- [Hardware Requirements](#hardware-requirements)
- [Reality Check: Performance Expectations](#reality-check-performance-expectations)
- [Installation Guide](#installation-guide)
- [Remote Access Setup (WireGuard VPN)](#remote-access-setup-wireguard-vpn)
- [Model Selection Guide](#model-selection-guide)
- [Security Hardening](#security-hardening)
- [Troubleshooting](#troubleshooting)
- [Maintenance & Backups](#maintenance--backups)

---

## What You're Building

A completely private AI assistant that:
- ✅ Runs 100% locally on your Raspberry Pi
- ✅ Never sends data to external servers
- ✅ Uses only open-source software
- ✅ Accessible securely from anywhere via VPN
- ✅ Costs ~$0/month to operate (just electricity)

**Tech Stack:**
- **LLM Engine:** [Ollama](https://github.com/ollama/ollama) (MIT License)
- **Web Interface:** [Open WebUI](https://github.com/open-webui/open-webui) (MIT License)
- **VPN:** WireGuard (GPLv2)
- **Container Platform:** Docker (Apache 2.0)

---

## Hardware Requirements

### Minimum Setup
- **Raspberry Pi 4 (32GB RAM)** - Required
- **64GB+ SSD or SD Card** - SSD strongly recommended for model loading speed
- **Power Supply** - Official Pi 4 power supply
- **Ethernet Connection** - WiFi works but ethernet is more stable

### Recommended Upgrade
- **Raspberry Pi 5 (8GB RAM)** - Can run larger 3B models smoothly

---

## Reality Check: Performance Expectations

### RAM Breakdown on Pi 4 (32GB)# Raspberry-Pi-AI-Server
local AI with Raspberry pi 4
### Model Performance Guide

| Model Size | RAM Usage | Speed on Pi 4 | Recommended? |
|------------|-----------|---------------|--------------|
| 0.5B params (Q4) | ~500 MB | Fast (2-3 tok/s) | ✅ Best for Pi 4 |
| 1.1B params (Q4) | ~1 GB | Good (1-2 tok/s) | ✅ Recommended |
| 1.5B params (Q4) | ~1.5 GB | Slow (0.5-1 tok/s) | ⚠️ Usable but slow |
| 3B+ params | ~3+ GB | ❌ Unusable | ❌ Too slow/OOM crashes |

**Recommended Models for Pi 4:**
- `qwen2.5:0.5b` - Apache 2.0 - Fast, good for quick queries
- `tinyllama:1.1b` - Apache 2.0 - Best balance of speed/quality
- `phi:2.7b` - MIT - Slow but better quality (Pi 5 recommended)

---

## Installation Guide

### Step 1: Install Raspberry Pi OS

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Flash **Raspberry Pi OS Lite (64-bit)** to your SSD/SD card
3. **Before writing**, click the gear icon ⚙️ and configure:
   - Set hostname: `pi-ai`
   - Create username/password
   - **Enable SSH**
   - Configure WiFi (if not using ethernet)
   - Set locale/timezone

4. Boot the Pi and SSH in:
````bash
ssh youruser@pi-ai.local
````

5. Update everything:
````bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
````

---

### Step 2: Secure SSH Access

**On your laptop/desktop** (not the Pi), create an SSH key:
````bash
# Generate key
ssh-keygen -t ed25519 -C "your-email@example.com"

# Copy key to Pi
ssh-copy-id youruser@pi-ai.local
````

**On the Pi**, disable password authentication:
````bash
sudo nano /etc/ssh/sshd_config
````

Find and set:

Restart SSH:
````bash
sudo systemctl restart ssh
````

---

### Step 3: Configure Firewall
````bash
sudo apt install ufw -y

# Allow SSH from your LAN (adjust to your network)
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp

sudo ufw enable
sudo ufw status
````

---

### Step 4: Install Docker
````bash
sudo apt install docker.io docker-compose-plugin -y
sudo usermod -aG docker $USER
newgrp docker

docker --version
````

---

### Step 5: Deploy Ollama + Open WebUI
````bash
mkdir -p ~/ai-stack && cd ~/ai-stack
nano docker-compose.yml
````

Paste this:
````yaml
services:
  ollama:
    image: ollama/ollama:0.1.29
    container_name: ollama
    volumes:
      - ./ollama:/root/.ollama
    ports:
      - "127.0.0.1:11434:11434"
    restart: unless-stopped

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama
    ports:
      - "127.0.0.1:3000:8080"
    volumes:
      - ./openwebui:/app/backend/data
    restart: unless-stopped
````

Start it:
````bash
docker compose up -d
docker ps
````

---

### Step 6: Download AI Models
````bash
# Fast model (500MB)
docker exec -it ollama ollama pull qwen2.5:0.5b

# Balanced model (1GB)
docker exec -it ollama ollama pull tinyllama

# Test it
docker exec -it ollama ollama run qwen2.5:0.5b
````

---

### Step 7: Access the Web UI

**From your laptop on the same network:**
````bash
# Create SSH tunnel (keep open)
ssh -L 3000:127.0.0.1:3000 youruser@pi-ai.local
````

**Open browser:** http://127.0.0.1:3000

Create your admin account and start chatting!

---

## Remote Access Setup (WireGuard VPN)

### Install WireGuard on Pi
````bash
sudo apt install wireguard -y
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
````

### Generate Keys
````bash
cd /etc/wireguard
sudo umask 077

# Server keys
wg genkey | sudo tee server_private.key | wg pubkey | sudo tee server_public.key

# Client keys
wg genkey | sudo tee client_private.key | wg pubkey | sudo tee client_public.key

# View keys
sudo cat server_private.key
sudo cat server_public.key
sudo cat client_private.key
sudo cat client_public.key
````

**Save these keys!**

### Configure Server
````bash
hostname -I  # Note your Pi's IP
sudo nano /etc/wireguard/wg0.conf
````

Paste (replace keys):
````ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY

PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
````

Start WireGuard:
````bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo wg show
````

### Configure Firewall
````bash
sudo ufw allow 51820/udp
sudo ufw reload
````

### Client Configuration

Find your public IP:
````bash
curl ifconfig.me
````

Create `wg0-client.conf` on your laptop:
````ini
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = YOUR_PUBLIC_IP:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
````

### Port Forward on Router

Forward UDP port 51820 to your Pi's local IP.

### Connect
````bash
sudo wg-quick up ./wg0-client.conf
ping 10.0.0.1
````

Access your AI: `http://10.0.0.1:3000`

---

## Security Hardening

### Automatic Updates
````bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
````

### Fail2Ban
````bash
sudo apt install fail2ban -y
sudo nano /etc/fail2ban/jail.local
````

Paste:
````ini
[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 3600
````
````bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
````

---

## Troubleshooting

### Web UI Won't Load
````bash
docker ps  # Check containers
cd ~/ai-stack
docker compose down
docker compose up -d
docker logs openwebui
````

### Out of Memory
````bash
free -h
docker exec -it ollama ollama rm large-model
# Use 0.5B or 1B models only
````

### VPN Won't Connect
````bash
sudo systemctl status wg-quick@wg0
sudo wg show
sudo ufw status | grep 51820
````

---

## Maintenance & Backups

### Weekly
````bash
df -h
docker ps
````

### Monthly
````bash
sudo apt update && sudo apt upgrade -y
cd ~/ai-stack
docker compose pull
docker compose up -d
````

### Backup
````bash
mkdir -p ~/backups
cd ~/ai-stack
tar -czf ~/backups/ai-stack-$(date +%Y%m%d).tar.gz ollama openwebui
````

---

## Resources

- [Ollama GitHub](https://github.com/ollama/ollama)
- [Open WebUI Docs](https://docs.openwebui.com)
- [WireGuard Documentation](https://www.wireguard.com/quickstart/)

---

## License

This guide is released under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

**Built with ❤️ for the open-source and privacy community**
