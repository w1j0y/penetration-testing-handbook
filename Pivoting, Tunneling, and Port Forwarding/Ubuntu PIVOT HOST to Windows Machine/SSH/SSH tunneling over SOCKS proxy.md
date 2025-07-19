# Attack Host
The attack host starts the SSH client and requests the SSH server to allow it to send some TCP data over the ssh socket. The SSH server responds with an acknowledgment, and the SSH client then starts listening on localhost:9050. Whatever data you send here will be broadcasted to the entire network (172.16.5.0/23) over SSH. We can use the below command to perform this dynamic port forwarding.
## Enabling Dynamic Port Forwarding with SSH
```
w1j0y@htb[/htb]$ ssh -D 9050 ubuntu@10.129.202.64
```
The -D argument requests the SSH server to enable dynamic port forwarding. Once we have this enabled, we will require a tool that can route any tool's packets over the port 9050
## Checking /etc/proxychains.conf
```
socks4 127.0.0.1 9050
```
## Tunneling workflow
### Using Nmap with Proxychains on our Attack Host
```
proxychains nmap -v -sT -Pn 172.16.5.1-200
```
### Route all the packets of nmap to the local port 9050 -> SSH client listening and will forwards all the packets over SSH to the 172.16.5.0/23 network (on Ubuntu Pivot Host) -> Scan results sent back to our Host from WIndows Host (172.16.5.0/23 network)
## Enumerating the Windows Target through Proxychains
One more important note to remember here is that we can only perform a full TCP connect scan over proxychains. The reason for this is that proxychains cannot understand partial packets. If you send partial packets like half connect scans, it will return incorrect results. We also need to make sure we are aware of the fact that host-alive checks may not work against Windows targets because the Windows Defender firewall blocks ICMP requests (traditional pings) by default.
Enumerating the Windows Target through Proxychains
```
proxychains nmap -v -Pn -sT 172.16.5.19
```
## Using Metasploit with Proxychains
```
w1j0y@htb[/htb]$ proxychains msfconsole
```
```
search rdp_scanner
```
## Using xfreerdp with Proxychains
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```