---
title: "Vulnserver - Stack Based Buffer Overflow"
date: 2020-01-19 
header:  
excerpt: "Vulnserver - Stack Based Buffer Overflow"
---

##  Intro

Vulnserver is an intentionally vulnerable software that can be found from [thegreycorner's blog](http://www.thegreycorner.com/p/vulnserver.html) or alternatively from their [GitHub](https://github.com/stephenbradshaw/vulnserver) The software has multiple different Buffer Overflow vulnerabilities of which the Stack Based Buffer Overflow using the TRUN command is the simplest.
To practice exploiting the software we need the following:
* Vulnserver
* Mona.py can be found from [corelan's GitHub](https://github.com/corelan/mona)
* Windows 10 / 7 (I will be using Windows 10 x64)
* Immunity Debugger or whichever debugger you wish to use. Immunity can be downloaded from [immunityinc.com](https://www.immunityinc.com/products/debugger/)
* Kali Linux which can be found from [Offensive Security's downlaod page](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/)

## Exploiting the TRUN Command