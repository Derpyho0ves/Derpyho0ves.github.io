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

1. Open VMware Workstation pro and click on "Create new Virtual Machine"
![vmware1](/images/vulnserver/stack/vmware1.PNG)

2. New window opens select "Typical" and click "next"
![vmware2](/images/vulnserver/stack/vmware2.PNG)

3. Next select the operating system ISO that you just downloaded and click "next"
![vmware3](/images/vulnserver/stack/vmware3.PNG)

4. Next give it some name, location can be left default and click "next"
![vmware4](/images/vulnserver/stack/vmware4.PNG)

5. Next choose the disk size, default is fine, make sure "Split virtual disk into multiple files" is selected by default since we donät want the VM to fill our file system instantly. Click "next".
![vmware5](/images/vulnserver/stack/vmware5.PNG)

6. Finally check that the settings are correct and click "Finish" to create the Virtual Machine
![vmware6](/images/vulnserver/stack/vmware6.PNG)

Repeat the steps for the other Virtual Machine.
### On VirtualBox

1. Open VirtualBox and click on the blue icon named "new"
![vb1](/images/vulnserver/stack/vb1.PNG)

2. New window opens, give the Virtual Machine a name, choose the version and click "Next"
![vb2](/images/vulnserver/stack/vb2.PNG)

3. Next you can incrase RAM on the operating system, default is 2GB I'll give it 6GB to make it bit more responsive click "Next"
![vb3](/images/vulnserver/stack/vb3.PNG)

4. Next create the Hard Disk file, default is fine and click "Create"
![vb4](/images/vulnserver/stack/vb4.PNG)

5. Next choose the Hard Disk file type, default VDI is fine and click "Next"
![vb5](/images/vulnserver/stack/vb5.PNG)

6. Next choose the storage type, use "Dynamically allocated" and click "Next" 
![vb6](/images/vulnserver/stack/vb6.PNG)

7. Next choose the hard disk size and location, defaults are fine and click "Create"
![vb7](/images/vulnserver/stack/vb7.PNG)

8. After creating the VM, start it up and it may ask you to select the start-up disk, choose the ISO you wish to install.
![vb8](/images/vulnserver/stack/vb8.PNG)

Repeat the steps for the other Virtual Machine.

### Setting up Vulnserver

On your newly installed Wndows VM, download the Vulnserver software and extract it somewhere where it is easy to access such as the Desktop.
![vuln1](/images/vulnserver/stack/vuln1.PNG)

In the vulnserver folder there a few files, start the vulnserver.exe by right clicking it and "run as administrator"
![vuln2](/images/vulnserver/stack/vuln2.PNG)

You now have vulnserver up and running
![vuln2](/images/vulnserver/stack/vuln2.PNG)