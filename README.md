# odoo-on-docker-on-ubuntu-server
# allow root user ubuntu 
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# Restart SSH service
sudo systemctl restart ssh
# set passwd SSH login as a root user
sudo passwd  
# remote to server with root user  
ssh root@ubuntu-server  
# Install Docker  
# Step 1: Install Docker on Ubuntu Server
# 🔹 1.1 Update and install dependencies 
  sudo apt update   
  sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg   
# 🔹 1.2 Add Docker GPG key
 
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg   
# 🔹 1.3 Add Docker APT repository
 
 
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
    https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null   
# 🔹 1.4 Install Docker Engine 
  sudo apt update
  sudo apt install -y docker-ce docker-ce-cli containerd.io
# 🔹 1.5 Enable and start Docker 
  sudo systemctl enable docker
  sudo systemctl start docker
# 🔹 1.6 Verify Docker is installed 
  docker --version
# ⚙️ Step 2: Install Docker Compose
# Note: Starting with Docker version 20.10, docker compose is a built-in plugin. If you want to use the legacy docker-compose binary, follow below
# 🔹 2.1 Install Docker Compose (plugin) 
  sudo apt install -y docker-compose-plugin
# Check version: 
  docker compose version
# 👤 Step 3: Optional – Run Docker without sudo 
  sudo usermod -aG docker $USER
# ✅ Step 1: Create Project Directory 
  mkdir -p ~/odoo-docker/config ~/odoo-docker/odoo
  cd ~/odoo-docker
# ✏️ Step 2: Create odoo.conf
  nano config/odoo.conf
  # Paste this: 
  [options]
  admin_passwd = admin
  db_host = db
  db_port = 5432
  db_user = odoo
  db_password = odoo
  addons_path = /mnt/extra-addons
  logfile = /var/log/odoo/odoo.log
# 🐳 Step 3: Create docker-compose.yml 
  nano docker-compose.yml
  # Paste this:
    version: '3.1'
    services:
      db:
        image: postgres:15
        container_name: odoo-db
        environment:
          - POSTGRES_DB=postgres
          - POSTGRES_USER=odoo
          - POSTGRES_PASSWORD=odoo
        volumes:
          - db-data:/var/lib/postgresql/data 
      odoo:
        image: odoo:18
        container_name: odoo-app
        depends_on:
          - db
        ports:
          - "8069:8069"
        volumes:
          - ./config/odoo.conf:/etc/odoo/odoo.conf
          - ./odoo:/mnt/extra-addons
        environment:
          - HOST=db
          - USER=odoo
          - PASSWORD=odoo
        command: ["odoo", "-c", "/etc/odoo/odoo.conf"]
    volumes:
      db-data:
      
# 🚀 Step 4: Start Odoo 
  docker compose up -d
# Check logs: 
  docker compose logs -f
# 🌐 Step 5: Access Odoo
  Open your browser 
  Go to: http://<YOUR_SERVER_IP>:8069

# Option 2: Access the log file inside the container
# You defined logfile = /var/log/odoo/odoo.log in odoo.conf, so you can access it by entering the container:
  docker exec -it odoo-app bash
# to watch it live:
  tail -f /var/log/odoo/odoo.log
# docker remove odoo run
  docker compose down
# ✅ Solution: Fix the odoo-docker.service File
# ✏️ Step 1: Edit the service file 
  sudo nano /etc/systemd/system/odoo-docker.service
# ✅ Replace the contents with this:
   # paste this
    [Unit]
    Description=Odoo Docker Compose Service
    Requires=docker.service
    After=docker.service
    
    [Service]
    Type=simple
    WorkingDirectory=/home/docker/odoo-docker
    ExecStart=/usr/bin/docker compose up
    ExecStop=/usr/bin/docker compose down
    Restart=always
    TimeoutStartSec=0
    
    [Install]
    WantedBy=multi-user.target
# 🔁 Replace /home/docker/odoo-docker with the actual directory of your docker-compose.yml. 
# 🔄 Step 2: Reload and restart the service 
  sudo systemctl daemon-reload
  sudo systemctl restart odoo-docker
# Check the status: 
  sudo systemctl status odoo-docker
# ✅ Optional: Enable Auto-Start on Boot 
  sudo systemctl enable odoo-docker
# Go into your Odoo container:
  docker exec -it odoo-app bash
# Run the update command:
  python3 odoo-bin -c /etc/odoo/odoo.conf -d dbodoo -u custom_module


