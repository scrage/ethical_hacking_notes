#programming #weak-credentials #rdp #reconnaissance 

# 0. Theory

Remote access software represents a legitimate way to connect to other hosts to perform actions or offer support. These tools use the same protocol at their base to communicate with the other hosts, which is `RDP` . **RDP** (**Remote Desktop Protocol**) operates on ports `3389 TCP` and `3389 UDP` . The only difference consists of how the information relayed by this protocol is presented to the end-user.

## CLI - Remote Access Tools
Command Line Interface-based Remote Access Tools have been around forever. A rudimentary example of this is `Telnet`. Telnet is considered insecure due to lacking the ability to encrypt the data being sent through it securely. This implies that an attacker with access to a network **TAP** (**Traffic Access Point**) could easily intercept the packets being sent through a Telnet connection and read the contents, be they login credentials, sensitive files, or anything else. Telnet, which runs on **port 23 TCP** by default, has mainly been replaced by its more secure counterpart, `SSH` , running on **port 22 TCP** by default.

**SSH** (**Secure Shell Protocol**) adds the required layers of authentication and encryption to the communication model, making it a much more viable approach to perform remote access and remote file transfers. It is used both for patch delivery, file transfers, log transfer, and remote management in today's environment.
![[Pasted image 20240827214955.png]]

SSH uses public-key cryptography to verify the remote host's identity, and the communication model is based on the `Client-Server architecture`, as seen previously with FTP, SMB, and other services. The local host uses the server's public key to verify its identity before establishing the encrypted tunnel connection. Once the tunnel is established, symmetric encryption methods and hashing algorithms are used to ensure the confidentiality and integrity of the data being sent over the tunnel.
![[Pasted image 20240827215212.png]]

However, both Telnet and SSH only offer the end-user access to the remote terminal part of the host being reached. This means that no display projection comes with these tools. In order to be able to see the remote host's display, one can resort to CLI-based tools such as `xfreerdp` . Tools such as this one are called `Remote Desktop Tools` , despite being part of the `Remote Access` family.

## GUI - Remote Access Tools
Graphical User Interface-based Remote Access Tools are newer to the family than the CLI-based ones. Albeit old themselves, the technology kept evolving, making user interaction with both the software and the remote host easier and more intuitive for the average user. This type of tool only allows for Remote Desktop connections.
Two of the most prevalent Remote Desktop tools are Teamviewer and Windows' Remote Desktop Connection, formerly known as Terminal Services Client.

Simpler software that runs natively can sometimes be misconfigured because user experience for the inexperienced user is not always a key factor when developing technical tools integrated with some operating systems. Take Windows' Remote Desktop Connection, for example. Microsoft Remote Desktop Connection runs natively on Windows, which means that it comes pre-installed with every Windows OS as a service, without the need from the end-user to perform any other actions other than activating the service and setting its parameters up. This is where some configuration errors come in, presenting us with cases such as the one below.


**What does the 3-letter acronym RDP stand for?**
*Remote Desktop Protocol*

**What is a 3-letter acronym that refers to interaction with the host through a command line interface?**
*CLI*

**What about graphical user interface interactions?**
*GUI*

**What is the name of an old remote access tool that came without encryption by default and listens on TCP port 23?**
*Telnet*

**What is the name of the service running on port 3389 TCP?**
*ms-wbt-server*

**What is the switch used to specify the target host's IP address when using xfreerdp?**
*/v:*


# 1. Recon

Verify target and its availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-4mztbwi7dn]─[~]
	└──╼ [★]$ ping 10.129.1.13 -c 5
	PING 10.129.1.13 (10.129.1.13) 56(84) bytes of data.
	64 bytes from 10.129.1.13: icmp_seq=1 ttl=127 time=145 ms
	64 bytes from 10.129.1.13: icmp_seq=2 ttl=127 time=146 ms
	64 bytes from 10.129.1.13: icmp_seq=3 ttl=127 time=145 ms
	64 bytes from 10.129.1.13: icmp_seq=4 ttl=127 time=145 ms
	64 bytes from 10.129.1.13: icmp_seq=5 ttl=127 time=145 ms
	
	--- 10.129.1.13 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4003ms
	rtt min/avg/max/mdev = 144.893/145.182/146.095/0.457 ms


# 2. Enumeration

We start with a usual `nmap` scan with the version scanning switch enabled to determine the exact versions of all the services running on open ports on the target.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-4mztbwi7dn]─[~]
	└──╼ [★]$ nmap -sV 10.129.1.13
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-27 15:08 CDT
	Nmap scan report for 10.129.1.13
	Host is up (0.15s latency).
	Not shown: 996 closed tcp ports (reset)
	PORT     STATE SERVICE       VERSION
	135/tcp  open  msrpc         Microsoft Windows RPC
	139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
	445/tcp  open  microsoft-ds?
	3389/tcp open  ms-wbt-server Microsoft Terminal Services
	Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 17.40 seconds

It is always a good idea to research the ports found in order to understand the big picture. **SpeedGuide** is a good resource for those just starting out with their networking basics and interested in understanding more common ports at a glance. Below are some examples:

	Port 135 TCP : https://www.speedguide.net/port.php?port=135
	Port 139 TCP : https://www.speedguide.net/port.php?port=139
	Port 445 TCP : https://www.speedguide.net/port.php?port=445
	Port 3389 TCP : https://www.speedguide.net/port.php?port=3389

Looking at the SpeedGuide entry for port 3389 TCP, we deem it of interest. It is typically used for Windows Remote Desktop and Remote Assistance connections (over RDP - Remote Desktop Protocol). We can quickly check for any misconfigurations in access control by attempting to connect to this readily available port without any valid credentials, thus confirming whether the service allows guest or anonymous connections or not.

# 3. Foothold\
xfreerdp will be used to connect from the HBT Parrot Security virtual machine. We can check if we have xfreerdp installed by typing the command name in the terminal.
If it needs to be installed, we just run the following command: `sudo apt-get install freerdp2-x11`.

We can first try to form an RDP session with the target by not providing any additional information for any switches other than the target IP address. This will make the script use our own username as the login username for the RDP session, thus testing guest login capabilities.

	/v:{target_IP} : Specifies the target IP of the host we would like to connect to.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-4mztbwi7dn]─[~]
	└──╼ [★]$ xfreerdp /v:10.129.1.13
	[15:17:28:375] [57571:57572] [INFO][com.freerdp.client.x11] - No user name set. - Using login name: scrage
	[15:17:28:985] [57571:57572] [INFO][com.freerdp.crypto] - creating directory /home/scrage/.config/freerdp
	[15:17:28:986] [57571:57572] [INFO][com.freerdp.crypto] - creating directory [/home/scrage/.config/freerdp/certs]
	[15:17:28:986] [57571:57572] [INFO][com.freerdp.crypto] - created directory [/home/scrage/.config/freerdp/server]
	[15:17:29:308] [57571:57572] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
	[15:17:29:308] [57571:57572] [WARN][com.freerdp.crypto] - CN = Explosion
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - The hostname used for this connection (10.129.1.13:3389) 
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - does not match the name given in the certificate:
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - Common Name (CN):
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - 	Explosion
	[15:17:29:309] [57571:57572] [ERROR][com.freerdp.crypto] - A valid certificate for the wrong name should NOT be trusted!
	Certificate details for 10.129.1.13:3389 (RDP-Server):
		Common Name: Explosion
		Subject:     CN = Explosion
		Issuer:      CN = Explosion
		Thumbprint:  f6:09:51:a0:da:ca:46:20:94:5d:03:e2:80:80:01:b7:bd:d6:66:8a:e7:c7:67:9f:de:bd:d8:d6:20:61:3b:60
	The above X.509 certificate could not be verified, possibly because you do not have
	the CA certificate in your certificate store, or the certificate has expired.
	Please look at the OpenSSL documentation on how to add a private CA to the store.
	Do you trust the above certificate? (Y/T/N) Y
	Domain:   
	Password: 
	[15:17:56:079] [57571:57572] [ERROR][com.freerdp.core] - transport_ssl_cb:freerdp_set_last_error_ex ERRCONNECT_PASSWORD_CERTAINLY_EXPIRED [0x0002000F]
	[15:17:56:079] [57571:57572] [ERROR][com.freerdp.core.transport] - BIO_read returned an error: error:0A000438:SSL routines::tlsv1 alert internal error

Our own username is not accepted for the RDP session login mechanism. We can try a myriad of other default accounts, such as `user`, `admin`, `Administrator`, and so on. In reality, this would be a time-consuming process. However, for the sake of RDP exploration, let's attempt logging in with the `Administrator` user, as seen from the commands below.
We will also be specifying to the script that we would like to bypass all requirements for a security certificate so that our own script does not request them. The target, in this case, already does not expect any. Let's take a look at the switches we will need to use with xfreerdp in order to connect to our target in this scenario successfully:

	/cert:ignore : Specifies to the scrips that all security certificate usage should be ignored.
	/u:Administrator : Specifies the login username to be "Administrator".
	/v:{target_IP} : Specifies the target IP of the host we would like to connect to.

Upon trying the following command and just hitting Enter on a blank password input, we are greeted with the following GUI remote desktop:

`xfreerdp /v:10.129.1.13 /cert:ignore /u:Administrator`

![[Pasted image 20240827222305.png]]

And upon opening the file `flag.txt` on the desktop we can read the flag hash:
`951fa96d7830c451b536be5a6be008a0`


https://www.hackthebox.com/achievement/machine/2057492/396
![[Pasted image 20240827223028.png]]