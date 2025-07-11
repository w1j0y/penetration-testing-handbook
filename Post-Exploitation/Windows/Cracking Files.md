# Protected Files
## Hunting for Encoded Files
```
cry0l1t3@unixclient:~$ for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
## Hunting for SSH Keys
```
grep -rnw "PRIVATE KEY" /* 2>/dev/null | grep ":1"
```
## Cracking with John
```
locate *2john*
```
We can convert many different formats into single hashes and try to crack the passwords with this. Then, we can open, read, and use the file if we succeed. There is a Python script called ssh2john.py for SSH keys, which generates the corresponding hashes for encrypted SSH keys, which we can then store in files.
```
ssh2john.py SSH.private > ssh.hash
```
### Cracking SSH Keys
```
john --wordlist=rockyou.txt ssh.hash
```
```
john ssh.hash --show
```
## Cracking Documents
John provides a Python script called office2john.py to extract hashes from all common Office documents that can then be fed into John or Hashcat for offline cracking. The procedure to crack them remains the same.
### Cracking Microsoft Office Documents
```
office2john.py Protected.docx > protected-docx.hash
```
```
cat protected-docx.hash
```
```
john --wordlist=rockyou.txt protected-docx.hash
```
```
john protected-docx.hash --show
```
### Cracking PDFs
```
pdf2john.py PDF.pdf > pdf.hash
```
```
cat pdf.hash
```
```
john --wordlist=rockyou.txt pdf.hash
```
```
john pdf.hash --show
```
# Protected Archives
## Download All File Extensions
```
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```
## Cracking Archives
### Cracking ZIP Using zip2john
```
zip2john ZIP.zip > zip.hash
```
### Viewing the Contents of zip.hash
```
cat zip.hash
```
### Viewing the Contents of zip.hash
```
john --wordlist=rockyou.txt zip.hash
```
### Viewing the Cracked Hash
```
d0x777@htb[/htb]$ john zip.hash --show
```
## Cracking OpenSSL Encrypted Archives
### file GZIP.gzip
When cracking OpenSSL encrypted files and archives, we can encounter many different difficulties that will bring many false positives or even fail to guess the correct password. Therefore, the safest choice for success is to use the openssl tool in a for-loop that tries to extract the files from the archive directly if the password is guessed correctly.
### Using a for-loop to Display Extracted Contents
```
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```
Once the for-loop has finished, we can look in the current folder again to check if the cracking of the archive was successful.
## Cracking BitLocker Encrypted Drives
BitLocker is an encryption program for entire partitions and external drives.
Again, we can use a script called `bitlocker2john` to extract the hash we need to crack. Four different hashes will be extracted, which can be used with different Hashcat hash modes. For our example, we will work with the first one, which refers to the BitLocker password.
### Using bitlocker2john
```
bitlocker2john -i Backup.vhd > backup.hashes
```
```
grep "bitlocker\$0" backup.hashes > backup.hash
```
```
cat backup.hash
```
Both John and Hashcat can be used for this purpose. This example will look at the procedure with Hashcat. The Hashcat mode for cracking BitLocker hashes is -m 22100
#### Using hashcat to Crack backup.hash
```
hashcat -m 22100 backup.hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt -o backup.cracked
```
#### Viewing the Cracked Hash
```
cat backup.cracked
```
#### Using John
```
john --format=bitlocker-opencl --wordlist=<pathtowordlist> hashes.txt
```
#### Then mount the Backup.vhd using the following commands
```
sudo mkdir /media/backup_bitlocker /media/mount
```
```
sudo losetup -P /dev/loop100 <your_file.vhd>
```
```
sudo dislocker -v -V /dev/loop100p2 -u -- /media/backup_bitlocker
```
```
sudo mount -o loop,rw /media/backup_bitlocker/dislocker-file /media/mount
```
```
ls -la /media/mount
```
After mounting the virtual disk, we have the SAM and System file so we used the following command to crack them
```
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save LOCAL
```
# id_rsa.bak
```
ssh2john id_rsa.bak > hash.txt
```
```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```