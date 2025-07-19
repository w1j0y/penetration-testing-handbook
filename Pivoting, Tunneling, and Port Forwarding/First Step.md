# First Step is performing a Ping Sweep
## Ping Sweep For Loop on Linux Pivot Hosts
```
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```
## Ping Sweep For Loop Using CMD on Windows Pivot Hosts
```
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```
## Ping Sweep Using PowerShell on Windows Pivot Hosts
```
1..254 | % {"172.16.6.$($_): $(Test-Connection -count 1 -comp 172.16.6.$($_) -quiet)"}
```