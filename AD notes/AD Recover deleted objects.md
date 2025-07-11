```
*Evil-WinRM* PS C:\\Users\\arksvc> Get-ADObject -filter 'isdeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property *
```
