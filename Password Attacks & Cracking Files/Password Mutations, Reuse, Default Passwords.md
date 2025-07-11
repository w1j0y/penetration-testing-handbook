# Hashcat
Hashcat uses a specific syntax for defining characters and words and how they can be modified
```
: Do nothing
l Lowercase all letters
U Uppercase all letters
C Capitalize the first letter and lowercase others
sXY Replace all intances of X with U
$! Add the exclamation character at the end
```
## Hashcat Rule File
```
cat custom.rule
:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```
Hashcat will apply the rules of custom.rule for each word in password.list and store the mutated version in our mut_password.list accordingly.
### Generating Rule-based Wordlist
```
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```
We could apply the following command to generate a wordlist based on a single word
```
hashcat --stdout <filecontainingtheword> -r /usr/share/hashcat/rule/base64.rule > pwlist
```
Hashcat and John come with pre-built rule lists that we can use for our password generating and cracking purposes. One of the most used rules is best64.rule, which can often lead to good results. It is important to note that password cracking and the creation of custom wordlists is a guessing game in most cases. 
```
ls /usr/share/haschat/rules/
```
# CeWL
We can now use another tool called CeWL to scan potential words from the company's website and save them in a separate list. We can then combine this list with the desired rules and create a customized password list that has a higher probability of guessing a correct password. We specify some parameters, like the depth to spider (-d), the minimum length of the word (-m), the storage of the found words in lowercase (--lowercase), as well as the file where we want to store the results (-w).
## Generating Wordlists Using CeWL
```
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```
# Password Reuse / Default Passwords
## Credential Stuffing
```
https://github.com/ihebski/DefaultCreds-cheat-sheet
```
## Credential Stuffing - Hydra Syntax
```
hydra -C <user_pass.list> <protocol>://<IP>
```
## Google Search - Default Credentials
```
https://raw.githubusercontent.com/ihebski/DefaultCreds-cheat-sheet/main/DefaultCreds-Cheat-Sheet.csv
```
Or we can use the following command after installing the cheat sheet
```
pip3 install defaultcreds-cheat-sheet
```