# Kioptrix 2- Walkthrough

First We check what's our ip with simple shell command:
```
ifconfig
```
![alt text](/screens/ifconfig2.png)

Then we need to check the hosts running on our IP address with another shell command:
```
netdiscover
```
![alt text](/screens/netdiscover2.png)

**PCS Systemtechnik GmbH** is Virtual Box Vendor so it must be our VM.

Let's scan it with **nmap**!

		* -sS stands for TCP SYN scan 
		* -A  DetectS OS and Services
    
    
 ```
 nmap -sS -A <ip address>
 ```
 
 ![alt text](/screens/nmap2.png)
 
 Port 631 caught my eye immediatly and CUPS 1.1 had some known vulnerabilities but noone of the exploits worked.
 Lets search further.
 
 Port 80, which is standard http port is open.
 When accessed it gives us a login panel.
 
  ![alt text](/screens/login1.png)

I tried some default credentials but they didnt work.
Lets try to bypass it with SQL Injection.

We can guess that the query looks something like

```
SELECT * FROM users WHERE username = <our input> AND password = <out input>
```	

If we pass ' or '1'='1'-- query will look now like this:

```
SELECT * FROM users WHERE username = '' or '1'='1'--  AND password = <out input>
```
	
"-- " (with space in the end) is MYSQL comment sign. Passing it in the end means that the end of the query will be commented out, so the query really looks like:

```
SELECT * FROM users WHERE username = '' or '1'='1'--
```

OR statement means that only one of the arguments needs to be True and as '1'='1' is always True - We bypassed the login form.
  
  ![alt text](/screens/login2.png)
  
After login we get redirected to a Panel which lets us Ping the machines over internet.

  ![alt text](/screens/panel.png)
  
  Lets add another command with ; (semicolon) and see if it works.
  
  ![alt text](/screens/commandinjection.png)
  
Yes! It gives us back the output from both commands so the panel is vulnerable to Command Injection.
As output shows we're logged in as apache, so now we need to use privilege escalation in order to get the root.

### Privilege Escalation

Linux command uname -a (Unix name) prints back info about the name and version of operating system running on machine.

  ![alt text](/screens/panel2.png)
