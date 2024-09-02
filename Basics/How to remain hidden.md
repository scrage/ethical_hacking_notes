# 1. Physical Security (Operations Security)
The very first line of defense in keeping our anonymity and security is Operations Security.

**Operations Security** (**OPSEC**) is the process by which we protect critical information whether it is classified or unclassified that can be used against us. It focuses on preventing our adversaries' access to information and actions that may compromise an operation.

Our online and offline habits and public interactions, even the most inconspicuous ones, can reveal our identity. We must always be mindful our writing style, social media posts, where we use and connect our computers, and social interactions in general.

# 2. The laptop
- A laptop purchased with the most untraceable and mobile way possible (e.g. buying it with a privacy-focused cryptocurrency (monero, zcash)).
- Once the laptop is acquired, wipe its OS and buy a USB stick to pre-load a live OS install. Still enable full disk encryption on the laptop, just in case of physical compromise.

# 3. Anonymity in network connections
To anonymize our identity in network connections, consider the following:
- Avoid using any type of unique or pseudo-unique identifiers.
- Never reveal any of your devices' Media Access Control (MAC) address publicly. (Randomize MAC address.)
- Anonymize IP address (VPNs (e.g. OpenVPN and tailscale are open source projects), TOR, proxies (e.g. proxychain) - but with caution, as each of these methods introduce intermediaries with assumptions of complete trust).

# 4. Separation of Environments
To make sure that we are separating our work, day-to-day environment from our hacking environment.
- Virtualization and containerization with the use of virtual machines.
- Bouncing servers method: ssh into the server environment after anonymization. Even if the environment is compromised, a new one can be curried up within minutes.
- Randomizing network connections with public wifis ([wifimap.io](https://www.wifimap.io) to search for public wifis locally).

# 5. Covering up tracks
- Limit offensive activities. - Generate as few logs and footprint as possible.
- Blend probing, offensive or fraudulent activities with common network connections and protocols such as DNS tunneling (e.g. dnscat2).