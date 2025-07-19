# Port Forwarding with Windows netsh
We can use netsh.exe to forward all data received on a specific port (say 8080) to a remote host on a remote port. This can be performed using the below command.
## Attack Host
1. RDP to the Windows Pivot Host machine: internal network (172.16.5.19) and external network (10.129.15.198)
## Windows Pivot Host 
1. Using Netsh.exe to Port Forward
```
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.25
```
3. The Windows Server (172.16.5.25) on the internal network will be reached by the Attack host by the port forward from netsh.exe
### Verifying Port Forward
```
netsh.exe interface portproxy show v4tov4
```
```
xfreerdp /v:10.129.42.198:8080 /u:victor /p:pass@123
```
