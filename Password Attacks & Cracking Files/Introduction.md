# Linux
/etc/passwd: The x in the password field indicates that the encrypted password is in the /etc/shadow file. However, the redirection to the /etc/shadow file does not make the users on the system invulnerable because if the rights of this file are set incorrectly, the file can be manipulated so that the user root does not need to type a password to log in. Therefore, an empty field means that we can log in with the username without entering a password.
# Windows
SAM Database: Credentials are encrypted and stored at the following location:
```
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```
# John The Ripper
## Attack Methods
### Dictionary Attacks
Dictionary attacks involve using a pre-generated list of words and phrases (known as a dictionary) to attempt to crack a password. The dictionary is then used to generate a series of strings which are then used to compare against the hashed passwords. If a match is found, the password is cracked, providing an attacker access to the system and the data stored within it.
### Brute Force Attacks
Brute force attacks involve attempting every conceivable combination of characters that could form a password. This is an extremely slow process, and using this method is typically only advisable if there are no other alternatives.
### Rainbow Table Attacks
Rainbow table attacks involve using a pre-computed table of hashes and their corresponding plaintext passwords, which is a much faster method than a brute-force attack.
Rainbow table attacks are only effective against hashes already present in the table, making the larger the table, the more successful the attack.

## Cracking Modes
### Single Crack Mode
Single Crack Mode is one of the most common John modes used when attempting to crack passwords using a single password list. It is a brute-force attack, meaning all passwords on the list are tried, one by one, until the correct one is found. This method is the most basic and straightforward way of cracking passwords and is thus a popular choice for those wishing to gain access to a secure system.
```
john --format=<hash_type> <hash or hash_file>
```
Download the cheat sheet from the module
### Wordlist Mode
Wordlist Mode is used to crack passwords using multiple lists of words. It is a dictionary attack which means it will try all the words in the lists one by one until it finds the right one. It is generally used for cracking multiple password hashes using a wordlist or a combination of wordlists. It is more effective than Single Crack Mode because it utilizes more words but is still relatively basic. The basic syntax for the command is:
```
john --wordlist=<wordlist_file> --rules <hash_file>
```
### Incremental Mode
`Incremental Mode` is an advanced John mode used to crack passwords using a character set. It is a hybrid attack, which means it will attempt to match the password by trying all possible combinations of characters from the character set. This mode works best when we know what the password might be, as it will try all the possible combinations in sequence, starting from the shortest one. Incremental mode generates the guesses on the fly, while wordlist mode uses a predefined list of words. At the same time, the single crack mode is used to check a single password against a hash.
```
john --incremental <hash_file>
```
## Cracking Files
It is also possible to crack even password-protected or encrypted files with John. We use additional tools that process the given files and produce hashes that John can work with. It automatically detects the formats and tries to crack them. The syntax for this can look like this:
```
<tool> <file_to_crack> > file.hash
```
```
pdf2john server_doc.pdf > server_doc.hash
```
Two commands available
```
john server_doc.hash
```
```
john --wordlist=<wordlist.txt> server_doc.hash
```
More of these tools can be found on Pwnbox in the following way:
```
locate *2john*
```
