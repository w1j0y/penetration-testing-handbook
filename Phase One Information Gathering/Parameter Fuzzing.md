## Parameter Fuzzing - GET
Similarly to how we have been fuzzing various parts of a website, we will use ffuf to enumerate parameters. Let us first start with fuzzing for GET requests, which are usually passed right after the URL, with a ? symbol, like:
```
http://admin.academy.htb>:PORT/admin/admin.php?param1=key.
```
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

## Parameter Fuzzing - POST
The main difference between POST requests and GET requests is that POST requests are not passed with the URL and cannot simply be appended after a ? symbol. POST requests are passed in the data field within the HTTP request. Check out the Web Requests module to learn more about HTTP requests.
So, let us repeat what we did earlier, but place our FUZZ keyword after the -d flag:
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
As we can see this time, we got a couple of hits, the same one we got when fuzzing GET and another parameter, which is id. Let's see what we get if we send a POST request with the id parameter. We can do that with curl, as follows:
```
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```
We get nothing so we need to do value fuzzing

## Value Fuzzing
**Custom Wordlist**
There are many ways to create this wordlist, from manually typing the IDs in a file, or scripting it using Bash or Python. The simplest way is to use the following command in Bash that writes all numbers from 1-1000 to a file:
```
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```
**Value Fuzzing**
Our command should be fairly similar to the POST command we used to fuzz for parameters, but our FUZZ keyword should be put where the parameter value would be, and we will use the ids.txt wordlist we just created, as follows:
```
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
