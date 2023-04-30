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
 
To install this project, you need to have an AWS account and access to CloudFormation service. You first have to launch the VPC stack using the KaliPurple-VPC.yml file. Write "vpc" in the stack name for simplicity. It will create all the necessary subnets and security groups. I used the N. Virginia region so all images belong to it. If you want to use a different region you need to use different images. 
 
Once this stack is created, you need to create the instances using the KaliPurple-NAT-EC2.yml file. For simplicity use "ec2" as a name. You need to input the name of the VPC stack that was created previously.

![AWS Cloudformation](https://user-images.githubusercontent.com/47893772/231020706-82afa33e-b182-4b1f-9ed3-4f71fe0f1b63.png)


 
The EC2 stack gives you the possibility of choosing the instances that you want to launch. This way you don't have to pay for services that you don't need. I used the Guacamole Bastion initially but there is no need for it once the firewall and its OpenVPN is configured unless there is a problem with one of the instances.
 
I left the option of launching it if needed but it is not necessary for most cases. I also left the possibility of using an internet gateway for the instances in the SOC and LAN subnets to have access to the internet. Again, this option should not be necessary once the firewall is configured.

 ![AWS Instances](https://user-images.githubusercontent.com/47893772/231023159-8aa8e92c-73b9-4765-be08-45234d8d7950.png)
 
The instance types defaults are the minimum required for each to work. You can choose a bigger type if desired.
 
## Configuration
 
To set up the SOC, I could not find any Kali Purple images in AWS without product codes, so I used a regular Debian image and manually installed only the required packages for each of the machines. Additionally, some of the packages are not yet available or not well configured int the Kali repositories so I had to go to the original repositories. One example of this is the Elastic Stack. 

The cost of running this setup is approximately $6 per day, and I use the instances for 5 hours each day, stopping them when not in use.

![Cost History](https://user-images.githubusercontent.com/47893772/231023307-d604dc42-dcd1-4a30-92eb-4d333c99df88.png)

Note that the SOC setup process is lengthy and nuanced, as the instructions in the Kali-Purple documentation are not very clear, resulting in lots of trial and error. However, it is possible to set up the same configuration as in the Kali-Purple instructions for all machines except Bizantium, which requires some tweaking to avoid using VLANs. I also omitted the use of a domain name for simplicity.

Currently, the cloud configuration and firewall accept packets from all over the internet and in all of the internal connections, making it unsuitable for production situations but acceptable for proof of concept. To make it as close to production as possible, I will be hardening the AWS security groups, routing tables, NACs, and firewall rules.

In the future, I will be publishing a tutorial to help others replicate this setup, starting with the firewall setup, which is necessary for the rest of the instances unless an internet gateway is used.

Lastly, I have not yet attempted an attack the vulnerable Kali-Pearly machine. As soon as I do, I will publish some screenshots of the SOC.

 
## Usage
 
To use this project, you need to connect to the firewall instance using its public IP address and configure its OpenVPN service. You can then download and install the OpenVPN client on your machine and connect to the firewall using its private IP address. 
 
You can then access all the other instances in the SOC and LAN subnets using their private IP addresses through SSH or RDP protocols.
 
You can use Kali Linux as your attack platform on the kali-pearly instance and run various tools such as Nmap, Metasploit, Burp Suite, etc. You can also upload other vulnerable machine AMIs to perform attacks. Once the simulated attack is complete, you can delete the stack and pay only for the time used. 

Please note that this repository is still work in progress.

Reference: https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/home

CloudFormation is an AWS service and cannot be used for provisioning infrastructure on other cloud platforms like Azure, Google Cloud, etc. For those platforms, you need to use Terraform instead. A tool to make this conversion is available on this page: https://discuss.hashicorp.com/t/tool-to-convert-cloudformation-to-terraform/46704. Keep in mind that the tool may require some tweaking to work properly.

## AIM Images
Due to AWS restrictions, AIM images with product codes cannot be made public. Therefore, the official Kali AIM cannot be used as a base. Instead, a Debian AIM with the Kali repository added was used. Several methods exist for achieving this, but the method outlined in this article was chosen https://miloserdov.org/?p=3609&PageSpeed=noscript. When installing Kali packages, follow the syntax 
```
sudo aptitude install -t <package-name>.
```

These machines can be accessed within the Cloud Formation VPC, or they can be launched individually. If you wish to access them in the Cloud Formation VPC and have not yet configured OPNSense with OpenVPN, use a Bastion such as Guacamole since the machines are created in a private subnet. Simultaneously launching Guacamole is an option when launching the instances in CloudFormation.

All these machines have a user named ´kali' with the password 'kali2023'. The 'root' user's password is also 'kali2023'. The Elastic-Agent is not installed on the images. Install the agent once you have Kali-Purple running. 

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
A Kali-Basic machine with XRDP and XFCE desktop installed and basic tools. This image can be found on AWS as 'ami-03d8d342a533f5c75'.

### Kali-Eminence
A Kali-Basic machine with Malcolm installed and running. This image can be found on AWS as 'ami-04563635f0999dc56'. To start Malcolm you need to input the following command. 

```
./scripts/start
```
Now you can access the dashboard and other pages (Kali-Purple's OpenVPN should take care of this).
```
https://192.168.253.103/dashboard
```


### The remaining images will be available soon.

## Tips
- The Byzantium machine needs 3 interfaces (LAN, WAN, and SOC). AWS may get them mixed up when it launches. Obtain the MAC address of the interfaces in the interfaces section of AWS and assign them to the appropriate subnet in the interfaces menu of OPNsense.

- When the Byzantium machine is stopped, it may lose the public IP address assigned. You need to create an Elastic IP and assign it to the WAN interface to solve this issue.

## Screenshots

![Screenshot 2023-04-10 at 10-44-47 Dashboard Lobby byzantium localdomain](https://user-images.githubusercontent.com/47893772/231025324-626561a3-dcc8-41b0-b57e-dfac77f23fed.png)



![Screenshot 2023-04-10 at 10-38-02 OpenCTI - Cyber Threat Intelligence Platform](https://user-images.githubusercontent.com/47893772/231025203-42793b70-dacb-4ff2-9793-a97d79280b7e.png)

![Screenshot 2023-04-10 at 10-44-17 Agents - Fleet - Elastic](https://user-images.githubusercontent.com/47893772/231025243-f8311ab8-cef1-4990-8c8b-eab654e8c0d1.png)

![Screenshot 2023-04-10 at 10-51-18 Elastic](https://user-images.githubusercontent.com/47893772/231025426-b07429a7-9545-4d3e-8cdb-d691dc91830d.png)

![Screenshot 2023-04-10 at 11-19-14 docker-cluster - Sessions](https://user-images.githubusercontent.com/47893772/231025535-7482c7d3-6ba7-4b97-a2f6-b61fc5c36640.png)


![Screenshot 2023-04-10 at 11-21-38 Home NetBox](https://user-images.githubusercontent.com/47893772/231025588-3894ed6c-d207-426b-b0ef-4a89e4a63e43.png)


![Screenshot 2023-04-10 at 11-22-54 CyberChef](https://user-images.githubusercontent.com/47893772/231025632-d2836ca4-a7d6-4519-adfa-9cc7e039af6a.png)



![Kali-Heliotrope-Autopilot](https://user-images.githubusercontent.com/47893772/231025109-7bc3b7c0-e33b-4724-bcfd-fff817849d7f.png)


