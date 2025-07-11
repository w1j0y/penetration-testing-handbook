# Host Discovery
```
sudo nmap 10.129.2.0/24 -sn
```
-sn Disables port scanning

## Host and Port Scanning
`-sT` uses the TCP three-way handshake to determine if a specific port on a target host is open or closed. The Connect scan is useful because it is the most accurate way to determine the state of a port, and it is also the most stealthy.
Using the following command to run a scan on all ports without taking too much time
```
nmap --min-rate 10000 -p- 10.129.201.127
```
```
nmap -sS -n -Pn -p- --min-rate 5000 IP
```
# Firewall and IDS/IPS Evasion
## Decoys -D
There are cases in which administrators block specific subnets from different regions in principle. This prevents any access to the target network. Another example is when IPS should block us. For this reason, the Decoy scanning method (-D) is the right choice.
```
sudo nmap <ip> -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```
`-sS`: Perform SYN scan on specified ports
`-Pn`: Disable ICMP Echo Requests
`-n`: Disable DNS resolution
`-D RND:5`: Generates five random IP addresses that indicates the souce IP the connection comes from.

## Scan by Using Different Source IP
Another scenario would be that only individual subnets would not have access to the server's specific services. So we can also manually specify the source IP address (-S) to test if we get better results with this one.
```
sudo nmap <ip> -n -Pn -p 445 -O -S <sourceip> -e tun0
```
`-S`: Scans the target by using different source IP address
`<sourceip>`: Specifies the source IP address (ex:10.129.2.200)
`-e tun0`: Sends all requests through the specified interface

## DNS Proxying
We can use TCP port 53 as a source port for our scans. If the administrator uses the firewall to control this port and does not filter IDS/IPS properly, our TCP packets will be trusted and passed through.
Example is running nmap scan and finding port 50000 as filtered, so we run the next command to find it open and then connect it to it
```
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```
Connect To The Filtered Port
```
ncat -nv --source-port 53 10.129.2.28 50000
```
