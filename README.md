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
Create a user and add to group `sudo`.
```
sudo adduser --system --quiet --shell=/bin/bash --home=/odoo --gecos 'ODOO' --group odoo 
sudo adduser odoo sudo
```
Create a log file.
```
sudo mkdir /var/log/odoo
sudo chown odoo:odoo /var/log/odoo
```
Custom configurations.
```
sudo vim /etc/odoo/odoo.conf
sudo chown -R odoo:odoo /etc/odoo
```
Paste following content in the file.
```
[options]
addons_path = /opt/odoo/addons,/opt/odoo/odoo/addons
admin_passwd = admin
http_port = 8069
logfile = /var/log/odoo/odoo-server.log
```
Configure startup file.
```
sudo cp /opt/odoo/debian/init /etc/init.d/odoo-init
sudo chown odoo:odoo /etc/init.d/odoo-init
sudo chmod 755 /etc/init.d/odoo-init
```
Configure to start Odoo on server startup.
```
sudo update-rc.d odoo-init defaults
```

## Startup
Start the Odoo server.
```
sudo /etc/init.d/odoo-init start
```
Check the server status.
```
systemctl status odoo-init.service
```
Stop the Odoo server.
```
sudo /etc/init.d/odoo-init stop
```
