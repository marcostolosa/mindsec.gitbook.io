---
icon: memory
---

# Memória

## Memória — Fundamentos Absolutos para Engenharia Reversa

> **Objetivo**: transformar o aluno em alguém capaz de **visualizar, mapear e manipular** qualquer layout de memória (user‑mode ou kernel) e compreender como isso se traduz em vulnerabilidades, exploits, depuração e reversing.

***

### 1. Físico × Virtual — a ilusão controlada

<table><thead><tr><th width="177.66668701171875">Camada</th><th width="225.66668701171875">Quem gerencia</th><th>Visão do reverser</th></tr></thead><tbody><tr><td><strong>RAM física</strong></td><td>Controlador de memória + firmware (UEFI)</td><td>Endereços 0‑N reais, raramente tocados diretamente</td></tr><tr><td><strong>Memória virtual</strong></td><td>MMU + SO</td><td>Cada processo enxerga <strong>espaço contínuo de 0 até 2ⁿ</strong> (4 GB em 32‑bit, 128 TB+ em 64‑bit)</td></tr><tr><td><strong>Regiões mapeadas</strong></td><td>Loader/<code>mmap</code></td><td>Segmentos ELF/PE, libs, heap, stack</td></tr></tbody></table>

> **INFO‑LEAK**: qualquer ponteiro virtual vazado permite calcular deslocamentos e contornar ASLR.

#### 1.1 Tabelas de páginas & TLB

* **Page** = bloco mínimo (4 KB padrão).
* **Page table** mapeia _virtual → físico_ + permissões (R/W/X).
* **TLB** cacheia as últimas traduções (reversers abusam de _Flush+Reload_ em side‑channels).

**Kernel trick**: muitos exploits escrevem diretamente na página de page‑tables para desativar NX; hoje isolado por KPTI/SMEP.

***

### 2. Layout típico de processo (Linux 64‑bit)

```
0x0000_0000_0000 -----> NULL page (não mapeada)
[text]     r-x  |  ↙ código do binário
[rodata]   r--  |
[data]     rw-  |  ↙ globais inicializadas
[bss]      rw-  |  ↙ globais zeradas
[heap]  ← malloc RW  (cresce ↑ via brk / mmap)
[mmap]  ← libs, anon RWX (aleatórios)
[stack] ← RSP   RW   (cresce ↓)
[vdso]   r-x   |   syscalls lidam com vsyscall
[guard]  ---   |   páginas c0‑canary para stack overflow
```

> **Reverser tip**: `cat /proc/$$/maps` ou _Process Hacker_ (Windows) para visualizar.

***

### 3. Segmentos e Seções

<table><thead><tr><th width="147.66668701171875">ELF Section</th><th width="252.66668701171875">Conteúdo</th><th>Importância no RE</th></tr></thead><tbody><tr><td><code>.text</code></td><td>instruções</td><td>buscar gadgets ROP</td></tr><tr><td><code>.plt/.got</code></td><td>trampolins de import</td><td>ret2plt &#x26; leak libc</td></tr><tr><td><code>.rodata</code></td><td>strings constantes</td><td>mensagens, form‑strings, chaves</td></tr><tr><td><code>.data</code> / <code>.bss</code></td><td>globais</td><td>config, vtables</td></tr></tbody></table>

PE (Windows) segue a mesma lógica (`.text`, `.rdata`, `.data`, `.reloc`).&#x20;

Mach‑O usa **load commands** — reverser precisa reconhecer regiões pelo comando LC\_SEGMENT.

***

### 4. Stack em profundidade

1. **Prólogo**: salva RBP, alinha RSP.
2. **Locais + canary**: GCC insere `__stack_chk_guard`.
3. **Epilogo**: verifica canary, `leave; ret`.

> **Exploit link**: overflow → sobrescrever RET ou canary. _leave; ret_ é pivot clássico.

#### Shadow Stack / CET

* Processadores Intel Tiger Lake+ têm **shadow stack** (duplicata somente‑leitura dos RETs).
* Bypass atual: data‑oriented or SROP (sigreturn).

***

### 5. Heap detalhado (ptmalloc — glibc)

* **Chunks**: `prev_size | size | fwd | bk | USERDATA`.
* **fastbin attack** → double‑free + overwrite FD ⇒ arbitrary write.
* **tcache (glibc ≥2.26)**: freelist por thread, muito usado em CTF.

> Ferramentas: `pwndbg heap`, `heaptrace`, `gef heap bins`.

Windows **LFH / Low‑Fragmentation Heap** e **segment heap** (Win10) têm header `HEAP_USERDATA_HEADER`; exploits usam `LookasideChains`, **FrontEndAlloc**.

***

### 6. Permissões & Proteções

<table><thead><tr><th width="159">Proteção</th><th width="143.3333740234375">Nível</th><th width="218.6666259765625">Efeito</th><th>RE/Exploit impacto</th></tr></thead><tbody><tr><td><strong>NX/DEP</strong></td><td>HW + SO</td><td>página não executável</td><td>força ROP/JOP/Shellcode RWX via mprotect</td></tr><tr><td><strong>ASLR</strong></td><td>SO</td><td>randomiza bases</td><td>necessita leaks</td></tr><tr><td><strong>Canary</strong></td><td>compilador</td><td>detecção de stack‑ovf</td><td>precisa leak ou partial overwrite</td></tr><tr><td><strong>CFI / CET</strong></td><td>compiler+CPU</td><td>verifica destinos indiretos</td><td>migra para JOP / data‑oriented</td></tr><tr><td><strong>PAC (ARMv8.3)</strong></td><td>HW</td><td>assina ponteiros</td><td>gadgets re‑sign</td></tr></tbody></table>

***

### 7. Endianness & Alinhamento

* **Little‑endian** (x86, ARM‑LE) → bytes invertidos; imprescindível p/ construir payload (`0x41424344` → "DCBA").
* **Alignment faults**: algumas archs crasham se 4‑byte value não alinhado (MIPS strict) — usado como mitigação.

***

### 8. Memory‑oriented Vulnerabilities

<table><thead><tr><th width="203.6666259765625">Classe</th><th width="210.66668701171875">Região alvo</th><th>Exemplo CVE</th></tr></thead><tbody><tr><td><strong>BoF (stack)</strong></td><td>stack</td><td><code>CVE‑2017‑9805</code> Struts2</td></tr><tr><td><strong>Heap OF / UAF</strong></td><td>heap</td><td><code>CVE‑2021‑3156</code> sudo Baron Samedit</td></tr><tr><td><strong>OOB Read/Write</strong></td><td>qualquer</td><td>Heartbleed (OpenSSL)</td></tr><tr><td><strong>Info‑leak</strong></td><td>stack/heap/rodata</td><td>Spectre‑style side channel</td></tr></tbody></table>

***

### 9. Ferramentas de Observação de Memória

<table><thead><tr><th width="213">Tarefa</th><th width="311.6666259765625">Ferramenta</th><th>Comando‑chave</th></tr></thead><tbody><tr><td>Dump segmentos</td><td><code>readelf</code>, <code>objdump</code>, <code>dumpbin</code></td><td><code>objdump -h</code></td></tr><tr><td>Ver mapas de processo</td><td>Linux <code>/proc/$pid/maps</code></td><td></td></tr><tr><td>Live heap viz</td><td><code>pwndbg heap</code></td><td></td></tr><tr><td>Track allocs</td><td><code>valgrind --tool=massif</code>, <code>Dr.Memory</code></td><td></td></tr><tr><td>Dynamic hook</td><td><code>Frida</code>, <code>ltrace/strace</code>, <code>DTrace</code></td><td>override malloc</td></tr></tbody></table>

***

### 10. Linha de raciocínio prática p/ RE

1. **Mapeie**: veja onde o binário, libs, stack e heap caem à primeira execução (usando `vmmap`, `Process Hacker`).
2. **Pergunte**: _Qual região posso controlar?_ (input buffer? heap chunk?)
3. **Procure leaks**: strings format, info‑leaks que revelam `libc` ou ponteiros stack.
4. **Planeje**: se controle for na stack → pivote; se no heap → use House‑x pattern.
5. **Implemente**: debugger + scripting (`pwndbg`, `pwntools`).

***

#### Checklist “Feynman” (auto‑teste)

* [ ] Consigo explicar diferença de endereço virtual e físico em ≤90 s.
* [ ] Descrevo o fluxo de tradução `RIP → TLB → RAM`.
* [ ] Posso apontar no GDB onde está o canário de stack em um binário compilado com `-fstack-protector`.
* [ ] Sei enumerar pelo menos **três** ataques de heap (fastbin dup, tcache poisoning, largebin attack).

Dominar esse panorama de memória é **pré‑requisito** para todas as técnicas de exploit (BoF, UAF, ROP, SROP, heap feng‑shui, kernel pwn). Sem ele, engenharia reversa é mero “tour guiado”. Com ele, você **assume o volante**. 🤘
