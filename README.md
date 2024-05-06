# Apache Tomcat & Nginx Server Configuration on Ubuntu DigitalOcean (Servlets & JSPs)

This repository contains step-by-step instructions for setting up Apache Tomcat and Nginx server configuration on Ubuntu DigitalOcean for serving Servlets & JSPs.

## Steps

### Step 0: Update APT
```bash
sudo apt update
```

### Step 1: Creating a non-root user
```bash
adduser millat
usermod -aG sudo millat
```

### Step 2: Install Java
```bash
sudo apt update
sudo apt install default-jdk
java -version
```

### Step 3: Install Tomcat
```bash
sudo useradd -m -d /opt/tomcat -U -s /bin/false tomcat
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.23/bin/apache-tomcat-10.1.23.tar.gz
sudo tar xzvf apache-tomcat-10.1.23.tar.gz -C /opt/tomcat --strip-components=1
sudo chown -R tomcat:tomcat /opt/tomcat/
sudo chmod -R u+x /opt/tomcat/bin
sudo nano /opt/tomcat/conf/tomcat-users.xml
```
Add the following lines inside `<tomcat-users>` tag:
```xml
<role rolename="manager-gui" />
<user username="manager" password="new-password" roles="manager-gui" />

<role rolename="admin-gui" />
<user username="admin" password="new-password" roles="manager-gui,admin-gui" />
```
```bash
sudo update-java-alternatives -l
sudo nano /etc/systemd/system/tomcat.service
```
Copy and paste the following configuration:
```bash
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-1.21.0-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl status tomcat
sudo systemctl enable tomcat
```

### Step 4: Install Nginx as a Proxy
```bash
sudo apt install nginx
sudo systemctl stop nginx.service
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
sudo nano /etc/nginx/sites-available/tomcat-server
```
Add the following configuration:
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  your-server-ip your-domain-name.com;

    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/tomcat-server /etc/nginx/sites-enabled/
sudo systemctl restart nginx.service
```

### Fix Error: 413 Request Entity Too Large on Nginx
```bash
sudo nano /etc/nginx/nginx.conf
```
Add or modify the following line under `http` block:
```nginx
http {
    ...
    # Set value 'client_max_body_size'
    client_max_body_size 100M;
    ...
}
```
```bash
sudo nginx -s reload
```

Follow these steps carefully to configure your Apache Tomcat and Nginx servers on Ubuntu DigitalOcean. Happy coding!
