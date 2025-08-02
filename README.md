
# 🟠 OPI Server

Mini server running on Orange Pi Zero 2W using Docker, Flask, Nginx, and ngrok.

---

## 📀 Ubuntu Image

Using the official Orange Pi Ubuntu Noble Server image:

🔗 [Download via Google Drive](https://drive.google.com/file/d/1336jjgNxg_dYrwFYVRt5GgQ4tJQBwQoC/view?usp=drive_link)  
🔗 [Official page](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2W.html)

### 📦 Flashing the SD Card

1. Extract the `.7z` file  
2. Use Balena Etcher (or similar) to flash the image to the SD card

---

## 👤 Initial Configuration

The image logs in automatically as the `orangepi` user.

### 1. Create a new user:

```bash
sudo useradd YOUR_USERNAME
````

### 2. Disable auto-login:

```bash
sudo nano /usr/lib/systemd/system/getty@.service.d/override.conf
```

Change this line:

```
ExecStart=-/sbin/agetty --noissue --autologin orangepi %I $TERM
```

To:

```
ExecStart=-/sbin/agetty --noissue %I $TERM
```

Then run:

```bash
sudo systemctl daemon-reload
sudo shutdown -r now
```

### 3. Log in with the new user

### 4. Set up keyboard and console (if needed):

```bash
sudo dpkg-reconfigure keyboard-configuration
sudo dpkg-reconfigure console-setup
sudo setupcon
```

---

## 🌐 Connect to Wi-Fi

```bash
sudo nmcli device wifi list
sudo nmcli device wifi connect "SSID_NAME" password "PASSWORD" ifname wlan0
```

---


## 🛡️ Enable Hardware Watchdog on Orange Pi
The watchdog helps automatically reboot the system in case of crashes or lost connectivity.

### Install and configure

```bash
sudo apt update
sudo apt install watchdog
sudo nano /etc/watchdog.conf
```
Uncomment or edit this lines:
```
#---The hardware timer settings
watchdog-device         = /dev/watchdog
watchdog-timeout        = 10

#---Other system settings
realtime                = yes
priority                = 1

#---Typical tests
ping                    = 8.8.8.8
ping                    = YOUR_ROUTER_IP_HERE
ping-count              = 3
interface               = eth0
```
    ⚠️ If you're using Wi-Fi instead of Ethernet, change interface = eth0 to interface = wlan0.

### Enable and start the service
```bash
sudo systemctl enable watchdog
sudo systemctl start watchdog
sudo systemctl status watchdog
```
---


## 🐳 Install Docker + UFW

### Docker:

```bash
sudo apt update && sudo apt install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $USER
```

### UFW (Firewall):

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

---

## 🚀 Clone and Run the Project

```bash
cd /opt
sudo git clone https://github.com/ander-yamamoto/opi_server.git
cd opi_server
```

### File permissions:

* **Single user:**

```bash
sudo chown -R $USER:$USER /opt/opi_server
```

* **Multi-user (team):**

```bash
sudo groupadd team_name
sudo usermod -aG team_name user1
sudo usermod -aG team_name user2

sudo chgrp -R team_name /opt/opi_server
sudo chmod -R 2775 /opt/opi_server
```

---

## 🔐 ngrok Configuration

Create a `.env` file:

```bash
sudo nano /opt/opi_server/.env
```

Add the following:

```env
NGROK_AUTHTOKEN="your_ngrok_token"
NGROK_HOSTNAME="your_ngrok_hostname"
```

---

## ▶️ Run with Docker Compose

```bash
docker compose up -d --build
```

### Access:

* Homepage:
  `http://[LOCAL_IP]` or `http://[YOUR_NGROK_HOSTNAME]`

* Flask App:
  `http://[LOCAL_IP]/flask_demo` or `http://[YOUR_NGROK_HOSTNAME]/flask_demo`

---

## 🔁 Autostart on Boot

```bash
sudo cp service/opi_server.service /etc/systemd/system/
sudo systemctl daemon-reexec
sudo systemctl enable opi_server.service
sudo systemctl start opi_server.service
sudo systemctl status opi_server.service
```
