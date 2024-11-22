# Incident Response Writeup - SQLi

This is a simulated SOC alert from LetsDefend.  

LetsDefend is a great Blue-team training platform that enables people to gain practical hands-on experience by investigating real cyber-attacks inside a simulated SOC.

### SOC165 - Possible SQL Injection Payload Detected 

---

![Screenshot from 2024-11-21 16-50-36](https://github.com/user-attachments/assets/ae7e0dc4-48a5-4037-9613-cba425efa284)

Based on the information the alert provided, a server in the environment may be compromised due to consecutive requests to the same page. 

The first thing to do is to take ownership of the ticket and create a case for this alert while utilizing the playbook.  Before creating a case, it’s important to review the ticket and take note of the following: 

- Hostnames 
- Destination IP address 
- Source IP address 
- Requested URL 
- File names 
- Hashes 

Since this is a web attack, we must know the IP address, HTTP request method, and URL to begin our investigation.  

Creating a case allows the SOC to prioritize incidents based on severity and potential impact.  This helps to ensure that the most critical issues are addressed first, which improves resource allocation and response times. 


<p align="center">
  <img width="460" height="300" src="https://github.com/user-attachments/assets/055f8ee6-145f-4403-9425-bf61f0db9ec7">
</p>


Since I have decided to take ownership of EventID: 115, clicking on the icon under “Action” in the investigation channel makes it official.  

Playbooks are crucial for a Security Operations Center for several reasons.  A couple of those reasons are consistency and standardization.  Playbooks ensure that incident responses are handled uniformly team-wide.  By standardizing procedures, SOC analysts can respond to threats in a uniform manner, reducing the chances of errors or missed steps. 


![Screenshot from 2024-11-21 16-39-53](https://github.com/user-attachments/assets/201ffbf0-bbce-4183-b9a7-98f98f0d53e7)

> Once I have created the case, I can begin running plays from the playbook.

<p align="center">
  <img width="550" height="390" src="https://github.com/user-attachments/assets/0d4ca549-0748-442e-8ef4-d13131230034">
</p>

As analysts, it’s important to verify whether the alert is a true positive or a false positive.  But it’s also important to understand why.

> Let’s verify some crucial information within our alert.

![Screenshot from 2024-11-21 16-53-03](https://github.com/user-attachments/assets/5f18840e-c941-4f2f-a670-d74ce75415b7)

> If you look at device action, it shows that this was allowed.

Analysis is so important in cybersecurity for maintaining a robust security posture, which enables organizations to: 

- Detect and respond to threats effectively 
- Manage risks 
- Ensure compliance 
- continuously improve their security strategies.

<p align="center">
  <img width="550" height="390" src="https://github.com/user-attachments/assets/67c4c208-50e0-4b6f-8d27-311124f193de">
</p>

The first task to do is to investigate the destination and source IP addresses.  After some sleuthing on the Log Management page, I have found that the destination IP address is an endpoint in our network, which is WebServer1001.   

As for the source IP address, we can use VirusTotal to get a better picture of who the actor is.  This resource can tell us things such as the GeoLocation to help us understand who the threat actor may be. 

![Screenshot from 2024-11-21 12-55-21](https://github.com/user-attachments/assets/08a1236b-542c-4550-bb91-8c70ae3a61ef)

As you can see from the results above, VirusTotal, returns that 6/94 security vendors have flagged this IP address as malicious. 

> Let’s decode the requested URL to get more information.

![Screenshot from 2024-11-21 17-04-17](https://github.com/user-attachments/assets/8535d804-5a87-40cb-a1a3-c2acc1237cb6)

> You can use https://www.urldecoder.org/ or another similar tool for this.

You can see SQL activity when you decode the URL in the alert details.   

> For context, this is an SQL statement used by attackers where a SQL injection vulnerability exists.  Since 1=1 is always true, this will return what comes before or 1=1 (i.e. dumping of the table) will be returned.  

Since this indeed looks like an SQL injection attack, we can head back to the log management page and take note of the information in the Raw Log.  

<p align="center">
  <img width="550" height="390" src="https://github.com/user-attachments/assets/d59bf101-ce99-4f9c-a025-ada2f2b22d04">
</p>

Based on the information here, it shows that the device action was permitted, the HTTP response size is 948, and HTTP response status is 500.  The HTTP response status indicates that this is an internal server error, meaning that the malicious payload was most likely unsuccessful.   

### Playbook

---

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/8d63db69-9aab-417a-8799-71c05584569c">
</p>

Based on the information we already have; we can mark this as ”Malicious.”  A regular user wouldn’t use URL encoding nor would they try to commit a SQLi against a database.

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/549e13d9-2df4-4056-8e01-63005aa56f6a">
</p>

We have confirmed that this was an SQL injection. 

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/f2df76db-3938-4ec4-b49c-196df7cf96d5">
</p>

After searching through the Email Exchange, I found no evidence of a planned test.  So this alert was not part of a test.  

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/b595b9a5-afff-4d9f-b1ea-ac577982572e">
</p>

The traffic came from the internet and into our company network.   

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/7149b13b-4bfd-4b47-8dc7-ba74baebb699">
</p>

Based on the HTTP response status of 500 and no evidence of compromise in the endpoint, we can assume that the attack was unsuccessful.   

<p align="center">
  <img width="700" height="390" src="https://github.com/user-attachments/assets/05da6e10-b5fc-4c8e-b863-975a7a16ecba">
</p>

Here we can add some important artifacts for the report. 

<p align="center">
  <img width="550" height="600" src="https://github.com/user-attachments/assets/a6255fde-6725-492c-ab94-be31bba65705">
</p>

Now it is time to close the alert as a True Positive while also adding a quick summary of your findings. 

![Screenshot from 2024-11-22 16-17-29](https://github.com/user-attachments/assets/35e7af3d-7d4d-4294-91f3-c54b88a4298c)

Looks like everything was correct!  On to the next one.  
