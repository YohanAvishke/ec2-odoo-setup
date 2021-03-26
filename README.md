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
Start the Odoo service and enabled it to start on boot by running.
```
sudo systemctl enable --now odoo14
```
View the Odoo log
```
sudo journalctl -n 50 -f -u odoo14
```
Stop the Odoo service and disable it from running on startup.
```
sudo systemctl disable --now odoo14
```
