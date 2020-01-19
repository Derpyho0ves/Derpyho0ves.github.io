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

After creating the VMs, start up Windows VM and download vulnserver, immunity debugger & mona.

### Setting up Immunity Debugger

Download [Immunity Debugger](https://debugger.immunityinc.com/ID_register.py), before you can download the site asks you to fill the form, you can put random information on fields.

![immu1](/images/vulnserver/stack/immu1.PNG)

Install Immunity Debugger and when the installer asks to install python 2.7.1 allow the installation.
After the installation is complete download [mona](https://github.com/corelan/mona) and copy the mona.py over to the Pycommands folder inside Immunity Debugger. In this case **C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands**

![mona1](/images/vulnserver/stack/mona1.PNG)

You can now open Immunity Debugger by right clicking the shortcut and "run as administrator"

![immu2](/images/vulnserver/stack/immu2.PNG)

### Setting up Vulnserver

Download the Vulnserver software and extract it somewhere where it is easy to access such as the Desktop.

![vuln1](/images/vulnserver/stack/vuln1.PNG)

In the vulnserver folder there a few files, start the vulnserver.exe by right clicking it and "run as administrator"

![vuln2](/images/vulnserver/stack/vuln2.PNG)

You now have vulnserver up and running

![vuln3](/images/vulnserver/stack/vuln3.PNG)


## Exploiting TRUN Command

Now that we have basic setup done, we can start exploiting vulnserver via the TRUN command. First we can connect  to the
server using netcat, by default vulnserver listens on port 9999. Connect with **nc -v 192.168.209.132 9999** we can also list
the comamnds by issuing the comamnd **HELP** after connecting.

![exp1](/images/vulnserver/stack/exp1.PNG)