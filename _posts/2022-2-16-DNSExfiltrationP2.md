---
layout: post
title: DNS Exfiltration Part2
---

In my [last blogpost](https://fanky.org/DNSExfiltrationP1/) I wrote about the data flow of DNS data exfiltration. This blogpost is about **detection** and **prevention**. Even if the specific prevention is not so simple. But also for this blogpost, a certain know-how is assumed. Also this time I will not go into detail about how DNS works. But don't worry, there are already many good articles on the internet. As always, I have linked all the pages that I found helpful. 

## Introduction
As already mentioned, its recommended to understand the basics of how DNS works. During my research work I found this great [article](https://www.plixer.com/blog/overview-of-dns-protocol-part-1-of-3/). Highly recommended to everyone who is not familiar with the DNS basics or to everyone like me, who needed a refresher. I can also recommend you wachting this two videos:

- [DNS Tunneling Identification and Defense](https://www.youtube.com/watch?v=CaFo83TlpPM&t=3s&ab_channel=TomOlzak) or [here](https://bit.ly/33aYOyK) if you prefere a PDF document. 
- [DNS tunneling down the rabbit hole](https://www.youtube.com/watch?v=ibRVf3NagBI&ab_channel=CarolinaConVideos)

Further on, there is a great [SANS paper](https://www.giac.org/paper/gcia/1116/detecting-dns-tunneling/108367) which was very helpful for me, although it is already almost 10 years old. 

*Even if the chance is small that ever someone of these people will read this blog here: thank you very much! These videos and documents helped me a lot with my studies!*

## Prevention
DNS firewalls are one of the best options to defense DNS exfiltration. Honestly, before I started to have a look at DNS exfiltration, I had never heart about DNS firewalls. So I did again some research and found a great [Infoblox article](https://blogs.infoblox.com/security/do-i-need-both-dns-firewall-and-next-generation-firewall/) which I recommend to everyone. For those of you who don't want to read this article, a very short summary of what a DNS firewall does: DNS firewalls uses response policy zone (RPZ). RPZ is a kind of a filtering mechanism. Domains which are already known as harmful are not allowed to communicate with. E.G if fank.org is already known as harmful and a client asks your recursive dns server who is \*.fanky.org (or just fanky.org), is blocked. RPZ are regularly updated (at least daily) by the threat intelligence organizations that create and manage them. *Please note that this is a simplification of DNS firewalls*

An other important step is hardening user devices. There are several ways to archive hardened devices. Some of them are:

- Application allow list or simply blocking users from installing (untrusted) software on their devices
- Removal of local admin access from users
- Host-based firewall
- Host-based IDS
- Network segmentation
- Endpoint protection 
- **User awareness** I know this is not a device hardening tip. But it is still very important one. 

## Detection 

As RPZ leaves gaps open, E.G when a bad domain is not known as harmful, a DNS firewall is not enough. We also need detection methods.

Luckily for use as good guys, attackers often don’t try to be stealthy while sending data over DNS. They are relying on the fact that DNS is often not monitored. As I learned during my research, the detection techniques will be separated in the two categories **Payload analyses** and **traffic analysis**. For payload analysis the DNS payload for one or more requests and responses will be analyzed for exfiltration indicators. For traffic analysis the traffic will be analyzed over time. The count, frequency and other request attributes will be considered. 

The hole description of the detection techniques can be found in the SANS paper. The following is a brief summary.

### Payload Analysis

- **Size of request and response** No exact number can be said, but URLs longer than 50 characters are uncommon. This is also impacted by [Search Engine Optimization](https://seopressor.com/blog/url-structure-affect-seo/). Consider alerting on querries over 50 characters long.
- **Entropy of hotsnames** Usually, DNS names have often dictionary words (E.G data.exfil.fanky.org). DNS exfiltration uses often encoded names. This also so, that certain special characters can be sent or received. (E.G Base64 JHQwbGVuUGEkJHcwcmQh for $t0lenPa$$w0rd!) 
- **Statistical analysis** There are specific characters which allows you to identify DNS traffic as malicious. Its well described in [this](http://www.syssec-project.eu/m/page-media/3/bilge-ndss11.pdf) document. 
- **Uncommon record types** Clients usually don't ask for TXT records. If clients are requesting for TXT records, that could by an indicator for [C2](https://csrc.nist.gov/glossary/term/command_and_control) communication.
- **Specific signatures** Some exfiltration or DNS tunneling can be recognized because of specific signatures. With these signatures we can E.G create snort rules to detect malicious traffic. Several sources also recommended using [N-gram](https://www.researchgate.net/publication/330843380_Malicious_Domain_Names_Detection_Algorithm_Based_on_N_-Gram). But I’m not sure yet if I can implement that in my detection rules. 

### Traffic Analysis

- **Volume of DNS traffic per IP client** If a client sends a large number of requests, then you should have a look at this. In an ideal world, you have also a baseline from a healthy client. Then you can compare the two traffic analysis. 
- **Volume of DNS traffic per domain** The average DNS [TTL](https://en.wikipedia.org/wiki/Time_to_live#DNS_records) of the [top500 moz]( https://moz.com/top500) is around 5000. If a client starts asking multiple times for a different subdomain, this could be an indicator of compromise. With [this script]( https://gist.github.com/mbuckbee/79b2e76bd9271bea38487defd8a9138b) you can check the average TTL by yourself. This method works worse when the attackers use multiple domains
- **Number of hotsnames per domain** The attackers don't want to sent each time the same data. Therefore, every request are unique. 
- **Volume of NXDomain responses** Looking for excessive [NXDomain](https://www.dnsknowledge.com/whatis/nxdomain-non-existent-domain-2/) responses. 
- **Orphan DNS requests** Normally, a DNS request is followed by an http request or some other type of communication to the requested page. If you see DNS requests, but no further communication afterwards, this can be another indicator of misuse of the DNS protocol. 

## Conclusion

![_config.yml]({{ site.baseurl }}/images/DNS-Exfiltration2/meme.jpg)

Unfortunately, there is no magic solution that solves all our problems realting to DNS exfiltration. The use of Response policy zone (RPZ) is a first step to block already known domain names. However, every environment should be proactively checked for DNS exfiltration on a regular basis. 

I hope you enjoyed my blogpost and you learned something. At least, I did! I can just recommend you reading and watching all linked articles and videos. But most important build your own lab and do there some DNS exfiltration. I have learned a lot this way. Stay tuned until my next blogpost where I will try to detect DNS exfiltration with Microsoft Sentinel and Snort rules. If you have any feedback, contact me on [twitter](https://twitter.com/FankyOrg)! 