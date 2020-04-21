# Kioptrix #3 

### Technical Issues

I posted a tutorial how to setup Kioptrix machine in my previous walkthroughs but this one generated a new kind of error to me.

If u get "unable to boot, please use kernel appriopriate for your cpu" error, check out these solutions.
http://www.linuxliveusb.com/en/help/faq/virtualization/154-unable-to-boot-please-use-a-kernel-appropriate-for-your-cpu
Solution #2 worked for me.

Another thing you need to do in order to work this machine:
![alt text](/screens/hosts.png)

So when everything is set and done we can start.

#
### Inspecting the application

In this machine we have an application running. We can access it through the IP at port 80 or by typing http://kioptrix3.com in a browser if we made the recommendd changes in **etc/hosts/**

![alt text](/screens/front.png)

Front page looks like this. We have 3 sections in here - **Home, Blog, Login** and a hyperlink at the bottom to **Gallery**.
Lets manually inspect the application first.

I went for the Gallery and while checking "Ligoat Press Room" i noticed **Sorting Options**.

![alt text](/screens/sorting3.png)

When u use it, the URL starts to use id parameter.

![alt text](/screens/id3.png)

#
### SQL Injection

Adding singlequote sign to id parameter generates an error

![alt text](/screens/mysqlerror3.png)

Application is vulnerable to **SQLi**
Lets use **sqlmap** in here.

```
sqlmap -u kioptrix3.com/gallery.php?id=1
```

In database **gallery** we gonna find table named **dev_accounts** which holds credentials for two developer accounts.
Passwords are hashed but weak enough for sqlmap to crack it.

![alt text](/screens/sqlmap3.png)

These are ssh credentials.
#
### Privilege escalation

**dreg** account has low privileges, lets see how about the other one.

![alt text](/screens/loneferret3.png)

**loneferret** is not the root we want but we have higher privileges and some interesting files in it's home directory.

![alt text](/screens/readme3.png)

I tried to access **/etc/shadow** directly but cannot.

![alt text](/screens/shadow3.png)

Let's stick to what **README** file says and use "sudo ht" for **"editing, creating and viewing files"**

When typing ``` sudo ht ``` in console I recive an error.
``` Error opening terminal: xterm-256color ```

Command ```export TERM=xterm``` fixed it for me.

We can access the **ht viewer** now.

![alt text](/screens/htviewer3.png)

**ht** is a file viewer, editor and analyzer for text, binary, and (especially) executable files.
documentation: http://hte.sourceforge.net/readme.html

Throught ht editor i could freely access /etc/shadow and edit it.

![alt text](/screens/htshadow3.png)

I copied encrypted **loneferret**'s password and inserted it as a new root password.
Lets try to login using new credentials.

![alt text](/screens/root3.png)

Got it! Root is ours!
