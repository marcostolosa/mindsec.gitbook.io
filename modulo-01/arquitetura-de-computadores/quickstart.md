---
icon: cabinet-filing
---

# Registradores

## Registradores — Guia Completo para Engenharia Reversa

> **Objetivo**: tornar o aluno capaz de ler e manipular qualquer registrador x86/x64 (e ARM básico), reconhecer seu papel em prólogos/epílogos, calling conventions, ROP e vulnerabilidades.

***

### 1 Visão‑geral (Arquitetura x86‑64)

| Grupo                | Registradores     | Função principal              | Usos em exploits                                    |
| -------------------- | ----------------- | ----------------------------- | --------------------------------------------------- |
| **Dados gerais**     | `RAX RBX RCX RDX` | Aritmética, I/O               | ↳ `RAX` para syscalls Linux; gadgets `pop rax; ret` |
| **Index / ponteiro** | `RSI RDI RBP RSP` | Endereços, stack              | ↳ `RSP` pivot; `RBP` leaks; `RDI` 1.º arg SystemV   |
| **Temporários**      | `R8–R11`          | Cálculos rápidos              | Gadgets livres de restrições                        |
| **Preservados**      | `RBX RBP R12–R15` | Devem ser salvos/ restaurados | Confiáveis p/ pivot JOP                             |
| **Instrução**        | `RIP`             | Próxima instrução             | Overwrite → controle de fluxo                       |
| **Segmento**         | `CS DS ES FS GS`  | TLS, modo                     | TLS leaks, TEB/PEB Windows                          |
| **Flags**            | `RFLAGS`          | CF/ZF/SF/OF …                 | Condições JNZ/JE; BIT flips ROP                     |

> **Dica**: _preserved/bcallee‑saved_ = sobreviver entre funções → ótimo para carregar valores em chains.

***

### 2 Detalhe de cada registrador crítico

#### 2.1 `RIP` — Instruction Pointer

* Armazenado implicitamente em `call` (`push rip`; `jmp`).
* Overwrite típico: `ret` para chain ROP.
* CET/Shadow‑stack mantém cópia → need JOP/DOP.

#### 2.2 `RSP` — Stack Pointer

* Alinha para 16 B antes de chamadas de função em SysV.
* Pivot clássico: gadget `pop rsp; ret` ou `mov rsp, [...]`.
* Leak → base da stack, útil para SROP.

#### 2.3 `RBP` — Base Pointer

* Em O2 compilado como frame‑pointer‑omit; ainda salvo em debug/‑fno‑omit‑frame‑pointer.
* Leak via format‑string (`%6$p`).
* Gadget `leave; ret` = `mov rsp, rbp` + `pop rbp` + `ret` ⇒ mini‑pivot.

#### 2.4 `RAX` — Accumulator

* System V: retorno de função + nº de syscall (`rax=0x3b → execve`).
* Gadget comum: `xor rax, rax` (cheap zero).
* _Write‑what‑where_ kernel: set `rax` pointer antes de `mov [rax], …`.

#### 2.5 `RDI/RSI/RDX/RCX/R8/R9` — Param regs

* **System V order**: `RDI`, `RSI`, `RDX`, `RCX`, `R8`, `R9`.
* **MS x64 order**: `RCX`, `RDX`, `R8`, `R9`.
* Construção de execve ROP:
  1. `pop rdi; ret` → `/bin/sh` ptr
  2. `pop rsi; ret` → 0
  3. `xor rdx, rdx`
  4. `mov rax, 0x3b; syscall`

#### 2.6 Flags (`RFLAGS`)

* `ZF` (zero), `CF` (carry) mudam após `cmp`/`test`.
* Gadgets `cmovz`, `setz` — Data Oriented Programming.

***

### 3 Registradores em ARM64 (visão 60 s)

| Reg     | Papel                    | Equivalente x86  |
| ------- | ------------------------ | ---------------- |
| `X0–X7` | Args/ret                 | `RDI–R9` / `RAX` |
| `SP`    | Stack pointer            | `RSP`            |
| `LR`    | Link register (ret addr) | RET push         |
| `PC`    | Program counter          | `RIP`            |
| `FP`    | Frame pointer            | `RBP`            |
| `CPSR`  | Flags                    | `RFLAGS`         |

PAC assina `LR` & `FP` → exploits precisam gadgets de re‑sign ou leaks de PAC key.

***

### 4 Como registradores aparecem no código Assembly

```asm
push   rbp           ; salva base
mov    rbp, rsp      ; novo frame
sub    rsp, 0x20     ; locals
mov    DWORD PTR [rbp-4], edi ; arg copy
call   printf        ; RDI ptr → format string
leave
ret
```

Observe `EDI` (arg1 32‑bit) copiado p/ stack; `leave` faz pivot.

***

### 5 Lab prático (10 min) — Trace registradores e fluxo

```bash
objdump -d basic > asm.txt         # veja prólogo
pwndbg> start
pwndbg> till call                 # pausa antes do call
pwndbg> info registers rdi rsi    # argumentos
pwndbg> stepi ; stepi             # execute call + ret
pwndbg> telescope $rsp            # visualize retorno
```

Objetivo:

1. Identificar `RIP` antes/depois de `call`.
2. Ver como `RBP`/`RSP` mudam ao entrar e sair da função.

***

### 6 Check‑list de maestria

* [ ] Citar ordem de argumentos em System V **e** MS x64.
* [ ] Usar gadget `pop rsi; pop r15; ret` para setar arg 2 em ROP.
* [ ] Mostrar diferença de `pushf / popf` (flags).
* [ ] Explicar papel de `LR` em overflow ARM (ret2plt style).

***

### 7 Leituras oficiais

1. Intel® 64 Manuals, Vol 1 § 3 (System Architecture).
2. SysV AMD64 ABI § 3 (calling conv).
3. MS x64 ABI (PE COFF spec § 5).
4. ARM A‑profile documentation — Procedure Call Std.
5. “x86‑64 Machine‑Level Programming” (CMU 15‑213 notes).

Com domínio total desses registradores, você transforma qualquer crash em oportunidade — e qualquer trace em roteiro de exploit.🤘
