---
description: 'Case 4: Data carving CTF. Analysis of a pcap file.'
---

# Exercise 4 - CTF data carving

### One of our agents managed to sniff an essential piece of data transferred transmitted via USB; he told us that this pcap file contains all that we need to recover the data.

### Question: can you recover the flag (AlexCTF)?

In this case, a pcap file is provided to analyze. There are different tools that can be used to retrieve information from this type of files so we are going to used some of them to show how they work.

#### Using `strings` command

One of the most common ways of inspecting the content of a file is using strings to print the printable strings of a file. In this case, it is used to get some general information or some hints of the content of the file to know what of the strings can be considered as interesting. The command used was:

```bash
strings fore2.pcap
```

The output of this command shows some interesting strings to take into account in later investigations. As we can see in the image below,&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 16.41.27.png" alt=""><figcaption></figcaption></figure>

#### Using `binwalk` command

This tool is used for analyzing, reverse engineering, and extracting firmware images. In this case, by running the command we can extract the files to a folder, something similar as the scalpel usage. From the output of the command we can see that there is a PNG file that was retrieved.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 16.55.31.png" alt=""><figcaption></figcaption></figure>

If we use `xxd` to print the signature of the file, we can ensure that it is a PNG file, so we can then change the filename to use the .png extension.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 16.57.36.png" alt=""><figcaption></figcaption></figure>

By opening the image, it can be seen the image with the flag inside.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 16.58.47.png" alt=""><figcaption></figcaption></figure>

#### Hand analysis of the PCAP file using Wireshark

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 17.41.26.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.01.13.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.01.53.png" alt=""><figcaption></figcaption></figure>

#### Using `scalpel` tool

Scalpel is another tool that can be used in this case. After some attempts to carve the png file, I already knew that there was an image file inside the pcap file but scalpel was not able to retrieve it until I noticed that the signtature of png files inside the configuration file was wrong. Once updated it, scalpel could carve the png file. The command used was:

```bash
scalpel fore2.pcap -o scalpel-ex 
```

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.13.04.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.13.42.png" alt=""><figcaption></figcaption></figure>

#### Using `foremost` tool

Foremost is another data carving tool that can be used similar to scalpel. In this case, as it can be seen in the image below, we only have to specify the same parameters as for scalpel; the image to analyze and the directory to store the carved files. The image shows how foremost is able to carve the png file.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.21.41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.22.11.png" alt=""><figcaption></figcaption></figure>

This is the same flag obtained by different ways. We can conclude that there are several ways and tools of analysing the content and carving files from different types of images and devices.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 18.22.42.png" alt=""><figcaption></figcaption></figure>
