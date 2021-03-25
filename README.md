# EC2 Odoo setup
All the steps I followed to setup a community version of Odoo on a free EC2 instance

## EC2
### Prerequisites
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) to setup the EC2 prerequisites.Details used,
 - **Region:** ap-south-1 (use closest region for the users)
 - **Key Pair:** groundstation-ec2-access-key-ap-south-1
 - **Security Group:** yohan\_SG_apsouth1

### Setup
Follow the [tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) to launch the instance. I used following configurations and details,
 - **OS type:** Amazon Linux 2 AMI (HVM), SSD Volume Type
 - **Processor:** 64 bit (x86)
 - **Storage:** 8GB

### Connect
Download and store the `.pem/.cert` file in `.ssh` directory. Use the following command to connect,

`ssh -i groundstation-ec2-access-key-ap-south-1.cer ec2-user@██████.ap-south-1.compute.amazonaws.com`
