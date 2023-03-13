# Jenkins Installation
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```

## Adoptium Java 11
``` shell title="Switch to root user" linenums="1"
sudo bash
```
### Add Adoptium repository
``` shell title="Add adoptium repository" linenums="1"
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
```
### Install Java 11
``` shell title="Update repository and install Java" linenums="1"
apt update
apt install temurin-11-jdk
update-alternatives --config java
/usr/bin/java --version
exit 
```

## Install Jenkins
First, add the repository key to the system:
``` shell title="Run from shell prompt" linenums="1"
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```
Next, let’s append the Debian package repository address to the server’s `sources.list`:
``` shell title="Run from shell prompt" linenums="1"
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
Next, we’ll run `update` so that `apt` will use the new repository.
``` shell title="Run from shell prompt" linenums="1"
sudo apt update
```
Finally, we’ll install Jenkins and its dependencies.
``` shell title="Run from shell prompt" linenums="1"
sudo apt install jenkins
```

### Starting Jenkins
Let’s start Jenkins by using systemctl:
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl start jenkins
```
Since systemctl doesn’t display status output, we’ll use the status command to verify that Jenkins started successfully:
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl status jenkins
```
If everything went well, the beginning of the status output shows that the service is active and configured to start at boot:
``` shell title="Run from shell prompt" linenums="1"
Output
● jenkins.service - LSB: Start Jenkins at boot time
   Loaded: loaded (/etc/init.d/jenkins; generated)
   Active: active (exited) since Fri 2020-06-05 21:21:46 UTC; 45s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 1137)
   CGroup: /system.slice/jenkins.service
```
## Setting Up Jenkins
To set up your installation, visit Jenkins on its default port, 8080, using your server domain name or IP address: http://your_server_ip_or_domain:8080

You should receive the Unlock Jenkins screen, which displays the location of the initial password: