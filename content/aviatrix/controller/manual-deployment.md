---
title: " Manual Process"
date: 2022-08-23T13:57:51-05:00
draft: false
---

In every modern SDDC solution, the first step is to deploy a controller. The aviatrix controller can be deployed in any Cloud service provider but this example will focus on deploying the Aviatrix controller on AWS. You have multiple ways to deploy an Aviatrix controller, you could either deploy it manually (the sole purpose of this post) or automate it using [AWS cloudformation](https://docs.aviatrix.com/StartUpGuides/aws_getting_started_guide.html) or Terraform (more on this one in a later post)

It is usually recommended to install the controller in a VPC that is dedicated to general infrastructure resources as described previous or here

Once you have the VPC and all its mandatory network constructs needed to provide internet connectivity, you can deploy a VPC

If you remember what we did in the previous article we will just install the Aviatrix Controller into that VPC.

The very simple topology will look like this ![AVX Controller](/images/aviatrix/controller/manual-deployment/Fig1-avxcontroller.png).

We will have install the custom [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) for the Aviatrix Controller in the VPC and we want to have an [elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) assigned to this EC2 instance. That way, if you need to reboot or stop the controller at any point, the IP address assigned will stay the same and you will not have to change your DNS record.


The first step is to deploy the Aviatrix EC2 instance using the AMI marketplace in the EC2 console as show below:
![AVX ControllerEC2](/images/aviatrix/controller/manual-deployment/Fig2-EC2launch.png).

Once you are in the AMI marketplace, you can search for a variety of images that are available for you. Just type "Aviatrix" and you will have the choise to deploy different solutions (controller or copilot.)
As I have my own license already, I am choosing the "Aviatrix Secure Networking Platform - BYOL" option. (Do not hesitate to [reach to Aviatrix](https://aviatrix.com/#live-chat) to understand the different licensing options).

![AVX ControllerAMI](/images/aviatrix/controller/manual-deployment/Fig3-AVXami.png).

Once you select the right option for you, you will have to configure your EC2 instance.
I gave it a name and an instance type (t3.large) and a previously created key-pair.

![AVX ControllerEC2Config](/images/aviatrix/controller/manual-deployment/Fig4-EC2Config01.png).

The networking config is critical as it will obviously provide the connectivity to the internet and other VPCs.
You need to select the VPC previously created and make sure to select a public subnet as well.
Make sure to not "auto-assign a public IP" to this EC2 instance, we will assign an elastic IP one. If you assign one now, the IP address will change if you stop the controller. 
Also, verify that the right security group is selected, it should authorize inbound SSH and TLS traffic from your management IP address (Home or on premise Data center for example).

![AVX ControllerEC2ConfigNetwork](/images/aviatrix/controller/manual-deployment/Fig5-EC2Config02.png).


Once we are done with the networking config, we can launch the controller and then assign an Elastic IP address:


![AVX ControllerEC2EIP](/images/aviatrix/controller/manual-deployment/Fig6-ElasticIP.png).

The process to assign an Elastic IP is very straightfoward, you just have to claim an IP address that you assign to the right VPC and configure that IP address to the EC2 instance of your choice (The AVX controller in our case).


![AVX ControllerEC2EIP](/images/aviatrix/controller/manual-deployment/Fig7-ElasticIPassigned02.png).


Once you have assigned the IP address, you should be able to access it using HTTPS (Make sure to login using the following credential, the username is "admin" and the password is the private IP address assigned by AWS within your VPC/Subnet):


![AVX Controller login](/images/aviatrix/controller/manual-deployment/Fig8-test.png).

Then, the controller will ask for your email to receive the initial notification and it will upgrade itself to the latest version.


![AVX Controller Email](/images/aviatrix/controller/manual-deployment/Fig9-email.png).
![AVX Controller upgrade](/images/aviatrix/controller/manual-deployment/Fig11-upgrade.png).


Now that your Aviatrix controller is deployed, you are almost ready to fly. You just need to make sure that your AWS (or other CSP) account is configured into the Aviatrix controller to orchestrate your CSP native constructs and deploy the Aviatrix gateways.


![AVX Controller Welcome](/images/aviatrix/controller/manual-deployment/Fig12-welcome.png).

Click on the AWS icon and fill the fields with your CSP account information. you can follow this [documentation](https://docs.aviatrix.com/StartUpGuides/aws_getting_started_guide.html) or even better this entire [chapter](https://docs.aviatrix.com/HowTos/onboarding_faq.html)



![AVX Controller AWS Setup](/images/aviatrix/controller/manual-deployment/Fig13-AWSaccount.png).

It is possible that your Aviatrix controller EC2 instance is missing some IAM roles at the begining ! Make sure to fix this using this [documentation](https://docs.aviatrix.com/HowTos/onboarding_faq.html#why-do-i-need-an-aws-account-credential) or you will end up getting this error message:

![AVX Controller AWS Setup](/images/aviatrix/controller/manual-deployment/Fig13-IAMrole.png).

Modify the IAM role into your EC2 instance:

![AVX Controller AWS Setup](/images/aviatrix/controller/manual-deployment/Fig14-IAMroleassigned.png).

![AVX Controller AWS Setup](/images/aviatrix/controller/manual-deployment/Fig15-IAMroleassignedec2.png).


Now you should be able to perform your first solo flight using the Aviatrix solution on AWS. Enjoy and have a relaxing flight :) .

