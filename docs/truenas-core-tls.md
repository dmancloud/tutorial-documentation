# TrueNAS (Core) Configure TLS Certificate

The first step is to update your network setting. Make sure you have a static IP address and update the hostname and domain you will need to change it to a Fully Qualified Domain Name (FQDN).

``` shell
truenas.dev.dman.cloud
```

## Install acme.sh shell script using the below command
Open a shell to your TrueNAS server

``` shell
curl https://get.acme.sh | sh -s email=xxxxxx@xxxxx.xxx
```
Next you will need to use the ACME DNS API wiki to determine what the correct syntax for your Domain service provider

``` shell
https://github.com/acmesh-official/acme.sh/wiki/dnsapi
```

The syntax below is for CloudFlare

``` shell
export CF_Token="sdfsdfsdfljlbjkljlkjsdfoiwje"
export CF_Account_ID="xxxxxxxxxxxxx"
```

In order to use the new token, the token currently needs access read access to Zone.Zone, and write access to Zone.DNS, across all Zones. 

The next step is to request a certificate from Let’s Encrypt

``` shell
acme.sh --issue --dns dns_cf --keylength 4096 -d truenas.dev.dman.cloud
```
## Create TrueNAS API Token
Next, you will need to generate a API Key on TrueNAS to deploy the certificate. You can obtain a API Key from your TrueNAS console

## Clone the below repository
This repository contains a python script which will help you install the TLS Certificate

``` shell
git clone https://github.com/dmancloud/letsencrypt-truenas.git
```
Once you’ve downloaded the script, you’ll need to create a configuration file called deploy_config. The git repo has an example (deploy_config.example) which you can copy and modify, or you can write your own from scratch. 

Insert your API key that you generated earlier

``` shell
[deploy]
api_key = CHANGE_ME
```

Next, you will install the certificate using the below command
``` shell
acme.sh --install-cert -d truenas.dev.dman.cloud --reloadcmd "~/letsencrypt-truenas/deploy_freenas.py"
```

* note if you get an error make sure the python script has the executable permission `chmod a+x deploy_freenas.py`

You shoulf see the message “Certificate import successfully.” Your Web Service will restart.

### Redirect http->https
Log back into your console after the system restarted. Then, navigate to System Settings > GUI > Settings and enable Web Interface HTTP -> HTTPS Redirect.

``` shell
It is a good idea to restart your TrueNas server again at this point
```

Lastly, you need to create a Cron Job to renew the certificate automatically, we can check `weekly`

``` shell
/root/.acme.sh/acme.sh --cron
```
Congratulation, you have successfully deployed Let’s encrypt Certificate on your TrueNAS.
