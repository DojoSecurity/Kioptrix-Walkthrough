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
