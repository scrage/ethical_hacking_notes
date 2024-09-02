#reconnaissance #databases #apache #mariadb #php #sql #sql-injection

# 0. Theory

This exercise will perform a `SQL Injection` against a `SQL Database` enabled web application.

The target is running a website with search capabilities against a back-end database containing searchable items vulnerable to this type of attack. Not all items in this database should be seen by any user, so different privileges on the website will grant you different search results.
For an attacker with knowledge on web application vulnerabilities - specifically SQL Injection, in this case - the separation between those privileges and the quarriable data tables can be blurred, as they will be able to exploit the web application to directly query any table found on the SQL Database of the webserver.

A good example of how an SQL Service typically works is the log-in process utilized for any user: Each time the user wants to log in, the web application sends the log-in page input (username/password combination) to the SQL Service, comparing it with stored database entries for that specific user. If the specified username and password match any entry in the database, then the SQL Service will report it back to the web application, which will, in turn, log the user in, giving them access to the restricted parts of the website.
After log-in, the web application will set the user a special permission in the form of a **cookie or authentication token** that associates their online presence with their authenticated presence on the website. This cookie is stored both locally, on the user's browser storage, and the webserver.

The reason websites use databases such as MySQL, MariaDB, or other kinds is that the data they collect or serve needs to be stored somewhere. Data could be usernames, passwords, posts, messages, or more sensitive sets such as **PII** (**Personally Identifiable Information**), which is protected by international data privacy laws.

SQL Injection is a common way of exploiting web pages that use SQL statements that retrieve and store user input data. If configured incorrectly, one can use this attack to exploit the well-known `SQL Injection` vulnerability.


**What does the acronym SQL stand for?**
*Structured Query Language*

**What is one of the most common type of SQL vulnerabilities?**
*SQL Injection*

**What is the 2021 OWASP Top 10 classification for this vulnerability?**
**[A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/)**

**What does Nmap report as the service and version that are running on port 80 of the target?**
*Apache httpd 2.4.38 ((Debian))*

**What is the standard port used for the HTTPS protocol?**
*443*

**What is a folder called in web-application terminology?**
*directory*

**What is the HTTP response code is given for 'Not Found' errors?**
*404*

**Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?**
*dir*

What single character can be used to comment out the rest of a line in MySQL?
`#`


# 1. Recon

Verify target and connectivity.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-uejrvhgdvs]─[~]
	└──╼ [★]$ ping 10.129.156.148 -c 5
	PING 10.129.156.148 (10.129.156.148) 56(84) bytes of data.
	64 bytes from 10.129.156.148: icmp_seq=1 ttl=63 time=7.94 ms
	64 bytes from 10.129.156.148: icmp_seq=2 ttl=63 time=8.13 ms
	64 bytes from 10.129.156.148: icmp_seq=3 ttl=63 time=8.06 ms
	64 bytes from 10.129.156.148: icmp_seq=4 ttl=63 time=7.99 ms
	64 bytes from 10.129.156.148: icmp_seq=5 ttl=63 time=7.87 ms
	
	--- 10.129.156.148 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4004ms
	rtt min/avg/max/mdev = 7.869/7.997/8.127/0.089 ms


# 2. Enumeration

We will need super-user privileges to run the `nmap` command below with the`-sC` or -`sV` flags. This is because script scanning (`-sC`) and version detection (`-sV`) are considered more intrusive methods of scanning the target. This results in a higher probability of being caught by a perimeter security device on the target's network.

	-sC: Performs a script scan using the default set of scripts. It is equivalent to -- script=default. Some of the scripts in this category are considered intrusive and should not be run against a target network without permission.
	-sV: Enables version detection, which will detect what versions are running on what port.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-uejrvhgdvs]─[~]
	└──╼ [★]$ sudo nmap -sC -sV 10.129.156.148
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-29 13:05 CDT
	Nmap scan report for 10.129.156.148
	Host is up (0.0097s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	80/tcp open  http    Apache httpd 2.4.38 ((Debian))
	|_http-server-header: Apache/2.4.38 (Debian)
	|_http-title: Login
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 6.96 seconds

The only open port we detect is port **80 TCP**, which is running the `Apache httpd` server version 2.4.38 .

Apache HTTP Server is a free and open-source application that runs web pages on either physical or virtual web servers. It is one of the most popular HTTP servers, and it usually runs on standard HTTP ports such as ports 80 TCP, 443 TCP, and alternatively on HTTP ports such as 8080 TCP or 8000 TCP.

The nmap scan provided us with the exact version of the Apache httpd service, which is 2.4.38. Usually, a good idea would be to search up the service version on popular vulnerability databases online to see if any vulnerability exists for the specified version. However, in our case, this version does not contain any known vulnerability that we could potentially exploit.

In order to further enumerate the service running on port 80, we can navigate directly to the IP address of the target from the browser.

![[Pasted image 20240829200912.png]]

Log-in forms are used to authenticate users and give them access to restricted parts of the website depending on the privilege level associated with the input username. Since we are not aware of any specific credentials that we could use to log-in, we will check if there are any other directories or pages useful for us in the enumeration process. It is always considered good practice to fully enumerate the target before we target a specific vulnerability we are aware of, such as the SQL Injection vulnerability in this case. We need the whole picture to ensure we are not missing anything and fall into a rabbit hole, which could quickly become frustrating.

We can perform a directory busting at this point, to uncover any existing web pages on the host.
Gobuster, Dirbuster, Dirb, and other directory busting tools come into play here.
These are brute-forcing tools.
Brute-force is a method of submitting data provided through a specially made list of variables known as the wordlist in an attempt to guess the correct input for it to be validated and access to be gained.

The same attack type comes into play for password brute-forcing - submitting passwords from a wordlist until we find the right one for the specified username. This method is prevalent for low-skilled attackers due to its low complexity, with the downside of being "noisy", meaning that it involves sending a large number of requests every second, so much that it becomes easily detectable by perimeter security devices that are fine-tuned to listen for non-human interactions with log-in forms.

For our case, we will be running `Gobuster`, which will brute-force the directories and files provided in the wordlist of our choice. Gobuster comes pre-installed with Parrot OS, which is running on the prepared HTB attacker box.

## Using Gobuster
### Installation

If we have a Go environment ready to go, then just:
`go install github.com/OJ/gobuster/v3@latest`

Otherwise, we can get the tool from its github repository, build it locally and run it:
	`git clone https://github.com/OJ/gobuster.git`
	`cd gobuster`
	`go get && go build`
	`go install`

To check how to use gobuster, see its help:
`gobuster --help`

### Usage
There is a dedicated folder with a myriad of wordlists, dictionaries, and rainbow tables that comes pre-installed with Parrot OS, found under the path /usr/share/wordlists . However, you can download the SecLists collection as well, it being one of the most famous wordlist collections in use today. SecLists can be downloaded by [following this link](https://github.com/danielmiessler/SecLists).

To download it, type the following command:
`git clone https://github.com/danielmiessler/SecLists.git`

The flags we will be using with `gobuster`:

	dir : Specify that we wish to do web directory enumeration.
	--url : Specify the web address of the target machine that runs the HTTP server.
	--wordlist : Specify the wordlist that we want to use.

We also use the `-x php` flag to tell gobuster to try words in the wordlist file with the ".php" suffix as well.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-uejrvhgdvs]─[~]
	└──╼ [★]$ gobuster dir --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php --url http://10.129.156.148/
	===============================================================
	Gobuster v3.6
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
	===============================================================
	[+] Url:                     http://10.129.156.148/
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
	/images               (Status: 301) [Size: 317] [--> http://10.129.156.148/images/]
	/.php                 (Status: 403) [Size: 279]
	/index.php            (Status: 200) [Size: 4896]
	/css                  (Status: 301) [Size: 314] [--> http://10.129.156.148/css/]
	/js                   (Status: 301) [Size: 313] [--> http://10.129.156.148/js/]
	/vendor               (Status: 301) [Size: 317] [--> http://10.129.156.148/vendor/]
	/fonts                (Status: 301) [Size: 316] [--> http://10.129.156.148/fonts/]
	/.php                 (Status: 403) [Size: 279]
	Progress: 175328 / 175330 (100.00%)
	===============================================================
	Finished
	===============================================================


A refresher on the relevant HTTP status codes:
(https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

| Status Code | Message           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200         | OK                | **`200 OK`** [successful response](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#successful_responses) status code indicates that a request has succeeded. A `200 OK` response is cacheable by default.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 301         | Moved Permanently | **`301 Moved Permanently`** redirect status response code indicates that the requested resource has been definitively moved to the URL given by the [`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) headers. A browser redirects to the new URL and search engines update their links to the resource.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| 403         | Forbidden         | **`403 Forbidden`** [client error response](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) status code indicates that the server understood the request but refused to process it. This status is similar to [`401`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401), except that for **`403 Forbidden`** responses, authenticating or re-authenticating makes no difference. The request failure is tied to application logic, such as insufficient permissions to a resource or action.<br><br>Clients that receive a `403` response should expect that repeating the request without modification will fail with the same error. Server owners may decided to send a [`404`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404) response instead of a 403 if acknowledging the existence of a resource to clients with insufficient privileges is not desired. |

After checking out the web directories, we have found no helpful information. The results present in our output represent default directories for most websites, and most of the time, they do not contain files that could be exploitable or useful for an attacker in any way. However, it is still worth checking them because sometimes, they could contain non-standard files placed there by mistake.


# 3. Foothold

Since Gobuster did not find anything useful, we need to check for any default credentials or bypass the login page somehow. To check for default credentials, we could type the most common combinations in the username and password fields, such as:

	admin:admin
	guest:guest
	user:user
	root:root
	administrator:password

After attempting all of those combinations, we have still failed to log in. We could, hypothetically, use a tool to attempt brute-forcing the log-in page. However, that would take much time and might trigger a security measure.

So, the next tactic would be to test the log-in form for a possible `SQL Injection` vulnerability.

When code (php, python, C#, Java, JavaScript, etc.) that queries a SQL database is written in a way that it string concatenates values of variables with the string of the actual SQL query, that makes the code vulnerable to `SQL Injection`.
An example in PHP for this scenario:

`<?php`
`mysql_connect("localhost", "db_username", "db_password"); # Connection to the SQL Database.`
`mysql_select_db("users"); # Database table where user information is stored.`

`$username=$_POST['username']; # User-specified username.`
`$password=$_POST['password']; #User-specified password.`

`$sql="SELECT * FROM users WHERE username='$username' AND password='$password'";`
`# Query for user/pass retrieval from the DB.`

`$result=mysql_query($sql);`
`# Performs query stored in $sql and stores it in $result.`

`$count=mysql_num_rows($result);`
`# Sets the $count variable to the number of rows stored in $result.`

`if ($count==1){`
	`# Checks if there's at least 1 result, and if yes:`
	`$_SESSION['username'] = $username; # Creates a session with the specified $username.`
	`$_SESSION['password'] = $password; # Creates a session with the specified $password.`
	`header("location:home.php"); # Redirect to homepage.`
`}`
`else { # If there's no singular result of a user/pass combination:`
	`header("location:login.php");`
	`# No redirection, as the login failed in the case the $count variable is not equal to 1, HTTP Response code 200 OK.`
`}`
`?>`

Notice the one-line comment symbol # in PHP. In order to set up an `SQL Injection`, we need to know the language being used on the back end to exploit it.

This code above is vulnerable to SQL Injection attacks, where you can modify the query (the `$sql` variable) through the log-in form on the web page to make the query do something that is not supposed to do - bypass the log-in entirely.

Note that we can specify the username and password through the log-in form on the web page. However, it will be directly embedded in the `$sql` variable that performs the SQL query without input validation. Notice that no regular expressions or functions forbid us from inserting special characters such as a single quote or hashtag. This is a dangerous practice because those special characters can be used for modifying the queries. The pair of single quotes are used to specify the exact data that needs to be retrieved from the SQL
Database, while the hashtag symbol is used to make comments. Therefore, we could manipulate the query command by inputting the following:

	Username: admin'#

We will close the query with that single quote, allowing the script to search for the admin username. By adding the hashtag, we will comment out the rest of the query, which will make searching for a matching password for the specified username obsolete. If we look further down in the PHP code above, we will see that the code will only approve the log-in once there is precisely one result of our username and password combination. However, since we have skipped the password search part of our query, the script will now only search if any entry exists with the username `admin` . In this case, we got lucky. There is indeed an account called admin , which will validate our SQL Injection and return the 1 value for the `$count` variable, which will be put through the `if statement`, allowing us to log-in without knowing the password. If there was no admin account, we could try any other accounts until we found one that existed. (`administrator`, `root`, `john_doe`, etc.) Any valid, existing username would make our SQL Injection work.

In this case, because the password-search part of the query has been skipped, we can throw anything we want at the password field, and it will not matter.

To be clear, this is how the query part of the PHP code gets affected by our input:

`SELECT * FROM users WHERE username='admin'#' AND password='a'`

Following our input, we have commented out the password check section of the query.

![[Pasted image 20240829210947.png]]
Flag exfiltrated: `e3d0796d002a446c0e622226f42e9672`


https://www.hackthebox.com/achievement/machine/2057492/402
![[Pasted image 20240829211938.png]]