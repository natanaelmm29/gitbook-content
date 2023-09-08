---
description: APT - Trends and insights
---

# Talk from the CCN

## Centro Criptológico Nacional

Attached to the Ministry of Defense, it is part of the CNI.&#x20;

## CCN Incidents

Focused on high risk and critical incidents. Some of the highest incidents regarding the ramsonware to the SEPE and some documents compromises&#x20;

## Threat Actors. APT

The most active threat actors are state-sponsor actors (related to governments and states). Their main objective is cyberspionage. The goals of APTs are:

* Theft of property, information --> threat
* Sabotage, disruption
* Stay undetected as long as possible --> persistence

They use very advanced techniquesm high resources and specialization, complex malware, and post exploitation techniques.&#x20;

## APTs in Spain

Geopolitical advantages and reduce the technological gap. Coming from North Korea, Russia, China, US.&#x20;

Panda --> China: Goblin, Vixen, Deep, Pirate...

Bird --> Russia: Energetic Bear, Octubre Rojo...

## How APTs work

#### Vulnerability exploitation

Both 0 days and 1+day vulnerabilities.&#x20;

#### Credential harvesting

Mimikatz, gsecdump. Using brute force techniques, default and password spraying.

#### Backdoor deployment

Remote shells on servers or other infectionss with other malware. Laterally movement.

{% hint style="info" %}
Nowadays APT groups are really interested in email accounts due to the remote work. Accessing email accounts of the employees could let them to get so valuable information
{% endhint %}

## Cyberkill chain

1. Reconnaissance
2. Weaponization --> creating the weapons (emails, exploits, etc.)
3. Delivery --> send emails, connect to servers
4. Exploitation
5. Installation
6. Command and Control
7. Actions on objectives --> taking information from the organization, etc.

## Case study. RU APT

APT29/Dukes/Nobelium --> to Solarwinds (2020) they were able to compromise the organization and trojanize a new version of the product so the customers were infected when upgraded. 18K customers were affected. HOW? The attackers injected code into the production pipeline in compilation time, then injected a malware in a software download and uploaded it to the servers and they moved laterally to other network assets.

## Case study.  CN incidents

The Hafnium campaign used a set of vulnerabilities on the Microsoft Exchange software to compromise a lot of organizations in 2021. Multiple 0 day were used that enabled Preauthentication RCE and authentication bypass and they remained undetected for months, with mailbox access and webshells deployed to exfiltration.&#x20;

## Case study. Cybercrime to SEPE in 2021

It is not the same as an APT but they are similar in the first stages. They differ on their goals, cybercrime want to disrupt the services and ask for money in change. In this case it was so important to kick out the attackers to restart the servers as it is an essential service on our country.&#x20;

{% hint style="info" %}
When APTs attack it is not that important to remove the attackers as soon as possible but to define a scope, learn how attackers move and act within the network to learn new procedures.
{% endhint %}

## Case study II. Work Ministry MITES in June 2021

The ransomware was deployed on the serves and not on the workstations as the cyberattack to SEPE.

#### Marketplace

Access Brokers --> sell access to organizations

Malware developers --> ransomware

Malware deployers --> takes the malware and deploys within the organization

### Work flow

1. Log in via citrix remotely with the bought credentials on the Dark Web
2. Privilege escalation
3. Use of RATs (Remote Access Tools) to move laterally to the DC
4. Information exfiltration
5. In the DC, encrypt the information and deploy the ransomware.

## Warfare

There have been deployed some attacks due to the warfare. Most of them to the critical infraestructures on the target country (attack to Viasat called AcidRain).&#x20;

{% hint style="info" %}
Another APTs: Snake/Turla and APT28/Sofacy
{% endhint %}

