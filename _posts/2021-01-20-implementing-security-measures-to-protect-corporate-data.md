---
layout: post
title:  "Implementing security measures to protect corporate data"
description: "Hardware security protection, Firewalls, Data breach mitigation and Access Control Lists"
date: 2021-01-20 16:00:00 +0000
categories: Blog
---

# Implementing security measures to protect corporate data

### Table of contents:


|           Key                         |                      Content                          |
|--------------------------------------:|:------------------------------------------------------|
| <a href="#hardware">1</a>             | Hardware security protection                          |
| <a href="#firewall">2</a>             | What is a firewall                                    |
|   <a href="#types">2.1</a>            |   Types of firewall                                   |
|   <a href="#filter">2.1.1</a>         |   Packet-filtering firewalls                          |
|   <a href="#gateways"> 2.1.2 </a>     |   Circuit-level gateways                              |
|   <a href="#stateful">2.1.3</a>       |   Stateful inspection firewalls                       |
|   <a href="#app">2.1.4</a>            |   Application-level gateways (a.k.a. proxy firewalls) |
|   <a href="#next">2.1.5</a>           |   Next-gen firewalls                                  |
| <a href="#ids">3</a>                  | What is an Intrusion Detection System (IDS)?          |
| <a href="#control_access">4</a>       | Control access                                        |
|   <a href="#whitelisting">4.1</a>     |   Whitelisting                                        |
|   <a href="#blacklisting">4.2</a>     |   Blacklisting                                        |
|   <a href="#greylisting">4.3</a>      |   Greylisting                                         |
| <a href="#mitigation">5</a>           | How to mitigate the damage after a data breach        |
| <a href="#refs">6</a>                 | References                                            |

---

<p>&nbsp;</p>

## <a id="hardware"></a> Hardware security protection:

- **<u>Physical preservation of the environment:</u>** In a corporation, there is very little to no room for error. A single slip can cost the company millions, so the obligatory first step in securing company hardware should be isolating the servers in secure, free of dust, free of smoke and air-conditioned rooms to avoid physical wearing due to natural factors. We don't want our networking equipment and servers to run slowly or shut down making us losing data, catch fire or allow any other unfortunate event to happen out of our control, so we must use electrical gadgets like UPS (Uninterrupted Power Supply), Volt Guards and Spike Guards, store the devices in a room with proper humidity to avoid electrostatic hazards, correct isolation and cooling to avoid overheating and adequate physical security to avoid theft or vandalism, like the [Protonmail datacenter](https://protonmail.com/security-details), which *"is located under 1000 meters of granite rock in a heavily guarded bunker which can survive a nuclear attack."*. Their servers also use *"fully encrypted hard disks with multiple password layers so data security is preserved even if our hardware is seized"*. Hold on, do you want to know more on encrypted email and its advantages? [I got you covered](https://0x5ubt13.github.io/blog/2020/11/17/encrypted-mail.html).

- **<u>Hardening the hosts:</u>** if by host we understand all the devices able to connect to the layer 3, we need to take care of securing all of them or we risk an automated cyberattack. No measures we impose will really harden a device connected to the internet enough to avoid being victims eventually, but at least we can stop the majority of random attacks by having a bit of maintenance and common sense. Keeping the OS or firmware of the host up-to-date, not opening unnecessary ports to the public, having backups, using a VPN, restricting physical access as seen above with Protonmail or having strong control over users and privileged users, as well as a good password policy in place, are only a few examples of the things we can do to ensure our protection is way stronger than the company across the road.   

- **<u>Installing firewalls, antivirus and IDS/IPS:</u>** explained in detail right below.

<p>&nbsp;</p>

## <a id="firewall"></a> What is a firewall?

A firewall is a type of cybersecurity tool, either a hardware device or a utility program, that is used to monitor a network, by accepting, rejecting or dropping packets passing through it in any way, either incoming or outgoing. If the firewall deems the packets are suspicious, it will block it regardless of its precedence. Firewalls can be used to separate network nodes from external traffic sources, internal traffic sources, or even specific applications. 

Firewalls can be software, hardware, or cloud-based, with each type of firewall having its own unique pros and cons. I like the analogy of it being like the high wall around a castle, blocking out the invading hordes, whilst still allowing friendly citizens in and out through its drawbridge. The firewall basically does the same, with the slight difference it also blocks suspicious activity from going outside, like a burglar after a bank heist that gets stopped right before passing the border trying to escape to a different country.

Therefore, we can say the primary goal of a firewall is to block malicious traffic requests and data packets while allowing legitimate traffic through.

There are 2 kinds of firewalls, software-based and hardware-based firewalls.

### Software firewall vs Hardware firewall:

The main difference between the hardware firewall and the software firewall is that the hardware firewall is an actual physical device that will sit between your local area network and the internet. Whereas the software firewall will be installed on each individual device.
This means the hardware firewall can stop bad data from ever entering the network, while software firewalls can offer closer controls over specific devices and how they interact with the network and internet.

There are a few different techniques that firewalls use to filter data, which I will try and cover in this blog post.

## <a id="types"></a> Types of firewall:
Depending on their general structure and approach of operation, firewalls can be divided in the following categories:

- <a href="#filter">Packet-filtering firewalls</a>
- <a href="#gateways">Circuit-level gateways</a>
- <a href="#stateful">Stateful inspection firewalls</a>
- <a href="#app">Application-level gateways (a.k.a. proxy firewalls)</a>
- <a href="#next">Next-gen firewalls</a>

### <a id="filter"></a> Packet-filtering Firewalls:
With packet filtering, the firewall inspects each packet of data and compares it to pre-defined security rules (known as the firewall policy, or ruleset). If the packet is flagged by the rules, then it is prevented from passing through the firewall. This is the most basic and precursor of other firewall architectures. These firewalls aren't resource-intensive, so you can plant them in any system regardless of the requisites. The con is they are easy to circumvent, compared to more modern ones.

### <a id="gateways"></a> Circuit-level Gateways:

[The Techopedia](https://www.techopedia.com/definition/24780/circuit-level-gateway) defines circuit-level gateways as follows:

>A circuit-level gateway is a firewall that provides User Datagram Protocol (UDP) and Transmission Control Protocol (TCP) connection security and works between an Open Systems Interconnection (OSI) network model’s transport and application layers such as the session layer. Unlike application gateways, circuit-level gateways monitor TCP data packet handshaking and session fulfilment of firewall rules and policies.

These firewalls check by verifying the TCP handshake. They are very efficient regarding resources but they don't check the packet, therefore, if the packet contains malware and had the right handshake, it would go through regardless, so you would need to add another layer of security.


### <a id="stateful"></a> Stateful inspection Firewalls:
Using the 2 previous technologies, packet inspection and TCP handshake verification, we can use them and combine them together to conceive superior protection than the two of them alone.
These firewalls combine both packet inspection technology and TCP handshake verification to create a level of protection greater than either of the previous two architectures could provide alone.

The con with these firewalls is that they put more of a strain on computing resources as well. This may slow down the transfer of legitimate packets compared to the other solutions in exchange for better security.


### <a id="app"></a> Application-level Gateways (a.k.a. Proxy Firewalls / Cloud Firewalls):
As their name suggests, they operate at the application layer to filter the traffic sitting in between your network and the outer network that the packet comes from. They are managed via a cloud-based solution or another proxy device. The proxy firewall inspects the incoming data packets and then allows them through. This kind of Firewalls is very similar to the category explored above this one, but application layer gateways can also perform deep-layer packet inspections


### <a id="next"></a> Next-gen firewalls:
Although there isn't a general consensus about what a Next-Gen firewall is, the majority of the most recently-released products are branded after this etiquette. A next-gen Firewall must include:

- Standard firewall capabilities like stateful inspection
- Integrated intrusion prevention
- Application awareness and control to see and block risky apps
- Upgrade paths to include future information feeds
- Techniques to address evolving security threats

<p>&nbsp;</p>

## <a id="ids"></a> Intrusion Detection Systems:
They are devices or applications that like a Firewall, as seen above, monitor the network they are implemented in for suspicious activity. If they detect malicious activity or a policy violation, these are reported or collected centrally using a security information and event management system. The most common classifications of IDS are:

- Network intrusion detection systems (NIDS): 
Systems like firewalls that analyze incoming network traffic.

- Host-based intrusion detection systems (HIDS): 
Systems like antivirus software that monitor important operating system files.

<p>&nbsp;</p>

## <a id="control_access"></a> Access Control Lists:

We have an additional networking card up our sleeves we can play to trick bad actors: control access with lists. For this, we will need to put a proper strategy in action. First, we need to define what our perimeter is and then, what are our resources to defend it. This means making a choice between "accepting", "blocking" or something in between. Very commonly seen in every-day computing, like email spam blocking, this constantly happens behind the curtains and it's that efficient that the user doesn't even notice it. Although all of them have their pros and cons, in this section I will be covering whitelisting, blacklisting and greylisting:

### <a id="whitelisting"></a> Whitelisting:

Only those who are worthy of your trust will be the chosen ones to connect to your network. That's how whitelisting works, blocking by default all incoming requests if we don't trust the sender. This is an approach classified, to some extent, as *extremist*. Whilst very secure, it's not always very convenient, see the Apple App Store for example: if you own an iPhone, you will only be able to install software filtered and approved by Apple. This might be seen from some part of the society as a secure approach, but the reality is [this isn't a perfect approach](https://www.wired.com/story/apple-app-store-malware-click-fraud/). This approach is called trust-centric.

### <a id="blacklisting"></a> Blacklisting:

If we invert everything we say earlier with whitelisting, we get blacklisting. It's based on allowing every connection, but setting rules blocking only known bad actors that we are certain will try and cause harm. This is how antivirus software works: your machine gets a new binary, the antivirus runs it in a sandbox for you and determines whether it may stay or shall leave, by cross-referencing the binary with the database they have on digital signatures and program behaviour. To put it in a less technical example, tt works the same way with the Disclosure and Barring Service in the UK. If someone has put you on the DBS list and you want to access a job, your employer might ask DBS for checking your profile. If matched, you are likely going to be rejected as you appeared in the "blacklist". This approach is called threat-centric.

### <a id="greylisting"></a> Greylisting:

A middle approach between whitelisting and blacklisting can be achieved by greylisting or, basically, using both at the same time. Why not? Think about this: in this case, we can use as an analogy the invitation to a Discord server. Anybody clicking the link will join, but if someone starts misbehaving, will most likely get banned and will never be able to come back in - let's analyse this. At the time of the entrance to the server, it was using the blacklist approach: only reject those who are already blacklisted. Then the ban falls upon your IP address, the server now will behave to you as a whitelist: only those who are trusted will have access.

<p>&nbsp;</p>

## <a id="mitigation"></a> How to mitigate the damage after a data breach?

It's a very unfortunate event, but we have to face it: we are going to be breached. No system is hard or secure enough and it is only a matter of time, [see the recent SolarWinds breach](https://arstechnica.com/information-technology/2020/12/feds-warn-that-solarwinds-hackers-likely-used-other-ways-to-breach-networks/). We **need** to be prepared to have a resilience plan after we suffer a breach, because the sooner we realise it's going to eventually happen, the better prepared we will be for mitigating the damage. Here are some tips:

- Gather information and Report it in less than 72 hours: under the [GDPR](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/), we have 3 days since we are bridged to report it to our authorities. Reporting includes all details we can provide the authorities with, like how the breach happened, who is involved, what are our incident response measures and what data was affected.

- Notifying those whose data was breached: following the supply chain attack mentioned above, Microsoft has felt rushed to [notify tens of their own customers](https://www.theverge.com/2020/12/17/22188060/microsoft-president-solarwinds-orion-hack-breach-brad-smith) about the attack that breached SolarWinds. If the data breaches happened where the users had credentials, the usual word of advice is to immediately change all the passwords related to the email account leaked in the breach, as well as taking a look at their bank accounts in case they had stored credit or debit cards with the company, to dodge any harm the delinquents may try to cause them.

- Leaving the authority to investigate: when the authority has been alerted, they will supervise the breach to assess whether our systems were strong enough to deem if it could have been prevented or was, in the end, a highly sophisticated attack enough, like, let's say, an APT using 0-day exploits. In the other hand, if our company is suspected to be negligent with our lacking security, we might get fined being the top penalty under the GDPR either €20 million or 4% of the annual global turnover of the company.

<p>&nbsp;</p>

# <a id="refs"></a> References:

Anon., 2013. United Kingdom Government - "DBS checks: detailed guidance" [Online] \
[https://www.gov.uk/government/collections/dbs-checking-service-guidance--2](https://www.gov.uk/government/collections/dbs-checking-service-guidance--2) \
[Last accessed Dec 18th, 2020] 

Anon., 2020. ICO - "Guide to the General Data Protection Regulation (GDPR)" [Online] \
[https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/) \
[Last accessed Dec 18th, 2020]

Anon., 12/08/2019. Consolidated Technologies - "Blacklisting vs. Whitelisting" [Online] \
[https://consoltech.com/blog/blacklisting-vs-whitelisting/](https://consoltech.com/blog/blacklisting-vs-whitelisting/) \
[Last accessed Dec 18th, 2020]

Sophie Meunier, 18/10/2018. IT Governance - "3 ways you can mitigate the damage of a data breach" [Online] \
[https://www.itgovernance.eu/blog/en/3-ways-you-can-mitigate-the-damage-of-a-data-breach](https://www.itgovernance.eu/blog/en/3-ways-you-can-mitigate-the-damage-of-a-data-breach) \
[Last accessed Dec 18th, 2020] 

Jakob Como, 7/06/2013. Plummer Slade, inc - "How to keep server rooms/equipment from overheating" [Online] \
[https://www.plummerslade.com/2016/06/how-to-keep-server-roomsequipment-from-overheating/](https://www.plummerslade.com/2016/06/how-to-keep-server-roomsequipment-from-overheating/) \
[Last accessed Dec 18th, 2020]

0x5ubt13, 17/11/2020. Subtle Labs - "Firewalls and Intrusion Detection Systems" [Online] \
[https://0x5ubt13.github.io/blog/2020/11/17/firewalls.html](https://0x5ubt13.github.io/blog/2020/11/17/firewalls.html) \
[Last accessed Dec 18th, 2020]

Anon., 2020. Cisco - "What Is a Firewall?" [Online] \
[https://www.cisco.com/c/en_uk/products/security/firewalls/what-is-a-firewall.html](https://www.cisco.com/c/en_uk/products/security/firewalls/what-is-a-firewall.html) \
[Last accessed Nov 29th, 2020]

Eric Dosal, 26/11/2019. Compuquip - "What is a Firewall? The Different Firewall Types & Architectures" [Online] \
[https://www.compuquip.com/blog/the-different-types-of-firewall-architectures](https://www.compuquip.com/blog/the-different-types-of-firewall-architectures) \
[Last accessed Nov 29th, 2020]

Eric Dosal, 17/09/2019. Compuquip - "Which of the Following Are Characteristics of a Circuit-Level Gateway?" [Online] \
[https://www.compuquip.com/blog/characteristics-of-a-circuit-level-gateway](https://www.compuquip.com/blog/characteristics-of-a-circuit-level-gateway) \
[Last accessed Nov 29th, 2020]

Anon., 2020. Techopedia - "Circuit-Level Gateway" [Online] \
[https://www.techopedia.com/definition/24780/circuit-level-gateway](https://www.techopedia.com/definition/24780/circuit-level-gateway) \
[Last accessed Nov 29th, 2020]

Anon., 2020. Barracuda - "What is a Intrusion Detection System?" [Online] \
[https://www.barracuda.com/glossary/intrusion-detection-system](https://www.barracuda.com/glossary/intrusion-detection-system) \
[Last accessed Nov 29th, 2020]
















