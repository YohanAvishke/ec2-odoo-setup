# EC2 Odoo setup
All the steps to setup a community version of Odoo on a free EC2 instance

# EC2
## Prerequisites
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) to setup the EC2 prerequisites. Names and configs used,
 - **Region:** `ap-south-1` (use closest region for the users)
 - **Key Pair:** `groundstation-ec2-access-key-ap-south-1`
 - **Security Group** 
   - **Name:** `yohan_SG_apsouth1`
   - **Inbound Rules:** Type: `Custom TCP` - Port: `8069`
 - **Storage:** 20GiB ([Guide to expand an existing storage size](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html#extend-file-system))

## Setup
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) to launch the instance. Names and configs used,
 - **OS type:** Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
 - **Processor:** 64 bit (x86)
 - **Storage:** 8GB

## Connect
Download and store the `.pem/.cert` file in `.ssh` directory. Use the following command to connect,

```
ssh -i groundstation-ec2-access-key-ap-south-1.cer ubuntu@|██████|.ap-south-1.compute.amazonaws.com
```


# Odoo
## Prerequisites
Update the server dependancies.
```
sudo apt-get update
sudo apt-get upgrade -y
```
Install pip3 package manager.
```
sudo apt install python3-pip -y
```
Install venv dependency.
```
sudo apt-get install python3-venv -y
```
Install PostgreSQL.
```
sudo apt install postgresql postgresql-client -y
```
Install rest of the development tools and native dependencies.
```
sudo apt install -y \
python3-dev libxml2-dev libxslt1-dev libldap2-dev \
libsasl2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev \
zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev \
libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev
```
Install wkhtmltopdf for headers and footers support.
```
sudo apt-get install wkhtmltopdf -y
```

## Installation
Create System user.
```
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
```
Create a new PostgreSQL user.
```
sudo -u postgres createuser -s odoo
```
Change user to Odoo user.
```
sudo su - odoo
```
Download Odoo from GIT
```
git clone https://github.com/odoo/odoo.git /opt/odoo/odoo14
```
Create a Virtual environment and activate it.
```
cd /opt/odoo/odoo14
python3 -m venv venv
source venv/bin/activate
```
Comment-out `libsass==0.17.0` since it gets stuck in the build stage.
Execute follow command to fix `libsass` issue.
But, beware, `0.20.0` does not work with Odoo Studio App.
```
pip3 install libsass==0.20.0
```
Install Python dependancies from pip.
```
pip3 install setuptools wheel
pip3 install -r requirements.txt
```
Create custom addons directory.
```
mkdir /opt/odoo/odoo14/custom-addons
```
After the installation deactivate the virtual environment and logout.
```
deactivate
exit
```

## Startup
Startup Odoo by provideing custom addon path and database.
```
python3 odoo-bin --addons-path=addons -d odoo
```
Odoo dashboard: [http://████.ap-south-1.compute.amazonaws.com:8069](http://████.ap-south-1.compute.amazonaws.com:8069)


# As a Background service
## Prerequisites
Create a custom configuration file.
```
sudo vim /etc/odoo/odoo14.conf
```
Paste following content in the file.
```
addons_path = /opt/odoo/odoo14/addons,/opt/odoo/odoo14/custom-addons
; This is the password that allows database operations:
admin_passwd = admin
db_host = False
db_port = False
db_user = odoo
db_password = False
```
Create Systemd Unit File.
```
sudo vim /etc/systemd/system/odoo14.service
```
Paste following content in the file.
```
[Unit]
Description=Odoo14
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo14
PermissionsStartOnly=true
User=odoo
Group=odoo
ExecStart=/opt/odoo/odoo14/venv/bin/python3 /opt/odoo/odoo14/odoo-bin -c /etc/odoo/odoo14.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

## Startup
Reload the Systemd to create the service.
```
sudo systemctl daemon-reload
```
Start the Odoo service.
```
sudo systemctl start odoo14
```
(optional) Start the Odoo service and enabled it to start on boot by running
```
sudo systemctl enable --now odoo14
```
Check the status of the service.
```
sudo systemctl status odoo14
```
View the Odoo log.
```
sudo journalctl -n 50 -f -u odoo14
```
Stop the Odoo service.
```
sudo systemctl stop odoo14
```
(optional) Stop the Odoo service and disable it from running on startup.
```
sudo systemctl disable --now odoo14
```


# NGINX
## Installation
Login as a root user. If root following command should display `root`.
```
sudo whoami
```
Download and Install NGINX
```
sudo apt install nginx -y
```
Verify if the service is up and running (`Active: active (running)`)
```
sudo systemctl status nginx
```

## Firewall setup
(Optional) Setup a firewall using `UFW`. 
Not required since EC2 provide security groups. but,
> "Having both is more secure and they can complement each other, 
> `IPTables` (or any other firewall) allows you to log posible atacks and even you can add dynamic rules"

[View Guide](https://linuxize.com/post/how-to-install-nginx-on-ubuntu-20-04/#configuring-firewall) 


# NGINX SSL cert securing
Install a free Let’s Encrypt SSL certificate and configure Nginx to use the SSL certificate and enable HTTP/2.
## Installation
Install `Certbot` to automates the tasks for obtaining and renewing SSL certificates and configuring web servers to use the certificates.
```
sudo apt install certbot -y
```

## Obtaining SSL certificate
Generate a new set of 2048 bit DH(Diffie–Hellman key exchange) parameters.
```
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
Make a directory to verify that the requested domain resolves to the server where certbot runs(using `Webroot` plugin).
```
sudo mkdir -p /var/lib/letsencrypt/.well-known
```
Make the directory writable by Nginx server.
```
sudo chgrp www-data /var/lib/letsencrypt
sudo chmod g+s /var/lib/letsencrypt
```
Create 2 files to include base snippets for all Nginx server blocks.
1. `sudo vim /etc/nginx/snippets/letsencrypt.conf`
   ```
   location ^~ /.well-known/acme-challenge/ {
     allow all;
     root /var/lib/letsencrypt/;
     default_type "text/plain";
     try_files $uri =404;
   }
   ```
2. `sudo vim /etc/nginx/snippets/ssl.conf` - Chippers recommended by Mozilla, Enables OCSP Stapling, HTTP Strict Transport Security (HSTS) and Enforces few security‑focused HTTP headers.
   ```
   ssl_dhparam /etc/ssl/certs/dhparam.pem;

   ssl_session_timeout 1d;
   ssl_session_cache shared:SSL:10m;
   ssl_session_tickets off;

   ssl_protocols TLSv1.2 TLSv1.3;
   ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
   ssl_prefer_server_ciphers on;

   ssl_stapling on;
   ssl_stapling_verify on;
   resolver 8.8.8.8 8.8.4.4 valid=300s;
   resolver_timeout 30s;

   add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
   add_header X-Frame-Options SAMEORIGIN;
   add_header X-Content-Type-Options nosniff;
   ```
Create the domain server block file.
```
sudo vim /etc/nginx/sites-available/██████.ap-south-1.compute.amazonaws.com.conf
```
Add following code to the file.
```
server {
  listen 80;
  server_name ██████.ap-south-1.compute.amazonaws.com www.██████.ap-south-1.compute.amazonaws.com;

  include snippets/letsencrypt.conf;
}
```
Enable the new server block by creating a symbolic link to `sites-enabled` directory.
```
sudo ln -s /etc/nginx/sites-available/.██████.ap-south-1.compute.amazonaws.com.conf /etc/nginx/sites-enabled/
```
(Optional) Add support for long domain names (Available sizes 64, 128, 256, 512, etc.)
```
sudo vim /etc/nginx/nginx.conf


http {
        ...
        server_names_hash_bucket_size 128;
        ...
```

Update the Nginx session with the changes.
```
sudo systemctl restart nginx
```
