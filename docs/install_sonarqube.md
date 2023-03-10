# How to Install Sonarqube in Ubuntu Linux
Code quality is an approximation of how useful and maintainable a specific piece of code is. Quality code will make the task of maintaining and expanding your application easier. It helps ensure that fewer bugs are introduced when you make required changes in the future.

SonarQube is an open-source tool that assists in code quality analysis and reporting. It scans your source code looking for potential bugs, vulnerabilities, and maintainability issues, and then presents the results in a report which will allow you to identify potential issues in your application.

In this guide, you will deploy a SonarQube server and scanner to analyze your code and create code quality reports.
## Prerequsites
To follow this tutorial, you will need:

- Virtual Machine running Ubuntu 22.04 or newer
- User account with sudo privileges 
## Update Package Repository & Install Updates
``` { .shell .copy }
sudo apt update && sudo apt upgrade
```
## Install Postgresql 15
First, add Postgresql's PPA, then update your package repository.

``` { .shell .copy }
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

``` { .shell .copy }

wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
```
``` { .shell .copy }
sudo apt update
```
Install PostgreSQL 
``` { .shell .copy }
sudo apt-get -y install postgresql postgresql-contrib
```
``` { .shell .copy }
sudo systemctl enable postgresql
```
## Create Database for Sonarqube
``` { .shell .copy }
sudo passwd postgres
```
``` { .shell .copy }
su - postgres
```
``` { .sql .copy }
createuser sonar
```
``` { .sql .copy }
psql 
```
``` { .psql .copy }
ALTER USER sonar WITH ENCRYPTED password 'sonar';
```
``` { .psql .copy }
CREATE DATABASE sonarqube OWNER sonar;
```
``` { .psql .copy }
grant all privileges on DATABASE sonarqube to sonar;
```
``` { .psql .copy }
\q
```
``` { .psql .copy }
exit
```
## Install Java 17
Next, install Java. Specifically, this will install the JDK (Java Development Kit).
``` { .shell .copy }
sudo bash
```
``` { .shell .copy }
apt install -y wget apt-transport-https
```
``` { .shell .copy }
mkdir -p /etc/apt/keyrings
```
``` { .shell .copy }
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
```
``` { .shell .copy }
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
```
``` { .shell .copy }
apt update
```
``` { .shell .copy }
apt install temurin-17-jdk
```
There can be multiple Java installations on one server. You can configure which version is the default for use in the command line by using update-alternatives, which manages which symbolic links are used for different commands.
``` { .shell .copy }
update-alternatives --config java
```
``` { .shell .copy }
/usr/bin/java --version
```
``` { .shell .copy }
exit 
```
## Increase Limux Kernel Limits

```
sudo vim /etc/security/limits.conf
```
Paste the below values at the bottom of the file
``` { .yaml .annotate }
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```
```
sudo vim /etc/sysctl.conf
```
Paste the below values at the bottom of the file
``` { .yaml .annotate }
vm.max_map_count = 262144
```
Reboot to set the new limits
```
sudo reboot
```
## Install Sonarqube
### Create a sonarqube user
``` { .yaml .annotate }
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```
### Downloading and Installing SonarQube
``` { .yaml .annotate }
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt install unzip
sudo unzip sonarqube-9.9.0.65466.zip -d /opt
sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
```
### Configuring the SonarQube Server

Update Sonarqube properties with DB credentials
```
sudo vim /opt/sonarqube/conf/sonar.properties
```
Find and replace the below values, you might need to add the sonar.jdbc.url
```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
Create service for Sonarqube
```
sudo vim /etc/systemd/system/sonar.service
```
Paste the below into the file
``` { .yaml .copy }
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
```
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
sudo tail -f /opt/sonarqube/logs/sonar.log
```
### Access the Sonarqube UI
```
http://<IP>:9000
```