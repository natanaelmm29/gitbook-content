---
description: 'Case 0: Metadata CTF for Computer Forensics. Drug dealer.'
---

# Exercise 0 - CTF metadata

The first step is to ensure that the file has not changed by computing the hash and comparing it with the one provided in the statement.

![](<.gitbook/assets/0 (1).png>)

### **Who is Joe Jacob's supplier of marijuana and what is the address listed for the supplier?**

Inspecting the supposedly erased file we determine that it covers 40 sectors in memory. If we inspect the first one and expand it to the whole size, using the ASCII display we can see the content of the file. As a result, Joe Jacob’s supplier of marijuana is Jimmy Jungle and the address is 626 Jungle Ave Apt 2, Jungle, NY 11111.

![](<.gitbook/assets/1 (11).png>)

![](<.gitbook/assets/2 (10).png>)

### **What crucial data is available within the coverpage.jpg file and why is this data crucial?**

This is how the image looks like when opening it with an image viewer. However, by inspecting the content of the image with commands like **strings** or **xxd**.

![](<.gitbook/assets/3 (7).png>)

![](<.gitbook/assets/4 (1).png>)

With the **strings** command we are able to print the strings of printable characters in files and as we can see in the image below, there is a string “pw=goodtimes” which may be a password for something.

![](<.gitbook/assets/5 (3).png>)

There is another way to dump the content of a file by using the command **xxd** which performs a hex dump as we can see in the image. Here the string “pw=goodtimes” appears again.

![](<.gitbook/assets/6 (4).png>)

### **What (if any) other high schools besides Smith Hill does Joe Jacobs frequent?**

If we analyse the file called “ScheduledVisits.exe”, it is supposed to have two sectors as we can see in the image below (sectors 104 and 105).

![](<.gitbook/assets/7 (6).png>)

However, from the image details tab we can determine that the size of the file occupies 5 sectors.

![](<.gitbook/assets/8 (2).png>)

Now, inspecting the content of the file in hex we found that the file signature corresponds with the file signature of zip file format (second image). At this point, the file is exported (the five sectors).

![](<.gitbook/assets/9 (5).png>)

![](<.gitbook/assets/10 (2).png>)

![](<.gitbook/assets/11 (3).png>)

After downloading the file, it is supposed to have a file inside called “Visits.xls” but the zip file is encrypted with a password. At this point we could use the password that was found inspecting the content of the cover file but there are some alternatives to get the password.

![](<.gitbook/assets/12 (4).png>)

With the tool **zip2john** the hash of the zip file is obtained and then with **john** we are able to get the password of the file, which is the same as the one obtained from the cover page image. Note that it is used a wordlist called “rockyou.txt” which is a massive password dictionary.

![](<.gitbook/assets/13 (1).png>)

Finally, by viewing the content of the .xls file inside the zip file we can determine that Smith Hill high school is not the only one that Joe Jacobs frequents. There are several high schools that he visits as the ones that can be seen in the image below: Key High School, Leetch High School, and many more.

![](<.gitbook/assets/14 (5).png>)

### **For each file, what processes were taken by the suspect to mask them from others?**

For the first file, the cover page, the suspect tried to mask it by changing the extension to .jpgc. Regarding the size of the file, the size of the file is supposed to be 15585 bytes which corresponds to the 31 sectors (73-103) from the image details. However, from the meta information of the file, it can be found that only one sector is used (sector 451).

The second file is called “JimmyJungle.doc” and it is not allocated so it is supposed to be erased. The size of the file is 20480 bytes which corresponds to the 40 sectors that appear in the meta information of the file (from sector 33 to 72). The suspect tried to mask this .doc file by deleting it.

The last file is called “ScheduledVisits.exe”. The suspect tried to mask it by changing the extension of the file from .zip to .exe. Moreover, the size of the file is supposed to be 1000 bytes which are the two sectors which appear in the file meta information (104 and 105), but in the image details tab we can see that the file occupies 5 sectors (from 104 to 108).

### **What processes did you (the investigator) use to successfully examine the entire contents of each file?**

For the first file, as from the image details tab we can see that the file size is 31 sectors, I got to the first one and by looking the hex dump I noticed that the first bytes corresponded to JPEG format (FF D8 FF E0 00 10 4A 46 49 46 00 01). At this point, I exported the file and then changed the extension to .jpg to then open the image. Then, using exiftool, strings and xxd I was able to get some interesting information from the file such as a possible password (goodtimes).

For the second file, the file is supposed to be deleted but we are able to retrieve the content simply by inspecting the content of the sectors that stored the information.

For the third file, from the fat contents tab I determined that the file size was 5 sectors, so by inspecting the content of the first sector and looking at the first bytes, I noticed that it was a zip file as the signature was 50 4B 03 04 which corresponds to zip format. After that, I exported the file and tried to decompress it but it was encrypted with a password. Then, using the password obtained from the cover page or using the tool john the ripper to crack the password, I was able to open the zip file and extract the .xls file inside.

