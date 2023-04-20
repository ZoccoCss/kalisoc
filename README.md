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
 
To install this project, you need to have an AWS account and access to CloudFormation service. You first have to launch the VPC stack using the KaliPurple-VPC.yml file. Write "vpc" in the stack name for simplicity. It will create all the necessary subnets and security groups.
 
Once this stack is created, you need to create the instances using the KaliPurple-NAT-EC2.yml file. For simplicity use "ec2" as a name. You need to input the name of the VPC stack that was created previously.

![AWS Cloudformation](https://user-images.githubusercontent.com/47893772/231020706-82afa33e-b182-4b1f-9ed3-4f71fe0f1b63.png)


 
The EC2 stack gives you the possibility of choosing the instances that you want to launch. This way you don't have to pay for services that you don't need. I used the Guacamole Bastion initially but there is no need for it once the firewall and its OpenVPN is configured unless there is a problem with one of the instances.
 
I left the option of launching it if needed but it is not necessary for most cases. I also left the possibility of using an internet gateway for the instances in the SOC and LAN subnets to have access to the internet. Again, this option should not be necessary once the firewall is configured.

 ![AWS Instances](https://user-images.githubusercontent.com/47893772/231023159-8aa8e92c-73b9-4765-be08-45234d8d7950.png)
 
The instance types defaults are the minimum required for each to work. You can choose a bigger type if desired.
 
## Configuration
 
To set up the SOC, I could not find any Kali Purple images in AWS, so I used a regular Kali Linux image and manually installed only the required packages. Additionally, I followed the www.elastic.co documentation to install the Elastic stack, which was not available in Kali at the time.

While I had originally planned to make the final AMI images publicly available via Kali to simplify the setup process, I discovered that AWS does not allow AMIs with product codes to be made public. Additionally, due to the current risk associated with the firewall's log collection, it is not advisable to share them publicly until logs that contain private information are cleared. Therefore, I will be creating images without product codes and ensure that any logs with personal information are cleared before sharing them in the future.

The setup cost is approximately $7 per day, and I use the instances for 5 hours each day, stopping them when not in use.

![Cost History](https://user-images.githubusercontent.com/47893772/231023307-d604dc42-dcd1-4a30-92eb-4d333c99df88.png)

Note that the SOC setup process is lengthy and nuanced, as the instructions in the Kali-Purple documentation are not very clear, resulting in lots of trial and error. However, it is possible to set up the same configuration for all machines except Bizantium, which requires some tweaking to avoid using VLANs. I also omitted the use of a domain name for simplicity.

Currently, the cloud configuration and firewall accept packets from all over the internet, making it unsuitable for production situations but acceptable for proof of concept. To make it as close to production as possible, I will be hardening the AWS security groups, routing tables, NACs, and firewall rules.

In the future, I will be publishing a tutorial to help others replicate this setup, starting with the firewall setup, which is necessary for the rest of the instances unless an internet gateway is used.

Lastly, I have not yet attempted an attack the vulnerable Kali-Pearly machine. As soon as I do, I will publish some screenshots of the SOC.

 
## Usage
 
To use this project, you need to connect to the firewall instance using its public IP address and configure its OpenVPN service. You can then download and install the OpenVPN client on your machine and connect to the firewall using its private IP address. 
 
You can then access all the other instances in the SOC and LAN subnets using their private IP addresses through SSH or RDP protocols.
 
You can use Kali Linux as your attack platform on the kali-pearly instance and run various tools such as Nmap, Metasploit, Burp Suite, etc. You can also upload other vulnerable machine AMIs to perform attacks. Once the simulated attack is complete, you can delete the stack and pay only for the time used. 

Please note that this repository is still work in progress.

Reference: https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/home

## Screenshots

![Screenshot 2023-04-10 at 10-44-47 Dashboard Lobby byzantium localdomain](https://user-images.githubusercontent.com/47893772/231025324-626561a3-dcc8-41b0-b57e-dfac77f23fed.png)



![Screenshot 2023-04-10 at 10-38-02 OpenCTI - Cyber Threat Intelligence Platform](https://user-images.githubusercontent.com/47893772/231025203-42793b70-dacb-4ff2-9793-a97d79280b7e.png)

![Screenshot 2023-04-10 at 10-44-17 Agents - Fleet - Elastic](https://user-images.githubusercontent.com/47893772/231025243-f8311ab8-cef1-4990-8c8b-eab654e8c0d1.png)

![Screenshot 2023-04-10 at 10-51-18 Elastic](https://user-images.githubusercontent.com/47893772/231025426-b07429a7-9545-4d3e-8cdb-d691dc91830d.png)

![Screenshot 2023-04-10 at 11-19-14 docker-cluster - Sessions](https://user-images.githubusercontent.com/47893772/231025535-7482c7d3-6ba7-4b97-a2f6-b61fc5c36640.png)


![Screenshot 2023-04-10 at 11-21-38 Home NetBox](https://user-images.githubusercontent.com/47893772/231025588-3894ed6c-d207-426b-b0ef-4a89e4a63e43.png)


![Screenshot 2023-04-10 at 11-22-54 CyberChef](https://user-images.githubusercontent.com/47893772/231025632-d2836ca4-a7d6-4519-adfa-9cc7e039af6a.png)



![Kali-Heliotrope-Autopilot](https://user-images.githubusercontent.com/47893772/231025109-7bc3b7c0-e33b-4724-bcfd-fff817849d7f.png)


