---
layout: post
title: Timestomp in NTFS
---
Today I want to write about timestomping in NTFS and how to investigate them in a forensic way.

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
- test.txt
- test - Copy.txt (nomen est omen - this file is a copy of test.txt)
- test2.txt
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/files1.png)
*Note, the creation time of Test - Copy.txt is when the file was copied. The last write attribute was copied from test.txt*
    get-item -Path *  | Ft FullName,CreationTime,CreationTimeUtc,LastWriteTime,LastWriteTimeUtc

As a next step, lets change the CreationTime and LastWriteTime. I do that via [PowerShell](https://www.ghacks.net/2017/10/09/how-to-edit-timestamps-with-windows-powershell/). But there are a lot of tools which you can also you for that. E.G [Meatasploit](https://www.offensive-security.com/metasploit-unleashed/timestomp/)
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/files2.png)
    (Get-Item "E:\test.txt").LastWriteTime=("12.09.2021 11:22:33")
    (Get-Item "E:\test.txt").CreationTime=("12.09.1848 11:22:33")

Lets check again how the time stamps of the files looks like
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/files3.png)

Hmmm, it seems, that the file explorer does not aaccept such an old date. Lets quickly change it to  a reasonable date
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/files4.png)

We know that these timestamps were manipulated. Its also easy to recognize, because of the obviously faked timestamps. But attackers aren’t dump and my files should just illustrate how it works. In the real-world, attackers would use a more realistic date. E.G they use the same timestamp as other files in the same folder / partition. The next step will show how we can recognize modified timestamps. But first a little excursion. If we would do a forensic investigating, we should always create an image of the original disk and verify with hashes, that nothing was modified. You can see the process here
![_config.yml]({{ site.baseurl }}/images/timestaps-in-NTFS/imaging-process.png)

