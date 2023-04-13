# How to Install Sonarqube in Ubuntu Linux
SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs and code smells on 29 programming languages.
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```
## PostgreSQL
### Add PostgresSQL repository
``` shell title="Run from shell prompt" linenums="1"
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
```
### Install PostgreSQL
``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql
```
### Create Database for Sonarqube
``` shell title="Set password for postgres user" linenums="1"
sudo passwd postgres
```
``` shell title="Change to the postgres user" linenums="1"
su - postgres
```
``` shell title="Create database user postgres" linenums="1"
createuser sonar
```
``` sql title="Set password and grant privileges" linenums="1"
createuser sonar
psql 
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
\q
exit
```
## Adoptium Java 17
``` shell title="Switch to root user" linenums="1"
sudo bash
```
### Add Adoptium repository
``` shell title="Add adoptium repository" linenums="1"
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
```
### Install Java 17
``` shell title="Update repository and install Java" linenums="1"
apt update
apt install temurin-17-jdk
update-alternatives --config java
/usr/bin/java --version
exit 
```
## Linux Kernel Tuning
### Increase Limits
``` shell title="Run from shell prompt" linenums="1"
sudo vim /etc/security/limits.conf
```
Paste the below values at the bottom of the file
``` shell title="Add these values" linenums="1"
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```
### Increase Mapped Memory Regions
``` shell title="Run from shell prompt" linenums="1"
sudo vim /etc/sysctl.conf
```
Paste the below values at the bottom of the file
``` shell title="Add these values" linenums="1"
vm.max_map_count = 262144
```
### Reboot System
``` shell title="Run from shell prompt" linenums="1"
sudo reboot
```
## Sonarqube
### Download and Extract
``` shell title="Run from shell prompt" linenums="1"
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt install unzip
sudo unzip sonarqube-9.9.0.65466.zip -d /opt
sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
```
### Create user and set permissions
``` shell title="Run from shell prompt" linenums="1"
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```
### Update Sonarqube properties with DB credentials
``` shell title="Run from shell prompt" linenums="1"
sudo vim /opt/sonarqube/conf/sonar.properties
```
Find and replace the below values, you might need to add the sonar.jdbc.url
``` shell title="Run from shell prompt" linenums="1"
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
Create service for Sonarqube
``` shell title="Run from shell prompt" linenums="1"
sudo vim /etc/systemd/system/sonar.service
```
Paste the below into the file
``` shell title="Paste the below contents" linenums="1"
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```
Start Sonarqube and Enable service
``` shell title="Paste the below contents" linenums="1"
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```
Watch log files and monitor for startup
``` shell title="Watch logs" linenums="1"
sudo tail -f /opt/sonarqube/logs/sonar.log
```
Access the Sonarqube UI
``` shell title="Paste the below contents" linenums="1"
http://<IP>:9000
```

# Optional Reverse Proxy and TLS Configuration

## Installing Nginx
``` shell title="Run from shell prompt"
sudo apt install nginx
```

``` shell title="create nginx config file" linenums="1"
vi /etc/nginx/sites-available/sonarqube.conf
```
Paste the contents below and be sure to update the domain name

``` shell title="Paste and update" linenums="1"
server {

    listen 80;
    server_name sonarqube.dev.dman.cloud;
    access_log /var/log/nginx/sonar.access.log;
    error_log /var/log/nginx/sonar.error.log;
    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
    }
}
```
Next, activate the server block configuration 'sonarqube.conf' by creating a symlink of that file to the '/etc/nginx/sites-enabled' directory. Then, verify your Nginx configuration files.

``` shell title="Enable virtual host and restart nginx" linenums="1"
sudo ln -s /etc/nginx/sites-available/sonarqube.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
That's it! You have now successfully installed Sonarque, if you found this tutotial helpful please consider subscribing to my YouTube Channel for more tutorials like this. https://www.youtube.com/@dineshmistry
