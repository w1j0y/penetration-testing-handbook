**All Adminer versions between 1.12.0 and 4.6.2 (included) are vulnerable:**
[https://podalirius.net/en/articles/writing-an-exploit-for-adminer-4.6.2-arbitrary-file-read-vulnerability/](https://podalirius.net/en/articles/writing-an-exploit-for-adminer-4.6.2-arbitrary-file-read-vulnerability/)
[https://github.com/p0dalirius/CVE-2021-43008-AdminerRead](https://github.com/p0dalirius/CVE-2021-43008-AdminerRead)
Improper Access Control in Adminer versions <= 4.6.2 (fixed in version 4.6.3) allows an attacker to achieve Arbitrary File Read on the server by connecting a remote MySQL database to the Adminer.

We can turn to the Internet and find an [article](https://medium.com/bugbountywriteup/adminer-script-results-to-pwning-server-private-bug-bounty-program-fe6d8a43fe6f) about being able to read arbitrary files from the file system. If we connect to a dummy database on our local machine, we can take advantage of the fact that SQL is able to read local files which would actually be from the remote machine. By starting the `mysql` service on our local machine and allowing `admirer.htb` to connect to it, we can get in to the Adminer service.

**Adminer version on the server is 4.7.8**
[https://ine.com/blog/adminer-ssrf-vulnerability-cve-202121311](https://ine.com/blog/adminer-ssrf-vulnerability-cve-202121311)
