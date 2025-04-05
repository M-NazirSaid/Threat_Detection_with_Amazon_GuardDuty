# Threat_Detection_with_Amazon_GuardDuty

<h2>Project Scope</h2>
This projects assumes that the reader is not an absolute beginner. That is the reader is already familiar with AWS terminologies and has access to AWS Management Console.


> [!NOTE]
> Make sure you terminate all services at the end of the practical to avoid incurring unnecessary cost at the end of the month.

In this project we will:-


1.	Start with an overview of GuardDuty and its features,
2.	Dive into the practical section and lunch an EC2 Instance with an EBS volume attached to it,
3.	Then connect to the EC2 instance via EC2 connect to download our sample infected file from IECR,
4.	Then we will enable GuarDuty and launch an “On-demand Malware scanning” on our EC2 Instance, and
5.	Finally analyze a sample finding.

<h2>Overview Of Amazon GuardDuty</h2>

<b>“Amazon GuardDuty is a threat detection service that continuously monitors your AWS accounts and workloads for malicious activity and delivers detailed security findings for visibility and remediation.”</b> - AWS Documentation

GuardDuty uses Machine Learning algorithms to perform anomaly detection using 3rd party data. As we will see, you just need a few clicks to enable it and run a 30 days free trial. This means that you do not need to install any 3rd party software like TrendMicro’s Deep Security. You also do not need to do any configurations as GuardDuty comes fully configured by AWS to look for the malicious activities, although you can specify certain IP’s you wish to include/exempt from your scanning.

<h3>Input Data</h3>

You are probably wondering where does the data comes from? Well GuardDuty cab be set to receive log data from:-
- <b>CloudTrail Events Logs</b> – such as unusual API calls, unauthorized deployment of services, etc.

  CloudTrail Management Events – such as VPC subnet creation, trail creation, etc.
  
  CloudTrail S3 Data Events – get object, list objects, delete object, put object, etc.

- <b>VPC Flow Logs</b> – unusual internal traffic, unusual IP addresses, etc
- DNS Logs – compromised EC2 instances sending encoded data within DNS queries.
- GuardDuty can also analyze EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events, etc to identify malicious activities. It can also be used to setup EventBridge rules to be notified in case of interesting findings.
- GuardDuty can help protect against <b>Cryptocurrency attacks</b> as it will generate a dedicated finding for such occasions when your resource is interacting with an <b>IP address</b> or a <b>Domain Name</b> that is know for Crypto related activities.

Finally, it should be noted that <b>“Malware Protection”</b> is one out of a five GuardDuty features known as Protection Plans. The other four are <b>S3 Protection, EKS Protection, RDS Protection & Lambda Protection</b>.

<h2>EC2 Instance Launch</h2>

To launch the EC instance with the EBS volume attached to it we will:-

1.	Search for <b>"EC2"</b> service in the <b>Services search bar</b>. 
2.	Click on <b>EC2</b> which is the first option on the list.  <img width="956" alt="1 - EC2 Launch" src="https://github.com/user-attachments/assets/6b0873a5-918b-420a-9682-bfd07266d875" />

3.	Click on <b>“Instances”</b> on the left pane of the EC2 service page.
4.	Click on either of the two <b>“Launch Instance”</b> buttons as circled on the screenshot below.  <img width="962" alt="2 - EC2 Launch" src="https://github.com/user-attachments/assets/470e079b-5d11-40b9-8fbe-67753c5c5ba0" />

5.	Type in the name of you want for your EC2 Instance in the <b>“Name and Tag”</b> textbox.
6.	Select the <b>OS Image type</b> you wish to use; <b>“Amazon Linux”</b> in our case.
7.	Expand the listbox and choose any of the free tier Amazon Machine Image; <b>Amazon Linux 2023 AMI</b> in our case.  <img width="959" alt="3 - EC2 Launch OS Spec" src="https://github.com/user-attachments/assets/e5a6e7bf-2866-480a-8c7e-addda2421f77" />

8.	Leave <b>Architecture</b> as is <b>64-bit (x86)</b> and <b>Instance Type</b> as <b>t2.micro</b>
9.	At this point we choose the <b>key pair</b> that will be used to access the EC2 and <b>“Allow SSH traffic from Anywhere 0.0.0.0/0”</b>.Now there are 2 very important things to note here.

>[!IMPORTANT]
>I already have a key pair created, so I just chose it. However, you can choose to proceed without key pair or click on “create new key pair” to create yours.
<b>#I will later make an illustration on “how to create new key pair” and will hopefully attach it here.#</b>

>[!WARNING]
>It is against AWS security Best Practice to launch an EC2 instance without an encryption key pair. Allow SSH traffic from anywhere. Doing so will allow anyone to have an unrestricted access to your EC2 instance.

10.	Then for the <b>Storage</b> leave everything as default, for we do not require huge storage for this practical and then click on Launch Instance.
11.	Allow the process to complete, and in case any of the steps failed you can <b><i>restart failed</i></b> steps.
12.	After completion, click on the <b>Instance ID i-01497d303bad2c421</b> in my case from the Success page displayed in order to start the instance. Also notice the <b>Launch log</b> showing that a <b>Security Group (AWS equivalent of endpoint Firewall)</b> has been configured for you based on the <b><i>allow SSH from 0.0.0.0/0</i></b> in <b>stage 9</b>. Also take note of the circled area showing your EC2 configurations.
13.	<b>Voila!</b> Your EC2 Instance has been launched.


<h2>Connecting to EC2 to Download</h2>

At this stage we will connect to the EC2 and download a sample malware from the internet. This will be the test material that GuardDuty will analyze and discover the malicious artifacts after we launched our scan. Below are the steps to follow:-

1.	Click on <b>Connect</b> from the upper right corner of the Instance page.
2.	
