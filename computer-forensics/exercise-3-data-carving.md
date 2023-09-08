---
description: 'Case 3: Floppy disk'
---

# Exercise 3 - Data carving

## Background

At 3:15 PM on September 15, 2004, you have been called by a Mr. Peter Szef, the owner of a small company named Small Business, Inc. Mr. Szef told you that he suspected that one of his assistants, Emma Crook, may be leaking company sensitive material to some of his big competitors. At 2:00 PM today he confronted Mss Crook with his suspicions and told her that she would be back at 3:00 PM for an explanation.

When Mr. Szef arrived back at Mss Crook’s office at 3:00 PM, she was gone. The office was completely cleaned out of all his belongings, including a personal laptop that Mss Crook used to bring to work. Mr. Szef turned on Mss Crook’s computer, only to find out that it would not boot. However, Mr. Szef found a floppy diskette in the trash can.

Mr. Szef wants you to examine the computer and the floppy diskette and to tell him exactly what Mss Crook was up to. You gave the computer a quick look and found that the hard drive was missing, so your only evidence, if any, will be on the floppy diskette.

### Question: what information can be found inside the floppy's image?

To start with this exercise, the first step is to ensure that the floppy disk image has not been modified by computing the sha-256 hash and compare with the given one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-01 a las 17.24.56.png" alt=""><figcaption></figcaption></figure>

To start the data carving process, we use **scalpel** passing as the argument the floppy image and the output directory where the carved files are going to be stored.&#x20;

```bash
scalpel floppy.dd -o /home/kali/Documents/Forensics/floppy_recovered
```

As we can see in the image below, scalpel was able to carve different files from the diskette. In this case, pgp, doc and mov files were retrieved.&#x20;

```bash
Scalpel version 1.60 audit file
Started at Sat May  6 06:18:57 2023
Command line:
scalpel floppy.dd -o /home/kali/Documents/Forensics/floppy_recovered 

Output directory: /home/kali/Documents/Forensics/floppy_recovered
Configuration file: /etc/scalpel/scalpel.conf

Opening target "/home/kali/Downloads/floppy/floppy.dd"

The following files were carved:
File		  Start			Chop		Length		Extracted From
00000030.pgp        85650		YES          100000		floppy.dd
00000029.pgp        64658		YES          100000		floppy.dd
00000028.pgp        18066		YES          100000		floppy.dd
00000027.pgp       107720		YES          100000		floppy.dd
00000026.doc       105984		YES         1368577		floppy.dd
00000025.doc        84480		YES         1390081		floppy.dd
00000024.doc        63488		YES         1411073		floppy.dd
00000023.doc        16896		YES         1457665		floppy.dd
00000022.doc       105984		NO          1368577		floppy.dd
00000021.doc        84480		NO            21504		floppy.dd
00000020.doc        63488		NO            20992		floppy.dd
00000019.doc        16896		NO            46592		floppy.dd
00000018.mov        87389		YES         1387172		floppy.dd
00000017.mov        66397		YES         1408164		floppy.dd
00000016.mov        43084		YES         1431477		floppy.dd
00000015.mov        41020		YES         1433541		floppy.dd
00000014.mov        31978		YES         1442583		floppy.dd
00000013.mov        31463		YES         1443098		floppy.dd
00000012.mov        30272		YES         1444289		floppy.dd
00000011.mov        29689		YES         1444872		floppy.dd
00000010.mov        28924		YES         1445637		floppy.dd
00000009.mov        27180		YES         1447381		floppy.dd
00000008.mov        26964		YES         1447597		floppy.dd
00000007.mov        26313		YES         1448248		floppy.dd
00000006.mov        26058		YES         1448503		floppy.dd
00000005.mov        25229		YES         1449332		floppy.dd
00000004.mov        25079		YES         1449482		floppy.dd
00000003.mov        20357		YES         1454204		floppy.dd
00000002.mov        20190		YES         1454371		floppy.dd
00000001.mov        20069		YES         1454492		floppy.dd
00000000.mov        19918		YES         1454643		floppy.dd


Completed at Sat May  6 06:18:57 2023

```

#### DOC files

Regarding the doc files, by looking at the contents using the **strings** command, we can see that three of them are .doc files with non interesting content.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.05.25.png" alt=""><figcaption></figcaption></figure>

This is the first doc file and through the **strings** command we can see that it is a Microsoft Word document.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.10.08.png" alt=""><figcaption></figcaption></figure>

This is the second doc file, which is another Microsoft Word document whose contents cannot prove that the investigated person is guilty.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.12.19.png" alt=""><figcaption></figcaption></figure>

This is the third Microsoft Word document, which contains the same text as the previous one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.12.34.png" alt=""><figcaption></figcaption></figure>

However, in the carved doc files there is a file which looks different. By using the **strings** command (second image) we can see that it is not a Word document but an Excel document.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.15.00.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.16.05.png" alt=""><figcaption></figcaption></figure>

If we try to open the file, we can see that it is password-protected, which raises suspicions as to whether or not it contains compromising information.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.19.13.png" alt=""><figcaption></figcaption></figure>

Using John The Ripper, we are able to crack the password of the file to retrieve the content of it. As we can see in the following image, we use the tool **office2john** to compute the hash of the file.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.21.45.png" alt=""><figcaption></figcaption></figure>

Then, using **john** an the **rockyou.txt** wordlist we are able to obtain the password of the Excel file, which seems to be "crook".

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.22.26.png" alt=""><figcaption></figcaption></figure>

Then, by opening the file, we can see the following sentense, determining that it corresponds to the evidence to demonstrate that Mss Crook was leaking sensitive information from the company.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.22.49.png" alt=""><figcaption></figcaption></figure>

#### MOV files

Regarding the mov files, there might be some error that is preventing from opening all the video files.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-05-06 a las 13.34.36.png" alt=""><figcaption></figcaption></figure>

Finally, regarding the PGP files, they are used to encrypt, decrypt and authenticate different file types and in this case they are not needed to demonstrate that Mss Crook was leaking information from the company.
