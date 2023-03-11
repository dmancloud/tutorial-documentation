# How to Install Sonarqube in Ubuntu Linux

SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs and code smells on 29 programming languages.
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages
``` shell
sudo apt update
sudo apt upgrade
```
## PostgreSQL
### Add PostgresSQL repository
``` shell
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
``` shell
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
```
### Install PostgreSQL
``` shell
sudo apt update
```
``` shell
sudo apt-get -y install postgresql postgresql-contrib
```
``` shell
sudo systemctl enable postgresql
```