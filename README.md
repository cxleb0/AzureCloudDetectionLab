I have recently completed the Azure Cloud Detection Lab by CyberWox Academy. This lab required me to complete the following: 
- Configure and Deploy Azure Resources such as Log Analytics Workspace, Virtual Machines, and Azure Sentinel
- Implement Network and Virtual Machine Security Best Practices
- Utilize Data Connectors to bring data into Sentinel for Analysis
- Understand Windows Security Event logs
- Configure Windows Security Policies
- Utilize KQL to query Logs
- Write Custom Analytic Rules to detect Microsoft Security Events
- Utilize MITRE ATT&CK to map adversary tactics, techniques, detection and mitigation procedures
Lab Network Design and Topolgy
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/fcefa2db-66ca-476b-9c54-390689253c82)

Initially, I created a free Microsoft Azure account and created the resource group called "labgroup".
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/73fb1de2-de8d-48a2-ad3b-d9b36c4977a3)

From there, I created the Windows 10 VM that I will be collecting the data from. I enabled an inbound port rule to allow access from port 3389(RDP).
To reduce the attack surface I enabled a security feature called Just In Time Access. What JIT access does it it protects our Azure VM from unauthorized network access.
JIT lets us access our VMs on the ports we need, for the period of time needed. JIT also implements principle of least privilege by giving the option to restrict access
to certain IP's as well as RBAC roles.
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/320755ed-05d7-4543-8010-f749fb56a400)

Next, I enabled Mirosoft Defender for Cloud and enabled Enhanced Security.
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/0521506b-3354-49f2-8a8d-4208e477cd38)

Then I configured a Log Analytic workspace. I configured it to use the same resource group as used for the Azure Virtual Machine. Once that was configured, I added Sentinel
to the workspace.
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/2e617604-154f-4cf4-929f-118e66ea19df)
Now initially I didnt see any incidents but Ill get into explaining why.

After the Sentinal Deployment, I went into the incidents tab and there was no incidents. That is due to the fact that there is no data being fed into sentinel.
I went to the Data connectors tab and this is where I selected what type of data I wanted to bring into my SIEM. I downloaded the Windows Security Events connector which
had Security Events via Legacy Agent and via AMA. Once we add collection rule, I named it and connected to the same resource group I have been using for this lab. I configured to 
collect all security events.

Now that I had everything configured, I needed to start generating security events to transport to my log. This is where the Windows 10 VM comes in as this is where I will be generating
events from. Once the VM is turned on, I grab the IP address for that device and remote in via Remote Desktop Connection on our host machine. Once entering the credentials that I used creating the VM
I entered in the IP and had access. 

From this point I opened the Event Viewer program and navigated to Windows Logs, then to the Security tab to view the events. I selected event with code 4624 as this was the code for successful logon attempt

After this I headed back to Azure Sentinel to pull the event from the Sentinel Logs.
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/5bf718bb-bddf-4dad-bdd8-55e998db72cb)
After using this KCL query (Kusto Query Language) I am able to pull all correlating events. 
Query Breakdown: 
Security Event - refers to the event table we are pulling the data from. All the events we observed in the event viewer are stored there.
Where - command filters on a specific category. In this case, we only want events that correspond to successful logins.
Project - command will specify what data to display when the query is run so, in this specific scenario, we want to only see the time the logon event occurred, what computer it came from and what account on this computer-generated the event.

After running the query, I was able to see a list of all the times I had a successful login on my VM.

In Sentinel I have the option to be alerted to certain events by setting up analytic rules. The analytic rule checks the VM for activity that matches the rule logic and will generate an alert once the activity occurs.
In the final part of the lab, I created my own custom rule to detect potentially malicious activity on my VM. To do this, I used Task Scheduler. Scheduled tasks can be a harmless event but could also be used as a 
persistence technique for malicous actors. 
According to the MITRE Attack Framework, “Adversaries may abuse task scheduling functionality to facilitate initial or recurring execution of malicious code. Utilities exist within all major operating systems to schedule 
programs or scripts to be executed at a specified date and time”.

The tasks demonstrated in this lab are not malicious. All the task does is open up Windows Explorer at a certain time but we will use this to create an analytic rule to monitor for that specific action so we can be alerted 
of the activity. 

By default, the Security Event ID that corresponds to a scheduled task creation is 4698 but these events are not logged by default in the Windows Event Viewer, so I had to make some changes in the Windows Security Policy on 
my VM. 
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/646a2c03-cef2-42d5-863f-b1f4cb0f0201)
I navigated to Local Security Policy then expanded Advanced Audit Policy Configuration. From there I expanded System Audit Policies and Select Object access. Then I selected the Aufit Other Object Access. I enabled success and failure and 
now logging is enabled for the schedueld task event.

To be able to detect a scheduled task creation, I needed to generate some activity in my VM. I created a task in Task Scheduler and configured it for Windows 10, then set the time I wanted it to execute then selected Internet Explorer
as the program to run everytime this task executed.

Lastyl, I wrote some KQL logic to aslert me when a scheduled task is created. I navigated to the Sentinel home page and clicked on Analytics Rules and create and selected the scheduled query option.
KCL query logic:
![initalsentinelviewaftergeneratingeventsinvm](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/1d776766-bed3-4d2f-8771-ac03c820d02a)

The project command displays data fields as columns when i=I run the query.
The parse command will allow me to extract data from the Event Data Field that I find important
SubjectUserName , TaskName, ClientProcessID is automatically displayed by the computer.
Next I enabled Alert Enrichment which is simply the process of adding context to the alert so it makes it easier to investigate.
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/545c5e0c-798d-48e9-876a-09d8e3175a22)
For query scheduling, I scheduled the query to run every 5 minutes and too look up data from the last to 5 minutes as well.

Now, I created another scheduled task in my VM and waited for it to appear Sentinel
![image](https://github.com/cxleb0/AzureCloudDetectionLab/assets/98493543/0769a540-ec71-4b5a-bdfa-10ba7b93a5e9)


This lab was extremely informative and easy to follow. I had a great time completing this lab as I finally got some exposure to Azure. Couldnt be more greateful for the guys at CyberWox.







