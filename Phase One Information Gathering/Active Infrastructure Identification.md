## HTTP Headers
```
curl -I http://10.129.79.254 -H "Host: dev.inlanefreight.local"
```
`X-Powered-By header`: This header can tell us what the web app is using. We can see values like PHP, ASP.NET, JSP, etc.
`Cookies`: Cookies are another attractive value to look at as each technology by default has its cookies. Some of the default cookie values are:
- `PHP`: `PHPSESSID=<COOKIE_VALUE>`
- `JAVA`: `JSESSION=<COOKIE_VALUE>`
- `.NET`: `ASPSESSIONID<RANDOM>=<COOKIE_VALUE>`

## WhatWeb
```
whatweb -a3 https://www.facebook.com -v
```

## WafW00f
WafW00f is a web application firewall (WAF) fingerprinting tool that sends requests and analyses responses to determine if a security solution is in place. We can install it with the following command:
```
sudo apt install wafw00f -y
```

## Aquatone
Aquatone is a tool for automatic and visual inspection of websites across many hosts and is convenient for quickly gaining an overview of HTTP-based attack surfaces by scanning a list of configurable ports, visiting the website with a headless Chrome browser, and taking a screenshot.
```
sudo apt install golang chromium-driver
```
```
go install github.com/michenriksen/aquatone@latest
```
```
export PATH="$PATH":"$HOME/go/bin"
```
⇒ To download aquatone for kali linux
```
https://medium.com/@sherlock297/install-aquatone-on-kali-linux-dd2a6850fd32
```
Now, it's time to use cat in our subdomain list and pipe the command to aquatone via:
```
cat facebook_aquatone.txt | aquatone -out ./aquatone -screenshot-timeout 1000
```
