## WI-FI Security Penetration Profiling

## Preface

WIFI can be found everywhere in our lives, and the routers that are visible in almost every home have become an inseparable part of our lives. But as the saying goes, where there is a network, there will be security problems, then the author of this article will be WIFI security in-depth and concise explanation, aims to analyze the common WIFI attacks from the perspective of the infiltrator, to analyze the attack process. So let's start now!

## Preparation

First of all, before testing, we need to configure the environment. First of all, we need a wireless card that can be turned on to listen to the mode, such a card in Taobao is about a few dozen dollars.

! [img](https://p1.ssl.qhimg.com/t011e4a7fcd869964b1.png)

You can choose from a wide variety of options, so I won't go into details here. But need to pay attention to is to pick the wireless network card band problem, I bought this wireless network card can only listen to the 2.4G network, but now the home router basically have two bands, so when we carry out such as deauth and other attacks, when the 2.4G band signal connection is abnormal will automatically jump to the 5G band resulting in the inability to capture the data packet.

Here are the main differences between 2.4GHz and 5GHz.
The two have their own advantages in general. The main advantages of 2.4GHz: strong signal, long coverage, low attenuation. Disadvantages: narrower bandwidth, slower speed, greater interference. 2.4GHz band has 14 channels, in China can use 1-13 channels. 5GHz advantages are: wider bandwidth, faster speed, less interference. Disadvantages:Smaller coverage area, high attenuation, small signal. Overall it is the opposite of the advantages of 2.4Ghz. 5G band has a total of about 200 channels, his interference can be said to be very small.

 

## Wireless card configuration

Before infiltration we need to configure our network card, after connecting the card to the virtual machine, enter the command iwconfig to see if the card is recognized.

! [img](https://p5.ssl.qhimg.com/t012b1d323777e2eec1.png)

Here wlan0 is our wireless NIC, turn on listen mode with the command.

``
airmon start wlan0
```

The following image shows that listening mode is enabled

! [img](https://p4.ssl.qhimg.com/t01734705bad149ec55.png)

Note:Why do we need to enable listen mode? Because by default, the card will only receive the data frames sent to itself, and discard the other data frames, which is the managed state. By default, the card only receives data frames sent to itself and discards all other data frames, which is the managed state.

With listening mode enabled, we use airodump-ng to scan for wireless networks.

! [img](https://p0.ssl.qhimg.com/t0170542e1598e6111c.png)

To protect the privacy of others, only the information of the router to be tested is shown.

BSSID: mac address of the wifi.
PWR: signal level reported by the NIC, the lower the value the better the signal, ignore the negative sign in front.
Beacons: the number of announcements sent out wirelessly
CH: Channel
MB: Maximum rate supported by the wireless
ENC: encryption algorithm system used, OPN means no encryption
CIPHER: detected encryption algorithm
AUTH: Authentication protocol used.
ESSID: wifi name
STATION: MAC address of the client, including those connected and those who want to search for wireless to connect. If the client is not connected, it will show “notassociated” under BSSID.
Rate: Indicates the transmission rate.
Lost: packets lost in the last 10 seconds.
Frames: number of packets sent by the client.
Probe: the name of the connected wifi.

 

## Deauth attack

A deauthentication flood attack, often referred to simply as a Deauth attack, is a form of denial of service attack on a wireless network, where a disconnect frame is sent to the router when the client clicks to disconnect, and the Deauth attack simulates such a link to disconnect the packet.

The following command attacks WI-FI signals on all channels.
`mdk3 wlan0mon d`.

There is no echo during the attack, but the attack is in progress.

! [img](https://p0.ssl.qhimg.com/t0190f1c1cd98a10f4c.png)

The effect is as follows

! [img](https://p2.ssl.qhimg.com/t011b49f2f655d106dd.jpg)

We found that the WIFI connection was disconnected and trying to connect again will show us that the password is wrong.

The aireplay-ng tool can also perform this operation.
-0 to specify the attack method as a de-authentication attack, 0 to send in a loop, specify another number for the number of offline packets to send, and -a to specify the target router mac address.

``
aireplay-ng -0 0 -a mac wlan0mon
```

After starting the attack it will keep sending offline packets to the target.

! [img](https://p5.ssl.qhimg.com/t011902a895abc01ebf.png)

Whether it's mdk3 or aireplay-ng they actually have about the same effect.
Currently the maintenance of mdk3 has been stopped, while some teams in China have developed mdk4 which supports 5GHz band you can pay attention to it.

 

## Auth attack

Authentication flood attack, the attacker will send a large number of fake authentication request frames (fake authentication service and status code) to the AP, when a large number of fake authentication requests are received more than the ability to withstand, the AP will disconnect other wireless services.

``
mdk3 wlan0mon a -a 50:2B:73:6A:18:81
```

After using the above command a lot of authentication information will be sent to the target AP, and soon it will disconnect other wireless connections, and the router will get stuck and crash if you continue to attack it.

Most mainstream routers are currently WPA2, so the above attack covers almost all current home routers, while the newly introduced WPA3 security standard mitigates e.g. Deauth attacks. However, the price of routers with WPA3 on the market is still a bit expensive in comparison.

 

## MAC Whitelist Bypass

In school, we must have experienced the campus network, for the teachers can use the campus network and ordinary users even if they know the password can not log in precisely because the network administrator filtered the MAC address of our devices. So there is no way to bypass it? In fact, it is just one more step. When we use airodump-ng to scan, we can see that the following clients are already connected.

! [image](https://user-images.githubusercontent.com/37897216/233993827-52b5e2bf-c898-4455-8e25-12fa8a27c009.png)

In the STATION item we can see the already connected clients. Here also from the MAC address randomization, we all know that the MAC address is the unique code and identifier of the terminal device network interface, in the production allocation can not be modified, so often device tracking. But with the emergence of the randomization of this mechanism to break this status quo. In windows 10, is the full randomization, that is to say, in the search for nearby wireless networks is a random MAC address, after the connection is equally random. But Android is not, he is semi-randomized, that is, when searching for wireless networks is randomized, but not after connecting, which allows us to have a chance to exploit. In the picture above, the STATION item is the client that is connected.

We can bypass the MAC whitelist by changing the MAC address of our own NIC to theirs.
The command is as follows.

``
macchanger -m mac wkan0
```

It is important to note that two devices online at the same time can cause a MAC conflict if modified in this way.

 

## Penetration Ideas in a Nutshell

Above we briefly explained the two common wireless attacks, so the guys must be asking if there are any portable devices?

! [img](https://p5.ssl.qhimg.com/t011d3052758e72e414.jpg)

I believe that partners with IoT experience must be able to see that this is an esp8266 board, we can burn wifi killer firmware through the wireless attack anytime, anywhere, the interface is as follows.

! [img](https://p4.ssl.qhimg.com/t01464b0dad5849898e.jpg)

Isn't it cool?

Of course, there are more than a few ways to attack, including wireless phishing, password blasting, drone attacks and so on.
Speaking of drones, this article would like to give you a demonstration of the drone attack process analysis, but why my drone is broken so there is no way. But the general principle of the attack is also through kismet and other software to detect drone signals, disconnection, you can also use radio frequency tools for GPS deception.

Here we also need to be vigilant outside the strange network, in the phishing network, the attacker can even change the access address, such as our normal access to Baidu's domain name, the domain name looks like there is no problem, but when you access through this router, the domain name has been pointing to is not the original web site, so malicious DNS servers in the field is almost a hundred hits.

For the auth flooding attack mentioned above, some routers currently come with a firewall in the firmware that can play a good preventive, as shown below.

[img]() [img](https://p5.ssl.qhimg.com/t019e201bd36f4c9ec2.jpg)

Such firewalls can provide good protection against ddos but not against auth attacks, so when WPA3-compliant routers are popularized, this problem will probably be a thing of the past!

 

## Phishing attacks WIFI Pineapple

Regarding phishing attacks, you can read my article about WIFI Pineapple.

``
https://blog.csdn.net/qq_42804789/article/details/105215524
ðŸ “ðŸ ‘ðŸ ’ðŸ ‘ðŸ ’ðŸ ”ñ

Phishing attacks are not particularly effective in terms of defense at this point in time, and it's mostly up to you to be aware of what you're doing. Try not to enter your account passwords on unfamiliar networks. If you have read the above, it is not difficult to realize that the attacker has access to all the information you access.

 

## Postscript

This is all for this article, because of the related regulations, it is not possible to carry out more demonstrations of the attack process, so I just picked a few more classic attack methods to explain, I hope you can get something out of it. If you are interested in this area, you can find me to discuss, welcome to ask questions or suggestions!

I wish you all the best of luck and a happy life!