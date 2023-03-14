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
```
Configure Nexus Docker Hosted Registry from the User Interface
In the example below we are assuming a docker hosted registry
was created on port 1111
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
sudo vi /etc/nginx/sites-available/nexus-repo.dev.dman.cloud
```
Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:
``` shell title="Paste the below (replace your domain)" linenums="1"


    server {
        listen   *:80;
        server_name  nexus-repo.dev.dman.cloud;

        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 1G;

        # optimize downloading files larger than 1G - refer to nginx doc before adjusting
        #proxy_max_temp_file_size 2G;

        location / {
	    proxy_pass http://127.0.0.1:8081;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

	# Docker /v2 and /v1 (for search) requests
	location /v2 {
	  proxy_set_header Host $host:$server_port;
	  proxy_set_header X-Real-IP $remote_addr;
	  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	  proxy_pass http://127.0.0.1:1111;
	}
	location /v1 {
	  proxy_set_header Host $host:$server_port;
	  proxy_set_header X-Real-IP $remote_addr;
	  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	  proxy_pass http://127.0.0.1:1111;
	}
    }
```  

Next, let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
``` shell title="Run from shell prompt (replace your domain)" linenums="1"
sudo ln -s /etc/nginx/sites-available/nexus-repo.dev.dman.cloud /etc/nginx/sites-enabled/
```
Next, test to make sure that there are no syntax errors in any of your Nginx files:
``` shell title="Run from shell prompt" linenums="1"
sudo nginx -t
```
If there aren’t any problems, restart Nginx to enable your changes:
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl restart nginx
```
Nginx should now be serving Nexus from your domain name. You can test this by navigating to http://your_domain


## Configure Nexus with SSL Using an Nginx Reverse Proxy
By default, Jenkins comes with its own built-in Winstone web server listening on port 8080, which is convenient for getting started. It’s also a good idea, however, to secure Jenkins with SSL to protect passwords and sensitive data transmitted through the web interface.

### Installing Certbot
The first step to using Let’s Encrypt to obtain an SSL certificate is to install the Certbot software on your server.
``` shell title="Run from shell prompt" linenums="1"
sudo apt install certbot python3-certbot-nginx
```
### Confirming Nginx’s Configuration
Certbot needs to be able to find the correct server block in your Nginx configuration for it to be able to automatically configure SSL. Specifically, it does this by looking for a server_name directive that matches the domain you request a certificate for.
``` shell title="Run from shell prompt (replace domain)" linenums="1"
sudo vi /etc/nginx/sites-available/nexus-repo.dev.dman.cloud
```
Find the existing server_name line. It should look like this:
``` shell title="Look for your domain"
...
server_name nexus-repo.dev.dman.cloud;
...
```
If it does, exit your editor and move on to the next step. If not review the installing Nginx Tutorial
### Obtaining an SSL Certificate
Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:
``` shell title="Run from shell prompt (replace domain)" linenums="1"
sudo certbot --nginx -d nexus-repo.dev.dman.cloud
```
If that’s successful, certbot will ask how you’d like to configure your HTTPS settings.

Select your choice then hit ENTER. The configuration will be updated, and Nginx will reload to pick up the new settings. certbot will wrap up with a message telling you the process was successful and where your certificates are stored:

### Verifying Certbot Auto-Renewal
Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The certbot package we installed takes care of this for us by adding a systemd timer that will run twice a day and automatically renew any certificate that’s within thirty days of expiration.

You can query the status of the timer with `systemctl`:
``` shell title="Run from shell prompt" linenums="1"
sudo systemctl status certbot.timer
```
``` shell title="Output should look like the below" linenums="1"
Output
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Mon 2020-05-04 20:04:36 UTC; 2 weeks 1 days ago
    Trigger: Thu 2020-05-21 05:22:32 UTC; 9h left
   Triggers: ● certbot.service
```   
To test the renewal process, you can do a dry run with `certbot`:
``` shell title="Run from shell prompt" linenums="1"
sudo certbot renew --dry-run
```
If you see no errors, you’re all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.

Nginx should now be serving your domain name. You can test this by navigating to https://your_domain