---
icon: layer-group
---

# Pilha (Stack)

## Stack — Anatomia, Exploração e Defesa (x86/x64)

> **Objetivo**: tornar o aluno capaz de visualizar, instrumentar e manipular a stack em qualquer cenário de engenharia reversa ou exploit development.

***

### 📚 1. A Pilha (The Stack)&#x20;

Quando um _thread_ (linha de execução) está rodando em um processo, ele executa instruções a partir da **Imagem do Programa** (código principal do binário carregado na memória) ou de diversas **DLLs (Dynamic Link Libraries)** – bibliotecas carregadas dinamicamente em tempo de execução.

Para armazenar **dados temporários de curto prazo**, como:

* parâmetros de função (arguments),
* variáveis locais,
* endereços de retorno (return addresses) e
* informações de controle de execução,

...cada _thread_ precisa de uma área própria de memória chamada **pilha (stack)**.

#### 🔄 Como funciona a pilha?

A pilha é organizada na arquitetura x86 usando uma estrutura de dados **LIFO (Last-In, First-Out)** — ou seja, o **último valor inserido é o primeiro a ser removido**.

O processador utiliza instruções específicas em Assembly, como:

* `PUSH` → adiciona (empilha) dados no topo da pilha
* `POP` → remove (desempilha) dados do topo da pilha

Essas operações são **cruciais em chamadas de função**, onde o endereço de retorno e os argumentos são manipulados diretamente pela pilha.

### 1.1. Conceito e Função

* **Pilha (Stack)** = estrutura LIFO mantida pela CPU/SO para gerenciar chamadas de função.
* Cria **frames** (blocos) contendo endereço de retorno, registradores salvos, variáveis locais e às vezes argumentos.

### 2. Layout de um Stack Frame (System V AMD64)

<figure><img src="../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

Stack Frame (Pilha) nada mais é do que partes da memória separadas pra cada função, com seus argumentos e variáveis.

```
              ↑ Endereços altos
┌──────────────┐  ← RBP (Base Pointer)
│ Parâmetros  │  (se passados na stack)
├──────────────┤
│ Retorno →   │  endereço para o chamador
├──────────────┤
│ RBP antigo  │  cadeado da stack
├──────────────┤
│ Locais      │  variáveis + canário (se -fstack-protector)
├──────────────┤
│ Alinhamento │  padding p/ 16‑byte
└──────────────┘  ← RSP (Stack Pointer)
              ↓ Endereços baixos
```

**Prólogo típico (gcc -O0):**

```asm
push   rbp          ; salva frame anterior
mov    rbp, rsp     ; novo frame base
sub    rsp, 0x50    ; reserva locais
```

**Epilogo:** `leave` (= mov rsp, rbp; pop rbp) → `ret`.

### 3. Escalonamento de Parâmetros

| ABI           | 1º–4º argumentos        | Demais | Retorno |
| ------------- | ----------------------- | ------ | ------- |
| SysV AMD64    | RDI RSI RDX RCX R8 R9   | stack  | RAX     |
| Microsoft x64 | RCX RDX R8 R9           | stack  | RAX     |
| x86 (cdecl)   | stack (último→primeiro) | —      | EAX     |

### 4. Exploração Clássica

#### 4.1 Buffer Overflow (Stack Smashing)

1. **Overwrite** variáveis locais → RBP → endereço de retorno.
2. Redirecione `RIP` para:
   * shellcode na própria stack (NX off).
   * ROP chain (NX on).

#### 4.2 Stack Pivot

* Utiliza gadget `pop rsp; ret` ou `leave; ret` para mover `RSP` para região controlada (heap, .bss, mmap).

#### 4.3 Stack Leak

* Formatação `%p` ou leitura out‑of‑bounds expõe ponteiros → base da stack → calculo de offsets.

### 5. Proteções Modernas

| Defesa                 | Mecanismo                                     | Bypass comum                |
| ---------------------- | --------------------------------------------- | --------------------------- |
| **Canary**             | valor random antes do RET; checado em epilogo | leak canary (FS:0x28 Linux) |
| **NX / DEP**           | marca stack `--x`                             | ROP, return‑to‐libc         |
| **ASLR**               | random base stack                             | stack‑leak ou brute‑force   |
| **Shadow Stack (CET)** | cópia RO de RETs                              | JOP / SROP                  |

### 6. Ferramentas e Comandos Essenciais

| Ferramenta                    | Comando               | Resultado          |
| ----------------------------- | --------------------- | ------------------ |
| **GDB + pwndbg**              | `telescope $rsp`      | dump stack         |
|                               | `context`             | visão regs + stack |
| **x64dbg**                    | CPU ‣ -> Stack window | visual interativo  |
| **Valgrind/AddressSanitizer** | crash log             | offset exato       |

### 7. Lab Hands‑on (5 Passos)

1. Compilar `vuln.c` com `-fno-stack-protector -z execstack`.
2. `gdb ./vuln` → break no `gets`.
3. Enviar 100 × "A" → observar `RIP = 0x41414141`.
4. Calcular offset (`pwntools.cyclic`).
5. Injetar shellcode ou ROP.

### 8. Checklist de Maestria

* [ ] Desenhar frame contendo canary.
* [ ] Identificar gadget `leave; ret` num binário.
* [ ] Explicar por que alinhamento de 16 B é obrigatório antes de `call` SysV.
* [ ] Demonstrar leak de `RBP` via `%p`.

### 9. Links Oficiais

1. **Intel® 64 Manual**, Vol. 1 §6 (Stack Operations)
2. **SysV ABI AMD64**, §3.2.2 (Process Stack)
3. **Microsoft PE / x64 ABI**, §5 (Prolog/Epilog)
4. **Phrack #49 – “Smashing the Stack”**
5. **Glibc Stack Guard Documentation**

Com domínio de stack, você transforma cada crash em oportunidade — aqui começa todo BoF, SROP, ROP, shadow‑pivot ou sigreturn café‑com‑leite. 🚀
