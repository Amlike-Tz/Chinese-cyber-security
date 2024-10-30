## hackrf receives ADS-B airplane signals

## Preface

This article explains how to use hackrf to receive ADS-B aircraft signals. By receiving ADS-B signals, you can get the flight number, altitude, latitude, longitude, trajectory and other information of an airplane within the vicinity. In this article, we will use hackrf to receive ADS-B signals, and explain its principle and experimental reproduction. I hope you can gain something from it!

Broadcast Automatic Dependent Surveillance (English: Automatic dependent surveillance - broadcast, abbreviated ADS-B) is an aircraft surveillance technology in which an aircraft determines its position by means of a satellite navigation system and broadcasts it periodically so that it can be tracked. Air traffic control ground stations can receive this information as an alternative to secondary radar, eliminating the need to send interrogation signals from the ground. Other aircraft could also receive this information to provide attitude awareness and autonomous avoidance.

ADS-B is an “automatic” system that does not require pilot or other external inputs, but relies on data from the aircraft's navigation system.

## Experimental Procedure

ADS-B signals are received in the 1090 MHZ frequency. Due to the large number of old wireless standards in the aviation CNS system, aircraft aviation has a set of standard protocols, and it is very difficult to modify them to make them widely available, which results in the current wireless standards still being relatively old. And between different airplanes by receiving ADS-B signals, you can get the information of other airplanes, so as to sense or avoid, ADS-B is an automatic broadcasting system, so we can also receive its signals on the ground.

**Installation environment**

$ apt update

$ apt install build-essential debhelper rtl-sdr libusb-1.0-0-dev librtlsdr-dev pkg-config dh-systemd libncurses5-dev libbladerf-dev git lighttpd -y

**Download dump1090**

$ git clone https://github.com/itemir/dump1090_sdrplus.git
$ make

Note that after installing dump1090, go into the folder and execute make to install it, then execute the following command inside the directory:

```
. /dump1090 --aggressive --net --interactive
``` .

After execution when an airplane passes us the terminal will output information, in the process of receiving the antenna as close as possible to the window or outdoors, and hackrf choose the best antenna that can receive 1090MHZ, to ensure that the experiment is carried out properly.

! [img](https://p2.ssl.qhimg.com/t01ac6a7d4be24f913f.png)

**Flight: flight number
Altitude: flight altitude
Speed
lat: longitude
lon: dimension
Track
sec: communication time

After opening the command dump1090 as above, it will automatically open a http service, enter the url address 127.0.0.1:8080 in the browser, it will automatically open a map, which shows the aircraft visualization of the heading information.

! [img](https://p3.ssl.qhimg.com/t0102a2f49f4dacd7de.png)

You can see that the map is constantly refreshing the coordinates of the aircraft, and the flight information is displayed on the left, doesn't it look more user-friendly?

But here you need to note that the default map of dump1090 is using google map, you need to hang the proxy to be able to access, here we can replace the gmap.html file under the folder of dump1090, replace it with the domestic map to use. gmap.html source code I've uploaded to the github, you can download it by yourself if you need it. I have uploaded the source code of gmap.html to github.

I have uploaded the source code of gmap.html to github.
https://github.com/wikiZ/dump1090
```

 

## modes_rx experiment and environment setup

When it comes to the construction of modes_rx, I still wiped a tear of sadness, and I really stepped on a lot of pits in the process of environment construction. Compared to dump1090, this tool can save the captured information as a kml file and import it into google earth or gpsprune.

**Environment setup

$ git clone https://github.com/bistromath/gr-air-modes.git

$ cd gr-air-modes

$ mkdir build

$ cd build

$ cmake ... /

$ make

$ sudo make install

$ sudo ldconfig

When running cmake ... / /, it is important to note that if your gnuradio is version 3.7.x then you may get the error shown below.

! [img](https://p3.ssl.qhimg.com/t01ec761de58296e2c3.jpg)

It will tell you that you need to install gnuradio 3.8 and you will not be able to run cmake properly. So what should we do? Don't worry, we can use the branch version of mades_rx, type git tag in the folder you can see there is a branch gr37, then we continue to type git checkout gr37 to switch to it and then install it normally!

! [img](https://p0.ssl.qhimg.com/t0133c17fc62e0e0740.png)

Once the installation is complete, enter the following command in the terminal.

``
modes_rx -g 60 -k air.kml
```

When the airplane passes by, save the information to air.kml. Then you can import it into google earth.

! [img](https://p4.ssl.qhimg.com/t01e44448ae447cfb97.png)

 

## tag1090

Compared with dump1090, I feel that tag1090 is a more user-friendly software, he clearly marked the flight track, the effect is as below:

! [img](https://p0.ssl.qhimg.com/t01ae397463a5d5c280.png)

If you are interested in this program, you can download it from github, so I won't repeat the configuration process here.

``
https://github.com/wiedehopf/tar1090
```

## Postscript

This article is here, in fact, to receive ADS-B there is no specific regulatory constraints, but we still need to pay attention to the information received, it is best not to send the information to foreign countries, then it is very likely to go to tea. After writing this article, I should not write again in recent weeks, because I need to prepare for the arrival of the antenna, ready to study the reception of meteorological maps, of course, will be written and published as an article. Stay tuned for that.

Finally, I wish you all success in your studies and work!