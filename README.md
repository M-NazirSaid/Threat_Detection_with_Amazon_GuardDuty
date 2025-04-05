# Threat_Detection_with_Amazon_GuardDuty

<h2>Project Scope</h2>
This projects assumes that the reader is not an absolute beginner. That is to say the reader is already familiar with AWS terminologies and has access to <b>AWS Management Console</b>.


> [!NOTE]
> Make sure you terminate all services at the end of the practical to avoid incurring unnecessary cost at the end of the month.

In this project we will:-


1.	Start with an <b>overview of GuardDuty</b> and some of its <b>features</b>,
2.	Dive into the practical section and <b>lunch an EC2 Instance</b> with an <b>EBS volume</b> attached to it,
3.	Then connect to the EC2 instance via <b>EC2 Instance Connect to download our sample infected file from EICAR</b>,
4.	Then <b>enable GuarDuty and launch an “On-demand Malware scanning”</b> on our EC2 Instance, and
5.	Finally analyze one finding using <b>Hybrid Analysis Cyber Threat Intelligence</b> platform.

<h2>Overview Of Amazon GuardDuty</h2>

<b>“Amazon GuardDuty is a threat detection service that continuously monitors your AWS accounts and workloads for malicious activity and delivers detailed security findings for visibility and remediation.”</b> - AWS Documentation

GuardDuty uses Machine Learning algorithms to perform anomaly detection using 3rd party data. As we will see, you just need a few clicks to enable it and run a 30 days free trial. This means that you do not need to install any 3rd party software like <b><i>TrendMicro’s Deep Security</i></b>. You also do not need to do any configurations as GuardDuty comes fully configured by AWS to look for the malicious activities, although you can specify certain IP’s you wish to include/exempt from your scanning.

<h3>Input Data</h3>

You are probably wondering "where does the log data comes from?" Well even though this is beyond the scope of our project, GuardDuty cab be set to receive logs from:-
- <b>CloudTrail Events Logs</b> – such as unusual API calls, unauthorized deployment of services, etc.

  <b><i>CloudTrail Management Events</i></b> – such as VPC subnet creation, trail creation, etc.
  
  <b><i>CloudTrail S3 Data Events</i></b> – get object, list objects, delete object, put object, etc.

- <b>VPC Flow Logs</b> – unusual internal traffic, unusual IP addresses, etc
- <b>DNS Logs</b> – compromised EC2 instances sending encoded data within DNS queries.
- GuardDuty can also analyze <b>EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events,</b> etc to identify malicious activities. It can also be used to <b>setup EventBridge rules</b> to be notified in case of interesting findings.
- GuardDuty can help protect against <b>Cryptocurrency attacks</b> as it will generate a dedicated finding for such occasions when your resource is interacting with an <b>IP address</b> or a <b>Domain Name</b> that is know for Crypto related activities.

Finally, it should be noted that at the time of this publication <b>“Malware Protection for EC2”</b> is one out of a seven GuardDuty features known as Protection Plans. The other six are <b>S3 Protection, EKS Protection, Extended Threat Detection, Runtime Monitoring, Malware Protection for S3, RDS Protection, Lambda Protection</b>.

<h2>EC2 Instance Launch</h2>

To launch the EC instance with the EBS volume attached to it we will:-

1.	Search for <b>"EC2"</b> service in the <b>Services search bar</b>. 
2.	Click on <b>EC2</b> which is the first option on the list.  <img width="956" alt="1 - EC2 Launch" src="https://github.com/user-attachments/assets/6b0873a5-918b-420a-9682-bfd07266d875" />

3.	Click on <b>“Instances”</b> on the left pane of the EC2 service page.
4.	Click on either of the two <b>“Launch Instances”</b> buttons as circled on the screenshot below.  <img width="962" alt="2 - EC2 Launch" src="https://github.com/user-attachments/assets/470e079b-5d11-40b9-8fbe-67753c5c5ba0" />

5.	Type in the name you want for your EC2 Instance in the <b>“Name and Tag”</b> textbox.
6.	Select the <b>OS Image type</b> you wish to use; <b>“Amazon Linux”</b> in our case.
7.	Expand the listbox and choose any of the free tier Amazon Machine Image; <b>Amazon Linux 2023 AMI</b> in our case.  <img width="959" alt="3 - EC2 Launch OS Spec" src="https://github.com/user-attachments/assets/e5a6e7bf-2866-480a-8c7e-addda2421f77" />

8.	Leave <b>Architecture</b> as is <b>64-bit (x86)</b> and <b>Instance Type</b> as <b>t2.micro</b> <img width="959" alt="4 - EC2 Launch OS Spec" src="https://github.com/user-attachments/assets/972ee195-7489-4a7a-9b70-6f670b996233" />


9.	At this point we choose the <b>key pair</b> that will be used to access the EC2 and <b>“Allow SSH traffic from Anywhere 0.0.0.0/0”</b>.Now there are 2 very important things to note here.  <img width="959" alt="5 - EC2 Launch OS key pair" src="https://github.com/user-attachments/assets/9fcb6d32-f8f1-48c0-a9bd-acc26c902187" />


>[!IMPORTANT]
>I already have a key pair created <b>"ec2-key"</b>, so I just chose it. However, you can choose to <b>Proceed without a key pair</b> or click on <b>create new key pair</b> to create yours.
#I will later make an illustration on <b>how to create new key pair</b> and will hopefully attach it here.

>[!WARNING]
>It is against <b>AWS Security Best Practice</b> to launch an EC2 instance without an encryption key pair or <b>Allow SSH traffic from anywhere</b> because doing so will allow anyone to have an <b>unrestricted access</b> to your EC2 instance.

10.	Then for the <b>Storage</b> leave everything as default, for we do not require huge storage for this practical and then click on <b>Launch Instance</b>.  <img width="959" alt="6 - EC2 Launch EBS" src="https://github.com/user-attachments/assets/a3fd21c4-4a2a-4eba-822d-8e4ab195a3ad" />


11.	Allow the process to complete, and in case any of the steps failed you can <b><i>restart failed</i></b> steps. <img width="959" alt="7 - EC2 Launch progress" src="https://github.com/user-attachments/assets/cda6577c-ef51-40f1-8ad4-a18a7def2314" />  

12.	After completion, click on the <b>Instance ID i-01497d303bad2c421</b> in my case from the Success page displayed in order to start the instance. Also notice the <b>Launch log</b> showing that a <b>Security Group (AWS equivalent of endpoint Firewall)</b> has been configured for you based on the <b><i>allow SSH from 0.0.0.0/0</i></b> in <b>stage 9</b>.   <img width="962" alt="8 - EC2 Launch Success" src="https://github.com/user-attachments/assets/39b7c45d-6c3f-4652-b8f4-cdbfb3a7f806" />

13.	<b>Voila!</b> Your EC2 Instance has been launched. Take note of the circled areas showing your <b>EC2 configuration</b>.
<img width="959" alt="9 - EC2 Launch Details" src="https://github.com/user-attachments/assets/b493d4d1-e916-40ff-b64d-93b2a220d1fc" />


<h2>Connecting to EC2 to Download Malware Sample EICAR</h2>

At this stage we will connect to the EC2 and download a sample malware from the internet. This will be the test material that GuardDuty will analyze and discover the malicious artifacts after we launched our scan. Below are the steps to follow:-

1.	Click on <b>Connect</b> from the upper right corner of the Instance page.  <img width="959" alt="10 - EC2 Connect" src="https://github.com/user-attachments/assets/9a0c6d18-e37c-4a59-b6b7-965ddf2e8187" />

2.	Connect using <b>EC2 Instance Connect</b> by clicking on <b>Connect</b> at the lower right corner and a <b>Command Line Interface</b> will be displayed.  <img width="958" alt="11 - EC2 Connect" src="https://github.com/user-attachments/assets/3529ee86-7b54-460d-ac15-5f487f706419" />

3.	Go to <b>https://www.eicar.org/download-anti-malware-testfile/</b> on your browser and copy the <b>EICAR.COM-ZIP</b> link address as seen from the below screenshot.  <img width="955" alt="12 - EICAR download" src="https://github.com/user-attachments/assets/9dad4334-1ca5-442a-8aea-1d6fc01fcbd6" />

4.	Go back to the EC2 Connect CLI and use the command <b>“wget”</b> and paste the copied link <b>(https://www.eicar.org/download/eicar_com-zip/)</b>, then hit <b>Enter</b>. This will download the Malware Test file on to your Instance.  <img width="959" alt="13 - EICAR download" src="https://github.com/user-attachments/assets/a96fad93-882c-4f73-8ea2-b7137113187a" />

5.	To verify the download success, you can do a linux <b><i>“ls command”</i></b> and you should see the <b>index.html</b> file.

<h2>Perform Malware Scanning with Amazon GuardDuty</h2>

At this stage we have already launched our EC2 instance and have also downloaded the malware sample on to the machine. Now what is left is to enable GuardDuty and start our scanning. It is worth mentioning at this stage that Amazon offers 2 types of malware scannings, <b>GuardDuty-Initiated malware scan</b> and <b>On-demand malware scan</b>. The former is automatic i.e GuardDuty starts the scanning once a certain <b><i>finding with potential presence of malware is discovered on an EC2 instance or a container</i></b>. However, the later can be initiated at anytime following the outlined steps below:-

1.	Open a new <b>AWS Management Console</b> window, search for and open <b>GuardDuty</b> just as we did to launch EC2 at the beginning and click on <b>Get Started</b>.  <img width="959" alt="14 - Enable GuardDuty" src="https://github.com/user-attachments/assets/7911695d-98c2-4851-b3ab-fc5f41a8fcb9" />

2.	Then scroll down and click on <b>Enable GuardDuty</b>.  <img width="959" alt="15 - Enable GuardDuty" src="https://github.com/user-attachments/assets/37ec11b1-ea89-4398-acc0-c0b290ffdd4e" />

3.	From the left pane of GuardDuty service page click on <b>EC2 Malware Scans</b>. Notice the <b>Summary dashboard</b> with the interesting things circled.  <img width="976" alt="16 - GuardDuty EC scan" src="https://github.com/user-attachments/assets/6e326d35-3eb9-4e5a-8925-b08460f95306" />

4.	Click on either of the two <b>Start a new on-demand scan</b> buttons.  <img width="959" alt="17 - GuardDuty EC scan" src="https://github.com/user-attachments/assets/db27af57-4ca7-4fc2-ae3c-d2cf879c9855" />

5.	From the next page, copy the sample <b>arn</b> and replace the <b>account ID & instance ID</b> with your specific account & instance IDs and click on <b>Connect</b> at the lower right corner of the page.  <img width="959" alt="18 - GuardDuty EC scan" src="https://github.com/user-attachments/assets/3faf0f0a-3820-450b-bcb4-f4e56c6c54dc" />

6.	Allow the scanning to run for a couple minutes and refresh the page. The <b>Scan result</b> will show <b>Infected</b>.  
<img width="958" alt="19 - GuardDuty EC scan completed" src="https://github.com/user-attachments/assets/37bbc7e4-f844-42c4-94bd-0883d13ae5e8" />


>[!NOTE]
>Usually GuardDuty will be configured with <b>EventBridge</b> to either <b>trigger a Lambda Function</b> to perform some action or <b>send SNS notifications</b> to Security Analysts.

<h2>Analysis of Findings with Hybrid Analysis CTI</h2>

1.	Click on <b>Findings</b> to see the results of the scan. 
2.	Click on one of the findings and analyze the <b>Severity level, Resources affected, Threats Detected,</b> etc.   <img width="959" alt="20 - GuardDuty Findings Analysis" src="https://github.com/user-attachments/assets/154301c4-2b85-4dac-8b3e-cd633c9bc421" />
 
3.	Copy the <b>hash</b> and got to <b>Hybrid Analysis</b> https://hybrid-analysis.com/ (or any other CTI platform like <b>VirusTotal</b>) and analyze the results of the file hash.  <img width="959" alt="21 - Hybrid Analysis" src="https://github.com/user-attachments/assets/3d9e2c52-7f41-48ed-a256-1f2ab237cca0" />

4.	Finally go ahead and Disable the GuardDuty service by clicking on <b>Settings</b> and then <b>Disable</b> button.  
<img width="959" alt="22 - Disable GuardDuty" src="https://github.com/user-attachments/assets/d3cd1f40-07a3-4b59-a7bf-812f61702fe9" />

THANKS FOR SEEING THIS THROUGH :blush:
