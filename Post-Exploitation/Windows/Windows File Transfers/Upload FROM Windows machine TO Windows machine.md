# PowerShell Session File Transfer
To create a PowerShell Remoting session on a remote computer, we will need administrative access, be a member of the Remote Management Users group, or have explicit permissions for PowerShell Remoting in the session configuration.
We have a session as Administrator in DC01, the user has administrative rights on DATABASE01, and PowerShell Remoting is enabled. Let's use Test-NetConnection to confirm we can connect to WinRM.
## From DC01 - Confirm WinRM port TCP 5985 is Open on DATABASE01.
```
PS C:\htb> Test-Connection -ComputerName DATABASE01 -Port 5985
```
Because this session already has privileges over DATABASE01, we don't need to specify credentials. In the example below, a session is created to the remote computer named DATABASE01 and stores the results in the variable named $Session.
## Create a PowerShell Remoting Session to DATABASE01
```
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```
We can use the Copy-Item cmdlet to copy a file from our local machine DC01 to the DATABASE01 session we have $Session or vice versa.
### Copy samplefile.txt from our Localhost to the DATABASE01 Session
#### Localhost —> DATABASE01
```
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```
### Copy DATABASE.txt from DATABASE01 Session to our Localhost
#### DATABASE01 —> Localhost
```
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```