## RTL-SDR reception of NOAA weather satellites

## Preface

I had the idea of writing about NOAA Weather Satellite a long time ago, but midway through the process, I had a lot of things to do (in fact, I was lazy), so this article is only released now. In the operation of the time, also read some articles on the Internet, summarized the experience, found that in our experimental process, some of the problems that occur do not have too much analysis and explanation, so this article will be in-depth on the operation process as well as the process of the problems that may occur to explain, I hope to be able to help you.

## NOAA Satellite

NOAA satellite is the National Oceanic and Atmospheric Administration's third generation of operational meteorological observation satellites, the first generation is called “TIROS” (TIROS) series (1960-1965), the second generation is called “ITOS” (ITOS) / NOAA series (1970-1976), and the third generation that operated thereafter was called the TIROS-N/NOAA series. The orbits are nearly circular sun-synchronous orbits, with orbital altitudes of 870 km and 833 km, orbital inclinations of 98.9° and 98.7°, and a period of 101.4 min. The purpose of NOAA's applications is daily meteorological operations, and there are two satellites in operation in normal times. Since one satellite can observe the same area on the ground at least twice a day, two satellites can make more than four observations.

## Experimental Environment

Software: WXtoimg
WXtoimg
Orbitron
SDRSharp
Tracking DDE plugin

Hardware: RTL2382U
RTL2382U
Yagi Antenna

## Plugin Installation

Before the experiment, we need to install the DDE plug-in for SDRSharp, because we can't use the professional equipment such as star finder during the usual experiment, and because of the Doppler effect caused by the satellite altitude, elevation angle and other factors, we need to adjust the frequency of the satellite constantly to achieve the best reception effect.

So what is Doppler shift?
The difference between the transmitted and received frequencies due to the Doppler effect is called Doppler shift. It reveals the Doppler shift caused by the Doppler effect of sound waves The Doppler shift caused by the Doppler effect of sound waves reveals the law that the properties of a wave change in motion.

Doppler shift and changes in signal amplitude, etc. When a train is approaching, the wavelength of the sound of the whistle is compressed and the frequency becomes higher, so the sound sounds thin. When the train is moving away, the wavelength of the sound is stretched and the frequency becomes lower, thus making the sound sound sound majestic.

**DDE plugin installation**
Copy NDde.dll, SDRSharp.DDETracker.dll, SDRSharp.PluginsCom.dll, SDRSharpDriverDDE.exe to the SDRSharp installation directory.
Then add the code to SDRSharp as shown below, which serves to add plugin options to the action page.

! [img](https://p0.ssl.qhimg.com/t0115a370422759b2da.png)
`<add key=“DDE Tracking Client” value=“SDRSharp.DDETracker.DdeTrackingPlugin,SDRSharp.DDETracker” />`

Then open Orbitron

! [img](https://p4.ssl.qhimg.com/t01c3e1f0f078be7e76.png)

Click on the Spinner/Radio option, then select the driver as MyDDE, the first time it will prompt that the plugin program can not be found, at this point we just need to follow the prompt to the directory we just copied to select SDRSharpDriverDDE.exe. Click the button on the right side of the driver selection window to use it.

! [img](https://p3.ssl.qhimg.com/t0115dd1ebea8a1d585.png)

Open SDRSharp again and find that the left option box has appeared the plugin we just added, this time it has not been installed successfully, we need to click on the arrow pointing to configure.

! [img](https://p4.ssl.qhimg.com/t01acba2ccb9cd4f75f.jpg)

Click the ADD button, enter the satellite name in the Satallite name box, and in the right box follow the picture above, what? Can't see it? Zoom in on your browser.

At this point, go back to Orbitron and update the ephemeris, I won't go into details on how to update it, let's continue with the main topic. In order to facilitate the download, DDE plug-ins on the Internet is more difficult to find, I have uploaded to the github interested partners can download.

DDE plug-in address:

```
https://github.com/wikiZ/DDE
```

 

## Experimental steps

Now back to business, after configuring the plug-in, we use the feeder to connect the Yagi antenna and the SDR device, and if you have the conditions, you can add an LNA for better results. After setting up, connect to our host, use Orbitron and other star-finding software to predict the satellite transit time in advance, turn on the SDRSharp, adjust the frequency to the transit range, it is worth noting that the NOAA series of satellites is not more than one, the frequency of different satellites is not the same.

The first time you open the WXtoimg software, you will be prompted to enter the latitude and longitude of the location of the city information, here we fill in accordance with their own city can be. The following figure:

! [image](https://user-images.githubusercontent.com/37897216/233991831-6cbe085c-e291-4932-8c27-b61b0244cbfa.png)

If you are not prompted to enter the location of the city, then in the following lat, lon custom latitude and longitude can be. After setting, if you want to adjust the location information again later, you can also set it in Options->Ground Station Location.

! [image](https://user-images.githubusercontent.com/37897216/233991874-d2d69581-cd13-4ead-9ff8-efcc10c7d37b.png)

Then Options->Recording Options to select the sound input, you can see that the option soundcard we chose is CABLE virtual sound card, about the virtual sound card installation and configuration of this is not much to say, you can Baidu. After downloading the virtual soundcard, select CABLE here. You can do it. Of course, you can also have an external sound card, but the virtual sound card will not be mixed with some ambient noise when recording, to ensure the quality of the audio when receiving.

Update Kepler in File->Update Keplers, it's better to update it regularly to make sure the satellite information is accurate, and check the transit time of satellites in File->Satellite Pass List. Of course the powerful Orbitron can also do star chasing and view the graphical positions of the satellites in real time.

! [img](https://p2.ssl.qhimg.com/t014428f1b7d390cffd.png)

When you get to the last step, click File->Record.

! [img](https://p0.ssl.qhimg.com/t01ce3e795323aca80a.png)

Remember to check the Create image(s) option to generate the image, and then click Auto Record to wait for the satellite transit, of course, the most intelligent point of this software is that he can dynamically get the satellite transit information automatically decoded.

The smartest thing about this program is that it can dynamically get satellite transit information and decode it automatically. [img](https://p1.ssl.qhimg.com/t0137984ed2e811889e.png)

You can see that at the bottom of the software there is information about the satellites you want to receive, the recording time and frequency, so you need to open SDRSharp to receive them in advance.

! [img](https://p3.ssl.qhimg.com/t01b4dcf1a1017cc457.png)

Note that the Scheduler must be checked or it will not work with Orbitron.

Then adjust the frequency to 137.9125MHZ and adjust the volume to maximum. Wait quietly.

After the arrival of the satellite, we can clearly observe the energy fluctuations of the waterfall chart, as well as frequency jumps, you can also see the left side of the DDE plug-in we set up a red banner, which means that as well as the successful linkage with Orbitron, you can also observe that the frequency of the above is constantly changing. The frequency at this point is the best frequency to receive NOAA satellites.

! [img](https://p2.ssl.qhimg.com/t019e2c63aade75a84a.png)

On the other side we see that WXtoimg is also receiving and decoding.

! [img](https://p0.ssl.qhimg.com/t01246d16a73ac84cbe.png)

You can see that the vol measurement in the bottom right corner of the image above is green, with a value of 57.1, which means that the quality of the audio is good, as a value of 40 or more is usually good. The blue progress bar is the decoding progress, and the information below can also see the satellite elevation angle and the remaining reception time. At the end of the recording, the software will automatically decode and save the effect image. In saved images you can see, if you want to decode to HD, you should click the option Enhancements MSA multispectral.

The final result is as follows:

! [img](https://p0.ssl.qhimg.com/t01a9f72c167fe2645f.png)

! [img](https://p5.ssl.qhimg.com/t01cca362ab39ec3531.jpg)

! [img](https://p0.ssl.qhimg.com/t01ca8010191d37fcc7.jpg)

! [img](https://p5.ssl.qhimg.com/t01120f20e7405a03f5.jpg)

 

## Problem Summary

Because we were in an urban area, the quality of the satellite signal was not very good due to the strong interference in the neighborhood, so we did not record HD cloud maps. So the following summarizes the problems that may occur in the experiment.

1, first of all, there must be a lot of partners using hackrf equipment, here hackrf itself is a very good device, but here his sensitivity is really a hard injury, which can also be felt when using the inexpensive RTL series of TV sticks to receive the difference in sensitivity, so it is not recommended that you use when receiving the cloud map.

2, and then a small ring antenna on Taobao, that is also a very good antenna, but does not apply to the experiment we are going to do, he receives short-wave effect is very good, but the reception of NOAA satellites when the effect is not good.

3, here is the most recommended for everyone to use is a four-armed spiral antenna, I think this is to receive satellite signals when the first choice, in the salted fish have, of course, you can also make their own, but it is more troublesome, do not recommend that newcomers to do. The second is the yagi antenna, in reception is also much better than some SDR equipment attached to the antenna.

4, pay attention to the feed line is too long will also lead to the quality of the received signal.
5, must be patient operation, because in the process of experimentation, there are really a lot of easy to overlook.
6, too lazy to configure their own plug-in partners can download the built environment, but it is still recommended to configure their own try again.

``
https://github.com/wikiZ/SDRSharpQPSK
```

## METEOR M2 satellite

In fact, NOAA is not the only weather satellite that can receive, such as METEOR M2, Japan's GMS series are very good choices, compared to NOAA, METEOR M2 satellite transit frequency is very poor, probably only every day at 7:00 or 8:00 a.m. when the satellite altitude as well as all aspects of the reception is suitable. Here will not explain separately, interested partners can read the following article to understand.

``
bilibili.com/read/cv2527746/
Â Â Â Â Â Â Â Â Â Â Â Â Â

## Postscript

Then we have this article here, here want to say is WXToimg has not been updated for several years, so there are a lot of bugs such as receiving color map, etc., here recommended that partners can go to look at the NOAA-APT and other decoding software, of course, including the SDRSharp these software are not fixed, you can still choose according to the actual situation. Like receiving software in linux gqrx is also a good choice, as for the previously mentioned connection to the LNA, I think it is not necessary, so it should still fit the specific environment configuration. Any questions, please feel free to leave a message.

**Finally, I wish you all the best of luck and your dreams come true! **