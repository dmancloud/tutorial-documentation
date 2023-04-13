# Jenkins Installation
!!! info "Prerequsites"
    Virtual Machine running Ubuntu 22.04 or newer

### Update Package Repository and Upgrade Packages

``` shell title="Become root"
sudo -i
```

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```

## Adoptium Java 11

### Add Adoptium repository
``` shell title="Add adoptium repository" linenums="1"
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
### Install Java 11
``` shell title="Update repository and install Java" linenums="1"
apt update
apt install temurin-11-jdk
/usr/bin/java --version
```

## Install Jenkins
First, add the repository key to the system:
``` shell title="Run from shell prompt"
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```
Next, let’s append the Debian package repository address to the server’s `sources.list`:
``` shell title="Run from shell prompt"
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
Next, we’ll run `update` so that `apt` will use the new repository.
``` shell title="Run from shell prompt"
sudo apt update
```
Finally, we’ll install Jenkins and its dependencies.
``` shell title="Run from shell prompt"
sudo apt install jenkins
```

### Starting Jenkins
Let’s start Jenkins by using systemctl:
``` shell title="Run from shell prompt"
sudo systemctl start jenkins
```
Since systemctl doesn’t display status output, we’ll use the status command to verify that Jenkins started successfully:
``` shell title="Run from shell prompt"
sudo systemctl status jenkins
```
If everything went well, the beginning of the status output shows that the service is active and configured to start at boot:
``` shell title="Run from shell prompt"
Output
● jenkins.service - LSB: Start Jenkins at boot time
   Loaded: loaded (/etc/init.d/jenkins; generated)
   Active: active (exited) since Fri 2020-06-05 21:21:46 UTC; 45s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 1137)
   CGroup: /system.slice/jenkins.service
```
## Access Jenkins User Interface
To set up your installation, visit Jenkins on its default port, 8080, using your server domain name or IP address: http://your_server_ip_or_domain:8080

## Example Pipeline
You should receive the Unlock Jenkins screen, which displays the location of the initial password:
``` groovy title="Sample Jenkinsfile" linenums="1"
pipeline {
    agent any
    stages {
        stage('Hello World') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```
