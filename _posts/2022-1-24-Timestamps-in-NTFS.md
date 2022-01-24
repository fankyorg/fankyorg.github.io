---
layout: post
title: Timestomp in NTFS
---
Today I want to write about timestomping in NTFS.

Let MITRE explain you why you should read this blog post:
> Adversaries may modify file time attributes to hide new or changes to existing files. Timestomping is a technique that modifies the timestamps of a file (the modify, access, create, and change times), often to mimic files that are in the same folder. This is done, for example, on files that have been modified or created by the adversary so that they do not appear conspicuous to forensic investigators or file analysis tools.
[Indicator Removal on Host: Timestomp](https://attack.mitre.org/techniques/T1070/006/)

For a better understanding of the article, you should be familiar with the following principles:
- MACB Timestamps
- Master File Table Record
- Standard Information Attribute
- File Name Attribute 
You can find an conclusion about this topics in my [public repo](). 

First of all, I created 3 files.
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/files1.png)



Please note:
In this case I did not follow the best practices. If you have a forensic investigating, you should always create an image of the original disk and verify with hashes, that nothing was modified. 
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/imaging-process.png)