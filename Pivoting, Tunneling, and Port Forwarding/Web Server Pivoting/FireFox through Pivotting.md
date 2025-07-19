1. Go to firefox `Settings`, `Network Settings`
2. Input the SOCKS host and Port, for example, if you logged in via ssh to a host machine and used ssh port forwarding to connect
```
ssh -D 9050 -i ~/.ssh/id_rsa root@10.129.194.108
```
⇒ Then you should be put the following settings
3. Select `Manual proxy configuration`: It can be either SOCKSv4 or v5
![[Pasted image 20250718115159.png]]
![[Pasted image 20250718115216.png]]
4. Then use the following command to open firefox through proxychains
```bash
proxychains firefox http://<ip:port>
```
5. You should delete the settings when you are done