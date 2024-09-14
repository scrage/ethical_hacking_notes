#burpsuite #proxy

## Set up a web proxy
- download [FoxyProxy](https://getfoxyproxy.org/) browser addon and in the Options, set up a new proxy for BurpSuite:
	- Hostname: 127.0.0.1
	- Type: HTTP
	- Port: 8080
- download the certificate from https://burp (while the proxy is being enabled in FoxyProxy) and import it into firefox
- set up a manual proxy under Network Settings for http and https protocols on 127.0.0.1:8080
- start BurpSuite