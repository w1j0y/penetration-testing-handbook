## Vhosts vs. Sub-domains
The key difference between `VHosts` and sub-domains is that a `VHost` is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be serving two or more different websites. In many cases, many websites would actually have sub-domains that are not public and will not publish them in public DNS records, and hence if we visit them in a browser, we would fail to connect, as the public DNS would not know their IP. Once again, if we use the sub-domain fuzzing, we would only be able to identify public sub-domains but will not identify any sub-domains that are not public.
This is where we utilize VHosts Fuzzing on an IP we already have. We will run a scan and test for scans on the same IP, and then we will be able to identify both public and non-public sub-domains and VHosts.

## vHost Fuzzing

```
cat ./vhosts | while read vhost;do echo "\n********\nFUZZING: ${vhost}\n********";curl -s -I http://192.168.10.10 -H "HOST: ${vhost}.randomtarget.com" | grep "Content-Length: ";done
```

## Vhost Fuzzing ffuf
To fuzz vhosts, we must first figure out what the response looks like for a non-existent vhost. We can choose anything we want here; we just want to provoke a response, so we should choose something that very likely does not exist.
```
curl -s -I http://10.129.203.101 -H "HOST: defnotvalid.inlanefreight.local" | grep "Content-Length:"
```
 Then we use `--fs <content-length>`

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' --fs <content-length>
```

We need to add all the discovered vhosts to the hosts file to run the Extension Fuzzing on all of them.
We can create a custom wordlist using cewl
```
cewl -w wordlist.txt -m 1 http://10.10.10.188/author.html
```
```
ffuf -w ./wordlist.txt -u http://192.168.10.10 -H "HOST: FUZZ.randomtarget.com" -fs 612
```
## Vhost Fuzzing gobuster
```
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```
```
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
```
gobuster dir -u http://10.129.78.12 -w /usr/share/seclists/Discovery/DNS/namelist.txt
```
`-t` increase the number of threads for faster scanning
`-k` ignore SSL/TLS certificate errors
`-o` save the output to a file