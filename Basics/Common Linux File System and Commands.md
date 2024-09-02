Every directory and file with a leading "." in their name is a hidden directory or file.

## Navigation on the file system

| command       | description                                             |
| ------------- | ------------------------------------------------------- |
| pwd           | print working directory                                 |
| cd            | change directory                                        |
| ls            | list current directory's content                        |
| ls -la        | list long all (also lists hidden files and directories) |
| mkdir         | make directory                                          |
| rmdir         | remove directory                                        |
| man [command] | prints the manual of the command                        |

## File management

| command                        | description                                                                                                                 |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| cp [file source] [destination] | copies file in source to destination<br>also creates new file in first argument's destination of second argument is omitted |
| mv [file source] [destination] | moves file in source to destination                                                                                         |
| rm [file]                      | removes file                                                                                                                |
| locate [file name]             | lists all the paths on the file system where the file in argument is located at                                             |
| cat [file]                     | print out the contents of a file                                                                                            |
| grep 'string' [file]           | pulling out specific string from a file                                                                                     |

## Users & Privileges

When listing long details of a directory, the first column will show 10 characters in each row:
First character denotes whether the element is a directory (**d**), a file (**-**), or a link (**l**).
Characters 2-3-4 denote read (**r**), write (**w**) and execute (**x**) privileges for the **owner**.
Characters 5-6-7 correspond to **group** privileges.
Characters 8-9-10 correspond to **all other users**.
A dash (**-**) denotes missing corresponding privilege at any place.
![[Pasted image 20240820123137.png]]

The **/tmp** folder has read-write-execute through privileges, and is an ideal place to drop anything while penetration testing.
![[Pasted image 20240820123644.png]]

| command                                                                    | description                                                           |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| passwd                                                                     | changes the active user's password                                    |
| chmod +[rwx] filename<br>chmod - [rwx] filename<br>chmod [bitmas] filename | change (permission) mode (add/remove or apply as bit mask)<br>to file |
| sudo adduser [user name]                                                   | adds a new user (requires sudo privilege)                             |
| su [user name]                                                             | switches user                                                         |

## Permission bit masks

| chmod mask<br>number | corresponding<br>permission | totals to<br>bit mask |
| -------------------- | --------------------------- | --------------------- |
| 0                    | - - -                       | 0+0+0                 |
| 1                    | - - x                       | 0+0+1                 |
| 2                    | - w -                       | 0+2+0                 |
| 3                    | - w x                       | 0+2+1                 |
| 4                    | r - -                       | 4+0+0                 |
| 5                    | r - x                       | 4+0+1                 |
| 6                    | r w -                       | 4+2+0                 |
| 7                    | r w x                       | 4+2+1                 |

## Common Networking commands

| command             | description                                                                                                                       |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| ip *object command* | assigns an address to a network interface and/or configures network interface parameters<br>replaces the old **ifconfig** command |
|                     |                                                                                                                                   |
*OBJECTS* can be any one of the following and may be written in full or abbreviated form:

|               |                      |                                                     |
| ------------- | -------------------- | --------------------------------------------------- |
| **Object**    | **Abbreviated form** | **Purpose**                                         |
| **link**      | l                    | Network device.                                     |
| **address**   | a  <br>addr          | Protocol (IP or IPv6) address on a device.          |
| **addrlabel** | addrl                | Label configuration for protocol address selection. |
| **neighbour** | n  <br>neigh         | ARP or NDISC cache entry.                           |
| **route**     | r                    | Routing table entry.                                |
| **rule**      | ru                   | Rule in routing policy database.                    |
| **maddress**  | m  <br>maddr         | Multicast address.                                  |
| **mroute**    | mr                   | Multicast routing cache entry.                      |
| **tunnel**    | t                    | Tunnel over IP.                                     |
| **xfrm**      | x                    | Framework for IPsec protocol.                       |
|               |                      |                                                     |


