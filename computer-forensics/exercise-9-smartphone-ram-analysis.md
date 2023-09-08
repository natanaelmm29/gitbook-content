---
description: 'Case 9: Analysis of the RAM memory of a smartphone'
---

# Exercise 9 - Smartphone RAM analysis

_This time your friend asks for your help in obtaining the user's unlock password of an Android mobile phone. To do so, we have a RAM image ("i900-CM.bin"). As clues for the case, we know that the password has the following format "INS{xxxxx}". We also have two files, "System.map" and a "module.dwarf", which will allow you to analyze the RAM image with the volatility tool._

### _With all this information, what is the user's unlock password?_

_For this activity the first step was to create a profile for Volatility using the two provided files, the System.map and the module.dwarf. I really tried it hard to make it work but it was not possible, so I finally got the filesystem from the professor._

_However, the process to get the file system of the mobile phone using Volatility would be very easy as there is a command that basically recovers the whole file system, which is called linux\_recover\_filesystem. To prove that I tried it hard, here is the output error that appeared many times. As it can be seen in the image below, the profile was not being created correctly._

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

_Moreover, here in the image below it can be seen the zip file containing the necessary files to create the profile._

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

_Once having the file system, I looked on the Internet where the passwords in Android file system are stored and apparently, they are stored under the /data/system directory. In fact, here we can find both the gesture key and the password key._

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

_The password is stored in the password.key file, but as we can imagine, it is not stored in plaintext, but it is hashed. The first step was to copy the hash and look on an online analyzer what type of hash it was._

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

_As we can see, the type of the hash was not detected. After a while, I found a very interesting blog about cracking Android passwords that said that the Android passwords were stored in a 72-character length string concatenating the SHA-1 hash and the MD5 hash for low powered devices. Moreover, in the blog it is said that the passwords were hashed using salt, whose type could be found in different locations depending on the version._

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

_In this case, as it can be seen in the image below, the database is stored under /data/system/ directory and by entering the following SQL query, we are able to get the salt value._

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

_The format of the salt value is a 64 bit or long integer, but we have to convert it to hex and then lower case (I used an online converter)._

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

_As the MD5 hash is easier to crack, the format in this case would be md5($pass.$salt), so as it can be seen in the image below, the hashcat mode has to be “10”._

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

_For the password cracking we have used the host machine instead of the virtual machine as the process is much quicker. As it can be seen in the image below, we have created a file with the MD5 hash with the salt concatenated._

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

_In the following image it is shown the cracked password, which is INS{t1MmY}. The arguments for the hashcat execution are -m 10 to indicate the type of the hash, -a 3 to indicat that the attack mode is brute force and finally the argument -1 to create a custom charset for the passwords cracking._

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

_The link to the used blog post is: https://www.pentestpartners.com/security-blog/crackingandroid-passwords-a-how-to/_
