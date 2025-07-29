---
icon: heading
---

# Stack & Heap

## Stack & Heap — Fundamentos, Diferenças e Exploração

> **Objetivo** – capacitar o aluno a distinguir de forma cirúrgica como **stack** e **heap** são criadas, usadas e corrompidas, e a empregar esse conhecimento em engenharia reversa, fuzzing e exploit development.

***

### 1 Visão de alto nível

| Aspecto     | **Stack**                                               | **Heap**                                                |
| ----------- | ------------------------------------------------------- | ------------------------------------------------------- |
| Criação     | Automática no início do processo                        | On‑demand (`malloc/new`, `HeapAlloc`, `VirtualAlloc`)   |
| Gerente     | CPU + ABI + SO                                          | Alocador (ptmalloc, jemalloc, LFH, Segment Heap)        |
| Crescimento | Endereços **decrescentes** (↑ altos → baixos)           | Endereços **cresc entes** (↓ baixos → altos)            |
| Finalidade  | Frames de função, variáveis locais, endereço de retorno | Objetos dinâmicos, buffers longos, estruturas complexas |
| Velocidade  | Muito rápida (ajuste de ponteiro)                       | Mais lenta (metadados, listas)                          |
| Limite      | Tamanho fixo por thread (ex.: 8 MB Linux)               | Somada à memória virtual disponível                     |

***

### 2 Stack em detalhe&#x20;

1. **Frame‑layout** → parâmetros ⟶ endereço de retorno ⟶ RBP anterior ⟶ locais.
2. **Prólogo/epílogo** controlam ponteiros `RSP/RBP`.
3. Vulnerabilidades clássicas: **stack overflow**, **SROP**, **stack pivot**.
4. Defesas: canário, NX/DEP, ASLR, shadow stack.

> **Lab relâmpago** – `pwndbg> telescope $rsp` e veja a torre da função atual.

***

### 3 Heap em detalhe (glibc ptmalloc)

#### 3.1 Estrutura `malloc_chunk`&#x20;

<pre><code><strong>(prev_size)  ─┐  usado apenas se chunk livre
</strong>size         ─┤  flag bits in‑use (prev_inuse, mmap, non_main)
fd / bk      ─┤  double‑linked lists (bins)
user data    ─┘  devolvido ao programa
</code></pre>

#### 3.2 Zonas e bins

| Bin                          | Alvo             | Ataques comuns                     |
| ---------------------------- | ---------------- | ---------------------------------- |
| **Fastbin** (≤0x80)          | listas LIFO      | double‑free, fastbin dup           |
| **tcache** (glibc ≥2.26)     | cache por thread | tcache poisoning                   |
| **Unsorted / Small / Large** | chunks > 0x80    | unsorted bin leak, largebin attack |

#### 3.3 Vulnerabilidades chave

* **Heap Overflow** → sobrescreve metadados ou vtables.
* **Use‑After‑Free (UAF)** → ponteiro usa memória já liberada.
* **Double‑Free** → chunk reinserido em lista; permite escrever ponteiros arbitrários.
* **House of系列** (Force, Spirit, Orange…) – técnicas clássicas de manipulação.

#### 3.4 Proteções modernas

| Proteção                   | Implementação       | Bypass típico                |
| -------------------------- | ------------------- | ---------------------------- |
| Safe‑linking (glibc 2.32+) | XOR fd com ASLR key | partial overwrite + leak key |
| Tcache key                 | 0x5555 XOR bitshift | leak via format‑string       |
| LFH cookies (Windows)      | Encod. ptr 8 bits   | spray + bruteforce           |

> **Lab relâmpago** – compile `heap_demo.c`, use `gef> heap bins` para visualizar fastbins → provoque double‑free.

***

### 4 Stack × Heap em Exploit Chains

1. **Leak** pointer da heap via UAF  → usa para calcular offset libc.
2. **Pivot** stack para página RWX na heap (`pop rsp`).
3. **ROP/Shellcode** reside na heap (menos restrições de tamanho).

#### Exemplo real (CVE‑2021‑3156 – sudoedit)

* Heap overflow altera `struct sudo_locale`.
* Leak ponteiro heap → base libc.
* ROP chain construído na stack mas salta para heap para contornar NX.

***

### 5 Ferramentas essenciais

| Tarefa          | Ferramenta            | Comando‑chave              |
| --------------- | --------------------- | -------------------------- |
| Visualizar bins | pwndbg / gef          | `heap` / `heap bins`       |
| Fuzz heap       |  AFL + jemalloc debug | custom hooks               |
| Heap spray      | pwntools              | `flat(b'A'*n)`             |
| Detect UAF      | AddressSanitizer      | compile ‑fsanitize=address |

***

### 6 Checklist de Maestria

* [ ] Explicar, em ≤90 seg, como fastbin dup vira write‑what‑where.
* [ ] Reconhecer frame stack no GDB e localizar endereço de retorno.
* [ ] Descrever diferença conceitual de overflow vs UAF.
* [ ] Usar `malloc_stats` para ver arenas em tempo real.

***

### 7 Referências oficiais

1. **glibc malloc.c** (comentado)
2. **ISO C11 §7.22.3** – Memory management
3. **“Advanced Heap Exploitation” – Google Project Zero blog**
4. **Windows Low‑Fragmentation Heap Internals (Alex Ionescu)**
5. **Heap Exploitation Handbook (RPISEC)**

Conhecer a **stack** ensina a pilotar o fluxo; dominar a **heap** dá espaço para criar infraestrutura de ataque. Juntas, formam o palco onde toda peça de exploração se apresenta. 🚀
