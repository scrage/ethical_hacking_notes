#theory
# 1. Reconnaissance
Both active and passive. Information gathering. Verifying target(s).

# 2. Scanning & Enumeration
Actively scanning a target. Usage of nmap, nessus, nikto, etc.
Enumerating on found items and looking if they have any value.

90% of penetration testing consists of research done on the internet about
the product you are testing. Since the technological ecosystem is continuously evolving, it is impossible to know everything about everything. The key is to know how to look for the information you need. The ability to research effectively is the skill you need to continuously adapt and evolve into your top quality.

The objective here is not speed but meticulousness. If a resource on the target is missed during the Enumeration phase of your test, you might lose a vital attack vector which would have potentially cut your worktime on the target in half or even less.

After identifying that we have a stable connection with target IP (at least 4 successful pings), the next step is scanning all of the target's open ports to determine the
services running on it. In order to start the scanning process, we can use the following command with the **nmap** script. nmap stands for Network Mapper, and it will send requests to the target's ports in hopes of receiving a reply, thus determining if the said port is open or not. Some ports are used by default by certain
services. Others might be non-standard, which is why we will be using the service detection flag **-sV** to determine the name and description of the identified services.

`sudo nmap -sV <target IP>`

# 3. Gaining Access (a.k.a. Exploitation)
Running an exploit against a vulnerability.

# 4. Maintaining Access
Keeping us logged in and/or access to the target system.

# 5. Covering Tracks
Cleaning up any tracks.


# Passive Recon
## Physical
- Satellite images
- Drone recon
- Building layout (badge readers, break areas, security, fencing, etc.)

## Social
- Employees (name, job title, phone number, manager, etc.)
- Pictures (badge photos, desk photos, computer photos, etc.)

## Web / Host
- Target validation (WHOIS, nslookup, dnsrecon)
- Finding subdomains (Google Fu, dig, Nmap, Sublist3r, Bluto, crt.sh, etc.)
- Fingerprinting (Nmap, Wappalyzer, WhatWeb, BuiltWith, Netcat)
- Data breaches (HaveIBeenPwned, Breach-Parse, WeLeakInfo)

## Discovering Emails

| tool                                         | for                                              |
| -------------------------------------------- | ------------------------------------------------ |
| https://hunter.io/                           | finding email addresses of companies and domains |
| https://phonebook.cz/                        | finding email addresses for domains              |
| https://www.voilanorbert.com/                | email finder                                     |
| https://clearbit.com/resources/tools/connect | finding verified B2B emails, chrome extension    |
| https://tools.emailhippo.com/                | verifying found email addresses                  |
