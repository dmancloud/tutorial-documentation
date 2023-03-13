# Configure Jenkins with SSL Using an Nginx Reverse Proxy
By default, Jenkins comes with its own built-in Winstone web server listening on port 8080, which is convenient for getting started. It’s also a good idea, however, to secure Jenkins with SSL to protect passwords and sensitive data transmitted through the web interface.
## Prerequsites 
- Jenkins installed 
- An A record with <domain> pointing to your server’s public IP address.
### Update Package Repository and Upgrade Packages
``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```
## Installing Certbot
The first step to using Let’s Encrypt to obtain an SSL certificate is to install the Certbot software on your server.
``` shell title="Run from shell prompt" linenums="1"
sudo apt install certbot python3-certbot-nginx
```
### Confirming Nginx’s Configuration
Certbot needs to be able to find the correct server block in your Nginx configuration for it to be able to automatically configure SSL. Specifically, it does this by looking for a server_name directive that matches the domain you request a certificate for.
``` shell title="Run from shell prompt (replace domain)" linenums="1"
sudo vi /etc/nginx/sites-available/jenkins.dev.dman.cloud
```
Find the existing server_name line. It should look like this:
``` shell title="Look for your domain"
...
server_name jenkins.dev.dman.cloud;
...
```
If it does, exit your editor and move on to the next step. If not review the installing Nginx Tutorial
### Obtaining an SSL Certificate
Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:
``` shell title="Run from shell prompt (replace domain)" linenums="1"
sudo certbot --nginx -d jenkins.dev.dman.cloud
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