# Windows Host
The Windows attack host starts a plink.exe process with the below command-line arguments to start a dynamic port forward over the Ubuntu server. This starts an SSH session between the Windows attack host and the Ubuntu server, and then plink starts listening on port 9050.
```
plink -ssh -D 9050 ubuntu@10.129.15.50
```
## Windows Attack Host
After configuring the SOCKS server for 127.0.0.1 and port 9050, we can directly start mstsc.exe to start an RDP session with a Windows target that allows RDP connections.
## Accepting the Host Key Without a Prompt
First run always stops to ask about the host key, which is a problem if this needs to run non-interactively.
```
echo y | plink -ssh -D 9050 ubuntu@10.129.15.50
```
Or skip the prompt and pass the password inline with -batch, useful if this is going into a scheduled task rather than run by hand:
```
plink -batch -ssh -D 9050 ubuntu@10.129.15.50 -pw "P@ssw0rd123"
```
## Local Port Forward: Skipping the SOCKS Setup Entirely
If we only need one specific port (RDP to the DC, say) the SOCKS proxy step isn't necessary at all: plink can forward it directly:
```
plink -ssh -L 127.0.0.1:3389:172.16.5.19:3389 ubuntu@10.129.15.50 -N
```
```
mstsc.exe /v:127.0.0.1:3389
```
## SOCKS Through a Second Proxy
If the Windows host's SOCKS proxy on 9050 needs to be reachable from our Linux attack box instead of used directly on Windows, point proxychains at the Windows host's IP and port instead of localhost:
```
[ProxyList]
socks4  10.129.15.50  9050
```
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```