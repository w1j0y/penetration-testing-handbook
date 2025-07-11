# ffuf
## Sub-domain Fuzzing
```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```
We see that we do not get any hits back. Does this mean that there are no sub-domain under `academy.htb`? - No. This means that there are no `public` sub-domains under `academy.htb`, as it does not have a public DNS record, as previously mentioned. Even though we did add `academy.htb` to our `/etc/hosts` file, we only added the main domain, so when `ffuf` is looking for other sub-domains, it will not find them in `/etc/hosts`, and will ask the public DNS, which obviously will not have them.
# Passive Subdomain Enumeration
## Automating Passive Subdomain Enumeration
### TheHarvester
To automate this, we will create a file called sources.txt with the following contents.
```
baidu 
bufferoverun 
crtsh 
hackertarget 
otx 
projectdiscovery 
rapiddns 
sublist3r 
threatcrowd 
trello 
urlscan 
vhost 
virustotal 
zoomeye
```
Once the file is created, we will execute the following commands to gather information from these sources.
```
cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
```
When the process finishes, we can extract all the subdomains found and sort them via the following command:
```
cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt”
```
Now we can merge all the passive reconnaissance files via:
```
cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt
```
```
cat facebook.com_subdomains_passive.txt | wc -l
```

### Certificate Transparency
```
curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"
```
```
openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\.||g' | tr -d ',' | sort -u
```
Example:
```
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | sort -u
```

# Active Subdomain Enumeration
## Zone Transfers
1. Identifying Nameservers
```
nslookup -type=NS zonetransfer.me
```
2. Perform the Zone transfer using -type=any and -query=AXFR parameters
```
nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja
```
3. Take every subdomain found in the first command and try to test for ANY and AXFR Zone Transfer


