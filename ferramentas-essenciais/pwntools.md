---
icon: code
---

# pwntools

## Cheatsheet Pwntools Shellcode

### `asm` → montar instruções em shellcode

Compila **assembly → opcode**.

```sh
# instrução simples (x86)
asm nop
90

# várias instruções
asm 'mov eax, 0xdeadbeef'
b8efbeadde

# saída em formato string estilo C
asm nop -f string
'\x90'

# saída em binário hexdump (mais bonito que xxd?)
asm 'push eax' | phd
00000000  50                                                  │P│
```

#### Arquitetura diferente

```sh
asm -c arm nop
00f020e3

asm -c mips nop
00000000
```

***

### `disasm` → desmontar opcodes em assembly

Converte **opcode → assembly legível**.

```sh
asm 'push eax' | disasm
   0:   50                      push   eax

asm -c arm 'bx lr' | disasm -c arm
   0:   e12fff1e        bx      lr
```

***

### `shellcraft` → gerador de shellcode pronto

Templates de _shellcode_ (syscalls, payloads, etc).

```sh
# listar tudo
shellcraft

# gerar shellcode /bin/sh (x86)
shellcraft i386.linux.sh
6a68682f2f2f73682f62696e89e331c96a0b5899cd80

# saída em assembly legível
shellcraft i386.linux.sh -f asm

# echo customizado
shellcraft i386.linux.echo "Hello, world" STDOUT_FILENO
```

***

### `phd` → hexdump melhorado

Visualizar bytes com destaque (ASCII, NULL, etc).

```sh
asm nop | phd
00000000  90                                                  │·│
```

***

### Depuração de shellcode

#### Gerar ELF executável

```sh
# shellcode /bin/sh → ELF
shellcraft i386.linux.sh -f elf > sh
chmod +x sh

# syscall exit(0)
asm 'mov eax, 1; int 0x80' -f elf > exit
chmod +x exit
```

#### Testar com strace

```sh
strace ./exit
_exit(0) = ?
```

#### Entrar direto no GDB

```sh
shellcraft i386.linux.sh --debug
asm 'mov eax, 1; int 0x80' --debug
```

***

### Dependências externas

* **Pwntools**: `pip install pwntools`
* **binutils-multiarch** (Linux): para suportar ARM, MIPS, PowerPC etc.
* **strace** e **gdb**: depuração e tracing de syscalls.

***

Esse cheatsheet cobre 90% do que você precisa em aula/lab: gerar shellcode, desmontar, visualizar bytes, montar ELF e depurar.
