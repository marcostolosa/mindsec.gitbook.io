# pwndbg

## Cheatsheet GDB + Pwndbg



### &#x20;Básico do GDB

```sh
gdb ./binário          # abre binário
gdb -q ./binário       # abre sem banner
r args                 # roda com argumentos
start                  # inicia no main
c                      # continue execução
s / si                 # step (entra na função, instrução única)
n / ni                 # next (pula função, instrução única)
finish                 # executa até retornar da função
q                      # sair
```

***

### Breakpoints

```sh
b main                 # breakpoint em função
b *0x400123            # breakpoint em endereço
r                      # roda até breakpoint
info b                 # lista breakpoints
d <n>                  # deleta breakpoint n
disable <n>            # desativa breakpoint
enable <n>             # reativa breakpoint
```

***

### Pwndbg Context

Pwndbg mostra automaticamente **contexto** (registradores, stack, código, memória) a cada breakpoint/step.

* **context**: mostra o painel completo (regs, stack, ASM).
* **context regs**: só registradores.
* **context code**: só desassembly.
* **context stack**: só pilha.
* **context off**: desliga o painel automático.

***

### Memória e Stack

```sh
x/10x $rsp             # examina 10 palavras em hex a partir do RSP
x/20i $rip             # examina 20 instruções a partir do RIP
x/s $rdi               # mostra string no endereço de RDI
hexdump $rsp 64        # dump da memória (64 bytes a partir de RSP)
stack                  # exibe pilha formatada
stack 20               # 20 entradas da pilha
telescope $rsp         # segue ponteiros na memória
```

***

### Registradores

```sh
info registers         # todos os registradores
p $rax                 # imprime valor do RAX
set $rax=0             # altera RAX
```

Extras Pwndbg:

```sh
regs                   # visão bonita dos registradores
reg rax                # só o RAX
```

***

### Funções úteis do Pwndbg

```sh
pattern create 100     # gera padrão cíclico de 100 bytes
pattern offset EIP     # encontra offset no padrão
telescope <addr>       # segue ponteiros (stack, heap)
hexdump <addr> [len]   # dump em hex/ASCII
nearpc                 # mostra instruções próximas ao RIP
search /bin/sh         # procura string na memória
```

***

### Exploit Dev

```sh
rop                   # mostra gadgets ROP do binário carregado
vmmap                 # mostra mapeamento de memória (heap, libs, stack)
elf                   # informações do ELF carregado
checksec              # mostra proteções (NX, PIE, RELRO, canário)
```

***

### Debugando com entrada/pipe

```sh
gdb -q ./vuln
r < input.txt          # roda com arquivo como stdin
r <<< "AAAA"           # roda com string direta
```

***

### Macros úteis

```sh
context                # mostra contexto todo
aslr off               # desativa ASLR (precisa de sudo)
```

***

### Workflow típico em CTF

1. `checksec` → ver proteções.
2. `r` / `start` → rodar até main.
3. `pattern create 200` → gerar padrão.
4. `r <<< $(pattern create 200)` → crash.
5. `info registers` ou `regs` → ver RIP/EIP.
6. `pattern offset <valor>` → achar offset.
7. Construir payload e testar.

***

### Instalação

```sh
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

***

👉 Esse cheatsheet cobre: comandos básicos do GDB, extensões do Pwndbg para contexto/stack, e funções específicas de exploit dev (pattern, rop, vmmap, checksec).

