# Bash Pitfalls

> **Fonte**: BashPitfalls - Greg's Wiki (mywiki.wooledge.org)\
> **Última Atualização**: Agosto 2025\
> **Autor**: Greg Wooledge (GreyCat)

### Introdução

Este é um guia abrangente dos erros mais comuns cometidos por programadores Bash. Cada exemplo mostrado aqui contém falhas de alguma forma. O objetivo é ajudar desenvolvedores a evitar essas armadilhas e escrever scripts Bash mais seguros e confiáveis.

***

### Detalhamento dos Pitfalls

## 1. for f in $(ls \*.mp3) {#1-for-f-in-ls-mp3}

**❌ Problema:**

```bash
for f in $(ls *.mp3); do # Errado!
    some command $f      # Errado!
done
```

**🔍 Por que está errado:**

1. **Word Splitting**: Nomes de arquivos com espaços são divididos
2. **Globbing**: Caracteres como `*` são expandidos
3. **Parsing**: Não há como distinguir onde um nome termina e outro começa
4. **ls behavior**: O comando `ls` pode alterar nomes de arquivos
5. **Newlines**: Arquivos podem conter quebras de linha
6. **Leading dashes**: Arquivos começando com `-` causam problemas

**✅ Solução correta:**

```bash
# Para arquivos no diretório atual
for file in ./*.mp3; do
    [ -e "$file" ] || continue  # Proteção se não há arquivos
    some command "$file"
done

# Para busca recursiva
find . -type f -name '*.mp3' -exec some command {} \;

# Ou com process substitution (Bash)
while IFS= LC_ALL=C read -r -d '' file; do
    some command "$file"
done < <(find . -type f -name '*.mp3' -print0)
```

## 2. cp $file $target {#2-cp-file-target}

**❌ Problema:**

```bash
cp $file $target  # Sem aspas!
```

**🔍 Por que está errado:**

* **Word Splitting**: Variáveis sem aspas sofrem divisão de palavras
* **Pathname expansion**: Caracteres como `*`, `?`, `[` são expandidos
* **Leading dashes**: Arquivos começando com `-` são interpretados como opções

**✅ Solução correta:**

```bash
cp -- "$file" "$target"  # Aspas duplas obrigatórias + -- para segurança
```

## 3. Filenames with leading dashes {#3-filenames-with-leading-dashes}

**❌ Problema:**

```bash
cp $filename /destination/  # Se filename = "-rf", vira cp -rf /destination/
```

**✅ Soluções:**

```bash
# Opção 1: Use -- para indicar fim das opções
cp -- "$filename" /destination/

# Opção 2: Use caminhos relativos
for i in ./*.mp3; do
    cp "$i" /target/
done

# Opção 3: Prefixe com ./
cp "./$filename" /destination/
```

## 4. \[ $foo = "bar" ] {#4-foo-bar}

**❌ Problema:**

```bash
[ $foo = "bar" ]  # Variável sem aspas
```

**🔍 Problemas:**

* Se `$foo` está vazio: `[ = "bar" ]` → erro "unary operator expected"
* Se `$foo` tem espaços: `[ hello world = "bar" ]` → muitos argumentos

**✅ Soluções:**

```bash
# POSIX compatível
[ "$foo" = bar ]

# Bash moderno (recomendado)
[[ $foo == bar ]]

# Forma antiga (compatibilidade extrema)
[ x"$foo" = xbar ]
```

## 5. cd $(dirname "$f") {#5-cd-dirname-f}

**❌ Problema:**

```bash
cd $(dirname "$f")  # Command substitution sem aspas
```

**✅ Solução:**

```bash
cd -P -- "$(dirname -- "$f")"
```

**🔍 Explicação das aspas aninhadas:**

* Aspas dentro de `$()` são um par separado
* Aspas fora de `$()` são outro par separado
* O parser trata como níveis de aninhamento diferentes

## 6. \[ "$foo" = bar && "$bar" = foo ] {#6-foo-bar-bar-foo}

**❌ Problema:**

```bash
[ "$foo" = bar && "$bar" = foo ]  # && não funciona dentro de [ ]
```

**✅ Soluções:**

```bash
# POSIX - dois comandos separados
[ "$foo" = bar ] && [ "$bar" = foo ]

# Bash - use [[ ]]
[[ $foo = bar && $bar = foo ]]
```

**❌ Evite (obsoleto):**

```bash
[ "$foo" = bar -a "$bar" = foo ]  # -a é obsoleto no POSIX
```

## 7. \[\[ $foo > 7 ]] {#7-foo-7}

**❌ Problema:**

```bash
[[ $foo > 7 ]]  # Comparação lexicográfica, não numérica!
```

**✅ Soluções:**

```bash
# Bash - aritmética
((foo > 7))

# Comparação numérica com [[ ]]
[[ foo -gt 7 ]]

# POSIX
[ "$foo" -gt 7 ]
```

**⚠️ Cuidado com injeção de código:** Se `$foo` vem de fonte externa não confiável, use apenas `[ "$foo" -gt 7 ]`.

## 8. grep foo bar | while read -r; do ((count++)); done {#8-grep-while-read}

**❌ Problema:**

```bash
count=0
grep foo bar | while read -r; do 
    ((count++))
done
echo $count  # Ainda é 0!
```

**🔍 Por que:** O `while` roda em uma subshell devido ao pipe, então mudanças em `count` não persistem.

**✅ Soluções:**

```bash
# Opção 1: Process substitution
count=0
while read -r; do 
    ((count++))
done < <(grep foo bar)

# Opção 2: Use here-string
count=0
while read -r; do 
    ((count++))
done <<< "$(grep foo bar)"

# Opção 3: Use um contador diferente
count=$(grep -c foo bar)
```

## 9. if \[grep foo myfile] {#9-if-grep-foo-myfile}

**❌ Problema:**

```bash
if [grep foo myfile]; then  # [ não é parte da sintaxe do if!
```

**🔍 Entendimento incorreto:** Muitos iniciantes pensam que `[` é parte da sintaxe do `if`, mas `[` é um comando!

**✅ Solução:**

```bash
if grep -q foo myfile; then
    # ...
fi
```

## 10. if \[bar="$foo"]; then … {#10-if-bar-foo}

**❌ Problema:**

```bash
if [bar="$foo"]; then  # Sem espaços!
```

**✅ Solução:**

```bash
if [ bar = "$foo" ]; then    # Espaços obrigatórios
if [[ bar = "$foo" ]]; then  # Espaços obrigatórios
```

**🔍 Explicação:** `[` é um comando que precisa de espaços entre seus argumentos, como qualquer comando.

## 11. if \[ \[ a = b ] && \[ c = d ] ]; then … {#11-if-a-b-c-d}

**❌ Problema:**

```bash
if [ [ a = b ] && [ c = d ] ]; then  # [ não é para agrupamento!
```

**✅ Soluções:**

```bash
# Dois comandos separados
if [ a = b ] && [ c = d ]; then

# Bash com [[ ]]
if [[ a = b && c = d ]]; then
```

## 12. read $foo {#12-read-foo}

**❌ Problema:**

```bash
read $foo  # $ não é usado com read!
```

**✅ Solução:**

```bash
read foo           # Simples
IFS= read -r foo   # Mais seguro
```

## 13. cat file | sed s/foo/bar/ > file {#13-cat-sed-same-file}

**❌ Problema:**

```bash
cat file | sed s/foo/bar/ > file  # Lê e escreve o mesmo arquivo!
```

**🔍 Resultado:** O arquivo é truncado antes da leitura, resultando em perda de dados.

**✅ Soluções:**

```bash
# Arquivo temporário
sed 's/foo/bar/g' file > tmpfile && mv tmpfile file

# GNU sed
sed -i 's/foo/bar/g' file

# Perl
perl -pi -e 's/foo/bar/g' file

# Editor ed
printf '%s\n' ',s/foo/bar/g' w q | ed -s file
```

## 14. echo $foo {#14-echo-foo}

**❌ Problema:**

```bash
echo $foo  # Sem aspas = word splitting + globbing
```

**✅ Soluções:**

```bash
echo "$foo"    # Bom
printf '%s\n' "$foo"  # Melhor (mais portável)
```

**🔍 Demonstração:**

```bash
foo="*.txt"
echo $foo     # Lista arquivos .txt
echo "$foo"   # Mostra literal "*.txt"
```

## 15. $foo=bar {#15-foo-bar-assignment}

**❌ Problema:**

```bash
$foo=bar  # $ não é usado em atribuições!
```

**✅ Solução:**

```bash
foo=bar        # Correto
foo="bar"      # Ainda melhor
```

## 16. foo = bar {#16-foo-bar-spaces}

**❌ Problema:**

```bash
foo = bar  # Espaços não são permitidos!
```

**✅ Solução:**

```bash
foo=bar     # Sem espaços
foo="bar"   # Com aspas para segurança
```

## 17. echo <\<EOF {#17-echo-eof}

**❌ Problema:**

```bash
echo <<EOF     # echo não lê stdin!
Hello world
EOF
```

**✅ Soluções:**

```bash
# Use cat
cat <<EOF
Hello world
EOF

# Ou aspas multilinha
echo "Hello world
How's it going?"

# Ou printf
printf '%s\n' "Hello world" "How's it going?"
```

## 18. su -c 'some command' {#18-su-c-some-command}

**❌ Problema:**

```bash
su -c 'some command'  # Falta o usuário!
```

**✅ Solução:**

```bash
su root -c 'some command'  # Usuário explícito
```

## 19. cd /foo; bar {#19-cd-foo-bar}

**❌ Problema:**

```bash
cd /foo; bar  # Se cd falhar, bar roda no diretório errado!
```

**✅ Soluções:**

```bash
# Verificação explícita
cd /foo && bar

# Ou com tratamento de erro
cd /foo || exit 1
bar

# Ou com subshell
(cd /foo && bar)
```

## 20. \[ bar == "$foo" ] {#20-bar-foo-double-equals}

**❌ Problema:**

```bash
[ bar == "$foo" ]  # == não é POSIX em [ ]
```

**✅ Soluções:**

```bash
[ bar = "$foo" ]     # POSIX
[[ bar == "$foo" ]]  # Bash
```

## 21. for i in {1..10}; do ./something &; done {#21-for-i-background}

**❌ Problema:**

```bash
for i in {1..10}; do ./something &; done  # ; após & é inválido
```

**✅ Soluções:**

```bash
for i in {1..10}; do ./something & done

# Ou
for i in {1..10}; do 
    ./something &
done
```

## 22. cmd1 && cmd2 || cmd3 {#22-cmd1-cmd2-cmd3}

**❌ Problema:**

```bash
# ERRADO! Não é equivalente a if-then-else
cmd1 && cmd2 || cmd3  # Se cmd2 falhar, cmd3 também executa!
```

**🔍 Demonstração:**

```bash
i=0
true && ((i++)) || ((i--))  # i++ executa, mas retorna 0 (false)
echo $i  # Resultado: 0 (ambos executaram!)
```

**✅ Solução:**

```bash
if cmd1; then
    cmd2
else
    cmd3
fi
```

## 23. echo "Hello World!" {#23-echo-hello-world}

**❌ Problema (apenas shells interativos):**

```bash
echo "Hello World!"  # ! pode triggerar history expansion
```

**✅ Soluções:**

```bash
echo 'Hello World!'              # Aspas simples
echo "Hello World"!              # ! fora das aspas
set +H                           # Desabilita history expansion
echo "Hello World!"
```

## 24. for arg in $\* {#24-for-arg-in-star}

**❌ Problema:**

```bash
for arg in $*; do  # Word splitting incorreto
```

**✅ Soluções:**

```bash
for arg in "$@"; do    # Forma correta
for arg; do            # Forma abreviada (equivalente)
```

## 25. function foo() {#25-function-foo}

**❌ Problema:**

```bash
function foo() {  # Mistura sintaxes diferentes
```

**✅ Soluções:**

```bash
foo() {        # POSIX (recomendado)
    # ...
}

function foo {  # Bash (válido mas menos portável)
    # ...
}
```

## 26. echo "\~" {#26-echo-tilde}

**❌ Problema:**

```bash
echo "~"  # ~ entre aspas não expande
```

**✅ Soluções:**

```bash
echo ~             # Sem aspas
echo "$HOME"       # Mais portável
echo "${HOME}"     # Ainda melhor
```

## 27. local var=$(cmd) {#27-local-var-cmd}

**❌ Problemas:**

```bash
local var=$(cmd)  # Exit status de cmd é perdido
```

**✅ Solução:**

```bash
local var
var=$(cmd)
rc=$?
```

## 28. export foo=\~/bar {#28-export-foo-tilde}

**❌ Problema:**

```bash
export foo=~/bar  # ~ pode não expandir em alguns shells
```

**✅ Soluções:**

```bash
foo=~/bar; export foo           # Seguro
export foo="$HOME/bar"          # Mais portável  
export foo="${HOME%/}/bar"      # Evita // se HOME=/
```

## 29. sed 's/$foo/good bye/' {#29-sed-variable-substitution}

**❌ Problema:**

```bash
sed 's/$foo/good bye/'  # Aspas simples = sem expansão
```

**✅ Solução:**

```bash
sed "s/$foo/good bye/"  # Aspas duplas
```

## 30. tr \[A-Z] \[a-z] {#30-tr-a-z}

**❌ Problemas:**

```bash
tr [A-Z] [a-z]  # 1. Globbing dos [ ]
                # 2. Notação incorreta para tr
                # 3. Problemas de locale
```

**✅ Soluções:**

```bash
# Para ASCII apenas
LC_COLLATE=C tr A-Z a-z

# Para locale-aware
tr '[:upper:]' '[:lower:]'
```

## 31. ps ax | grep gedit {#31-ps-grep}

**❌ Problema:**

```bash
ps ax | grep gedit  # grep aparece nos resultados
```

**✅ Soluções:**

```bash
# Trick com regex
ps ax | grep '[g]edit'

# Melhor: use pgrep
pgrep gedit

# Ou ps com filtros
ps -C gedit
```

## 32. printf "$foo" {#32-printf-foo}

**❌ Problema:**

```bash
printf "$foo"  # Vulnerabilidade de format string
```

**✅ Solução:**

```bash
printf '%s' "$foo"      # Sempre forneça format string
printf '%s\n' "$foo"    # Com newline
```

## 33. for i in {1..$n} {#33-for-i-brace-expansion}

**❌ Problema:**

```bash
for i in {1..$n}; do  # Brace expansion antes da variável
```

**✅ Solução:**

```bash
for ((i=1; i<=n; i++)); do
    # ...
done
```

## 34. if \[\[ $foo = $bar ]] (depending on intent) {#34-pattern-matching}

**❌ Problema:**

```bash
if [[ $foo = $bar ]]  # $bar é tratado como pattern!
```

**✅ Soluções:**

```bash
if [[ $foo = "$bar" ]]   # String literal
if [[ $foo = $pattern ]] # Pattern matching intencional
```

## 35. if \[\[ $foo =\~ 'some RE' ]] {#35-regex-quotes}

**❌ Problema:**

```bash
if [[ $foo =~ 'some RE' ]]  # Aspas fazem regex virar string literal
```

**✅ Soluções:**

```bash
# Sem aspas
if [[ $foo =~ some\ RE ]]

# Ou use variável
re='some RE'
if [[ $foo =~ $re ]]
```

## 36. \[ -n $foo ] or \[ -z $foo ] {#36-test-n-z}

**❌ Problema:**

```bash
[ -n $foo ]  # Sem aspas pode quebrar
[ -z $foo ]  # Sem aspas pode quebrar
```

**✅ Soluções:**

```bash
[ -n "$foo" ]    # Com aspas
[ -z "$foo" ]    # Com aspas
[[ -n $foo ]]    # Bash - aspas opcionais
[[ -z $foo ]]    # Bash - aspas opcionais
```

## 37. \[\[ -e "$broken\_symlink" ]] returns 1 {#37-broken-symlink}

**❌ Problema:**

```bash
[[ -e "$broken_symlink" ]]  # Falha para symlinks quebrados
```

**✅ Solução:**

```bash
# Testa symlink OR arquivo existente
[[ -e "$broken_symlink" || -L "$broken_symlink" ]]

# POSIX
[ -e "$broken_symlink" ] || [ -L "$broken_symlink" ]
```

## 38. ed file <<<"g/d{0,3}/s//e/g" fails {#38-ed-regex}

**❌ Problema:**

```bash
ed file <<<"g/d\{0,3\}/s//e/g"  # ed não aceita {0,3}
```

**✅ Solução:**

```bash
ed file <<<"g/d\{1,3\}/s//e/g"  # Use {1,3} ao invés de {0,3}
```

## 39. expr sub-string fails for "match" {#39-expr-match}

**❌ Problema:**

```bash
word=match
expr "$word" : ".\(.*\)"  # "match" é palavra-chave do expr
```

**✅ Soluções:**

```bash
# GNU expr
expr + "$word" : ".\(.*\)"

# Melhor: use parameter expansion
echo "${word#?}"     # Remove primeiro caractere
echo "${word:1}"     # Substring a partir do índice 1
```

## 40. On UTF-8 and Byte-Order Marks (BOM) {#40-utf8-bom}

**⚠️ Problema:** Arquivos com BOM podem quebrar scripts que esperam caracteres ASCII específicos no início.

**✅ Dica:**

* Scripts Unix/UTF-8 normalmente NÃO usam BOM
* Se encontrar BOM, trate como arquivo estrangeiro (tipo Windows)
* Remove BOM se necessário antes de processar

## 41. content=$(\<file) {#41-content-file}

**⚠️ Cuidado:**

```bash
content=$(<file)  # Remove trailing newlines!
```

**✅ Para preservar exatamente:**

```bash
# Workaround feio mas funcional
content_x=$(cat file; printf x)
content=${content_x%x}
```

## 42. for file in ./\* ; do if \[\[ $file != _._ ]] {#42-glob-pattern}

**❌ Problema:**

```bash
for file in ./*; do
    if [[ $file != *.* ]]; then  # Pattern não considera o ./
```

**✅ Soluções:**

```bash
# Ajustar o pattern
if [[ $file != ./*.* ]]; then

# Ou remover o prefixo
if [[ ${file##*/} != *.* ]]; then
```

## 43. somecmd 2>&1 >>logfile {#43-redirection-order}

**❌ Problema:**

```bash
somecmd 2>&1 >>logfile  # Ordem errada! stderr vai para terminal
```

**✅ Solução:**

```bash
somecmd >>logfile 2>&1  # Ordem correta
```

**🔍 Explicação:** Redirecionamentos são avaliados da esquerda para direita:

* `2>&1 >>logfile`: stderr → terminal, depois stdout → logfile
* `>>logfile 2>&1`: stdout → logfile, depois stderr → onde stdout aponta (logfile)

## 44. cmd; (( ! $? )) || die {#44-exit-status}

**❌ Desnecessário:**

```bash
cmd; (( ! $? )) || die  # $? é desnecessário
```

**✅ Solução:**

```bash
cmd || die              # Mais simples e direto
if ! cmd; then die; fi  # Ainda mais claro
```

## 45. y=$(( array\[$x] )) {#45-array-arithmetic}

**❌ Problema:**

```bash
x='$(date >&2)'
y=$((array["$x"]))  # Code injection!
```

**✅ Soluções:**

```bash
# Validação antes do uso
[[ $x == +([0-9]) ]] || exit 1
y=$((array[x]))

# Ou escape
y=$((array[\$x]))
```

## 46. read num; echo $((num+1)) {#46-read-arithmetic}

**❌ Problema:**

```bash
read num
echo $((num+1))  # Code injection se num não for número!
```

**✅ Solução:**

```bash
read num
# Validação
case $num in
    *[!0-9]*) echo "Not a number" >&2; exit 1 ;;
    *) echo $((num+1)) ;;
esac
```

## 47. IFS=, read -ra fields <<< "$csv\_line" {#47-ifs-csv}

**❌ Problema:**

```bash
IFS=, read -ra fields <<< "a,b,"  # Campo vazio final é perdido!
```

**✅ Solução:**

```bash
input="a,b,"
IFS=, read -ra fields <<< "$input,"  # Adiciona delimitador extra
```



