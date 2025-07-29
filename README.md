# opi_server
Mini server running on Orange PI zero 2w

### Ubuntu Image
Running Ubuntu Noble Server from Orangepi official image:
https://drive.google.com/file/d/1336jjgNxg_dYrwFYVRt5GgQ4tJQBwQoC/view?usp=drive_link

For other options, check official website
http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2W.html

Unzip 7z file and flash sdcard using balena-etcher or any other sd-card flash tool.

At first login, orange pi image will auto login to user orangepi.

first add new user:
```
sudo useradd "new-user"
```
then edit:
```
sudo nano /usr/lib/systemd/system/getty@.service.d/override.conf
```
change line:  
ExecStart=-/sbin/agetty --noissue --autologin orangepi %I $TERM  
for:  
ExecStart=-/sbin/agetty --noissue %I $TERM
```
sudo systemctl daemon-reload
sudo shutdown -r now
```
This will prevent autologin.
Login with your "new-user" after reboot.  
If needed, configure keyboard and console:
```
sudo dpkg-reconfigure keyboard-configuration
sudo dpkg-reconfigure console-setup
sudo setupcon
```
To connect to wifi use:
```
sudo nmcli device wifi list
sudo nmcli device wifi connect "WIFI-ID" password "PASSWORD" ifname wlan0
```

### Install docker and UFW
```
# Docker installation
sudo apt update && sudo apt install -y \
    ca-certificates curl gnupg lsb-release

# Adiciona a chave oficial do Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Adiciona o repositório do Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instala Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Permite usar docker sem sudo
sudo usermod -aG docker $USER
```

```
# install ufw
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing

# open permission for ssh and http
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443

sudo ufw enable
```

### Clone and run this project
```
cd /opt
sudo git clone https://github.com/ander-yamamoto/opi_server.git
cd opi_server
```

To work with vscode using ssh, change permission on this folder:
If working as single user:
```
sudo chown -R $USER:$USER /opt/opi_server
```
If working as a team:
```
# create group
sudo groupadd team_name

# add users to the team group
sudo usermod -aG team_name "team_user_1"
sudo usermod -aG team_name "team_user_2"
#...

# give write permission to team
sudo chgrp -R team_name /opt/opi_server
sudo chmod -R 2775 /opt/opi_server
```
```
# create env file
sudo nano /opt/opi_server/.env
```
add lines:  
NGROK_AUTHTOKEN="your_ngrok_auth_token"
NGROK_HOSTNAME="you_ngrok_hostname"

```
docker compose up -d --build
```
Orange PI should start serving webpage on:
[orangepi IP]
and
[your_ngork_hostname]

Orange PI should start serving flask app on:
[orangepi IP]/flask_demo
and
[your_ngork_hostname]/flask_demo


