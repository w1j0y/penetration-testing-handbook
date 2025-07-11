Crawling, often called spidering, is the automated process of systematically browsing the World Wide Web.
```
cewl -m5 --lowercase -w wordlist.txt http://192.168.10.10
```
```
ffuf -w ./folders.txt:FOLDERS,./wordlist.txt:WORDLIST,./extensions.txt:EXTENSIONS -u http://192.168.10.10/FOLDERS/WORDLIST/EXTENSIONS
```

## Well-Known URIs
The .well-known standard, defined in RFC 8615, serves as a standardized directory within a website's root domain. This designated location, typically accessible via the /.well-known/ path on a web server, centralizes a website's critical metadata, including configuration files and information related to its services, protocols, and security mechanisms.
The openid-configuration URI is part of the OpenID Connect Discovery protocol, an identity layer built on top of the OAuth 2.0 protocol. When a client application wants to use OpenID Connect for authentication, it can retrieve the OpenID Connect Provider's configuration by accessing the https://example.com/.well-known/openid-configuration endpoint. This endpoint returns a JSON document containing metadata about the provider's endpoints, supported authentication methods, token issuance.

## ReconSpider
```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
```
```
unzip ReconSpider.zip
```
```
python3 ReconSpider.py http://<URL>
```
After running ReconSpider.py, the data will be saved in a JSON file, `results.json`. This file can be explored using any text editor.

## Automating Recon
These frameworks aim to provide a complete suite of tools for web reconnaissance:
### FinalRecon
```
git clone https://github.com/thewhiteh4t/FinalRecon.git
```
```
cd FinalRecon
```
```
pip3 install -r requirements.txt
```
```
chmod +x ./finalrecon.py
```
```
./finalrecon.py --help
```

### Recon-ng
### theHarvester
### OSINT Framework
