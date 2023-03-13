# Install Nginx Reverse Proxy for Jenkins
Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is a lightweight choice that can be used as either a web server or reverse proxy.

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

In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives.
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo vi /etc/nginx/sites-available/jenkins.dev.dman.cloud
```
Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:
``` shell title="Paste the below (replace your domain)" linenums="1"
upstream jenkins{
    server 127.0.0.1:8080;
}

server{
    listen      80;
    server_name jenkins.dev.dman.cloud;

    access_log  /var/log/nginx/jenkins.access.log;
    error_log   /var/log/nginx/jenkins.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto https;
    }

}
```
Next, let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo ln -s /etc/nginx/sites-available/jenkins.dev.dman.cloud /etc/nginx/sites-enabled/
```
Next, test to make sure that there are no syntax errors in any of your Nginx files:
``` shell title="Run from shell prompt" linenums="1"
sudo nginx -t
```
If there aren’t any problems, restart Nginx to enable your changes:
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl restart nginx
```
Nginx should now be serving Jenkins from your domain name. You can test this by navigating to http://your_domain