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

### On VMware

1. Open VMware Workstation pro and click on "Create new Virtual Machine".

![vmware1](/images/vulnserver/stack/vmware1.PNG)
2. New window opens select "Typical" and click "next".
![vmware2](/images/vulnserver/stack/vmware2.PNG)

3. Next select the operating system ISO that you just downloaded and click "next".			 
![vmware3](/images/vulnserver/stack/vmware3.PNG)

4. Next give it some name, location can be left default and click "next".
![vmware4](/images/vulnserver/stack/vmware4.PNG)

5. Next choose the disk size, default is fine, make sure "Split virtual disk into multiple files" is selected by default since we donät want the VM to fill our file system instantly. Click "next".
![vmware5](/images/vulnserver/stack/vmware5.PNG)

6. Finally check that the settings are correct and click "Finish" to create the Virtual Machine.
![vmware6](/images/vulnserver/stack/vmware6.PNG)
### On VirtualBox