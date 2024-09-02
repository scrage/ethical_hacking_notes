# Things to look for methodically
- service version information
- main backend directories
- source code
- potential vulnerabilities scanning (i.e. with nikto)


# Scanning the local network
`sudo arp-scan -l` to scan local network and look for IP addresses
`sudo netdiscover -r <IP>/24`  to start capturing ARP req/rep packets from host to find target IPs

 T4

| `nmap -T4 -p- -A <IP>` | toggle meaning                       |
| ---------------------- | ------------------------------------ |
| `-T4`                  | speed between T1 and T5 (T4 is fine) |
| `-p-`                  | scans all ports                      |
| `-p X,Y,Z`             | specific ports to scan listed        |
| -`A`                   | scan for every information (all)     |
| `<IP>`                 | the target IP address                |
# Enumerating HTTP/HTTPS
#theory

Always check the source code of web pages you find.
## Taking notes
When doing reconnaissance or enumerating on a target, taking notes in the following, or in similar format, is advised:
	80/443 - 192.168.57.134 - (10:58 PM)
	Default webpage found on opened port - Apache - PHP - RedHat Linux Server

Finding error pages on default webpages can potentially lead us to more information when too much info is disclosed
	Information Disclosure - 404 page - Apache 1.3.20 is running - kioptrix.level1 host name with port 443 open

Taking screenshots is also advised.

## Directory Busting
`dirbuster`, `dirb` and `gobuster` are built-in tools on kali linux for directory busting.
Start it with an ampersand at the end of the command.
1. Paste the target IP address with the port.
2. Check the "Go Faster" checkbox.
3. Check "List based brute force" and pick a file from /usr/share/wordlist/dirbuster folder.
4. Use the appropriate file extension for the enumeration (based on info gathered about the web host engine earlier). Check other extensions too: "php,txt,zip,rar,pdf,docx...", but this will make the process slower.


`dirsearch` for directory busting

## Burp Suite for Enumeration
Set up the local proxy and start BurpSuite. Intercept once request, then send it to a repeater (ctrl+R) (right click on Raw request body and "Send to Repeater").
The **Repeater** tab will show up. You can edit the request and see the results there.

Server Response headers can disclose information.

# Enumerating SMB
`msfconsole` is a tool to run a wide array of exploits and other activities. (Auxiliary, post, payload, exploit services.)
https://docs.metasploit.com/

Running the `search smb` command in the msf console will show a number of results.

`use auxiliary/scanner/smb/smb_version` will for example to run an SMB version detection.

`info` prints general info on the service currently in use
`options` prints the available options for the service to run

To start a service, `set` the required options for the service first, then execute the `run` command.

![[Pasted image 20240824150851.png]]

Another tool is `smbclient` that can connect to SMB file shares. Once connected, we can discover files there.
`smbclient -L \\\\<IP address>\\`

# Enumerating SSH
Once we have a version number of SSH being used on a server, we can start enumerating SSH connections to the server.
`ssh <IP address>`

If we get an error "UNable to negotiate with *IP-address* port 22: no matching key exchange method found. Their offer: *algo1,algo2,...* ", then run with the following flag:
`ssh <IP address> -oKexAlgorithms=+algoN` where *algoN* is an algorithm the server offers.

If another error says "...no matching cipher found. Their offer: cipher1,cipher2,...", then run it with a `-c` flag:
`ssh <IP address> -oKexAlgorithms=+algoN -c cipherN`

Upon successful connection attempt, the server may still be asking for a root password, and we may not connect successfully, but sometimes a **banner** text is printed onto the console that can give us more information.


# Scanning with Nessus
Nessus is a vulnerability scanner.

Download Nessus from its official page.
Extract and install it: `sudo dpkg -i Nessus-10.8.2-debian10_amd64.deb`

Follow its prompt and start the service by typing `/bin/systemctl start nessusd.service`, then got to https://kali:8834/ to configure the scanner.

