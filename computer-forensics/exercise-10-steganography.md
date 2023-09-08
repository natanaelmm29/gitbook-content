---
description: 'Case 10: Some steganography CTFs'
---

# Exercise 10 - Steganography

Exercise 10 - Stego Exercises

Case 10: Steganography CTFs.

### Funny pictures with steganography are often used to broadcast messages without being detected. Can you find the hidden message(pCTF{message}) inserted in the following image? <a href="#_b5dcs3g5m5lo" id="_b5dcs3g5m5lo"></a>

The first step is to compute the hash of the file that is going to be analyzed.&#x20;

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

An 8-bit colormap is a typical place where things are hidden using steganography.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

By using Gimp as an image editor, we are able to change the colormap palette used by the image. Changing it to a high contrast one, we can see the flag in the image. As we can see in the image, the flag is **pctf{keep\_doge\_alive\_2014}**.

&#x20;

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### My friend visited a lab and sent me this photo of them 3D printing something.  Pretty cool, eh?. Can you find the hidden message (GCTF{message}) inserted in the mentioned picture?

Firstly we check that the hash value of the file has not changed from the provided one to assure that the file has not been modified.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Using the command file we can get some information about the file which is being analyzed to see if there is something interesting that could be a clue for the steganography technique that is being used, as it was in the case of the colormap.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Using the command `strings` and `xxd` we can notice that there is something suspicious inside the image file. As we can see in the image below, there are some “_jpeg_” strings so we can imagine that there are some images inside the image file.

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

To retrieve the different images inside, we can use the command `binwalk` that is used to analyze and extract the content of binary files. As we can see in the image below, there are three JPEG images extracted.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

By changing the images name to add the correct extension, we can now open them to see the content. The first image is the default one, the second one is a hint and the last one is the flag, which is **GCTF{d474\_c4rv1n6\_15\_345y}**.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

![](<../.gitbook/assets/image (34).png>)![](<../.gitbook/assets/image (4).png>)

### A colleague sent you a picture (lol.png) and gave you the following instructions: "It's not LSB, it's MSB! Red is Random, Green is Garbage, Blue is Boring. Hint: Only one channel is correct. Also, I like doing things top-down." Can you find the hidden message (flag{message}) inserted in the mentioned picture?

The first step is to check that the hash of the files is the same to assure that the file has not been modified.

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

With the command file we obtain the basic information of the PNG file.&#x20;

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

In the statement it says that it is not LSB but MSB, which means Most Significant Bit. For example, in a binary number, it is the bit with the largest value, which is usually located farthest to the left. It also says that only one channel is correct so it is basically testing all of them, although the sentence says that red is random, green is garbage and blue is boring, which seems to be the correct one. Finally, the statement mentions that things are done top-down, which means that the order is made by columns.

\
With this information clear, using the steganography tool called Stegosolve, we can see in the flag, which is **flag{MSB\_really\_sucks}**.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

### Elliot (Mr Robot) sent you two audio files. He did not provide any other information. Can you find the hidden message4 (CTF{message}) inserted in the mentioned wav files?

The first step is to check the hash of the files to ensure that they have not been modified.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Firstly, by trying to open both the audio files with DeepSound, we are not able to open any of them, as one is encrypted with a password and the other has no compatible format.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Adding a spectrogram to the other .wav file and some filters to clear the password, we can see that there is what seems to be a password, as it does not have the flag format.

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

By checking if the password works for the protected audio file, we can see that it is decrypted and it is shown the secret file.

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

By extracting the secret file we can see the following message:

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

After analyzing the audio files, I remembered that I could use `strings` an `exiftool` to analyze the strings and metadata of the doc. As it can be seen in the image below, the flag is in a metadata field. The flag is: **GCTF{57364n06r4phy\_15n7\_7h47\_h4rd**}.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

### Your ex-coworker, Zizi, discreetly changed your wallpaper on your desktop before leaving the company... Can you find the hidden message (CTF{message}) inserted in the mentioned image file?

Checking the integrity of the file:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

The first step is to execute the command `file` to get general information about the file.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

As the hint is that there is a file inside the image, I use `binwalk` to retrieve the files inside the image. As it can be seen in the image, there is a zip file inside the image.

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Uncompressing the zip file, we can see that there is a file called _pass.txt_ but it is not the flag.

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

I have tried for hours to install JPHide but I gave up. Using the command `xxd` I can see that there is another JPEG file inside the image, which is supposed to be retrieved using JPHide and the password obtained in the pass.txt file. In the image itself or another file it might be the flag.
