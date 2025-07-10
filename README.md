# Lab-Project---DNS

This lab project was created as a simple showcase of how I can troubleshoot networking issues.

Scenario: Multiple users on a particular subnet are being redirected to a suspicious website. Server team has reported updates to DMZ server that acts as the DNS Server for the organization.

Tools Used:
- Ping | Ipconfig /all | NSlookup | Browser | Windows command prompt

TROUBLESHOOTING EXPLANATION and SOLUTION

When I approached the problem, I focused on a few keywords that popped out to me (redirect, multiple users, recent updates on the DNS server), which alerted me of a potential issue with the DNS server that is causing the issue. The ticket also stated that the issue only affected www.wgu.edu, which narrowed the issue down to possibly only a single site having the issue. I mainly used the "Windows Desktop 1" device in USER_Net for troubleshooting. The issue affected multiple users, so I am sure to resolve it for all by resolving it for one. 

First, I started by ensuring that the issue reported by the ticket was accurate. I checked the "Windows Desktop 1" ethernet card configuration using the command >ipconfig /all for correctness. Then, I used the ping command to ping the default gateway and DNS server. The command was successful for the default gateway but not the DNS server; specifically, the response was "Request Timed Out". I wanted to test this further since it was unclear whether the issue was a firewall setting or a route misconfiguration. To do this, I opened a web browser and accessed google.com and Wikipedia.com. The ability to access various websites made it clear to me that the next step was to inspect the DNS server. I knew I could query the DNS server with the nslookup command. I used this command >nslookup wgu.edu, and as a result, I received wgu.edu = 10.10.20.2. I knew this was wrong since the IP pointed to a local server, and it was a private IP address. I went straight to the server and opened the DNS record for wgu.edu. The record showed what the query showed: a private IP address. I just needed to update this record with the correct IP address. I found the address by going to the VM hosting GNS3 "WGU-WIN10-GNS3", opening the command prompt, and using the command >nslookup wgu.edu. The result was a public IP address (151.101.194.224). Returning to the DNS server, I updated the record with the public IP address. Lastly, I returned to the "Windows Desktop 1" VM and did another >nslookup wgu.edu command. This time, the result was the public IP address. I now opened a web browser and went to wgu.edu, and the client browser went straight there without being redirected. Based on the steps taken, updating the DNS record for wgu.edu resolved the issue described by the ticket.


TROUBLESHOOTING STEPS

“Windows Desktop 1”

Step 1.	Open the command prompt and use the command >ipconfig /all. From here, I determined that the PC had the proper configurations.
              
    i.	IP: 10.10.40.2/24
    ii.	Default Gateway: 10.10.40.254
    iii.	DNS Server: 10.10.20.3

Step 2.	Ping Default Gateway from “Windows Desktop 1” VM. >ping 10.10.40.254  Success.

Step 3.	Ping DNS server from “Windows Desktop 1” VM using the command >ping 10.10.20.3. 

Step 4.	Command returned “Request timed out”. I know a message like this generally means no connectivity to the device, however specifically it can also mean a few more things.

    i.	There is a route missing either to the DNS server from the default gateway (router) or from the DNS server to the default gateway (router).
    ii.	There is a firewall setting blocking ICMP.

![Picture11](https://github.com/user-attachments/assets/c19c6eb3-d64b-4395-9683-23d11a26e2f1)

Step 5.	To confirm that there was a connection to the DNS and that the ticket issue is mainly with www.wgu.edu, from the “Windows Desktop 1” I used a web browser (Firefox) to see if any other sites were reachable. Sites searched (Google.com and Wikipedia.com).
Success.

    i.	This confirmed that there was access to the Internet from the “Windows Desktop PC”, however this did not mean the issue with the site www.wgu.edu was resolved. This pointed me in the direction of inspecting the DNS record.
    
    *The previous steps were tests to confirm that what the ticket reported was accurate. Moving forward the troubleshooting is mainly focus on resolving any DNS issues. *

![Picture12](https://github.com/user-attachments/assets/d4b9d3ca-3dfa-4d15-9001-81fdfeb662d3)

Step 6.	The next step is determining what is received when Windows Desktop 1 queries the DNS       
                server. I used the command >nslookup www.wgu.edu. This tool lets me enter a 
                hostname or FQDN to query the DNS server for an associated IP address (DNS record).
                
    i.	Result: www.wgu.edu -> 10.10.20.2
    
Step 7.	The result shows the IP address 10.10.20.2, a private IP address (thus not a globally 
               routable IP) to "DMZ_Server_2". This pointed me to check the DNS record for 
               www.wgu.edu at "DMZ_Server_3" (DNS server).

![Picture13](https://github.com/user-attachments/assets/ad9d2d30-681b-480c-aae8-e8d6099af172)

“DMZ Server 3:

Step 8.	After logging into “DMZ_Server_3”, I opened active directory and went to the DNS 
              setting to find the DNS record for www.wgu.edu. (Server Manager  Tools  DNS  
              Forward Lookup Zones  wgu.edu). The A records report the following.
              
    i.	Host (A): 10.10.20.2
    
Step 9.	I can see from this that the server team should have put the accurate IP address for 
               www.wgu.edu during the update. I need to input the correct IP address in the DNS 
               record here to fix this issue. First, I needed to learn the actual IP for www.wgu.edu.
               
“WGU-Win10-GNS3 Lab VM”

Step 10.	I used the fact that the VM hosting GNS3 was connected to the Internet with an 
               accurate DNS server to perform a nslookup using the command prompt there.
               
    i.	Actual www.wgu.edu IP address: 151.101.194.224

![Picture14](https://github.com/user-attachments/assets/dc5bb9d7-9ab6-44ef-a950-d6e3f5b1e782)

“DMZ Server 3”

Step 11.	Back on the DNS server, I changed the A record IP addresses in the Active Directory to 
               the correct ones found before. At this point, I am confident the issue is resolved. I close 
               out of Active Directory and return to the end-user desktop.

![Picture15](https://github.com/user-attachments/assets/19e6cc86-be47-4c23-a3b1-c4882493ac3f)

“Windows Desktop 1”

Step 12.	On “Windows Desktop 1”, I again open command prompt and use the command 
    
    >nslookup www.wgu.edu to confirm changes.
    i.	Result: www.wgu.edu -> 151.101.194.224. Success.

![Picture16](https://github.com/user-attachments/assets/097ae0b9-5044-4e8b-b90d-2d35c3126806)

Step 13.	With the result of the nslookup command being accurate now, I open a web browser 
               and attempt to access www.wgu.edu. 

![Picture17](https://github.com/user-attachments/assets/7436ac0e-01fc-4d39-8feb-99e24880a1fa)

