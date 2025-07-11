## crt.sh 
Find subdomains
```
curl -s https://crt.sh/\\?q\\=example.com\\&output\\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
Save them into a list called subdomainlist then determine the ip address for each one
```
for i in $(cat subdomainlist);do host $i | grep "has address" | grep example.com | cut -d" " -f1,4;done
```

## Shodan
Shodan can be used to find devices and systems permanently connected to the Internet like Internet of Things (IoT)
```
for i in $(cat subdomainlist);do host $i | grep "has address" | grep example.com | cut -d" " -f4 >> ip-addresses.txt;done
```
```
for i in $(cat ip-addresses.txt);do shodan host $i;done
```

## DNS Records
```
dig any example.com
```

## whois
```
whois <IP>
```
