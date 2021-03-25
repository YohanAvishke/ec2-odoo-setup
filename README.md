# EC2 Odoo setup
All the steps I followed to setup a community version of Odoo on a free EC2 instance

## EC2
### Prerequisites
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) to setup the EC2 prerequisites. Names and configs used,
 - **Region:** `ap-south-1` (use closest region for the users)
 - **Key Pair:** `groundstation-ec2-access-key-ap-south-1`
 - **Security Group:** `yohan_SG_apsouth1`
 - **Storage:** 20GiB ([Guide to expand an existing storage size](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html#extend-file-system))

### Setup
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) to launch the instance. Names and configs used,
 - **OS type:** Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
 - **Processor:** 64 bit (x86)
 - **Storage:** 8GB

### Connect
Download and store the `.pem/.cert` file in `.ssh` directory. Use the following command to connect,

```
ssh -i groundstation-ec2-access-key-ap-south-1.cer ubuntu@|██████|.ap-south-1.compute.amazonaws.com
```


## Odoo
### Prerequisites
Update the server dependancies.
```
sudo apt-get update
sudo apt-get upgrade -y
```
Install pip3 package manager.
```
sudo apt install python3-pip -y
```
Install PostgreSQL.
```
sudo apt install postgresql postgresql-client -y
```
Create new PostgreSQL user.
```
sudo -u postgres createuser -s $USER
createdb $USER
```
Install development tools and native dependencies.
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

### Installation
Download Odoo from GIT
```
git clone https://github.com/odoo/odoo.git
```
