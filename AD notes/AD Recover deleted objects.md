# Enumerating Recently Deleted Objects
```
*Evil-WinRM* PS C:\\Users\\arksvc> Get-ADObject -filter 'isdeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property *
```
Narrowing to objects deleted in the last 90 days and pulling out the fields worth mining for leftover creds or prior group membership:
```
Get-ADObject -filter 'isdeleted -eq $true -and whenChanged -gt (Get-Date).AddDays(-90)' -includeDeletedObjects -property samaccountname,lastKnownParent,description,memberOf,whenChanged
```
A deleted user's `description` field or former `memberOf` list often outlives the account itself, worth checking even when the account isn't worth restoring.

# Restoring a Deleted Object
Requires Reanimate-Tombstone rights over the deleted object, or membership in a group that has them (Domain Admins by default).
```
Restore-ADObject -Identity "<ObjectGUID or DistinguishedName>"
```
Restored objects come back disabled, in their original OU (or Deleted Objects if that OU no longer exists), with group memberships intact. A restored user can be re-enabled and lands straight back in whatever privileged group it used to belong to.

# Enabling the AD Recycle Bin
Objects are only restorable this way if the AD Recycle Bin feature is enabled on the forest. Worth flagging as a finding if it's off: deleted objects otherwise sit as tombstones for 180 days by default with most attributes already stripped, which is worse for us and for the business.
```
Get-ADOptionalFeature -Filter 'name -like "Recycle Bin Feature"' | select Name,EnabledScopes
```
```
Enable-ADOptionalFeature 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target <forest-fqdn>
```
