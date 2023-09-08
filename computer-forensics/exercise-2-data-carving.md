---
description: >-
  Case 2: Data carving. Recovering information from deleted files from USBs,
  quickly formatted USBs and securely wiped USBs
---

# Exercise 2 - Data carving

### Copy several files (e.g., PDF or MS Office files) to a USB memory and move to the trash some of these files (you can also empty the trash). Can you recover the deleted files via data-carving techniques?

For this activities we are going to use the pdf file corresponding to the module 2 of the Forensic subject. As it can be seen in the image below, there is no file inside the USB memory at the time of performing the data carving because we moved it to the trash previously.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 17.21.25.PNG" alt=""><figcaption></figcaption></figure>

In order to specify **scalpel** what files to look for, it was needed to uncomment the signatures of the types of files that were going to be retrieved. In this case, as we wanted to look for **pdf** files, we uncommented the signatures shown in the following image:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 19.51.42.PNG" alt=""><figcaption></figcaption></figure>

At this point, we just have to execute scalpel specifying what block device is going to be read (in this case the USB is in `/dev/sdb1`) and with the option `-o <path>` the location where the recovered files are going to be stored.&#x20;

As it can be seen in the image below, by the execution of scalpel two files were carved, one corresponding to the first signature and one corresponding to the second one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 17.54.52.PNG" alt=""><figcaption></figcaption></figure>

If we look at the content of the directory which contains the carved files, we can see the result of the execution in the file `audit.txt` and the two recovered files.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 17.56.22.PNG" alt=""><figcaption></figcaption></figure>

By opening the first one, we can see that it corresponds with the deleted pdf of the module 2.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 17.55.50.PNG" alt=""><figcaption></figcaption></figure>

As a result, we can determine that the simple deletion of files from a USB memory is not permanent. However, what really happens is that the deleted files are still in the USB storage but the portion of the memory used to store this file is marked as empty to the user. As a result, the file gets deleted once it gets overwritten.&#x20;

### Copy several files (e.g., PDF or MS Office files) to a USB memory and format the USB memory using a “QuickFormat”. Can you recover the deleted files via data-carving techniques?

In the case of performing a quick format, it does happen the same as in the previos section. In reality, a quick format does erase the FAT (File Allocation Table) but not the sectors containing the file information, so it is possible to carve the files inside the USB memory.&#x20;

To test it, we first have to perform a quick format on the USB. As it can be seen in the image below, at the moment of performing the format, the USB memory had the pdf file inside.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 23.20.33.png" alt=""><figcaption></figcaption></figure>

Then, we check the box of the Quick Format to select the type of format we want to execute.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 23.20.52.png" alt=""><figcaption></figcaption></figure>

Once the Quick Format have finished, we can start the data carving process.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-28 a las 23.21.22.png" alt=""><figcaption></figcaption></figure>

Once the USB memory is formatted, we start the data carving process following the same steps as in the first section exeuting **scalpel** with the same arguments but in this case we are going to store the result of the command in another directory.

The result of the data carving is that two files were retrieved, one for each of the signatures.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-29 a las 15.33.27.png" alt=""><figcaption></figcaption></figure>

The result of the data carving process is stored under the specified directory and as it can be seen, there are two pdf files recovered.&#x20;

<figure><img src="../.gitbook/assets/Imagen 29-4-23 a las 15.41.jpg" alt=""><figcaption></figcaption></figure>

By opening the first pdf, we can see that it is the original pdf used for this practice.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-29 a las 15.33.59.png" alt=""><figcaption></figcaption></figure>

### Copy several files (e.g., PDF or MS Office files) to a USB memory. Wipe in a secure-way the entire USB memory (or some of the files). Can you recover the deleted files via data-carving techniques?

The last alternative is to securely erase the USB memory. In this case, it is very unlikely that any data can be recovered using standard recovery methods. The secure way not only involves destroying the file system but overwritting all the memory sectors with random data or zeros multiple times, making it really difficult to recover any data.&#x20;

For this task it has been tested several ways of wiping the entire USB memory in a secure way and all of them have prevented **scalpel** from carving any file. The first way was using the Windows utility unuchecking the Quick Format option.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-29 a las 21.22.50.png" alt=""><figcaption></figcaption></figure>

The second way was using:

```bash
# if=<FILE> --> read from FILE instead of stdin.
# of=<FILE> --> write to file instead of stdout.
# bs        --> block size (default=512)

dd if=/dev/urandom of=/dev/sdb1 bs=1M
```

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-29 a las 19.03.30.png" alt=""><figcaption></figcaption></figure>

The last way of wiping the USB memory that we performed was using:

```bash
# -n N --> overwrite N times (number of iterations)
# -u   --> truncate and remove the file after overwriting
# -v   --> verbose
# -z   --> add a final of zeros to hide shredding  

shred –n 5 –uvz /dev/sdb1
```

<figure><img src="../.gitbook/assets/Imagen 29-4-23 a las 19.52.jpg" alt=""><figcaption></figcaption></figure>

The output of all of these techniques to perform a secure way wipe of the USB memory resulted in that no file was carved. As it can be seen in the image below, the output of the **scalpel** operation was unsuccessful.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-29 a las 19.43.00.png" alt=""><figcaption></figcaption></figure>

&#x20;
