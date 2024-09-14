# Breached Credentials
https://hashes.com/en/decrypt/hash to decrypt a hash.

# Subdomains
- sublist3r for kali linux to hunt for subdomains
- any certificate checker
- OWASP AMASS for hunting subdomains

# Web Applications
- https://builtwith.com/ to check what technologies a web page or site was built with
- https://addons.mozilla.org/hu/firefox/addon/wappalyzer/ firefox addon 
- `whatweb` is a built-in command on kali linux for web scanning URLs, hostnames, IP addresses, filenames or IP ranges in CIDR, `x.x.x-x`, or `x.x.x.x-x.x.x.x` format
- BurpSuite is a web proxy on kali linux
	- download [FoxyProxy](https://getfoxyproxy.org/) browser addon and in the Options, set up a new proxy for BurpSuite:
		- Hostname: 127.0.0.1
		- Type: HTTP
		- Port: 8080
	- download the certificate from https://burp (while the proxy is being enabled in FoxyProxy) and import it into firefox
	- set up a manual proxy under Network Settings for http and https protocols on 127.0.0.1:8080
	- start BurpSuite
- Google search syntax is invaluable to finding all information

Social media like FB, twitter, or linkedin are gold mine for finding photos of badges, names of employees and managers, credentials and other social engineering related information.