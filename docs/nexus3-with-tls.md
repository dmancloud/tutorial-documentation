# Install Nexus3 Repository Manager with TLS
Manage components, binaries and build artifacts across your entire software supply chain.
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```
## Adoptium Temurin Java 8
``` shell title="Switch to root user" linenums="1"
sudo bash
```
### Add Adoptium repository
``` shell title="Add adoptium repository" linenums="1"
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
```
### Install Java 8
``` shell title="Update repository and install Java" linenums="1"
apt update
apt install temurin-8-jdk
```
``` shell title="Check the Java Version is installed correctly" linenums="1"
/usr/bin/java -version
exit 
```

## Install Nexus3 Repository Manager
At time of writing v3.49.0 is the latest version. You can check if there is a newer version available by visiting - https://help.sonatype.com/repomanager3/product-information/download

### Download Nexus

``` shell title="Run from shell prompt" linenums="1"
wget https://download.sonatype.com/nexus/3/nexus-3.49.0-02-unix.tar.gz
```

### Extract Nexus
``` shell title="Run from shell prompt" linenums="1"
sudo tar -xzvf nexus-3.49.0-02-unix.tar.gz -C /opt
```
``` shell title="Run from shell prompt" linenums="1"
cd /opt
```
``` shell title="Run from shell prompt" linenums="1"
sudo mv nexus-3.49.0-02 nexus
```

### Create User to run Nexus3
``` shell title="Run from shell prompt" linenums="1"
sudo adduser nexus
```
### Update and Grant Permissions to Nexus user
``` shell title="Run from shell prompt" linenums="1"
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
```
### Change default run_as user
``` shell title="Run from shell prompt" linenums="1"
sudo vi /opt/nexus/bin/nexus.rc
```
Uncomment `#run_as_user=` and modify to set nexus as user. It should read `run_as_user=”nexus”`

### Configure Nexus to run as a service
``` shell title="Run from shell prompt" linenums="1"
sudo vi /etc/systemd/system/nexus.service
```
``` shell title="Paste the below" linenums="1"
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
[Install]
WantedBy=multi-user.target
```
### Start and Enable Nexus
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl enable nexus
```
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl start nexus
```
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl status nexus
```
### Monitor Startup
``` shell title="Run from shell prompt" linenums="1"
tail -f /opt/sonatype-work/nexus3/log/nexus.log
```
Wait until you see Nexus3 has started, you should see something like below
``` shell title="Run from shell prompt" linenums="1"
Started @50347ms
2023-03-14 13:36:15,995+0000 INFO  [jetty-main-1]  *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer -
-------------------------------------------------

Started Sonatype Nexus OSS 3.49.0-02

-------------------------------------------------
```
### Access User Interface
Once Nexus is successfully installed, you can access it in the browser by
``` shell title="Run from shell prompt" linenums="1"
http://IP_Address:8081
```

You can obtain the initial password by issuing the following command:
``` shell title="Run from shell prompt" linenums="1"
cat /opt/sonatype-work/nexus3/admin.password
```
