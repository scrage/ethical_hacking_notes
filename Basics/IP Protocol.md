**Internet Protocol** (**IP**) is a [network layer](https://en.wikipedia.org/wiki/Network_layer "Network layer") communication protocol in the [Internet protocol suite](https://en.wikipedia.org/wiki/Internet_protocol_suite "Internet protocol suite") for relaying datagrams across network boundaries. Working on **Layer 3 (Routing)** of the **OSI** model. 

IP has the task of delivering [packets](https://en.wikipedia.org/wiki/Packet_(information_technology)) from the source [host](https://en.wikipedia.org/wiki/Host_(network)) to the destination host solely based on the [IP addresses](https://en.wikipedia.org/wiki/IP_address) in the packet headers.

# IPv4
The first version of the [Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol "Internet Protocol") as a standalone specification. It is one of the core protocols of standards-based [internetworking](https://en.wikipedia.org/wiki/Internetworking) methods in the Internet and other packet-switched networks. Its first version was deployed for production on SATNET in 1082 and on ARPANET in January 1983. Is still in use to route most Internet traffic today, even though all the available IPv4 addresses has been taken and used up already. Its successor is the IPv6 specification, with currently ongoing deployment.

IPv4 uses **32-bit** addresses which limits the address space to 4294967296 (2^32) addresses.
It uses four octets to express individual decimal numbers separated by periods ([dot-decimal notation](https://en.wikipedia.org/wiki/Dot-decimal_notation)).

# IPv6
The most recent version of the [Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol "Internet Protocol"). It was developed by the Internet Engineering Task Force to deal with the long-anticipated problem of IPv4 address exhaustion, and was introduced to replace IPv4. It became a Draft Standard for the IETF in December 1998, which subsequently ratified it as an Internet Standard on 14 July 2017.

IPv6 uses **128-bit** addresses, which are represented as eight groups of four hexadecimal digits each, separated by colons.


Upon executing `ifconfig` on unix:

| parameter | what it represents                    |
| --------- | ------------------------------------- |
| inet      | IP v4 address in decimal notation     |
| inet6     | IP v6 address in hexadecimal notation |
![[Pasted image 20240817133858.png]]