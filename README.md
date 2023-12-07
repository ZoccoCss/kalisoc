# KaliPurple SOC in AWS
 
This project is a cost effective solution for launching a Security Operations Center (SOC) in AWS using Kali Linux and Purple Team tools. 

![New Cost](https://user-images.githubusercontent.com/47893772/231023009-617e5cf0-25ae-46b5-931d-75f8918bcfc9.png)


It consists of two files:
 
- KaliPurple-VPC.yml - This file will configure the Virtual Private Cloud (VPC) with the necessary subnets and security groups.
- KaliPurple-NAT-EC2.yml - This file will configure the instances. You can select the ones that you need to lauch. 
 
## VPC Diagram
 
![template1-designer(4)](https://user-images.githubusercontent.com/47893772/230233768-5f024316-4f0f-4b70-9310-0891ceb63d9a.png)
 
This is the diagram of the VPC. I reduced the number of subnets for simplicity by grouping all the VLANs into one. This reduced the number of required interfaces in the firewall from 5 to 3 and this allowed me to choose an instance type that was more cost effective. I know that the solution is not the ideal but it was a compromise that I had to make. Also, I didn't use the kali.purple domain name so all the references to the machines are done through their respective private ip addresses. 

![VPC](https://user-images.githubusercontent.com/47893772/231020516-6c6cc77f-19f5-480d-a762-50b57fb26450.png)
 
## Installation
 
To install this project, you need to have an AWS account and access to CloudFormation service. You first have to create two IAM roles named 'KaliPurpleGuacamole' and 'KaliPurpleSSMFullAccess'. Using the 'AmazonSSMFullAccess' should be OK for a testing environment. 

Launch the VPC stack using the KaliPurple-VPC.yml file. Write "vpc" in the stack name for simplicity. It will create all the necessary subnets and security groups. I used the N. Virginia region so all images belong to it. If you want to use a different region you need to use different images. 
 
Once this stack is created, you need to create the instances using the KaliPurple-NAT-EC2.yml file. For simplicity use "ec2" as a name. You need to input the name of the VPC stack that was created previously.

![AWS Cloudformation](https://user-images.githubusercontent.com/47893772/231020706-82afa33e-b182-4b1f-9ed3-4f71fe0f1b63.png)
 
The EC2 stack gives you the possibility of choosing the instances that you want to launch. This way you don't have to pay for services that you don't need. I used the Guacamole Bastion initially but there is no need for it once the firewall and its OpenVPN is configured unless there is a problem with one of the instances. I left the option of launching it if needed but it is not necessary for most cases. I also left the possibility of using an internet gateway for the instances in the SOC and LAN subnets to have access to the internet. Again, this option should not be necessary once the firewall is configured.

 ![AWS Instances](https://user-images.githubusercontent.com/47893772/231023159-8aa8e92c-73b9-4765-be08-45234d8d7950.png)
 
The instance types defaults are the minimum required for each to work. You can choose a bigger type if desired.
 
## Configuration

The setup is similar to that described in Kali-Purple SOC instructions, with some modifications made to avoid the use of VLANs. For simplicity, the domain name was not utilized.

Currently, the cloud configuration and firewall accept packets from all over the internet and in all of the internal connections, making it unsuitable for production situations but acceptable for proof of concept.
 
## Usage
 
To use this project, you need to connect to the firewall instance using its public IP address and configure its OpenVPN service. You can then use the OpenVPN client on your machine to connect to the firewall. You can then access all the other instances in the SOC and LAN subnets using their private IP addresses through http. https, SSH or RDP protocols. 

You need to enter into Elasticsearch and then install an Elastic-Agent of each of the other machines except for Kali-Pearly. Otherwise, data will not be ingested into Elasticsearch. 
 
You can use Kali-Heliotrope as your attack platform on the kali-pearly instance. You can also upload other vulnerable machine AMIs. Once you are done, delete the stack. 

## AIM Images
Due to AWS restrictions, AIM images with product codes cannot be made public. Therefore, the official Kali AIM cannot be used as a base. Instead, a Debian AIM with the Kali repository added was used. Several methods exist for achieving this, but the method outlined in [this](https://miloserdov.org/?p=3609&PageSpeed=noscript) article was chosen. When installing Kali packages, follow the syntax 
```
sudo aptitude install -t <package-name>.
```
Some of the packages are not yet available or not well-configured in the default Kali repositories, so I had to install them from the original repositories. An example of this is the Elastic Stack. Only the minimum required packages were installed in each of the images.

These machines can be accessed within the Cloud Formation VPC, or they can be launched individually. If you wish to access them in the Cloud Formation VPC and have not yet configured OPNSense with OpenVPN, use a Bastion such as Guacamole since the machines are created in a private subnet. Simultaneously launching Guacamole is an option when launching the instances in CloudFormation.

To login to all EC2 instances, except for Bizantium, use the following credentials. 
```
Username: kali
Password: kali2023
```
### Kali-Basic
This machine is not part of the SOC, but it has been included to assist users with customizations. It does not have any Kali tools installed. This image can be found on AWS as 'ami-0dbd53121fa982e40'.

### Kali-Pearly
I decided not to publish a vulnerable machine as an AMI image in AWS, so the image that I included in the CloudFormation script is Kali-Basic. This is the simplest of the Kali-Purple machines, but the instructions provided to set up dvwa did not work for me. I had to go to the official repository to obtain it at https://github.com/digininja/DVWA. I used the following commands to set it up:

```
sudo apt update
sudo apt upgrade
sudo apt install -t kali-rolling apache2 mariadb-server mariadb-client php php-mysqli php-gd libapache2-mod-php php8.2-mysql git
cd Downloads/
git clone https://github.com/digininja/DVWA.git
sudo mv DVWA/ /var/www/html/
cd /var/www/html/DVWA/
cp config/config.inc.php.dist config/config.inc.php
```
Database setup is also necessary:
```
sudo mysql

mysql> create database dvwa;
```
Query OK, 1 row affected (0.00 sec)
```
mysql> create user dvwa@localhost identified by 'p@ssw0rd';
```
Query OK, 0 rows affected (0.01 sec)
```
mysql> grant all on dvwa.* to dvwa@localhost;
```
Query OK, 0 rows affected (0.01 sec)
```
mysql> flush privileges;
```
Query OK, 0 rows affected (0.00 sec)

Assuming you have Kali-Purple ready, use a desktop viewer and xrdp to connect to 192.168.1.20 (Kali-Purple's OpenVPN should take care of this). Once connected, open Heliotrope's browser and navigate to <kali-Pearly's IP address>/DVWA/login.php. Log in with the username 'admin' and the password 'password', 

### Kali-Heliotrope
A Kali-Basic machine with XRDP and XFCE desktop installed and basic tools. This image can be found on AWS as 'ami-03d8d342a533f5c75'. Point a desktop viewer with the XRDP protocol to 192.168.1.20.

### Kali-Eminence
A Kali-Basic machine with Malcolm installed and running. This image can be found on AWS as 'ami-04563635f0999dc56'. To start Malcolm you need to input the following command. 

```
./scripts/start
```
Now you can access the dashboard and other pages (Kali-Purple's OpenVPN should take care of this).

Point your browser to the following links:

https://192.168.253.103/

https://192.168.253.103/dashboard

https://192.168.253.103/upload

https://192.168.253.103/netbox

https://192.168.253.103/cyberchef

https://192.168.253.103/readme

https://192.168.253.103/name-map-ui

https://192.168.253.103:488

https://192.168.253.103:9443

https://192.168.253.103:8022/files


### Kali-Bizantium
Rather than a machine image, in this case the most efficient way to share this machine is via it's configuration file ![Bizantium Config](/config-byzantium.localdomain.xml). You need to edit the file and paste your ip address in the places were 'XXX.XXX.XXX.XXX' appear. Launch the default OPNsense image in the stack and upload the configuration file. The passwords for this configuation are the following. 
```
**********************************************************************************************************

*** set initial ec2-user password to : PHbbdPhJjUfq4J43R8SLepe2JDWGGiXpDRJNzEocz7YVlXKMY5

*** !!! remember to change this immediately

*** openssh-key provided, set to ec2-user

*** set initial root password to : 6s89E3cPU9oE3uRHN59H4jEkQ7n6udvV8nGxpocXRKRyHNMwHo

*** remember to change this immediately

**********************************************************************************************************
```          
I left the OpenVPN configuration, you need to change the hostname in the client export section to the ip address of the ec2 instance. I left the certificates, to help you see how I configured it. You might want to change this. The username and password for the OpenVPN are the following.
```
Username: kaliopenvpn
Password: bizantium
```
### Kali-Purple
The Kali-Purple image is 'ami-0f0e440cf357443a4' and the Elasticsearch credentials are the following:
```
Username: elastic
Password: 9voOW_WV6AO3EifKz=uu
```

Point the browser to https://192.168.253.105:5601

### Kali Violet
The Kali-Violet image is 'ami-0ea952e3e2d36ebad'. The credentials are he following.

OpenCTI
```
Username: admin@opencti.io
Password: kalipurpleSOCCTI
```
OpenCTI Portainer
```
Username: admin
Password: kalipurpleSOCPortainer
```
GVM
```
Username: admin
Password: efa72ac9-95fe-496e-b110-e68baa757ea5
```

The links are:

https://192.178.253.107:8080

https://192.178.253.107:9392

## Tips
- The Byzantium machine needs 3 interfaces (LAN, WAN, and SOC). OpnSense may get them mixed up when it launches. Obtain the MAC address of the interfaces in the interfaces section of AWS and assign them to the appropriate subnet in the interfaces menu of OPNsense. If you can't access the login screen, relaunch the stack.

- When the Byzantium machine is stopped, it may lose the public IP address assigned. You need to create an Elastic IP and assign it to the WAN interface to solve this issue.

## Notes

- Please note that this repository is still work in progress.

- CloudFormation is an AWS service and cannot be used for provisioning infrastructure on other cloud platforms like Azure, Google Cloud, etc. For those platforms, you need to use Terraform instead. A tool to make this conversion is available on this page: https://discuss.hashicorp.com/t/tool-to-convert-cloudformation-to-terraform/46704. Keep in mind that the tool may require some tweaking to work properly.

- Reference: https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/home

## Cost

The cost of running this setup is approximately $6 per day, and I use the instances for 5 hours each day, stopping them when not in use.

![Cost History](https://user-images.githubusercontent.com/47893772/231023307-d604dc42-dcd1-4a30-92eb-4d333c99df88.png)

The VPC stack in AWS is free, so you can leave it running indefinitely. However, keep in mind that AWS will charge for services in the EC2 stack, so be sure to delete it once you no longer needed to avoid unnecessary charges.


## Screenshots

![Screenshot 2023-04-10 at 10-44-47 Dashboard Lobby byzantium localdomain](https://user-images.githubusercontent.com/47893772/231025324-626561a3-dcc8-41b0-b57e-dfac77f23fed.png)



![Screenshot 2023-04-10 at 10-38-02 OpenCTI - Cyber Threat Intelligence Platform](https://user-images.githubusercontent.com/47893772/231025203-42793b70-dacb-4ff2-9793-a97d79280b7e.png)

![Screenshot 2023-04-10 at 10-44-17 Agents - Fleet - Elastic](https://user-images.githubusercontent.com/47893772/231025243-f8311ab8-cef1-4990-8c8b-eab654e8c0d1.png)

![Screenshot 2023-04-10 at 10-51-18 Elastic](https://user-images.githubusercontent.com/47893772/231025426-b07429a7-9545-4d3e-8cdb-d691dc91830d.png)

![Screenshot 2023-04-10 at 11-19-14 docker-cluster - Sessions](https://user-images.githubusercontent.com/47893772/231025535-7482c7d3-6ba7-4b97-a2f6-b61fc5c36640.png)


![Screenshot 2023-04-10 at 11-21-38 Home NetBox](https://user-images.githubusercontent.com/47893772/231025588-3894ed6c-d207-426b-b0ef-4a89e4a63e43.png)


![Screenshot 2023-04-10 at 11-22-54 CyberChef](https://user-images.githubusercontent.com/47893772/231025632-d2836ca4-a7d6-4519-adfa-9cc7e039af6a.png)



![Kali-Heliotrope-Autopilot](https://user-images.githubusercontent.com/47893772/231025109-7bc3b7c0-e33b-4724-bcfd-fff817849d7f.png)


