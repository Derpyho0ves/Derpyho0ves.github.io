---
title: "Vulnserver - Stack Based Buffer Overflow"
date: 2020-01-19 
header:  
excerpt: "Vulnserver - Stack Based Buffer Overflow"
---

##  Intro

Vulnserver is an intentionally vulnerable software that can be found from [thegreycorner's blog](http://www.thegreycorner.com/p/vulnserver.html) or alternatively on their [GitHub](https://github.com/stephenbradshaw/vulnserver).

The software has multiple different Buffer Overflow vulnerabilities of which the Stack Based Buffer Overflow using the TRUN command is the simplest.

To practice exploiting the software we need the following
* [Immunity Debugger](https://www.immunityinc.com/products/debugger/)
* [Kali](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/)
* [Mona.py](https://github.com/corelan/mona)
* Vulnserver
* Windows 7 / 10 (I will be using Windows 10 x64)

## Setup

I will be using [VMware Workstation pro](https://my.vmware.com/en/web/vmware/info/slug/desktop_end_user_computing/vmware_workstation_pro/15_0), you could alternatively use [VirtualBox](https://www.virtualbox.org/wiki/Downloads) which is free. Select the version  based on your operating system e.g. Windows.
After downloading the preferred Virtualization platform we need to create the two Virtual Machines Windows & Kali.



### Setting up Vulnserver

On your newly installed Wndows VM, download the Vulnserver software and extract it somewhere where it is easy to access such as the Desktop.

![vuln1](/images/vulnserver/stack/vuln1.PNG)

In the vulnserver folder there a few files, start the vulnserver.exe by right clicking it and "run as administrator"

![vuln2](/images/vulnserver/stack/vuln2.PNG)

You now have vulnserver up and running

![vuln3](/images/vulnserver/stack/vuln3.PNG)
