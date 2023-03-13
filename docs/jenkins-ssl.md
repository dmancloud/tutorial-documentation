# Configure Jenkins with SSL Using an Nginx Reverse Proxy
By default, Jenkins comes with its own built-in Winstone web server listening on port 8080, which is convenient for getting started. It’s also a good idea, however, to secure Jenkins with SSL to protect passwords and sensitive data transmitted through the web interface.

In this tutorial, you will configure Nginx as a reverse proxy to direct client requests to Jenkins.
## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
- Jenkins installed
### Update Package Repository and Upgrade Packages

``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```

## Installing Nginx
``` shell title="Run from shell prompt" linenums="1"
sudo apt install nginx
```
### Checking your Web Server
We can check with the `systemd` init system to make sure the service is running by typing:
``` shell title="Run from shell prompt" linenums="1"
systemctl status nginx
```
We can check with the systemd init system to make sure the service is running by typing:
``` console title="Output should look similar to the below" linenums="1"
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-04-20 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   Memory: 3.5M
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```

Check you Web Server is running
``` shell title="Access your web server by visiting"
http://your_server_ip
```