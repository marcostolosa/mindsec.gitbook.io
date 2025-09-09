---
icon: backward
---

# Radare2

## Cheatsheet Radare2 (r2)&#x20;

***

### Inicialização

```sh
r2 ./binario            # abre binário
r2 -d ./binario         # abre em modo debug
r2 -d -A ./binario      # debug + análise inicial
r2 -A ./binario         # análise completa automática
r2 -nn ./binario        # sem análise automática
```

***

### Navegação

```sh
s main                  # ir para símbolo main
s 0x00400550            # ir para endereço
s-                      # voltar para posição anterior
s+                      # ir para próxima posição
```

***

### Análise

```sh
aa                      # auto-análise
aaa                     # análise agressiva (funções, calls, strings, xrefs)
afl                     # lista funções
afn novo_nome 0x00400   # renomeia função
afi                     # info da função atual
afv                     # variáveis locais e argumentos
afvn novo_nome arg0     # renomear argumento
```

***

### Listagem de Código

```sh
pdf                     # print disassembly da função atual
pdf @ main              # disassembly do main
pd 20                   # disassembly de 20 instruções
pdj 20                  # saída em JSON
pdc                     # decompile (quando disponível, ex: r2ghidra)
pdr                     # decompile (modo rápido/alternativo)
```

***

### Strings & Imports

```sh
iz                      # lista strings
izz                     # lista strings com análise profunda
ii                      # lista imports
iij                     # imports em JSON
iE                      # exports
```

***

### Info do Binário

```sh
i                       # info do binário
ie                      # info de entradas (entrypoint, init, etc.)
iS                      # seções
iSj                     # seções em JSON
ir                      # registradores
```

***

### Referências (XREFs)

```sh
axt @ sym.func          # quem referencia essa função
axt 0x00401010          # referências ao endereço
axf                     # funções chamadas pela função atual
```

***

### Stack & Memória

```sh
px 64 @ rsp             # hexdump de 64 bytes a partir do RSP
pxa 32 @ 0x601000       # hexdump anotado
psz 64 @ 0x601000       # imprime string de 64 bytes
dm                      # lista regiões de memória (no debug)
dmm                     # mostra mappings detalhados
```

***

### Debugging

```sh
ood                     # reabra o processo
dc                      # continue execução
ds                      # step into (instrução)
dso                     # step over
db 0x00400550           # breakpoint
db sym.main             # breakpoint na main
db- 0x00400550          # remove breakpoint
dr                      # mostra registradores
dr rax=0                # seta valor no registrador
dpt                     # mostra threads
```

***

### Exploit Dev

```sh
aaa                     # análise agressiva
afl ~strcpy             # procura chamadas a strcpy
iz ~flag                # procura strings "flag"
wx 9090 @ 0x00400550    # patch: escreve NOPs no endereço
wa jmp 0x00400500       # patch com assembly (write asm)
waf                     # escreve função no lugar atual
```

***

### Busca

```sh
/ str flag              # busca string “flag”
/x deadbeef             # busca sequência hex
/ra mov eax, 1          # busca instrução
//c /bin/sh             # busca string com cache
```

***

### Visual Modes

```sh
V                       # modo visual (navegação interativa)
VV                      # visual graph (CFG)
V!                      # painel múltiplo
Vpp                     # visual de patches
```

Atalhos no **visual mode**:

* `p` → troca painel (hex, disasm, gráfico)
* `?` → ajuda
* `q` → sair

***

### Scripting & JSON

* Radare2 é **100% scriptável**:

```sh
pdj 10                  # disasm em JSON
aflj                    # funções em JSON
izj                     # strings em JSON
```

* Pode exportar e usar em Python:

```python
import r2pipe
r2 = r2pipe.open("./binario")
r2.cmd("aaa")
print(r2.cmdj("aflj"))
```

***

### Plugins e Extensões

* **r2ghidra-dec** → decompilador do Ghidra dentro do r2 (`pdc`)
* **r2frida** → integração com Frida para análise dinâmica
* **r2pipe** → interface Python, Go, Rust, etc.
* **Cutter** → GUI do Radare2

***

### Fluxo típico CTF/Malware

1. `r2 -A ./bin` → análise inicial
2. `afl` → lista funções
3. `pdf @ sym.main` → olhar main
4. `iz` / `ii` → procurar strings/imports
5. `axt` → ver XREFs em funções críticas
6. `VV` → gráfico do fluxo
7. `pdc` → decompilar função
8. `wx` ou `wa` → testar patch

***

### Dicas Ninja

* **Comando mágico**: `aaa` (faz quase tudo de análise)
* **Aliases úteis**:
  * `s` → seek (posicionar)
  * `pdf` → disasm função
  * `pdc` → decompile
  * `VV` → grafo interativo
* **Undo/Redo**: `u` e `U` (patches)
* **Macros**: scripts `.r2` podem automatizar sequência de comandos
* **Histórico**: tudo no `~/.radare2rc`

***

### Instalação

```sh
git clone https://github.com/radareorg/radare2
cd radare2
./sys/install.sh
```

GUI recomendada: [Cutter](https://cutter.re/)

***

👉 Esse cheatsheet cobre **análise estática**, **debugging dinâmico**, **exploit dev**, **malware RE**, **patching**, **visual mode**, **JSON scripting**, **extensões** e até o fluxo típico de CTF.

