# Adding an SSH Based Agent to Jenkins
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```

## Create Jenkins User
``` shell title="Run from shell prompt" linenums="1"
sudo adduser jenkins
```
Grant Sudo Rights to Jenkins User
``` shell title="Run from shell prompt" linenums="1"
sudo usermod -aG sudo jenkins
```
Logout and ssh back as user Jenkins
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
## Docker
### Install using the repository
Update the apt package index and install packages to allow apt to use a repository over HTTPS:
``` shell title="Run from shell prompt" linenums="1"
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
Add Docker’s official GPG key:
``` shell title="Run from shell prompt" linenums="1"
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
Use the following command to set up the repository:
``` shell title="Run from shell prompt" linenums="1"
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install Docker Engine
``` shell title="Run from shell prompt" linenums="1"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Manage Docker as a non-root user
Create the docker group.
``` shell title="Run from shell prompt" linenums="1"
sudo groupadd docker
```
Add your user to the docker group.
``` shell title="Run from shell prompt" linenums="1"
sudo usermod -aG docker $USER
```
Run the following command to activate the changes to groups:
``` shell title="Run from shell prompt" linenums="1"
newgrp docker
```
Verify that you can run docker commands without sudo.
``` shell title="Run from shell prompt" linenums="1"
docker run hello-world
```

## Connect to Remote SSH Agent
From the Jenkins UI (Controller)
``` shell title="Run from shell prompt" linenums="1"
ssh jenkins@$AGENT_HOSTNAME
```
Create private and public SSH keys. The following command creates the private key jenkinsAgent_rsa and the public key jenkinsAgent_rsa.pub. It is recommended to store your keys under ~/.ssh/ so we move to that directory before creating the key pair.
``` shell title="Run from shell prompt" linenums="1"
 mkdir ~/.ssh; cd ~/.ssh/ && ssh-keygen -t rsa -m PEM -C "Jenkins agent key" -f "jenkinsAgent_rsa"
```
Add the public SSH key to the list of authorized keys on the agent machine
``` shell title="Run from shell prompt" linenums="1"
cat jenkinsAgent_rsa.pub >> ~/.ssh/authorized_keys
```
Ensure that the permissions of the ~/.ssh directory is secure, as most ssh daemons will refuse to use keys that have file permissions that are considered insecure:
``` shell title="Run from shell prompt" linenums="1"
chmod 700 ~/.ssh
 chmod 600 ~/.ssh/authorized_keys ~/.ssh/jenkinsAgent_rsa
```
Copy the private SSH key (~/.ssh/jenkinsAgent_rsa) from the agent machine to your OS clipboard
``` shell title="Run from shell prompt" linenums="1"
cat ~/.ssh/jenkinsAgent_rsa
```

Now you can add the Agent on the Jenkins UI (Controller)