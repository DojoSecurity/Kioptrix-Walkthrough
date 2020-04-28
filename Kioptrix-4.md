# Kioptrix #4

This machine is harder than the last three but I enjoyed it a lot and surely learned some new stuff.

First We check what's our ip address with simple shell command ```ifconfig```
![alt text](/screens/ifconfig2.png)

Then we need to check the hosts running on our IP address with another shell command ```netdiscover```
![alt text](/screens/netdiscover2.png)

**PCS Systemtechnik GmbH** is Virtual Box Vendor so it must be our VM.

Let's use **nmap** to see what's running on the machine.
```nmap -sS -A <ip address>```
* **-sS** stands for TCP SYN scan 
* **-A**  DetectS OS and Services

![alt text](/screens/nmap4.png)

So there's an application running on port 80, let's see what it holds.

![alt text](/screens/login4.png)

We have a basic login page in here.

#

### SQL Injection
SQL query on login page looks something like

```SELECT * FROM users WHERE username = 'username' AND password = 'password'```

Injecting **' or 'a'='a** inside password field works just fine and bypasses the login.

![alt text](/screens/admin4.png)

But how about the **Username** field? Our payload bypasses the Password field but username have to be valid anyway. 
Did i just guessed the **Admin** username right?
Well, not really.

![alt text](/screens/random4.png)

Entering any random username works. Weird.. but the content of the page doesnt help us anyway.
Let's use the SQL Injection not only to bypass the login but to get the content of the database.

Using **sqlmap** on POST request requires a longer and more complicated command but it can be simplified with **Burp Suite**.
Intercept the request and save it to a file.
Now all we need is **-r** flag and path to a file.
``` sqlmap -r <path to file> ```

We get some interesting stuff.

![alt text](/screens/sqlmap41.png)

Using these credentials in login panel works but again gives us nothing.
Fortunately they're SSH credentials also.

#
### Escaping the restricted shell

After login through ssh we get access to a restricted shell.

![alt text](/screens/shell4.png)

Both accounts seems to have the same privileges.
It gives us few commands like **echo ls ll**.
We get kicked out if we try to use **cd** or some other forbidden signs and commands. (Warning for the first try, then kick).

At this point I got stuck for quite a while. 
I switched back to **sqlmap** and used the **--os-shell** which gave me a shell with **var/www** user and way more privileges.
``` sqlmap -r <file with request> --os-shell ```

![alt text](/screens/varwww4.png)

With more privileged shell I noticed a **MySQL** process running with a root privileges.
While inspecting files in the **var/www**  i found a **checklogin.php** file.

![alt text](/screens/checklogin4.png)

It shows us the MySQL login credentials and that the password field is totally empty!

I tried getting a superior shell with ``` --os-pwn ``` option in sqlmap to access the MYSQL but it crashed everytime. (read the text in the end).
After feeling so close.. I had to go back to the restricted shell.

A good hour of research paid back in Python looking command:

``` echo os.shell('/bin/bash') ```

![alt text](/screens/binbash4.png)

It worked but I wasn't sure why.
Credits to some fellow stackoverflow user for explaining.
https://unix.stackexchange.com/questions/370769/why-does-echo-os-system-bin-bash-work
For more info on escaping restricted shells: https://www.sans.org/blog/escaping-restricted-linux-shells/

Finally! After escaping the restricted shell I could finally get my hands on MySQL.

#
### MySQL UDF - User Defined Functions

MySQL has something called User Defined Functions.
It let's MySQL execute commands on system and so if MySQL runs with root privileges.. that's the place for us.

Let's login to mysql
``` mysql -h localhost -u root -p ```

![alt text](/screens/mysql4.png)

Works well!

Now let's change the privileges of the user **John** we're using:

``` SELECT sys_exec('usermod -a -G admin john'); ```

Where: 
  * **sys_exec**: Returns the exit code of the external program.
  * **usermod**:  Modificate the existing user.
  * **-a flag**:  Assigns user to a new group
  * **-G**:       Specifies a group to assign to.

![alt text](/screens/usermod4.png)

So it should add user **John** to the group with admin privileges.
Let's check if it worked.

![alt text](/screens/root4.png)

Got it!

#
Escaping the restricted shell.

I had some issues that I couldn't fix when trying to access **--os-pwn** shell but I see that it worked for others.
It's worth trying out both ways.














