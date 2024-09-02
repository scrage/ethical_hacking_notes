#common-applications #magneto #reconnaissance #web-site-structure-discovery #weak-credentials 

# 0. Theory

Networking knowledge plays a tremendous part for the overall readiness of a cybersecurity engineer. Features such as Active Directory, Kerberos Authentication, Server Message Block, Hypertext Transfer Protocol, Secure Shell can all be dissected into their (almost) simplest form if enough networking knowledge is applied

To brush up on Networking knowledge, [here's an introduction to Networking](https://academy.hackthebox.com/module/details/34) on HTB Academy.


**What is the full path to the file on a Linux computer that holds a local list of domain name to IP address pairs?**
*/etc/hosts*



# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-2aru0rhvm9]─[~]
	└──╼ [★]$ ping 10.129.158.243 -c 4
	PING 10.129.158.243 (10.129.158.243) 56(84) bytes of data.
	64 bytes from 10.129.158.243: icmp_seq=1 ttl=63 time=9.04 ms
	64 bytes from 10.129.158.243: icmp_seq=2 ttl=63 time=8.25 ms
	64 bytes from 10.129.158.243: icmp_seq=3 ttl=63 time=8.60 ms
	64 bytes from 10.129.158.243: icmp_seq=4 ttl=63 time=8.29 ms
	
	--- 10.129.158.243 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3005ms
	rtt min/avg/max/mdev = 8.245/8.542/9.036/0.315 ms


# 2. Enumeration

Upon conducting a port scan with version and service name discovery, we notice that only **port 80** is open with `nginx/1.14.2` running on it.
We also notice from the output that right below that the http-title returns `Did not follow redirect to http://ignition.htb/`. We keep this URL in mind for now.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-2aru0rhvm9]─[~]
	└──╼ [★]$ nmap -sV -sC 10.129.158.243
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-02 05:03 CDT
	Nmap scan report for 10.129.158.243
	Host is up (0.010s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	80/tcp open  http    nginx 1.14.2
	|_http-title: Did not follow redirect to http://ignition.htb/
	|_http-server-header: nginx/1.14.2
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 8.17 seconds


Upon attempting to access the webpage through a browser window, we are presented with an error. The *Check if there is a typo in `ignition.htb`* references the same URL we found during our `nmap` scan, but without further details as to what might cause this error to pop up when simply attempting to access the website. Below, a more detailed error code is displayed: `DNS_PROBE_FINISHED_NXDOMAIN`.

## Adding target's IP address and host name to the local /etc/hosts file

After a quick Google search of the error, we learn that there might be two underlying reasons to this error appearing.
- We've mistyped the `ignition.htb` address in our URL search bar, and the DNS servers can't find the associated IP address for the mistyped name.
- We never entered any hostname such as `ignition.htb` into the search bar, but the website expects us to.

Since we know for a fact that we never entered any hostname into the search bar, we will be exploring the second option only. This option refers to an issue with what is known as name-based VHosting (or Virtual Hosting). According to [the Wikipedia article on Virtual Hosting](https://en.wikipedia.org/wiki/Virtual_hosting), we have the following statements:

>Virtual hosting is a method for hosting multiple domain names (with separate handling of each name) on a single server (or pool of servers). This allows one server to share its resources, such as memory and processor cycles, without requiring all services provided to use the same host name. The term virtual hosting is usually used in reference to web servers but the principles do carry over to other Internet services.
>[..] 
>A technical prerequisite needed for name-based virtual hosts is a web browser with HTTP/1.1 support (commonplace today) to include the target hostname in the request. This allows a server hosting multiple sites behind one IP address to deliver the correct site's content. More specifically it means setting the Host HTTP header, which is mandatory in HTTP/1.1.
>[..]
>Furthermore, if the Domain Name System (DNS) is not properly functioning, it is difficult to access a virtually-hosted website even if the IP address is known. If the user tries to fall back to using the IP address to contact the system, as in http://10.23.45.67/, the web browser will send the IP address as the host name. Since the web server relies on the web browser client telling it what server name (vhost) to use, the server will respond with a default website—often not the site the user expects. A workaround in this case is to add the IP address and host name to the client system's hosts file. Accessing the server with the domain name should work again. [..]

In short, multiple websites can share the same IP address, allowing users to access them separately by visiting the specific hostnames of each website instead of the hosting server's IP address. The webserver we are making requests to is throwing us an error because we haven't specified a certain hostname out of the ones that could be hosted on that same target IP address. From here, we'd think that simply inputting `ignition.htb` instead of the target IP address into the search bar would solve our issue, but unfortunately, this is not the case. When entering a hostname instead of an IP address as the request's destination, there is a middleman involved that is worth mentioning:

>The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities. Most prominently, it translates more readily memorized domain names to the numerical IP addresses needed for locating and identifying computer services and devices with the underlying network protocols. By providing a worldwide, distributed directory service, the Domain Name System has been an essential component of the functionality of the Internet since 1985.

Because DNS is involved when translating the hostnames to the one IP address available on the server's side, this will prove to be an issue once the target is isolated, such as in our case. In order to solve this, we can edit our own local hosts file which includes correlations between hostnames and IP addresses to accomodate for the lack of a DNS server doing it on our behalf.

Until then, we must first confirm that we are correct. In order to get a better view of the exact requests and responses being made and to confirm our suspicion, we will need to make use of a popular and easy to use tool called `cURL` . This tool will allow us to manipulate HTTP requests made to a server and receive the responses directly in the terminal, without the latter being interpreted by our browser as generic error messages such as in the example above.

Looking through the help menu options, we decide to simply make the output more detailed, in order to learn as much as possible from the target's responses. We can achieve this by increasing the verbosity of the script's output.

	-v : Make the operation more talkative. More detailed output will be displayed during runtime.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-2aru0rhvm9]─[~]
	└──╼ [★]$ curl -v http://10.129.158.243/
	*   Trying 10.129.158.243:80...
	* Connected to 10.129.158.243 (10.129.158.243) port 80 (#0)
> 	GET / HTTP/1.1
> 	Host: 10.129.158.243
> 	User-Agent: curl/7.88.1
> 	Accept: */*
> 	
	< HTTP/1.1 302 Found
	< Server: nginx/1.14.2
	< Date: Mon, 02 Sep 2024 10:37:23 GMT
	< Content-Type: text/html; charset=UTF-8
	< Transfer-Encoding: chunked
	< Connection: keep-alive
	< Set-Cookie: PHPSESSID=egnllc6aou5os52v6ksgrth49d; expires=Mon, 02-Sep-2024 11:37:23 GMT; Max-Age=3600; path=/; domain=10.129.158.243; HttpOnly; SameSite=Lax
	< Location: http://ignition.htb/
	< Pragma: no-cache
	< Cache-Control: max-age=0, must-revalidate, no-cache, no-store
	< Expires: Sat, 02 Sep 2023 10:37:23 GMT
	`[*** SNIP ***]`


We see that our request contains a `Host` field which is home to the target's IP address instead of the hostname. The `302 Found` response, together with the Location header, [indicates](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302) that the resource we requested (`/`) has been (temporarily) moved to `http://ignition.htb/`. This means that our assumptions were true.

To solve the issue we are currently facing here, we will modify our local DNS file `/hosts/etc`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-2aru0rhvm9]─[~]
	└──╼ [★]$ echo "10.129.158.243 ignition.htb" | sudo tee -a /etc/hosts
	10.129.158.243 ignition.htb

Once this configuration is complete, we can proceed to reload the target's webpage and verify if it loads successfully.

![[Pasted image 20240902131506.png]]


After exploring the landing page for a short period of time, we can deduce that nothing helpful can be leveraged here. The only option of exploring the website further is using `gobuster`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.32]─[scrage@htb-2aru0rhvm9]─[~]
	└──╼ [★]$ gobuster dir --url http://ignition.htb/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt 
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://ignition.htb/
	[+] Method:                  GET
	[+] Threads:                 10
	[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
	[+] Negative Status codes:   404
	[+] User Agent:              gobuster/3.6
	[+] Timeout:                 10s
	===============================================================
	Starting gobuster in directory enumeration mode
	===============================================================
	/contact              (Status: 200) [Size: 28673]
	/home                 (Status: 200) [Size: 25802]
	/media                (Status: 301) [Size: 185] [--> http://ignition.htb/media/]
	/0                    (Status: 200) [Size: 25803]
	/catalog              (Status: 302) [Size: 0] [--> http://ignition.htb/]
	/static               (Status: 301) [Size: 185] [--> http://ignition.htb/static/]
	/admin                (Status: 200) [Size: 7095]
	/Home                 (Status: 301) [Size: 0] [--> http://ignition.htb/home]
	/cms                  (Status: 200) [Size: 25817]
	Progress: 1210 / 87665 (1.38%)^C
	[!] Keyboard interrupt detected, terminating.
	Progress: 1210 / 87665 (1.38%)
	===============================================================
	Finished
	===============================================================


Gobuster found us an /admin page. Upon entering it to the end of the host URL in the browser, we are faced with a login page.

![[Pasted image 20240902132321.png]]

A username and password are being requested. Normally, we would go off credentials we extracted through other means, such as an FTP server left unsecured, as seen before. This time, however, we will attempt some default credentials for the `Magento` service, as there is a logo for Magento boasting in the middle of the page, since there is no other basis upon which we can rely.

>The Magento Admin is protected by multiple layers of security measures to prevent unauthorized access to your store, order, and customer data. The first time you sign in to the Admin, you are required to enter your username and password and to set up twofactor authentication (2FA).
>
>Depending on the configuration of your store, you might also be required to resolve a CAPTCHA challenge such as entering a series of keyboard characters, solving a puzzle, or clicking a series of images with a common theme. These tests are designed to identify you has human, rather than an automated bot. For additional security, you can determine which parts of the Admin each user has permission to access, and also limit the number of login attempts. By default, after six attempts the account is locked, and the user must wait a few minutes before trying again. Locked accounts can also be reset from the Admin.
>
>An Admin password must be seven or more characters long and include both letters and numbers.


# 3. Foothold

According to the documentation, we should not attempt to brute force this login form because it has anti-bruteforce measures implemented, we will need to guess the password. Since the password must be seven or more characters long & to include both letters and numbers, we can attempt to use the [most common passwords of the year](https://cybernews.com/best-password-managers/most-common-passwords/) as well as a common username, such as `admin`.

Some combinations that fulfil the requirement of the password being at least 7 characters and including both letters and numbers:

	admin admin123
	admin root123
	admin password1
	admin administrator1
	admin changeme1
	admin password123
	admin qwerty123
	admin administrator123
	admin changeme123

After manually attempting a number of these credentials, we land on a successful login. The correct combination is: `admin:qwerty123`.
We are presented with the Magento administrative panel, where the flag can be found under the `Advanced Reporting` section of the Dashboard.

![[Pasted image 20240902133654.png]]

Flag exfiltrated: `797d6c988d9dc5865e010b9410f247e0`



https://www.hackthebox.com/achievement/machine/2057492/405
![[Pasted image 20240902134329.png]]