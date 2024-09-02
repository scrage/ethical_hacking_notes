### [IP command examples](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/)

| example command                                                                                                                          | effect                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `ip a`                                                                                                                                   | Displays info about all network interfaces                                                 |
| `ip -4 a`                                                                                                                                | Only show TCP/IP IPv4                                                                      |
| `ip -6 a`                                                                                                                                | Only show TCP/IP IPv4                                                                      |
| `ip a show eth0`<br>`ip a list eth0`<br>`ip a show dev eth0`                                                                             | Only show eth0 interface                                                                   |
| `ip link show up`                                                                                                                        | Only show running interfaces                                                               |
| `ip a add {ip_addr/mask} dev {interface}`<br><br>`ip a add 192.168.1.200/255.255.255.0 dev eth0`<br>`ip a add 192.168.1.200/24 dev eth0` | Assigns the IP address to the interface<br><br>assigns 192.168.1.200/255.255.255.0 to eth0 |
| `ip n`<br>`ip n show`<br>`ip neigh show`                                                                                                 | Display neighbour/arp cache                                                                |
| `ip r`<br>`ip r list`<br>`ip route`<br>`ip route list`                                                                                   | Displays the contents of the routing tables                                                |
| `route`                                                                                                                                  | Displays the kernel IP routing table                                                       |
| `ping {ip address}`                                                                                                                      | Pings the address                                                                          |
| `ping`{ip address} -c 1                                                                                                                  | Pings the address once (count 1)                                                           |


# Example Exercises
## Ping a range of IP addresses
