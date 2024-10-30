# Preface

Hello everyone, I am the wind up, this time to share with you **Serverless scanning technology ** is also a hidden scanning technology I talked about in the SecTime salon speech, through the **Serverless (cloud function)** to realize a variety of scanner detection function, in order to achieve the bypassing of situational awareness, WAF and other security equipment, to Increase the difficulty of traceability of the blue team researchers and judges, to achieve the effect of sealing no seal, checking no check. I hope that you can gain by reading this article, then the following by me to give you the secret of this interesting attack.

# Serverless directory scanning implementation

First create a cloud function, here to Tencent Cloud as an example.

! [1](. /images/1.png)

Then choose custom create, run environment choose python3.6, function name is arbitrary, geographic area is randomly selected, will be randomly assigned to the geographic area under the IP as our export IP address, the configuration and the cloud function on the line in the same way as the CS.

! [1](. /images/2.png)

The cloud function code is the following, whether it is a cloud function or domain front or reverse proxy and other means, the essence is traffic forwarding, so the core code we wrote on the cloud function is to realize the function of a single scan to return information, and the rest of the control code is executed by our locally written code.

```python.
# -*- coding: utf8 -*-
import requests

def main_handler(event, context).
    headers=event[“headers”]
    url = event[“queryString”][“url”]
    path = event[“queryString”][“path”]
    crake_url=str(url+path)
    try.
        r = requests.get(crake_url,timeout=5,headers=headers,verify=False)
        status = r.status_code
    except Exception: status = None
        status = None
        pass

    return status,crake_url
```

After configuring the cloud function code, proceed to the **Trigger Management** option.

! [1](. /images/3.png)

Triggers are configured as shown, note the unchecked Collective Response.

! [1](. /images/4.png)

Edit the API configuration by setting the path to **/** and clicking **Do it now**!

! [1](. /images/5.png)

Then we get the public domain names of the two API gateways, and we have completed the basic configuration. You can use these two public domain names to realize the trigger implementation of the cloud function function we wrote above.

! [1](. /images/6.png)

Here to further write a simple Demo, in the local implementation of a directory scanning function, through the acquisition of the public address to pass the parameter, the realization of the cloud function service to the specified site directory scanning, here I pass the url address to be scanned as well as the dictionary, the code is as follows:

! [1](. /images/7.png)

Execute the code locally for scanning, we will get the response to us and output the scan result.

! [1](. /images/8.png)

The microblogging intelligence reveals that the IDC servers in Tencent Cloud - Guangzhou are indeed

! [1](. /images/9.png)

However, please note that **Apache access.log log ** header header User-Agent for python-requests/2.26.0, there is a very obvious characteristics, then we continue to modify the local control code.

! [1](. /images/10.png)

We create a get_ua-header.py file, which creates an array of UA's holding a large number of different User-Agents.

! [1](. /images/11.png)

Here we import the UA array we just created, and then set the header User-Agent in the second arrow to randomize the values in the UA array each time, and then specify the header in the send request. look more like a legitimate request.

! [1](. /images/12.png)

Here the cloud function gets the headers of our local request API gateway domain and then replaces its header with the header of our local request packet when forwarding it to achieve the effect of customizing the header, the modified scanning scenario is shown in the figure below:

! [1](. /images/13.png)

found in the Apache logs, User-Agent has been random information for our local request, then by continuing to customize the local control code in the header information to make him look more reasonable, you can achieve a more invisible scanning.

Continue to repeat the above operation, create an identical cloud function, API gateway is also the same configuration, but need to pay attention to is the choice of geographic area to be a different place, because ** a single user in a geographical area can only have five random export IP **, that is, if these five IP are blocked then our scanning can not be continued, but the attack and defense itself is reciprocal, so we can not continue, but the attack and defense itself is reciprocal. But the attack and defense itself is equal, so what is the way to bypass it?

We added two identical cloud function configuration, only for the selection of different geographic areas, but also to bypass this restriction above, modify our code to add another API gateway domain name address to the dictionary for the division of labor scanning, the two cloud function to intercept a part of the dictionary for scanning, here I chose the geographic area is Guangzhou and Shanghai, and then start directory scanning.

! [1](. /images/14.png)

Now continue to check the access.log log, found some more Shanghai IP addresses, is not more confused? In the target business system every day in the face of a large number of requests to use, the previous idea of troubleshooting and traceability is to filter the IP address of frequent visits or suspicious operations, but for this case, usually mixed with some normal requests, the defense side of the log audit will often think that these are normal HTTP requests.

! [1](. /images/15.png)

The idea of implementing batch scanning here is explained in depth in **Extended Extensions**.

# Serverless Port Scanning Implementation

Then we continue to implement the port scanner implementation, cloud function configuration and API gateway configuration here will not do more details, with the same above, here we only focus on the cloud function code and local control code implementation.

Cloud function code is as follows:


```python
# -*- coding: utf8 -*-
from socket import *

def main_handler(event, context).
	IP=event[“queryString”][“ip”]
	port=event[“queryString”][“port”]
	try.
		conn=socket(AF_INET,SOCK_STREAM)
		res=conn.connect_ex((str(IP),int(port)))
		conn.send('Hello,World!'.encode(“utf8”))
		results=conn.recv(25)
		if res==0.
			conn.close()
			return port
	except Exception as err.
		print(err)
	finally.
		print(err) finally: print(“”)
		conn.close()
	
	return None
``

Again here the cloud function is only needed to implement a simple single port scan, and our control code is as follows:

! [1](. /images/16.png)

As you can see, here the same application of the above mentioned batch scanning, if the above mentioned is through the path in the dictionary to distinguish between the use of two different cloud function API gateway domain name, then here the port scanning is through the port to distinguish.

Because port scanning will be slower, so here I use the way of concurrent scanning, scanning speed will be faster, the use of grequests module.

Here to execute a local scanning script, successfully get the target host port open, but this is not the focus.

! [1](. /images/17.png)

Here's how to demonstrate port scanning in situational awareness using Hfish

! [1](. /images/18.png)

As you can see, the IPs scanned on these ports are all random IPs from Tencent Cloud, so is there no way to get started?

! [1](. /images/19.png)

Through Kunyu on these IP batch query found that the location of Shanghai and Chengdu, China, also confirmed that the attack IP for our cloud function of the export IP address, and these data Time are relatively early, and in the threat intelligence to see these IP for the IDC server, but also aided in explaining that these IP for the recovery of the current cloud function for the application of the request. Applied to the cloud function to request the IP address of the cloud server, the time factor here is also the key to our subsequent judgment to trace back this attack.

! [1](. /images/20.png)

Welcome to learn about Kunyu, is a very good use of spatial mapping data analysis tool, which is full of highlights Oh ~ very suitable for you to work in the red and blue confrontation information collection or traceability and other work, unlike other tools is that Kunyu's design is more closely related to the work of data analysis rather than just IP export.

**Download: https://github.com/knownsec/Kunyu**

# Extension

As mentioned above **a single user can only have 5 random export IPs in a certain geographic region** So in the case of larger traffic scanning, a single time if only 5 scanned IP addresses, it is still very easy to be detected and thus blocked IPs. so in order to solve this problem, we can be scanned in batches, and if it is a port scanning, we can We can create multiple cloud functions and scan different ports for different geographic regions, in terms of situational awareness is the n * 5 IP addresses in the scanning, which also solves the dilemma of a single geographic region can only have 5 IP.

The same further thinking, we can also create multiple geographic cloud function under the same cloud function, and then create other vendors cloud function, to achieve the diversity of IP addresses, and then create different geographic areas on this basis, theoretically can be superimposed, to achieve the single large traffic scanning.

For large group targets, usually every day posture awareness will have a large scanning traffic, then in this case, if we give our cloud function scanning and then add a certain degree of random delay scanning, then what we see in the posture list is a bunch of discontinuous single request, if it is a directory scanning will be directly considered as a normal request, because of the randomness of the IP and the Legitimacy of the request packet, discontinuity simply can not associate this is a scanning event, not to mention the issue of traceability, there is no way to restore the attack chain, such an attack will only be more difficult than the C2 stealth bounce.

The whole realization of the mind map is as follows:

! [1](. /images/21.png)

Partners can also be based on the basis of what I have said, further refine the request packet as rationalized as possible, so that in the face of firewalls and other equipment, in the face of a large number of IP scanning can not be through the IP frequent request and thus blocking IP, resulting in blocking can not be blocked, not to mention the issue of traceability. The local control code can help us better control the entire scanning process, so that the entire scanning is more personalized, the cloud function code is only a single traffic forwarding scanning, while the local code is responsible for what kind of scanning through the problem.

But for the blue team members do not need to panic too much, after all, there is an attack there is a defense, for this kind of scanning, if you find out the suspicious traffic directly blocked IP can be, this situation will lead to inaccurate results of the red team scanning, a certain IP is blocked, then his subsequent requests through the IP will be scanned failed.

Back to the red team's point of view, in the face of the above response, we can through the mind map about the division of labor scanning, multiple cloud function alternately repeat the same scanning task to prevent the ban, so that a certain IP is blocked without fear of scanning the failure, of course, generally speaking, the banning of the IP is the end of the scanning thing, usually the defense is not so fast response speed! I've always thought that the two sides of the offense and defense are the same.

I have always believed that the attack and defense against the two sides is the position of reciprocity, each has its own advantages, and how to use their own advantages to achieve the goal is the need to think about the problem, in the test of corporate security at the same time also honing the technical level of technicians, which is the significance of the red and blue confrontation. So when we think about attack or defense issues must stand in the opposing point of view to think about the other side of the response means to find a breakthrough point.

The above mentioned codes have been uploaded to Github, interested masters can download by themselves or write their own.

**Download address: https://github.com/wikiZ/ServerlessScan**

# Postscript

Looking back to the month of November 2021, I have been conducting an average of one speaking event per week, either as a team share, internal share, or salon share, and I am thankful to all of you for your support. I think in this kind of sharing scenario, it is very improve the communication and communication skills. In the docking with the operation lady, the preview and other work, as well as the preparation of PPT, conceptualization of how to better speak out what I have learned, to be accepted by others, these are things that I have not thought of doing in the past, I think it is precisely in each of these experiences that people continue to grow up, and I am also well aware of my own lack of ability, so I have never dared to slacken off.

To date, Kunyu has a lot of Title, complementary white hat conference of the year the best weapon award, 404 star chain program, Kcon weapon spectrum, etc., but also very grateful for your support, I did not think that when I was mindlessly do a thing can get so many people's support, I will insist on updating to listen to everyone's needs, to better improve Kunyu, I see that the domestic masters and even foreign countries. Seeing the attention of domestic masters and even foreign practitioners is really a great fulfillment.

As for the ID “Wind Rising”, it was originally inspired by **Wind Rising from the End of the Green Weed**. Because I am not from the science class, so the meaning of the ID is also to hope that I can gradually grow in the continuous accumulation, the real wind rises, of course, from this goal there is still a very long way to go hahaha. 

Looking back to 2021, I output a total of 8 articles, this number is actually not much, but I always insist on outputting the knowledge points that I think are useful, even if I output one article in a few months, I insist on not going to create some repetitive work, and each article will bring in some of my own ideas to you. I think that if writing an article becomes to write an article to write then it loses its significance, so I have always insisted on writing in a safe article platform, see some public paid articles can only be put into a smile, I agree with the payment of knowledge, but personally, I can let more people to see my article to gain understanding of the wind, far more important than the benefit itself, so I will continue to adhere to the concept, adhere to the output, continue to progress! This concept, adhere to the output, continue to progress.

Perhaps because this is the last article of the year, so in the postscript to talk about some more, and I hope you understand, here to thank you in this year to my support, I wish you all wish, dream come true.

Want to exchange network-related cases, red team attack and defense technology master can contact me, usually I also very much like to make friends.

! [1](. /images/x.jpg)