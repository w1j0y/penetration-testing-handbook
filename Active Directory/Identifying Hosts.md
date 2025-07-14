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
## Active
# fping
```
https://fping.org/
```
```
fping -asgq 172.16.5.0/23
```
nmap scan to the hosts discovered above