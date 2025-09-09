# GEF

## Cheatsheet GDB + GEF

### Básico do GDB

```sh
gdb ./binario          # abre binário
gdb -q ./binario       # abre sem banner
r args                 # roda com argumentos
start                  # inicia no main
c                      # continua execução
s / si                 # step (entra na função, instrução única)
n / ni                 # next (pula função, instrução única)
finish                 # roda até retornar da função
q                      # sair
```

***

### Breakpoints

```sh
b main                 # breakpoint em função
b *0x400123            # breakpoint em endereço
info b                 # lista breakpoints
d <n>                  # deleta breakpoint
disable <n>            # desativa breakpoint
enable <n>             # reativa breakpoint
```

***

### Contexto (auto-painel do GEF)

GEF mostra automaticamente:

* Registradores coloridos
* Código desassemblado próximo do RIP
* Stack (valores + ASCII)
* Info da heap, se existir

Atalhos:

```sh
context                # mostra contexto
context disable        # desliga painel
```

***

### Memória e Stack

```sh
x/10x $rsp             # examina 10 palavras em hex
x/20i $rip             # examina 20 instruções
x/s $rdi               # mostra string no endereço de RDI
hexdump $rsp 64        # dump da memória (64 bytes a partir de RSP)
dereference $rsp       # segue ponteiros da pilha
```

***

### Registradores

```sh
info registers         # todos os regs
p $rax                 # imprime RAX
set $rax=0             # altera RAX
```

Extras do GEF:

```sh
registers              # visão colorida
```

***

### Funções úteis do GEF

```sh
pattern create 100     # gera padrão cíclico de 100 bytes
pattern search <val>   # procura valor na memória
pattern offset <val>   # acha offset exato no padrão

dereference $rsp 20    # mostra 20 endereços seguidos (como telescope)
hexdump $rbp-0x50 128  # dump bonitinho
search-pattern AAAA    # procura sequência na memória
```

***

### Exploit Dev

```sh
vmmap                  # mostra mapeamento de memória (stack, heap, libs)
elf-info               # infos sobre ELF
checksec               # mostra proteções (NX, PIE, RELRO, canário)
aslr                   # mostra estado do ASLR
rop                    # busca gadgets ROP
```

***

### Debugando entrada/pipe

```sh
r < input.txt
r <<< "AAAA"
```

***

### Workflow típico CTF

1. `checksec` → ver proteções.
2. `r` / `start` → rodar.
3. `pattern create 200` → gerar padrão.
4. `r <<< $(pattern create 200)` → crash.
5. `registers` → ver RIP/EIP.
6. `pattern offset <valor>` → calcular offset.
7. Criar payload.

***

### Instalação

```sh
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"
```

***

### Diferenças Pwndbg x GEF

* **GEF** → comandos curtos, painel mais limpo, scripts em Python 3.
* **Pwndbg** → painel mais rico, mais integração com rop/gadgets.
* Ambos podem ser usados junto com o **pwndbg + peda + gef** (via [pwndbg-gef-peda](https://github.com/apogiatzis/pwndbg-gef-peda)).

***

👉 Esse cheatsheet cobre: comandos do GDB, extensões do GEF para contexto, memória, registradores e funções específicas de exploit dev.
