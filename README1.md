# Kioptrix 1 - Walkthrought

## Setting up 

[Tutorial that I used](https://medium.com/@obikag/how-to-get-kioptrix-working-on-virtualbox-an-oscp-story-c824baf83da1)

First We check what's our ip with simple shell command:
```
ifconfig
```
![alt text](/screens/ifconfig.png)

Then we need to check the hosts running on our IP address with:
```
netdiscover
```
![alt text](/screens/netdiscover.png)
PCS Systemtechnik GmbH is Virtual Box Vendor so it must be our VM.


Let's scan it with nmap!

		* -sS stands for TCP SYN scan 
    
		* -A  DetectS OS and Services
    
```
nmap -sS -A <ip address>
```
![alt text](/screens/nmap.png)

We gonna set port 139 as our target.
We see that the process running is netbios-ssn - Samba but no exact version is described.

After quick research I found a Metasploit scanner for Samba Version.
To access Metasploit in Kali Linux type:
```
msfconsole #to access Metasploit
use auxiliary/scanner/smb/smb_version #chosing the module
exploit #start exploit
```
![alt text](/screens/msfscan.png)


So it tells us that the Samba version is 2.2.1a
Lets look for exploit!
Metasploit had some  exploits but somehow they failed for me.
Exploit-db came for the resque! [exploit-db](https://www.exploit-db.com/exploits/10)


We can download the exploit script from the exploit-db.
It comes as a .c file so we have to compile it first using gcc(GNU Compiler Collections).
```
gcc <name of the C file> -o <name of the file compiled>
```
I called mine: smbexploit
To use it type: ./<name of file>
![alt text](/screens/smbexploit.png)

Console will display the options available.
```
./<name of the file> <options> <target ip address>
```
![alt text](/screens/root.png)

Got it! Root obtained!




