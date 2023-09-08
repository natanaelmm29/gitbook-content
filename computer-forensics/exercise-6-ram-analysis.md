---
description: >-
  Case 6: Using Volatility to analyze a memory dump image. Specific information
  about the memory image.
---

# Exercise 6 - RAM analysis

The first step is checking the integrity of the file to analyze.

![](<../.gitbook/assets/0 (2).png>)

### What profile is the most appropriate for this machine? (ex: Win10x86\_14393) <a href="#_441x083qf7qc" id="_441x083qf7qc"></a>

The command _imageinfo_ only provides the suggested profiles whereas the command _kdbgscan_ determines the correct profile. In the following images we can determine the differences provided by both commands.

`python2 vol.py -f ../adam.mem imageinfo`

![](<../.gitbook/assets/1 (1).png>)

`python2 vol.py -f ../adam.mem kdbgscan`

![](<../.gitbook/assets/2 (1).png>)

### What was the process ID of notepad.exe? <a href="#_jx8nf2t8bcvb" id="_jx8nf2t8bcvb"></a>

There are several commands that can be used to list the processes of the memory dump image. In this case, the command _pslist_ or _pstree_ are two of the alternatives. Using the command _pslist_ we obtain the list of processes in the memory image provided, as it can be seen in the image below. The PID of the process notepad.exe is 3032. The command used is the following:

`python2 vol.py -f ../adam.mem --profile=Win7SP1x64_23418 pslist`

![](<../.gitbook/assets/3 (5).png>)

### Name the child processes of wscript.exe. <a href="#_pz8nm033h806" id="_pz8nm033h806"></a>

There are two ways of obtaining the child processes of wscript.exe, using the command _pslist_ and looking for every process having the Parent PID of wscript.exe or using the command _pstree_ which shows in a graphical way the child processes of every process if they have.

![](<../.gitbook/assets/4 (5).png>)

Using the following command, we can see in the image below how the child processes are represented. In this case, the direct child process of wscript.exe is UWkpjFjDzM.exe.

`python2 vol.py -f ../adam.mem --profile=Win7SP1x64_23418 pstree`

![](<../.gitbook/assets/5 (6).png>)

### What was the IP address of the machine at the time the RAM dump was created? <a href="#_btjf7hlv0rar" id="_btjf7hlv0rar"></a>

To scan for network artifacts the command _netscan_ can be used. Thus, to get the IP address of the local machine the following command is used. As it can be seen in the image below, the IP address of the machine is **10.0.0.101**.

`python2 vol.py -f ../adam.mem --profile=Win7SP1x64_23418 netscan`

![](<../.gitbook/assets/6 (2).png>)

### Based on the answer regarding the infected PID, can you determine what was the IP of the attacker? <a href="#_uwh6hftokz6j" id="_uwh6hftokz6j"></a>

As we can see, there is a connection performed by the process UWkpjFjDzM.exe from the local IP address to the address 10.0.0.106, which might be the attacker’s IP address.

![](<../.gitbook/assets/7 (1).png>)

### What process name is VCRUNTIME140.dll associated with? <a href="#_p4epqicbrz6n" id="_p4epqicbrz6n"></a>

To list the DLLs that have been loaded by a process, the command _dlllist_ can be used. In this case, with grep we can get the times that the DLL was loaded but the PID of the processes is not shown. If we inspect the whole response of the command, which is massive, we can determine that there are five associated processes to this DLL, including Excel, Powerpoint and Outlook, but there is a suspicious one called OfficeClickToR with PID 1136.

`python2 vol.py -f ../adam.mem --profile=Win7SP1x64_23418 dlllist`

![](../.gitbook/assets/8.png)

![](<../.gitbook/assets/9 (1).png>)

### What is the md5 hash value of the potential malware on the system? <a href="#_svg4xb8v1mwz" id="_svg4xb8v1mwz"></a>

The potential malware sample seems to be UWkpjFjDzM.exe. There is a command called _procdump_ that can be used to dump the process executable to then compute the hash.

`python2 vol.py -f ../adam.mem --profile=Win7SP1x64_23418 -D ../recovered -p 3496 procdump`

![](<../.gitbook/assets/10 (4).png>)

Then, to generate the MD5 hash of the malware sample is the following:

![](<../.gitbook/assets/11 (1).png>)

### An application was run at 2019-03-07 23:06:58 UTC, what's the name of the program? (Include extension) <a href="#_5bu6tpz11pt1" id="_5bu6tpz11pt1"></a>

There is a command called _shimcache_ that can be used to list the executed applications in addition to the timestamps of their executions. Thus, as it can be seen in the image below, we are using grep to get the specific application, which in this case is Skype.exe.

![](<../.gitbook/assets/12 (2).png>)

### What is the short name of the file at file record 59045? <a href="#_1t4dven6msjk" id="_1t4dven6msjk"></a>

Volatility provides a command called _mftparser_ which scans for the entries in the Master File Table. In this case, we are using grep to reduce the amount of printed information and to be more specific. As it can be seen, the shortname of the file is Users\Bob\DOCUME\~1\EMPLOY\~1\EMPLOY\~1.XLS

![](<../.gitbook/assets/13 (2).png>)

### This box was exploited and is a running meterpreter. What PID was infected? <a href="#_uvjyfrxxio5t" id="_uvjyfrxxio5t"></a>

As we stated before, the malware seems to be UWkpjFjDzM.exe. Furthermore, by inspecting the result of the _netscan_ command, as we can see in the image below, the port to which the connection is established on the attacker’s machine is 4444, which is the default port used by Metasploit. This might be a clue to determine that the meterpreter session is carried out in this connection. The infected PID is then 3496.

![](<../.gitbook/assets/14 (1).png>)
