---
layout: post
title: DNS Exfiltration Part1
---

A few weeks ago, I received a file with recorded network traffic from a teacher, with the hint that a password was hidden in this file. As it turned out, the password was sent via the DNS protocol (decoded). Even though I had already heard about DNS exfiltration, I was not able to find the password. That was very annoying. So that this does hopefully not happen to me again, I started to look deeper into the topic DNS exfiltration. 

## Introduction
As often, as a good guy in IT security, you must first understand how the bad guys act. Therefore, I recommend reading the following article:

- [Exfiltration](https://attack.mitre.org/tactics/TA0010/) - MITRE (tactic)
- [Automated Exfiltration](https://attack.mitre.org/techniques/T1020/) - MITRE (technique)
- [Application Layer Protocol: DNS](https://attack.mitre.org/techniques/T1071/004/) - MITRE (technique)
- [Exfiltration Over Alternative Protocol](https://attack.mitre.org/techniques/T1048/) - MITRE  (technique)
- [Introduction to DNS Data Exfiltration](https://www.akamai.com/blog/news/introduction-to-dns-data-exfiltration) - Akamai

After reading these articles (especially the one from akamai), I felt I had a general idea of DNS exfiltration. However, since this was not enough for me, I decided to rebuild the whole thing in my LAB. 

## LAB setup and exfiltration
 To have the full control over the data stream, I decided using only DNS servers inside my (segmented) network. My planned setup should at the end look like this. 

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/laboverview.png) 

 First, I created an A record (capturesdns.exfil.fanky.org), which points to my Ubuntu DNS. After that, I created a new zone (exfil.fanky.org) on my Microsoft DNS Server (which is the primary DNS of my client). At the end, I created a delegated zone (data.exfil.fanky.org).
 
 *Note: The fact that I do not use an external DNS does not affect the functionality. The behavior would be exactly the same even with an external DNS server.*

 Here you can see an nslookup on my client:

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/nslookup.png)

*I receive the information from my primary DNS server (10.20.50.10), that I can find the dns zone data.exfil.fanky.org on capturedns.exfil.fanky.org with the address 10.20.48.10 (Ubuntu DNS).*

With this delegation every DNS query to data.exfil.fanky.org is send to my Ubuntu server. 

And that is the crucial point of DNS exfiltration. If I want to send data outside a protected network. I can abuse DNS queries. Even If I have blocked DNS to the internet on my clients. I just need to initiate a domain name resolution request. It doesn’t matter how I sent the data. I can do that for example via browser request, nslookup, curl, or, or. Its only important that I trigger a dns request to something.data.exfil.fanky.org. 

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/data-nslookup-curl.png)

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/data-browser.png)

On my ubuntu server I see the following:

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/ubuntu-capture.png)

*Note: attackers would "hide" the sent data by using encoding like Base64. Then you are also able to send spaces, commas, dots and so on*

The firewall logs shows that there was no direct communication between the client (10.20.49.103) and the Ubuntu DNS server (10.20.48.10). The hole data was send via my "good" dns servver (10.20.50.10).

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration1/fw-view.png)

**Credits**

To configure my Ubuntu DNS server, I used [this article](https://hinty.io/devforth/dns-exfiltration-of-data-step-by-step-simple-guide/). If you want to build in your own LAB with a DNS server which is reachable via internet, then I highly recommend following this article. 

Thats it for today. The topic of part 2 will be how to discover and prevent data exfiltartion via DNS. Thanks for reading!