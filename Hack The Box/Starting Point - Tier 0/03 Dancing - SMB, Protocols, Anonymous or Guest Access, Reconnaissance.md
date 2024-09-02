#smb #protocols #anonymous-access #reconnaissance 

# 0. Theory

There are multiple ways to transfer a file between two hosts (computers) on the same network. One of these protocols is **SMB** (**Server Message Block**). This communication protocol provides shared access to files, printers, and serial ports between endpoints on a network. We mostly see SMB services running on Windows machines.

During scanning, we will typically see port 445 TCP open on the target, reserved for the SMB protocol. Usually, SMB runs at the Application or Presentation layers of the OSI model. Due to this, it relies on lower-level protocols for transport. The Transport layer protocol that Microsoft SMB Protocol is most often used with is NetBIOS over TCP/IP (NBT). This is why, during scans, we will most likely see both protocols with open ports running on the target. We will see this during the enumeration phase of the writeup.
![[Pasted image 20240827112033.png]]

An SMB-enabled storage on the network is called a `share`.
Despite having the ability to secure access to the share, a network administrator can sometimes make mistakes and accidentally allow logins without any valid credentials or using either `guest accounts` or `anonymous log-ons` .


**What does the 3-letter acronym SMB stand for?**
*Server Message Block*

**What port does SMB use to operate at?**
*[TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol "Transmission Control Protocol") [port](https://en.wikipedia.org/wiki/Port_(computer_networking) "Port (computer networking)") 445*
	*(SMB relies on the [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol "Transmission Control Protocol") and [IP](https://en.wikipedia.org/wiki/Internet_Protocol "Internet Protocol") protocols for transport.)*

**What is the service name for port 445 that came up in our Nmap scan?**
*microsoft-ds*

**What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?**
*-L*

**What is the command we can use within the SMB shell to download the files we find?**
*get*

# 1. Recon
Verify the target by pinging it.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ ping 10.129.154.132 -c 5
	PING 10.129.154.132 (10.129.154.132) 56(84) bytes of data.
	64 bytes from 10.129.154.132: icmp_seq=1 ttl=127 time=8.24 ms
	64 bytes from 10.129.154.132: icmp_seq=2 ttl=127 time=8.24 ms
	64 bytes from 10.129.154.132: icmp_seq=3 ttl=127 time=8.24 ms
	64 bytes from 10.129.154.132: icmp_seq=4 ttl=127 time=8.83 ms
	64 bytes from 10.129.154.132: icmp_seq=5 ttl=127 time=8.34 ms
	
	--- 10.129.154.132 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4006ms
	rtt min/avg/max/mdev = 8.236/8.376/8.829/0.229 ms


# 2. Enumeration

Finding out what services are running on the target:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ sudo nmap -sV 10.129.1.12
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-27 08:35 CDT
	Nmap scan report for 10.129.1.12
	Host is up (0.0091s latency).
	Not shown: 997 closed tcp ports (reset)
	PORT    STATE SERVICE       VERSION
	135/tcp open  msrpc         Microsoft Windows RPC
	139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
	445/tcp open  microsoft-ds?
	Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds

We can see that port **445 TCP** for SMB is up and running, which means that we have an active share that we could potentially explore. In order to do so, we will need the appropriate services and scripts installed.

In order to successfully enumerate share content on the remote system, we can use a script called `smbclient`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ sudo apt-get install smbclient
	Reading package lists... Done
	Building dependency tree... Done
	Reading state information... Done
	smbclient is already the newest version (2:4.17.12+dfsg-0+deb12u1).
	The following packages were automatically installed and are no longer required:
	  espeak-ng-data geany-common libamd2 libbabl-0.1-0 libbrlapi0.8 libcamd2
	  libccolamd2 libcholmod3 libdotconf0 libept1.6.0 libespeak-ng1 libgegl-0.4-0
	  libgegl-common libgimp2.0 libmetis5 libmng1 libmypaint-1.5-1
	  libmypaint-common libpcaudio0 libsonic0 libspeechd2 libtorrent-rasterbar2.0
	  libumfpack5 libwmf-0.2-7 libwpe-1.0-1 libwpebackend-fdo-1.0-1 libxapian30
	  node-clipboard node-prismjs python3-brlapi python3-louis python3-pyatspi
	  python3-speechd sound-icons speech-dispatcher-audio-plugins xbrlapi xkbset
	Use 'sudo apt autoremove' to remove them.
	0 upgraded, 0 newly installed, 0 to remove and 156 not upgraded.

We can see from the output that we already had the latest version of `smbclient`.
The next step is to start enumerating the contents of the share found on our target in both cases.

Smbclient will attempt to connect to the remote host and check if there is any authentication required. If there is, it will ask you for a password for your local username. We should take note of this. If we do not specify a specific username to smbclient when attempting to connect to the remote host, it will just use your local machine's username. That is the one you are currently logged into your Virtual Machine with. This is because SMB authentication always requires a username, so by not giving it one explicitly to try to login with, it will just have to pass your current local username to avoid throwing an error with the protocol.
![[Pasted image 20240827154107.png]]

Let's use our local username since we do not know about any remote usernames present on the target host that we could potentially log in with. After that, we will be prompted for a password. This password is related to the username you input before. Hypothetically, if we were a legitimate remote user trying to log in to their resource, we would know our username and password and log in normally to access our share. In this case, we do not have such credentials, so what we will be trying to perform is any of the following:
- Guest authentication
- Anonymous authentication

Any of these will result in us logging in without knowing a proper username/password combination and seeing the files stored on the share. Let's try that. We leave the password field blank, simply hitting Enter to tell the script to move along.

***(There was a restart of the target machine at this point.)***

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ ping 10.129.154.189 -c 5
	PING 10.129.154.189 (10.129.154.189) 56(84) bytes of data.
	64 bytes from 10.129.154.189: icmp_seq=1 ttl=127 time=8.02 ms
	64 bytes from 10.129.154.189: icmp_seq=2 ttl=127 time=7.75 ms
	64 bytes from 10.129.154.189: icmp_seq=3 ttl=127 time=7.86 ms
	64 bytes from 10.129.154.189: icmp_seq=4 ttl=127 time=7.99 ms
	64 bytes from 10.129.154.189: icmp_seq=5 ttl=127 time=8.01 ms
	
	--- 10.129.154.189 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4006ms
	rtt min/avg/max/mdev = 7.745/7.924/8.017/0.106 ms
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ smbclient -L 10.129.154.189
	Password for [WORKGROUP\scrage]:
	
		Sharename       Type      Comment
		---------       ----      -------
		ADMIN$          Disk      Remote Admin
		C$              Disk      Default share
		IPC$            IPC       Remote IPC
		WorkShares      Disk      
	Reconnecting with SMB1 for workgroup listing.
	do_connect: Connection to 10.129.154.189 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
	Unable to connect with SMB1 -- no workgroup available

We know that we should use the -L flag by checking the script's help:

	[-L|--list=HOST]

We see that four separate shares are displayed. Let us go through each of them and see what they mean.
- `ADMIN$` - Administrative shares are hidden network shares created by the Windows NT family of operating systems that allow system administrators to have remote access to every disk volume on a network-connected system. These shares may not be permanently deleted but may be disabled.
- `C$` - Administrative share for the C:\ disk volume. This is where the operating system is hosted.
- `IPC$` - The inter-process communication share. Used for inter-process communication via named pipes and is not part of the file system.
- `WorkShares` - Custom share.

# 3. Foothold

We will try to connect to each of the shares except for the `IPC$` one, which is not valuable for us in this exercise since it is not browsable as any regular directory would be. Using the same tactic as before, attempting to log in without the proper credentials to find improperly configured permissions on any of these shares. We'll just give a blank password for each username to see if it works.

- For `ADMIN$` we don't have permission:
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ smbclient \\\\10.129.154.189\\ADMIN$
	Password for [WORKGROUP\scrage]:
	tree connect failed: NT_STATUS_ACCESS_DENIED
	
- For `C$` we don't have permission:
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ smbclient \\\\10.129.154.189\\C$
	Password for [WORKGROUP\scrage]:
	tree connect failed: NT_STATUS_ACCESS_DENIED
	
- But we can access WorkShares as guest user:
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ smbclient \\\\10.129.154.189\\WorkShares
	Password for [WORKGROUP\scrage]:
	Try "help" to get a list of possible commands.
	smb: \> 

Any folders that are human-made, are prone to misconfiguration. And we've just exploited that.

We can use the help command to see what we can do within this shell.

	smb: \> help
	?              allinfo        altname        archive        backup       
	blocksize      cancel         case_sensitive cd             chmod        
	chown          close          del            deltree        dir          
	du             echo           exit           get            getfacl      
	geteas         hardlink       help           history        iosize       
	lcd            link           lock           lowercase      ls           
	l              mask           md             mget           mkdir        
	more           mput           newer          notify         open         
	posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir  
	posix_unlink   posix_whoami   print          prompt         put
	pwd            q              queue          quit           readlink
	rd             recurse        reget          rename         reput
	rm             rmdir          showacls       setea          setmode
	scopy          stat           symlink        tar            tarmode
	timeout        translate      unlock         volume         vuid         
	wdel           logon          listconnect    showconnect    tcon
	tdis           tid            utimes         logoff         ..
	!
	smb: \> dir
	  .                                   D        0  Mon Mar 29 03:22:01 2021
	  ..                                  D        0  Mon Mar 29 03:22:01 2021
	  Amy.J                               D        0  Mon Mar 29 04:08:24 2021
	  James.P                             D        0  Thu Jun  3 03:38:03 2021
			5114111 blocks of size 4096. 1750591 blocks available

We'll use these commands:
`ls : listing contents of the directories within the share
`cd : changing current directories within the share`
`get : downloading the contents of the directories within the share`
`exit : exiting the smb shell`

When we look around in the two directories we just found, we can see the following files:

	smb: \> cd Amy.J\
	smb: \Amy.J\> ls
	  .                                   D        0  Mon Mar 29 04:08:24 2021
	  ..                                  D        0  Mon Mar 29 04:08:24 2021
	  worknotes.txt                       A       94  Fri Mar 26 06:00:37 2021
			5114111 blocks of size 4096. 1750590 blocks available
	smb: \Amy.J\> cd ..\James.P\
	smb: \James.P\> ls
	  .                                   D        0  Thu Jun  3 03:38:03 2021
	  ..                                  D        0  Thu Jun  3 03:38:03 2021
	  flag.txt                            A       32  Mon Mar 29 04:26:57 2021
	
			5114111 blocks of size 4096. 1750559 blocks available

We can download `flag.txt` to our machine. The `get` command will download it to the folder where we are running the SMB client from locally.

	smb: \> get Amy.J\worknotes.txt 
	getting file \Amy.J\worknotes.txt of size 94 as Amy.J\worknotes.txt (1.8 KiloBytes/sec) (average 1.3 KiloBytes/sec)
	smb: \> exit
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ ls
	'Amy.J\worknotes.txt'   Desktop     Downloads   Music     Pictures   Templates
	 cacert.der             Documents   flag.txt    my_data   Public     Videos
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ cat Amy.J\\worknotes.txt 
	- start apache server on the linux machine
	- secure the ftp server
	- setup winrm on dancing ┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v2cvbixxdl]─[~]
	└──╼ [★]$ cat flag.txt 
	5f61c10dffbc77a704d76016a22f1664


https://www.hackthebox.com/achievement/machine/2057492/395
![[Pasted image 20240827161330.png]]