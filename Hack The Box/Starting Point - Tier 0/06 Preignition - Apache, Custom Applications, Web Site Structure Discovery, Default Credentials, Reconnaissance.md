#apache #custom-applications #reconnaissance #web-site-structure-discovery #default-credentials

# 0. Theory

Web servers can sometimes be used strictly internally by employees, but most of the time can be found to be public-facing, meaning anyone from the Internet can access them to retrieve information and files from their hosted web pages. For the most part, the web pages hosted on the web servers are managed through their administrative panels, locked behind a log-in page.

Let's consider the following example:
We have decided to start your own blog and use WordPress to achieve this. Once installed, your WordPress website will have a public-facing side and a private-facing one, the latter being your administrative panel hosted at www.yourwebsite.com/wp-admin . This page is locked behind a log-in screen.

Once we, as an administrators of the WordPress site, log into its' admin panel, we will have access to a myriad of controls, ranging from content uploading mechanisms, to theme selection, custom script editing for specific pages, and more. The more we learn about WordPress, the more we can see how this is a vital part of a successful pentest, as some of these mechanisms could be outdated and come with critical flaws that would allow an attacker to gain a foothold and subsequently pivot through the network with ease.

Thus, we conclude that Web enumeration, specifically **directory busting** (**dir busting**), is one of the most essential skills any Penetration Tester must possess. While manually navigating websites and clicking all the available links may reveal some data, most of the links and pages may not be published to the public and, hence, are less secure. Suppose we did not know the `wp-admin` page is the administrative section of the WordPress site we exemplified above. How else would we have found it out if not for web enumeration and directory busting?


**Directory Brute-forcing is a technique used to check a lot of paths on a web server to find hidden pages. Which is another name for this?**
*dir busting*

What switch do we use for nmap's scan to specify that we want to perform version detection
`-sV`

**What does Nmap report is the service identified as running on port 80/tcp?**
*http*

**What server name and version of service is running on port 80/tcp?**
*nginx 1.14.2*

**What switch do we use to specify to Gobuster we want to perform dir busting specifically?**
*dir*

**When using gobuster to dir bust, what switch do we add to make sure it finds PHP pages?**
`-x php`


# 1. Recon

Verify connectivity to the target by using `ping`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ ping 10.129.155.91
	PING 10.129.155.91 (10.129.155.91) 56(84) bytes of data.
	64 bytes from 10.129.155.91: icmp_seq=1 ttl=63 time=8.11 ms
	64 bytes from 10.129.155.91: icmp_seq=2 ttl=63 time=8.05 ms
	64 bytes from 10.129.155.91: icmp_seq=3 ttl=63 time=8.16 ms
	^C
	--- 10.129.155.91 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2002ms
	rtt min/avg/max/mdev = 8.052/8.108/8.163/0.045 ms

# 2. Enumeration

Scan available ports using `nmap` with the service version discovery switch.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ nmap -sV 10.129.155.91
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-27 16:15 CDT
	Nmap scan report for 10.129.155.91
	Host is up (0.0094s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	80/tcp open  http    nginx 1.14.2
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 6.75 seconds

There is an http service running on port 80, signaling that this target might be hosting some explorable web content. To look at the contents ourselves, we can open a web browser of our choice and navigate to the target's IP address in the URL bar at the top of the window. This will automatically address the target's port 80 for the client-server communication and load the web page's contents.

`http://10.129.155.91:80`

![[Pasted image 20240827231935.png]]

At the top of the page, we observe the mention of the nginx service. After researching basic information about nginx and its purpose, we conclude that our target is a web server. Web servers are hosts on the target network which have the sole purpose of serving web content internal or external users.

What we are looking at on our browser screen is the default post-installation page for the nginx service, meaning that there is the possibility that this web application might not be adequately configured yet, or that default credentials are used to facilitate faster configuration up to the point of live deployment. This, however, also means that there are no buttons or links on the web page to assist us with navigation between web directories or other content.

When browsing a regular web page, we use these elements to move around on the website. However, these elements are only links to other directories containing other web pages, which get loaded in our browser as if we manually navigated to them using the URL search bar at the top of the browser screen. Knowing this, could we attempt to find any "hidden" content hosted on this webserver?

The short answer is yes, but to avoid guessing URLs manually through the browser's search bar, we can find a better solution. This method is called dir busting, short for directory busting. For this purpose, we will be using the tool called `gobuster` , which is written in Go.

## Installing gobuster
First, we need to make sure you have Go installed on our Linux distribution, which is the programming language used to write the gobuster tool. Once all the dependencies are satisfied for Go, you can proceed to download and install gobuster. In order to install Go, you need to input the following command in our terminal window:

	sudo apt install golang-go

Once that installation is complete, we can proceed with installing gobuster. If we have a Go environment ready to go (at least go 1.16), it is as easy as typing in the following command in our terminal:

	go install github.com/OJ/gobuster/v3@latest

In case this fails, we can always compile the tool from its' source code by running the following commands:

	sudo git clone https://github.com/OJ/gobuster.git
	cd gobuster
	go get && go build
	go install

## Using gobuster
By looking at the tool's help page, by typing in the `gobuster -h` command in our terminal, we receive a list of all possible switches for the tool and their description.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ gobuster -h
	Usage:
	  gobuster [command]
	
	Available Commands:
	  completion  Generate the autocompletion script for the specified shell
	  dir         Uses directory/file enumeration mode
	  dns         Uses DNS subdomain enumeration mode
	  fuzz        Uses fuzzing mode. Replaces the keyword FUZZ in the URL, Headers and the request body
	  gcs         Uses gcs bucket enumeration mode
	  help        Help about any command
	  s3          Uses aws bucket enumeration mode
	  tftp        Uses TFTP enumeration mode
	  version     shows the current version
	  vhost       Uses VHOST enumeration mode (you most probably want to use the IP address as the URL parameter)
	
	Flags:
	      --debug                 Enable debug output
	      --delay duration        Time each thread waits between requests (e.g. 1500ms)
	  -h, --help                  help for gobuster
	      --no-color              Disable color output
	      --no-error              Don't display errors
	  -z, --no-progress           Don't display progress
	  -o, --output string         Output file to write results to (defaults to stdout)
	  -p, --pattern string        File containing replacement patterns
	  -q, --quiet                 Don't print the banner and other noise
	  -t, --threads int           Number of concurrent threads (default 10)
	  -v, --verbose               Verbose output (errors)
	  -w, --wordlist string       Path to the wordlist. Set to - to use STDIN.
	      --wordlist-offset int   Resume from a given position in the wordlist (defaults to 0)
	
	Use "gobuster [command] --help" for more information about a command.

In our case, we will only need to use the following:

	dir : specify we are using the directory busting mode of the tool
	-w : specify a wordlist, a collection of common directory names that are typically used for sites
	-u : specify the target's IP address

There is a list of various helpful files, thematically ordered and placed in different directories under the /usr/share/ folder, that come preinstalled with every Kali linux distribution, and on some other linux distros, such as the HTB Parrot box we are using here.

A group of such helpful files are so-called wordlists. These literally contain a list of words. These are either common words, or user names, passwords that were leaked in large volumes earlier in history. These wordlists, along with other useful files, are of great help for penetration testing, as they make certain steps and operations easier and quicker, often even possible to automate them.

**However**, even though the older HTB exercise write-up document refers to a wordlist file located under a directory named "dirb" (`/usr/share/wordlists/dirb/common.txt`), today we can see that such directory does not exist in the given Parrot machine. Instead, we are presented with the following directory structure and wordlist files:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ ls /usr/share/wordlists/
	dirbuster/      metasploit/     seclists/       
	dnsmap.txt      nmap.lst        sqlmap.txt      
	john.lst        rockyou.txt.gz  wfuzz/          
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ ls /usr/share/wordlists/dirbuster
	apache-user-enum-1.0.txt  directory-list-2.3-medium.txt
	apache-user-enum-2.0.txt  directory-list-2.3-small.txt
	directories.jbrofuzz      directory-list-lowercase-2.3-medium.txt
	directory-list-1.0.txt    directory-list-lowercase-2.3-small.txt

We will use the wordlists located under the `/usr/share/wordlists/dirbuster/` directory.

The command could look like this:
`gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u 10.129.155.91:80`

But upon running `gobuster` this way, it would not yield us any result, because although the word "admin" is in both the small and medium wordlist files, "admin.php" is missing. So we just simply create a txt file that contains "admin.php" in one line, and we provide that file as the wordlist parameter instead.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ echo "admin.php" > simple_wordlist.txt

And we can see that we have a finding:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ gobuster dir -w ~/simple_wordlist.txt -u 10.129.155.91:80
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://10.129.155.91:80
	[+] Method:                  GET
	[+] Threads:                 10
	[+] Wordlist:                /home/scrage/simple_wordlist.txt
	[+] Negative Status codes:   404
	[+] User Agent:              gobuster/3.6
	[+] Timeout:                 10s
	===============================================================
	Starting gobuster in directory enumeration mode
	===============================================================
	Progress: 1 / 2 (50.00%)
	/admin.php            (Status: 200) [Size: 999]
	===============================================================
	Finished
	===============================================================

**OR BETTER**, there is a switch we can use here that tells gobuster to also look for php pages or any other extensions such as asp, aspx, etc. when trying to bust the directories with the wordlist provided:
`-x php`

So a more life-like example of this exercise would look like this:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-6m9y8uitpk]─[~]
	└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -u 10.129.155.91
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://10.129.155.91
	[+] Method:                  GET
	[+] Threads:                 10
	[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
	[+] Negative Status codes:   404
	[+] User Agent:              gobuster/3.6
	[+] Extensions:              php
	[+] Timeout:                 10s
	===============================================================
	Starting gobuster in directory enumeration mode
	===============================================================
	/admin.php            (Status: 200) [Size: 999]
	Progress: 11886 / 175330 (6.78%)^C
	[!] Keyboard interrupt detected, terminating.
	Progress: 12423 / 175330 (7.09%)
	===============================================================
	Finished
	===============================================================

Always use the `-x` switch when you have an educated guess about what kind of web server would be running on the host machine! Just bear in mind that this also increases the command's execution time.

Once we navigate to the `/admin.php` page of the target webserver, we are met with an administrative panel for the website. It asks us for a username and password to get past the security check, which could prove problematic in normal circumstances.

![[Pasted image 20240828001402.png]]

Usually, in situations such as this one, we would need to fire up some brute-forcing tools to attempt logging in with multiple credentials sets for an extended period of time until we hit a valid log-in since we do not have any underlying context about usernames and passwords that might have been registered on this web site as valid administrative accounts. But first, we can try our luck with some default credentials since this is a fresh nginx installation. We are betting that it might have been left unconfigured at the time of our assessment. Let's try logging in with the following credentials:

	admin admin

And success.
![[Pasted image 20240828001544.png]]

Flag is exfiltrated: `6483bee07c1c1d57f14e5b0717503c73`


https://www.hackthebox.com/achievement/machine/2057492/397
![[Pasted image 20240828002502.png]]