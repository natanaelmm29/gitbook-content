---
description: 'Case 7: Using Volatility to analyze a memory dump image. Atenea.'
---

# Exercise 7 - RAM analysis

### Can you identify the attacker’s domain? <a href="#_wazdvectcu5l" id="_wazdvectcu5l"></a>

The first step is to compute the hash to ensure that the file has not been modified.

![](<../.gitbook/assets/0 (8).png>)

Then, to determine the profile we execute the command _imageinfo_ to get general information about the memory image.

`python2 vol.py -f ../atenea.img imageinfo`

![](<../.gitbook/assets/1 (10).png>)

To get the connections we can use the command _sockets_. In this case, there is only an IP address which is 169.254.240.50, but we are not sure that it is the IP of the attacker’s domain.

`python2 vol.py -f ../atenea.img --profile=WinXPSP2x86 sockets`

![](<../.gitbook/assets/2 (9).png>)

Using the command _pslist_ we can list the processes of the system. In the image below we can see the list and as it can be seen there are some cmd.exe, which are suspicious. There is a process called Memoryze.exe that after searching on the Internet it is a Mandiant software used for forensic analysis.

`python2 vol.py -f ../atenea.img --profile=WinXPSP2x86 pslist`

![](<../.gitbook/assets/3 (11).png>)

As there are some cmd.exe processes, we execute the commands _cmdscan_ and _consoles_ to get the information about the executed commands and their outputs. As it can be seen in the image below, it is a ping request to a domain named [**www.go0gle.com**](http://www.go0gle.com/).

`python2 vol.py -f ../atenea.img --profile=WinXPSP2x86 cmdscan`

![](<../.gitbook/assets/4 (8).png>)

From the output of the _consoles_ command, we can see that the ping request was not successful, so we could think that it is the attacker’s domain as it is a weird domain but as it is not resolved, we cannot be sure.

`python2 vol.py -f ../atenea.img --profile=WinXPSP2x86 consoles`

![](<../.gitbook/assets/5 (9).png>)

_Looking at the processes with the command pstree, we can see that there is an explorer.exe process whose children processes seem to be suspicious, as they are cmd.exe and regedit.exe._

_The regedit.exe is a Windows executable file that allows to view and edit the keys and entries in the Windows registry database, which is a known technique for achieving persistence._

_With the command memdump we can use strings to identify some interesting printable strings from the regedit.exe._

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

_One of the most used persistence techniques is the use of the key ‘\Software\Microsoft\Windows\CurrentVersion\’. These registry keys are used to set startup folder for persistence. In order to print the value of the key, the command printkey can be used:_

_`python2 vol.py -f ../atenea.img --profile=WinXPSP2x86 printkey -K “Microsoft\Windows\CurrentVersion\Run”`_

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

_As it can be seen in the image above, the value of the registry clearly shows the domain of the attacker, which in this case is wiki-read.com._
