# Install Nginx
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
### Setting Up Server Blocks (Recommended)
Create the directory for your_domain as follows, using the -p flag to create any necessary parent directories:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo mkdir -p /var/www/jenkins.dev.dman.cloud/html
```
Next, assign ownership of the directory with the $USER environment variable:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo chown -R $USER:$USER /var/www/jenkins.dev.dman.cloud/html
```
To ensure that your permissions are correct and allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others, you can input the following command:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo chmod -R 755 /var/www/jenkins.dev.dman.cloud
```
Next, create a sample index.html page using `vi` or your favorite editor:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo vi /var/www/jenkins.dev.dman.cloud/html/index.html
```
Inside, add the following sample HTML:
``` html title="Replace your domain" linenums="1"
<html>
    <head>
        <title>Welcome to jenkins.dev.dman.cloud!</title>
    </head>
    <body>
        <h1>Success!  The jenkins.dev.dman.cloud server block is working!</h1>
    </body>
</html>
```
Save and close the file

In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives.
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo vi /etc/nginx/sites-available/jenkins.dev.dman.cloud
```
Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:
``` shell title="Paste the below (replace your domain)" linenums="1"
server {
        listen 80;
        listen [::]:80;

        root /var/www/jenkins.dev.dman.cloud/html;
        index index.html index.htm index.nginx-debian.html;

        server_name jenkins.dev.dman.cloud;

        location / {
                try_files $uri $uri/ =404;
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
Nginx should now be serving your domain name. You can test this by navigating to http://your_domain