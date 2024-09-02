#scripting 
# Creating files and manipulating their content

| command                     | derscription                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------------------- |
| `echo "hello" > hello.txt`  | prints "hello" into the file hello.txt<br>creates the file if not existing, or overwrites it |
| `echo "hello" >> hello.txt` | prints "hello" into the file hello.txt<br>appends the file                                   |
| `touch newfile.txt`         | creates the file                                                                             |

# Starting and stopping services

| command                                                          | description                                                                      |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `sudo service apache2 start`                                     | spins up a new apache2 web server                                                |
| `sudo service apache2 stop`                                      | stops the service                                                                |
| `python3 -m http.server 80`                                      | starts a python3 http web server on port 80                                      |
| `sudo systemctl enable {service}`<br>`sudo systemctl enable ssh` | sets a service to start with the system startup<br>enables ssh at system startup |

# Manipulating data

| command                                                    | description                                                                                                                                                        |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `grep "string" {file}`                                     | grabs only the line containing the "string" in the given file                                                                                                      |
| `cat {file} \| grep "string"`                              | reads the file,<br>then outputs only the line containing the "string"                                                                                              |
| `cat {file} \| grep "string" \| cut -d "{delimiter}" -f N` | reads the file, grabs only the line containing the "string",<br>then cuts the whole output string by the provided delimiter, and outputs only the Nth token string |

# Exercise Examples
---
## Sweep a range of IP addresses and save the list of available ones into a text file
### Lead-up exercise
Take IP 10.0.2.15 as a single example first:
`└─$ ip a`
`1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000`
    `link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00`
    `inet 127.0.0.1/8 scope host lo`
       `valid_lft forever preferred_lft forever`
    `inet6 ::1/128 scope host noprefixroute` 
       `valid_lft forever preferred_lft forever`
`2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000`
    `link/ether 08:00:27:18:61:d8 brd ff:ff:ff:ff:ff:ff`
    `inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0`
       `valid_lft 62909sec preferred_lft 62909sec`
    `inet6 fe80::a00:27ff:fe18:61d8/64 scope link noprefixroute` 
       `valid_lft forever preferred_lft forever`

**To ping the address only once and save the result into a file:**
`cat ping 10.0.2.15 -c 1 > ping.txt`
**Result:**
`PING 10.0.2.15 (10.0.2.15) 56(84) bytes of data.`
`64 bytes from 10.0.2.15: icmp_seq=1 ttl=64 time=0.017 ms`

`--- 10.0.2.15 ping statistics ---`
`1 packets transmitted, 1 received, 0% packet loss, time 0ms`
`rtt min/avg/max/mdev = 0.017/0.017/0.017/0.000 ms`

**To see only the relevant line in that file:**
`cat ping.txt | grep "64 bytes"`
**Result:**
`64 bytes from 10.0.2.15: icmp_seq=1 ttl=64 time=0.017 ms`

**To see only the IP address pinged successfully:**
`cat ping.txt | grep "64 bytes" | cut -d " " -f 4  `
**Result:**
`10.0.2.15:`

**To use `translate` to remove the colon at the end of the IP (`-d` deletes the parameter):**
`cat ping.txt | grep "64 bytes" | cut -d " " -f 4 | tr -d ":"`
**Result:**
`10.0.2.15`

### Solution
To make an IP sweeping script, create a new sh file (with GUI editor, optionally):
`mousepad ipsweep.sh

Content:
`#!/bin/bash`

`if ["$1" == ""]`
`then`
  `echo "Error: Missing IP address to sweep."`
  `echo "Syntax: ./ipsweep.sh 10.0.2"`

`else`
  `for ip in seq 1 254; do`
  `ping -c 1 $1.$ip | grep "64 bytes" | cut -d " " -f 4 | tr -d ":" &`
  `done`
`fi`

**Note:**
*The ampersand character at the end of the ping command allows the loop to run parallelly, and it improves the script's performance greatly.*

Save it and give it execute permission:
`chmod +x ipsweep.sh `

Running it with 10.0.2 passed as parameter:
`./ipsweep.sh 10.0.2`
Result:
	`10.0.2.3`
	`10.0.2.2`
	`10.0.2.4`
	`10.0.2.15`


## Scan the ports of given IP addresses
### Solution
Using the previous solution, save the available IP addresses' list into a file to work with:
**./ipsweep.sh 10.0.2 > ips.txt**

Then probe each address with the `nmap` command:
`for ip in $(cat ips.txt); do nmap $ip; done;`
or for better performance:
`for ip in $(cat ips.txt); do nmap $ip & done;` 

OR

`#! /bin/python3`
`import sys`
`import socket`
`from datetime import datetime`

`# define the target`
`if len(sys.argv) == 2:`
	`target = socket.gethostbyname(sys.argv[1]) # translate hostname to IPv4`
	
`else:`
	`print('Invalid number of arguments.')`
	`print('Syntax: python3 scanner.py <ip>')`

`print("-" * 40)`
`print("# Scanning target: " + target)`
`print(f"Time started: {datetime.now()}")`
`print("-" * 40)`

`try:`
	`are_open_ports_found = False`
	`for port in range(50, 85):`
		`s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
		`socket.setdefaulttimeout(1)`
		`result = s.connect_ex((target, port))`
		
		if result == 0:
			print(f"Port {port} is open.")
			are_open_ports_found = True
		s.close()
	if not are_open_ports_found:`
		print("No open ports were found on target.")
	print("Done.")

`except KeyboardInterrupt:`
	`print("\nInterrupted by user, exiting.")`
	`sys.exit()`

`except socket.gaierror:`
	`print("Host name could not be resolved.")`
	`sys.exit()`
	
`except socket.error:`
	`print("Could not connect to server.")`
	`sys.exit()`

## Send files between devices quickly
### Solution
On the destination (linux) machine, set up `netcat` to listen to an arbitrary port and direct the incoming data into a local file:
`netcat -l 1234 > SECRETS.txt`

On the source (linux) machine, cat the file's content and pipe it into a netcat command, parametrized with the receiver's IP address and port:
`cat SECRETS.txt | netcat <IP> <port> -q 0` 

The `-q 0` flag closes the connection when the transfer is complete.

## Scan a web server for vulnerabilities
### Solution
`nikto` is a basic tool for vulnerability scanning. A web host with good firewall or security settings will catch it and you can get blocked using it, but it is rarely the case that such level of security is in fact in place for a real world webpage.

Good tool for a beginning and practicing.

`nikto -h http[s]://<website or IP address>`
The `-h` stands for "host"