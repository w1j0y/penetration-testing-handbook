1. Capture the request and save it into a file called upload.req
2. Add the FUZZ at the location of the extension
```
ffuf -request upload.req -request-proto http -w /usr/share/seclists/Fuzzing/extensions-most-common.fuzz.txt -mr success
```
3. The word success is displayed in the response when a file is successfully uploaded