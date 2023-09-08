---
description: 'Case 8: Analysis of a smartphone memory image with Autopsy'
---

# Exercise 8 - Smartphone analysis with Autopsy

![](<../.gitbook/assets/0 (6).png>)

### What is the phone owner's full name (first name and last name)? <a href="#_9li1ewkz4tub" id="_9li1ewkz4tub"></a>

There are several ways to obtain the owner’s name. In this case, if we look into the Whatsapp preferences, there is a .xml file containing the preferences in which we can see the full name, Derek Hor.

![](<../.gitbook/assets/1 (6).png>)

Almost finishing this activity I came across this image (below) which contains an X-Ray of a knee. During this exercise we have investigated the web history and the messages and we have come across some messages where the arrested man was complaining about a knee pain and some web searches related to medical centers and knee pain. Taking this into account, we can suppose that the X-Ray belongs to the arrested man and we can see his full name in the image, which corresponds with the one found in the Whatsapp preferences, Derek Hor

![](<../.gitbook/assets/2 (6).png>)

### What is the phone number he reported about the drug delivery? <a href="#_eyj08g77unq5" id="_eyj08g77unq5"></a>

If we look at the contacts tab, we can see that there are three registered contacts, one related to a Medical Center, another called Arnie and the suspicious one which is called Bos and whose phone number is +12395104974. In the image below we can see the contact tab of the Boss.

![](<../.gitbook/assets/3 (6).png>)

### What were the suspect's delivery locations on the night of arrest? <a href="#_d4lcyewma4gx" id="_d4lcyewma4gx"></a>

The man was arrested on April 18, 2021. If we inspect the messages with the Boss contact, it is very easy to notice that the arrested man sends certain photos regularly. It seems that these photos are used as the evidence of every delivery. As we can see in the image below, it is shown a message that says, “Today’s photos”, which seems to prove our theory.

![](<../.gitbook/assets/4 (7).png>)

If we now move to the messages sent on that day, we can see the URL which seems to be the URL of the folder with the images sent to the Boss. In the image below we can see the content of the message.

![](<../.gitbook/assets/5 (8).png>)

Now we know that the images of the delivery that night are supposed to be stored in that album. If we look at the browser history (the one provided in the campus, as the autopsy web history does not have timestamps), we can see that there are three accesses to that album in the URL “https://ibb.co/album/rvD0OF” , so this might mean that there were three locations to deliver and those three images were uploaded to the album. If we take a look at the Google Maps searches, we can see three locations, right before the uploading of each photo. This might mean that the arrested man always looks for the location right before going to deliver the drugs and after that he uploads the corresponding image to the album. In result, the delivery locations that night were:

* Camelback Golf Club” in 7847 N Mockingbird Ln, Scottsdale
* 2013 W Harwell Rd., Phoenix
* 33°29'04.2"N 111°52'38.0"W

![](<../.gitbook/assets/6 (3).png>)

### How long has the suspect been acting as a drug dealer? <a href="#_xtpdcp1g1nai" id="_xtpdcp1g1nai"></a>

We have proven that the last working night of the drug dealer was the night of the arrest, as that night he made three drug deliveries. Now, if we take a look at all the messages, we can see that a message was sent to Arnie saying that he just came back from an interview. In the message he also asks Arnie to look where the boss hangs, but this message has no images attached. However, the two consecutive messages are images, which cannot be seen as they are encrypted by Whatsapp. However, if we inspect the Whatsapp media folder, inside the sent images, there are only two of them which corresponds to the number of images sent to Arnie.

![](<../.gitbook/assets/7 (4).png>)

Although the images have no timestamp, it is not difficult to infer that those images are the sent images to Arnie to show the places where the boss usually hangs. With this information, we can determine that the interview was with the drug provider because of the luxury. Thus, the arrested man allegedly started working as a drug dealer on that day, on June 6, 2020.

![](<../.gitbook/assets/8 (6).png>) ![](<../.gitbook/assets/9 (2).png>)

### From what bitcoin wallet did he get paid the last time for his job? <a href="#_5kwpq3py4ccc" id="_5kwpq3py4ccc"></a>

This was a difficult task to solve. The first part is really easy, the last message related to a payment is on April 15, 2021, as it can be seen in the image below, with an amount of 0.0913 btc.

![](<../.gitbook/assets/10 (6).png>)

In the following messages, we can see how the arrested man declares to the boss that he paid a bit less than that amount stated in the message.

![](<../.gitbook/assets/11 (2).png>)

After some time trying to determine the bitcoin wallet of the drug dealer, I remembered that there was a hint in the campus which was a link to Blockchair, which is a blockchain search and analytics engine for Bitcoin and some other cryptocurrencies. Then, we only have to look for transactions that were made on April 15, 2021 with an amount of approximately 0.0913 btc. Furthermore, the message was sent at 10:35 PDT, which is 17:35 UTC, so we have to look for transactions around that time. After some time trying to apply the correct filters (they were not applied well by the platform), finally we got the whole list of the transactions between 17:30 and 17:36 UTC whose amount was between 0.09 and 0.0913 BTC. As we knew that the boss sent a bit less than the accorded quantity, we can see that there is only one transaction fulfilling those requirements.

<figure><img src="../.gitbook/assets/12 (3).png" alt=""><figcaption></figcaption></figure>

If we take a look at the transaction, we can see both the wallet of the sender and the receiver.

![](<../.gitbook/assets/13 (4).png>)

### What is the phone number of the drug supplier? <a href="#_n3ldw8zd0o6x" id="_n3ldw8zd0o6x"></a>

By looking at the contacts and the messages, there is an unknown phone number. By inspecting the messages exchanged with that number, we notice that it does not seem to be a drug supplier as the messages have nothing to do with that topic, as they are talking about a virtual sex game.

![](<../.gitbook/assets/14 (2).png>)

Then, navigating through all the images we can see some screenshots of an app called Signal and something related to its backups. Signal is an instant messaging app similar to Whatsapp but it is supposed to be more secure and private.

![](<../.gitbook/assets/15 (2).png>) ![](<../.gitbook/assets/16 (2).png>)

By further investigation, we can find in the images three screenshots containing the passwords of the backups of the conversations.

![](<../.gitbook/assets/17 (1).png>) ![](<../.gitbook/assets/18 (1).png>) ![](<../.gitbook/assets/19 (1).png>)

At this point and with any other clue, we might think that the communications with the supplier were carried out through this application. Using the keyword search in Autopsy, we entered the word signal to look for all files containing that name and reached a signal backup. As we can deduce, the backups are encrypted with passwords as the screenshots obtained previously were related to the local backups encryption.

![](<../.gitbook/assets/20 (1).png>)

At this time, we have to look for a way of decrypting that backup in order to see the content. It was a bit difficult to find a tool for decrypting _.backup_ files until I entered the word signal in Google. Then some tools appeared to solve that problem so it was a matter of trying those tools to check if they work. There was a python tool to show using HTML the content of the backup by using another tool called “_signalbackup-tools_” that can be found in this repository: [https://github.com/bepaald/signalbackup-tools](https://github.com/bepaald/signalbackup-tools).

After a lot of time trying to make this tool work from the source code, I finally downloaded the Windows binary and directly executed it. As it can be seen in the image below, I tested all the passwords from the screenshots and none of them seemed to work.

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

After this, I did suppose that the backup password was the one in the first screenshot that is covered so we cannot get it. I started then from the first step to search in every file. I looked over all the images again and then started with different files. Within the databases and looked around the names and noticed that there was one called keep.db, which is the same name that is in the screenshot. By looking at the content of the table we see that there is a row whose content seems like a backup password.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

By trying to dump the backup by using the signal backup tools we could see the content of the backup with SQLite.

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

With SQLite we can list the database. Within the _recipient_ table we can see that there are two rows with two phone numbers, one corresponding to horatio (which is the arrested man, as it is the email account found in another backups as the calendar) and another number, which is supposed to be the drug supplier, whose number is +14233767293.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

### When was the last time the suspect met his supplier? <a href="#_nu9aj41xbqq8" id="_nu9aj41xbqq8"></a>

When looking for the backup password I took a look at the calendar, I quickly noticed that there were regular events called the same, “_Pizza delivery_”. By searching for the last “_Pizza delivery_” event, filtering by title and converting the _dtstart_ column to date format, we can see that it was on April 10, 2021 00:30:00.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### What is the supplier's phone IMEI identifier? <a href="#_xamre3yo066n" id="_xamre3yo066n"></a>

Using the keyword search, I did not find any IMEI number. Asking the professor I was provided with a hint, which is using the provided tool to locate the devices connected at a specific time in a specific location. By entering the different coordinates, we got a huge list of connected devices. The idea was to get that list and look for common devices connected on those days. By using the commands _sort_ and _uniq_ and the list of the whole results for every coordinate, we are able to see that the two numbers matching the requirements are 332182208414842 and 350236009513272, so one of them should be the IMEI of the drug supplier and the other of Derek Hor.

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>
