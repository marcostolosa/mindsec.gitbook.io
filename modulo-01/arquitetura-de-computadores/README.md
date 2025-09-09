---
description: (Fundação absoluta para todo exploit e engenharia reversa)
icon: microchip
---

# Arquitetura de Computadores

> **Meta do módulo**: ao final, você será capaz de **desenhar** a organização de memória de um processo, **identificar** onde variáveis vivem, **explicar** a função dos registradores principais e **demonstrar** com debugger a criação e destruição de stack frames, distinguindo claramente **stack** de **heap**.

## Módulo 1 — Memória, Registradores, Stack & Heap

_(Fundação absoluta para todo exploit e engenharia reversa)_

> **Meta do módulo**: ao final, você será capaz de **desenhar** a organização de memória de um processo, **identificar** onde variáveis vivem, **explicar** a função dos registradores principais e **demonstrar** com debugger a criação e destruição de stack frames, distinguindo claramente **stack** de **heap**.

***

### 1. Panorama Geral — “O Prato Feito Digital”

<table><thead><tr><th width="100.33331298828125">Camada</th><th width="222.99993896484375">Analogia visual</th><th width="279">Conteúdo típico</th><th>Permissões padrão</th></tr></thead><tbody><tr><td><code>.text</code></td><td>Receita fixada</td><td>Instruções</td><td><code>r-x</code></td></tr><tr><td><code>.rodata</code></td><td>Etiquetas</td><td>Strings/constantes</td><td><code>r--</code></td></tr><tr><td><code>.data</code></td><td>Arroz pronto</td><td>Globais inicializadas</td><td><code>rw-</code></td></tr><tr><td><code>.bss</code></td><td>Panela vazia</td><td>Globais zeradas</td><td><code>rw-</code></td></tr><tr><td><strong>Heap</strong></td><td>Bufê à la carte</td><td>Alocações via <code>malloc/new</code></td><td><code>rw-</code></td></tr><tr><td><strong>Stack</strong></td><td>Torre de pratos</td><td>Frames de função</td><td><code>rw-</code></td></tr></tbody></table>

> **Lab‑flash 01** (3 min): `readelf -S hello | grep -E '\.(text|data|bss|rodata)'` → anote tamanho e offsets.

***

### 2. Registradores Essenciais — “O Painel de Controle da CPU”

<table><thead><tr><th width="200.33331298828125">Família</th><th>Propósito</th><th>Exemplo (x86‑64)</th></tr></thead><tbody><tr><td><strong>Dados</strong></td><td>Operações aritméticas/lógicas</td><td><code>RAX</code>, <code>RBX</code>, <code>RCX</code>, <code>RDX</code></td></tr><tr><td><strong>Indexação</strong></td><td>Ponteiros/arranjos</td><td><code>RSI</code>, <code>RDI</code></td></tr><tr><td><strong>Stack Pointer</strong></td><td>Topo da torre</td><td><code>RSP</code> (<code>ESP</code> 32‑bit)</td></tr><tr><td><strong>Base Pointer</strong></td><td>Base do frame atual</td><td><code>RBP</code> (<code>EBP</code>)</td></tr><tr><td><strong>Instruction Pointer</strong></td><td>Próxima instrução</td><td><code>RIP</code> (<code>EIP</code>)</td></tr></tbody></table>

> **Lab‑flash 02** (2 min):
>
> ```
> gdb -q ./hello
> start
> info reg rip rsp rbp rax
> ```
>
> Explique em voz alta o que cada valor representa.

***

### 3. A Stack em Detalhes — “Pratos que entram, pratos que saem”

```
Endereços altos ↓
┌──────────────────────────────┐
│ Args da função            │
│ Endereço de retorno ← alvo│
│ RBP anterior              │
│ Variáveis locais          │
└──────────────────────────────┘
Endereços baixos ↑
```

#### 3.1 Ciclo de vida do frame

1. **Prólogo**: `push rbp; mov rbp, rsp; sub rsp, X` → reserva espaço.
2. **Corpo**: usa variáveis locais / chamadas aninhadas.
3. **Epilogo**: `leave; ret` → descarta frame e volta ao chamador.

#### 3.2 Por que importa?

* Sobrescrever o **endereço de retorno** = **controle total do fluxo (BoF)**.
* Leak de `rbp`/`rsp` ajuda em **info‑leaks e ROP**.

> **Lab‑guiado**: compile `vuln.c` com `-fno-stack-protector -g`, trace no GDB e observe a mudança de `RSP` ao entrar/sair de `vuln()`.

***

### 4. A Heap em Detalhes — “Bufê Sob Demanda”

| Conceito             | x86‑64/Linux (glibc)          | Analogias               |
| -------------------- | ----------------------------- | ----------------------- |
| **Arena**            | Região gerenciada pelo malloc | Salão do bufê           |
| **Chunk**            | Bloco individual              | Porção no prato         |
| **Free list / bins** | Listas de chunks livres       | Pilhas de pratos limpos |

* Cresce **para cima** (end. maiores).
* Vulnerabilidades típicas: **heap overflow, use‑after‑free, double‑free**.

> **Lab‑flash 03** (5 min): código `heap_demo.c` com dois `malloc`, `free`, breakpoint em `free` e olhar `glibc` bins via pwndbg `heap bins`.

***

### 5. Stack vs Heap — Comparação Rápida

| Atributo    | **Stack**               | **Heap**               |
| ----------- | ----------------------- | ---------------------- |
| Gerência    | Automática (CPU)        | Manual (`malloc/free`) |
| Crescimento | ↓ (endereços menores)   | ↑ (endereços maiores)  |
| Tamanho     | Limitado (default 8 MB) | Virtualmente ilimitado |
| Velocidade  | Muito rápido            | Mais lento             |
| Riscos      | Buffer/stack overflow   | Heap overflow, UAF     |

***

### 6. Exercícios Práticos

1. **Mapeie** seu processo: `cat /proc/$(pidof hello)/maps` e marque stack & heap.
2. **Desmonte** uma função `foo` e identifique cada campo do frame.
3. **Exploit básica**: modifique exemplo `gets()` e provoque SIGSEGV → capture offset com pwntools `cyclic`.

***

### 7. Checklist Feynman

* [ ] Posso desenhar de cabeça a ordem das seções `.text` → `.bss` → heap → stack.
* [ ] Sei explicar diferença entre `RBP` e `RSP`.
* [ ] Consigo demonstrar overflow que altera endereço de retorno.
* [ ] Localizo no pwndbg a arena principal do heap e interpreto um chunk.

***

### 8. Referências Oficiais Essenciais

1. Intel® 64 Manuals, Vol. 1 §§ 2–3 (registradores, modos).
2. _Operating Systems: Three Easy Pieces_ – Cap. 14 (Virtual Memory).
3. _The Art of Exploitation_ § Stack & Heap.
4. pwndbg docs – `heap` commands.
5. glibc malloc internals (ptmalloc3 wiki).

***

**Pronto!** Esta base sólida permitirá quebrar limites (BoF, heap feng‑shui) nos módulos seguintes.🤘

## Módulo 1 – Fundamentos da Engenharia Reversa

> **Meta do módulo**: construir a base conceitual e prática indispensável para todo o restante do curso, respondendo quatro perguntas‑chave:
>
> 1. **Como o computador armazena e move dados (memória, registradores, pilha, heap)?**
> 2. **Como o sistema operacional conversa com o hardware e com os processos?**
> 3. **Como ler e escrever Assembly (x86/x64) de forma fluida?**
> 4. **Quais ferramentas vou usar no resto do curso e como iniciar nelas?**

***

### 1 Arquitetura de Computadores

#### 1.1 Mapa da Memória

<table><thead><tr><th width="116">Segmento</th><th width="147.66668701171875">Permissões típicas</th><th width="224">Conteúdo</th><th>Analogia</th></tr></thead><tbody><tr><td><code>.text</code></td><td><code>r‑x</code></td><td>Código</td><td>Receita fixa na parede</td></tr><tr><td><code>.rodata</code></td><td><code>r--</code></td><td>Constantes/strings</td><td>Etiquetas</td></tr><tr><td><code>.data</code></td><td><code>rw-</code></td><td>Globais inic.</td><td>Arroz pronto</td></tr><tr><td><code>.bss</code></td><td><code>rw-</code></td><td>Globais zeradas</td><td>Panela vazia</td></tr><tr><td><strong>heap</strong></td><td><code>rw-</code></td><td><code>malloc/new</code></td><td>Bufê à la carte</td></tr><tr><td><strong>stack</strong></td><td><code>rw-</code></td><td>Frames de função</td><td>Torre de pratos (LIFO)</td></tr></tbody></table>

**Hands‑on (5 min)**\
`readelf ‑S hello`   → marque endereço e tamanho de cada segmento.

#### 1.2 Registradores essenciais

<table><thead><tr><th width="174.3333740234375">Arquitetura</th><th>Propósito</th><th>Exemplos</th></tr></thead><tbody><tr><td>x86 (32 bit)</td><td>Dados</td><td><code>EAX</code>, <code>EBX</code>, <code>ECX</code>, <code>EDX</code></td></tr><tr><td>x86‑64 (64 bit)</td><td>Dados estendidos</td><td><code>RAX</code>, <code>RBX</code>, … <code>R15</code></td></tr><tr><td>Ambos</td><td>Stack ptr</td><td><code>ESP</code> / <code>RSP</code></td></tr><tr><td>Ambos</td><td>Base ptr</td><td><code>EBP</code> / <code>RBP</code></td></tr><tr><td>Ambos</td><td>Próxima instrução</td><td><code>EIP</code> / <code>RIP</code></td></tr></tbody></table>

_Micro‑lab_:

```
gdb ./hello
start
info reg rip rsp rbp
```

#### 1.3 Stack × Heap visual

```
Endereços altos ↓
|   Stack   |  ← cresce para baixo
|-----------|
|   Heap    |  ← cresce para cima
|-----------|
| .bss/.data|
|-----------|
|   .text   |
Endereços baixos ↑
```

***

### 2 Sistemas Operacionais em 10 minutos

1. **Modo usuário × modo kernel** ‑ privilégio e interrupções.
2. **Syscalls** ‑ ponte de user para kernel (Linux: `int 0x80`, `syscall`; Windows: ntdll → KiFastSystemCall).
3. **Carregador (loader)** ‑ mapeia ELF/PE na memória.
4. **Gerência de memória virtual** ‑ páginas, proteção (`rwx`).

_Exercício‑flash_: use `strace ls` (Linux) ou Process Monitor (Win) e liste cinco syscalls mais comuns de um “Hello World”.

***

### 3 Assembly Essencial (x86/x64)

#### 3.1 Sintaxe AT\&T × Intel

* AT\&T (prefixo %) → `mov %eax, %ebx`
* Intel → `mov ebx, eax`

#### 3.2 Conjunto mínimo de instruções

| Grupo         | Exemplo                    | Uso                  |
| ------------- | -------------------------- | -------------------- |
| Transferência | `mov`, `lea`, `xchg`       | Dados                |
| Aritmética    | `add`, `sub`, `inc`, `dec` | Contadores           |
| Lógica        | `and`, `or`, `xor`, `not`  | Flags / criptografia |
| Controle      | `call`, `ret`, `jmp`, `je` | Fluxo                |
| Pilha         | `push`, `pop`, `leave`     | Frames               |

#### 3.3 Calling conventions

* **System V AMD64** (Linux/macOS): argumentos → `RDI RSI RDX RCX R8 R9`.
* **Microsoft x64**: `RCX RDX R8 R9`, resto stack.

_Lab rápido_: compile `gcc -fasynchronous-unwind-tables -S hello.c` e abra o `.s`.

***

### 4 Ferramentas Essenciais (setup mínimo)

| Categoria               | Ferramenta           | Tarefa inicial              |
| ----------------------- | -------------------- | --------------------------- |
| Disassembler/Decompiler | **Ghidra**           | Abrir ELF/PE, renomear main |
| Disassembler GUI        | **IDA Free**         | Navegar gráfico de funções  |
| CLI + scriptável        | **Radare2/iaito**    | `aaa`, `pdf`, `VV`          |
| Debugger Linux          | **GDB + GEF/Pwndbg** | Break main, print stack     |
| Debugger Windows        | **x64dbg**           | Trace `printf`              |
| Helper Exploit          | **pwntools**         | `cyclic()` & socket         |



> **Dica de Feynman**: se você não consegue explicar o comando `pdf @ main` para outra pessoa, volte e pratique até fazer sentido.

***

### 5 Checkpoint de Aprendizagem

* [ ] Desenhei à mão um diagrama completo stack/heap/segments.
* [x] Pausar um programa no GDB/x64dbg e localizar `RIP`.
* [ ] Explicar a diferença entre modo usuário e modo kernel em ≤ 60 s.
* [ ] Converter pseudo‑código simples em assembly mentalmente.



***

#### Prova Relâmpago (auto‑cheque)

1. Qual registrador recebe o **1.º argumento** em System V AMD64?
2. O que acontece internamente quando você faz `call func`?
3. Cite duas diferenças entre heap e stack.

Se travar em qualquer pergunta, revise a seção correspondente.

