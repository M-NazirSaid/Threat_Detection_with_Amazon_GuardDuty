# Threat_Detection_with_Amazon_GuardDuty

<h2>Project Scope</h2>
This projects assumes that the reader is not an absolute beginner. That is the reader is already familiar with AWS terminologies and has access to AWS Management Console.
In this project we will:-
1.	Start with an overview of GuardDuty and its features,
2.	Dive into the practical section and lunch an EC2 Instance with an EBS volume attached to it,
3.	Then connect to the EC2 instance via EC2 connect to download our sample infected file from IECR,
4.	Then we will enable GuarDuty and launch an “On-demand Malware scanning” on our EC2 Instance, and
5.	Finally analyze a sample finding
Overview Of Amazon Guardduty
 “Amazon GuardDuty is a threat detection service that continuously monitors your AWS accounts and workloads for malicious activity and delivers detailed security findings for visibility and remediation.”
- AWS Documentation
GuardDuty uses Machine Learning algorithms to perform anomaly detection using 3rd party data. As we will see, you just need a few clicks to enable it and run a 30 days trial. This means that you do not need to install any 3rd party software like TrendMicro’s Deep Security. You also do not need to do any configurations as GuardDuty comes fully configured by AWS to look for the malicious activities, although you can specify certain IP’s you wish to include/exempt from your scanning.
Input Data
You are probably wondering where does the data comes from. Well GuardDuty cab be set to receive log data from
•	CloudTrail Events Logs – such as unusual API calls, unauthorized deployment of services, etc.
-	CloudTrail Management Events – such as VPC subnet creation, trail creation, etc.
-	CloudTrail S3 Data Events – get object, list objects, delete object, put object, etc.
•	VPC Flow Logs – unusual internal traffic, unusual IP addresses, etc
•	DNS Logs – compromised EC2 instances sending encoded data within DNS queries
GuardDuty can also analyze EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events, etc to identify malicious activities. It can also be used to setup EventBridge rules to be notified in case of interesting findings. GuardDuty can help protect against Cryptocurrency attacks as it will generate a dedicated finding for such occasions when your resource is interacting with a IP or Domain Name that is know for Crypto mining activities.
Finally, it should be noted that “Malware Protection” is one out of a five GuardDuty features known as Protection Plans. The other four are S3 Protection, EKS Protection, RDS Protection and Lambda Protection.
EC2 Instance Launch
To launch the EC instance with the EBS volume attached to it we will
1.	Search for EC2 service in the services search bar. 
2.	Click on EC2 which is the first option on the list.
3.	Click on “Instances” on the left pane of the EC2 service page.
4.	Click on either of the two “Launch Instance” buttons as circled on the screenshot below.
5.	Type in the name of your EC2 Instance in the “Name and Tag” textbox.
6.	Select the OS Image type you wish to use; “Amazon Linux” in our case.
7.	Expand the listbox and choose any of the free tier Amazon Machine Image; Amazon Linux 2023 AMI in our case
8.	Leave Architecture as it is 64-bit (x86) and Instance Type as t2.micro
9.	At this point we choose the key pair that will be used to access the EC2 and “Allow SSH from traffic from Anywhere 0.0.0.0/0”. Now there are 2 very important things to note here.
a.	I already have a key pair created, so I just chose it. However, you can choose to proceed without key pair or click on “create new key pair” to create yours.
# I will later make an illustration on “how to create new key pair” and will hopefully attach it here. #
b.	It is against AWS security Best Practice to:
i.	Launch an EC2 instance without an encryption key pair
ii.	Allow SSH traffic from anywhere. Doing so will allow anyone to have an unrestricted access to your EC2 instance.
10.	Then for the Storage leave everything as default, for we do not require huge storage for this practical and then click on Launch Instance.
11.	Allow the progress to complete, and in case any of the steps failed you can restart failed steps.
12.	After completion, click on the Instance ID i-01497d303bad2c421 in my case from the Success page displayed. Also notice the Launch log showing that a Security Group (AWS equivalent of endpoint Firewall) has been configured for you based on the allow SSH from 0.0.0.0/0 in stage 9.
13.	Voila! Your EC2 Instance has been launched. Take note of the circled areas showing our configurations
Connecting to EC2 to Download
At this stage we will connect to the EC2 and download a sample malware from the internet. This will be the test material that GuardDuty will analyze and discover the malicious artifacts after we launched our scan. Below are the steps to follow:-
1.	Click on Connect from the upper right corner of the Instance page.
2.	
