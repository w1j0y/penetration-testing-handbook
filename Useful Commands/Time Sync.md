- To sync your attack host with the machine you are attacking (probably AD error), run the following commands
- If you are not currently running as the root user, switch to the root user by running the “su” command
- Run “timedatectl set-ntp off” to disable the Network Time Protocol from auto-updating
```
timedatectl set-ntp off
```
- Run “rdate -n [IP of Target]” to match your date and time with the date and time of the your target machine
```
rdate -n [IP of Target]
```
- The above command did not work while inputting the IP of an AD machine, it worked when i input the FQDN of the domain
[https://medium.com/@danieldantebarnes/fixing-the-kerberos-sessionerror-krb-ap-err-skew-clock-skew-too-great-issue-while-kerberoasting-b60b0fe20069](https://medium.com/@danieldantebarnes/fixing-the-kerberos-sessionerror-krb-ap-err-skew-clock-skew-too-great-issue-while-kerberoasting-b60b0fe20069)
