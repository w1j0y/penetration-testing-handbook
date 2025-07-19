Pivoting's primary use is to defeat segmentation (both physically and virtually) to access an isolated network. Tunneling, on the other hand, is a subset of pivoting. Tunneling encapsulates network traffic into another protocol and routes traffic through it. 
# Pivoting
Utilizing multiple hosts to cross network boundaries you would not usually have access to. This is more of a targeted objective. The goal here is to allow us to move deeper into a network by compromising targeted hosts or infrastructure.
# Tunneling
We often find ourselves using various protocols to shuttle traffic in/out of a network where there is a chance of our traffic being detected. For example, using HTTP to mask our Command & Control traffic from a server we own to the victim host. The key here is obfuscation of our actions to avoid detection for as long as possible.
# Routing
```
netstat -r
```
# Protocols, Services & Ports
Protocols are the rules that govern network communications. Many protocols and services have corresponding ports that act as identifiers. When we see an IP address, we know it identifies a computer that may be reachable over a network. When we see an open port bound to that IP address, we know that it identifies an application we may be able to connect to. Connecting to specific ports that a device is listening on can often allow us to use ports & protocols that are permitted in the firewall to gain a foothold on the network.