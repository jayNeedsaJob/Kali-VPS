<h1 align="center">
  Github-VPS
</h1>
<br>
<br>

### 🐳 Installation

> [!NOTE] 
> [Github codespace terminal](https://docs.github.com/en/codespaces/developing-in-a-codespace/using-github-codespaces-with-github-cli)
---
```bash
# pulling images 
$ docker pull docker.io/kalilinux/kali-rolling

# Priviliged mode (recommended for ctf players)
$ docker run --privileged -it kalilinux/kali-rolling /bin/bash
```
## Kali default
```bash
$ apt update && apt install -y kali-linux-default
```
### Automation in new terminal session 
> kali_privs.sh
```sh
#!/bin/bash

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m'

# Checking Kali-linux docker ID
kali_id=$(docker ps -a -q)
bash='/bin/bash'

echo -e '${YELLOW}Starting another terminal kali privs${NC}'
docker start $kali_id
docker exec -it $kali_id $bash
echo -e "${YELLOW}Success..${NC}"
sleep 1.5
```
## Adding Graphical User Interface (noVNC)
> [!IMPORTANT]
> Run this script in the terminal of your Github Codespace, which is using Ubuntu OS

> setup-noVNC.sh 
```sh
#!/bin/bash

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m' 

error_exit() {
    echo -e "${RED}Error: $1${NC}" >&2
    exit 1
}
set -e

echo -e "${GREEN}Starting setup of VNC and noVNC on github codespace terminal...${NC}"

# Update and install necessary packages
echo -e "${YELLOW}1. Updating system and installing required packages...${NC}"
{
    sudo apt update
    sudo apt install -y xfce4 xfce4-goodies novnc python3-websockify python3-numpy tightvncserver htop nano neofetch
} || error_exit "Failed to update and install packages."

# Generate SSL certificate
echo -e "${YELLOW}2. Generating SSL certificate for noVNC...${NC}"
{
    mkdir -p ~/.vnc
    openssl req -x509 -nodes -newkey rsa:3072 -keyout ~/.vnc/novnc.pem -out ~/.vnc/novnc.pem -days 3650 -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=localhost"
} || error_exit "Failed to generate SSL certificate."

# Start VNC server to create initial configuration files
echo -e "${YELLOW}3. Starting VNC server to create initial configuration files...${NC}"
{
    vncserver
} || error_exit "Failed to start VNC server."

# Kill the VNC server to edit the configuration
echo -e "${YELLOW}4. Stopping VNC server to modify configuration files...${NC}"
{
    vncserver -kill :1
} || error_exit "Failed to kill VNC server."

# Backup and create new xstartup file
echo -e "${YELLOW}5. Backing up old xstartup file and creating a new one...${NC}"
{
    mv ~/.vnc/xstartup ~/.vnc/xstartup.bak

    cat <<EOL > ~/.vnc/xstartup
#!/bin/sh
xrdb \$HOME/.Xresources
startxfce4 &
EOL

    chmod +x ~/.vnc/xstartup
} || error_exit "Failed to back up and create xstartup file."

echo -e "${GREEN}Succesfully configured please run ${YELLOW}start-novcn.sh${NC}"
```
### Starting noVNC Web access 
> start-novnc.sh
```sh
#!/bin/bash

NC="\e[0m"        
RED="\033[0;31m"      
GREEN="\033[0;32m"    
YELLOW="\033[1;33m"   
BLUE="\033[1;34m"     
CYAN="\033[1;36m"     
WHITE="\033[1;37m"    
MAGENTA="\033[1;35m"  

WEB_DIR="/usr/share/novnc/"
CERT_FILE="$HOME/.vnc/novnc.pem"
LOCAL_PORT="5901"
LISTEN_PORT="6080"


# Check if the cert file exists
if [ ! -f "$CERT_FILE" ]; then
    echo -e "${RED}Error: Certificate file not found: ${BLINK}$CERT_FILE${NC}"
    exit 1
fi

# Start noVNC
echo -e "${YELLOW} Starting noVNC to enable web-based VNC access...${NC}"
websockify -D --web="$WEB_DIR" --cert="$CERT_FILE" $LISTEN_PORT localhost:$LOCAL_PORT

# Start vncserver
# Note: adjust the resolution if applicable
echo -e "${YELLOW} Starting novncserver${NC}"
vncserver -geometry 1920x1080

echo -e "${GREEN}noVNC server started on port ${WHITE}$LISTEN_PORT${WHITE}, forwarding to localhost:${WHITE}$LOCAL_PORT${NC}
```
