```
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```
We can even make it go faster if we are in a hurry by increasing the number of threads to 200, for example, with -t 200, but this is not recommended, especially when used on a remote site, as it may disrupt it, and cause a Denial of Service, or bring down your internet connection in severe cases.

# Gobuster
```
gobuster dir -u http://<target ip>/ -w /usr/share/dirb/wordlists/common.txt
```
