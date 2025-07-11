One common way to identify that is by finding the server type through the HTTP response headers and guessing the extension. For example, if the server is apache, then it may be .php, or if it was IIS, then it could be .asp or .aspx, and so on. There is one file we can always find in most websites, which is index, so we will use it as our file and fuzz extensions on it.
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```
