# Linux
## Enumerating the Password Policy - Credentialed
```
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```
## Enumerating the Password Policy
### SMB NULL Sessions
```
rpcclient -U "" -N <ip>
```
```
querydominfo
```
### enum4linux
```
enum4linux -P 172.16.5.5
```
### enum4linux-ng
```
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```
### LDAP Anonymous Bind
```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```
### crackmapexec
```
crackmapexec smb ip --pas-pol -u '' -p ''
```
# Windows
## Enumerating Null Session
```
net use \\DC01\ipc$ "" /u:"”
```
## Enumerating the Password Policy
### PowerView
```
import-module .\PowerView.ps1
```
```
Get-DomainPolicy
```
```
net accounts
```