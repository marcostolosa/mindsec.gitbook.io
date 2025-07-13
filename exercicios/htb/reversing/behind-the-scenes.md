---
description: Reversing Challenge - Very Easy
---

# Behind the Scenes

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 🏆 HACKTHEBOX - BEHINDTHESCENES WRITEUP COMPLETO

> **"Algumas vezes a melhor maneira de derrotar o anti-debug é nunca debuggar"**

***

### 🎯 OBJETIVO

Extrair a flag escondida de um binário ELF64 (protegido por técnicas anti-debug avançadas).

***

### 🔍 RECONHECIMENTO INICIAL

#### Verificação Básica do Arquivo

```bash
# Identificar tipo do arquivo
$ file behindthescenes
behindthescenes: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c20c22d2fb4b38ad37f5b2e5adf8b5b1f3f4e3d5, for GNU/Linux 3.2.0, not stripped

# Verificar permissões e tamanho
$ ls -la behindthescenes
-rwxr-xr-x 1 user user 14856 Dec 15 10:30 behindthescenes

# Verificar se há símbolos de debug
$ readelf -S behindthescenes | grep debug
# Resultado: Nenhuma seção de debug (stripped parcialmente)

# Testar execução básica
$ ./behindthescenes
./challenge <password>
```

**📊 Primeiras Observações:**

* Binário ELF64 dinâmico padrão
* Não stripped (símbolos disponíveis)
* Requer argumento de senha
* Mensagem de uso limpa

#### Análise de Strings

```bash
# Extrair strings legíveis
$ strings behindthescenes
/lib64/ld-linux-x86-64.so.2
libc.so.6
strncmp          # ← CRÍTICO: Comparação de strings
puts
__stack_chk_fail
printf           # ← CRÍTICO: Output
strlen
sigemptyset      # ← SUSPEITO: Signal handling
memset
sigaction        # ← RED FLAG: Anti-debug
__cxa_finalize
__libc_start_main
./challenge <password>
GLIBC_2.4
GLIBC_2.2.5

# Extrair strings com offset para localização
$ strings -o behindthescenes | grep -E "(password|flag|htb|challenge)"
    8196 ./challenge <password>
```

**🚨 Red Flags Identificados:**

* `sigaction` + `sigemptyset` = Manipulação de signals
* `strncmp` = Comparação de senha
* `printf` = Output de resultado
* Nenhuma string óbvia de senha

#### Análise de Imports/Exports

```bash
# Verificar símbolos importados
$ objdump -T behindthescenes
DYNAMIC SYMBOL TABLE:
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 strncmp
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 puts
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.4   __stack_chk_fail
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 printf
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 strlen
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 sigemptyset
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 memset
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 sigaction

# Verificar símbolos exportados
$ nm behindthescenes | grep -E "(main|sigaction|handler)"
0000000000101261 T main
0000000000101229 t sigill_sigaction  # ← FUNÇÃO SUSPEITA!
```

**🎯 Análise dos Imports:**

* Signal handling é confirmado (`sigaction`, `sigemptyset`)
* String processing presente (`strncmp`, `strlen`)
* Output capability (`printf`, `puts`)
* Stack protection ativa (`__stack_chk_fail`)

***

### 🧪 TESTE DINÂMICO INICIAL

#### Testes de Execução

```bash
# Teste sem argumentos
$ ./behindthescenes
./challenge <password>

# Teste com senha simples
$ ./behindthescenes test
# Resultado: Nada (silêncio total)

# Teste com senha comum
$ ./behindthescenes password
# Resultado: Nada

# Teste com buffer overflow
$ ./behindthescenes $(python3 -c "print('A'*100)")
# Resultado: Nada (sem crash)

# Teste com caracteres especiais
$ ./behindthescenes "!@#$%^&*()"
# Resultado: Nada
```

**🧠 Observação Crítica:** O programa tem **falha silenciosa** - não dá feedback de erro nem sucesso. Isso é extremamente suspeito e indica proteção.

#### Verificação de Proteções

```bash
# Verificar proteções de binário
$ checksec --file=behindthescenes
RELRO           : Full RELRO
Stack           : Canary found
NX              : NX enabled
PIE             : PIE enabled
RUNPATH         : No RUNPATH
Symbols         : 73 Symbols

# Verificar se há anti-debug óbvio
$ ltrace ./behindthescenes test 2>&1 | head -10
sigaction(SIGILL, { 0x101229, [], SA_SIGINFO }, NULL) = 0  # ← SIGILL!
# Processo para aqui...

# Verificar system calls
$ strace -e trace=signal ./behindthescenes test 2>&1
rt_sigaction(SIGILL, {sa_handler=0x555555555229, sa_mask=[], sa_flags=SA_SIGINFO}, NULL, 8) = 0
--- SIGILL {si_signo=SIGILL, si_code=ILL_ILLOPN, si_addr=0x555555555274} ---
# SIGILL disparado!
```

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

**🔥 DESCOBERTA CRUCIAL:**

* `sigaction(SIGILL, ...)` = Registra handler para instrução ilegal
* `SIGILL` é disparado durante execução
* Isso indica **técnica anti-debug UD2**!

***

### 🛡️ IDENTIFICAÇÃO DA TÉCNICA ANTI-DEBUG

#### Análise Estática com Ghidra

**Setup do Ghidra**

```bash
# Abrir Ghidra
$ ghidra

# Processo no Ghidra:
# 1. File → Import File → behindthescenes
# 2. Analysis → Auto Analyze → marcar todas opções
# 3. Aguardar análise completa
```

**Análise da Função Main**

Navegando para a função `main` no Ghidra:

```c
void main(void) {
  code *pcVar1;
  long in_FS_OFFSET;
  sigaction local_a8;
  undefined8 local_10;
  
  // Stack canary setup
  local_10 = *(undefined8 *)(in_FS_OFFSET + 0x28);
  
  // Limpa estrutura sigaction
  memset(&local_a8,0,0x98);
  sigemptyset(&local_a8.sa_mask);
  
  // Configura handler personalizado
  local_a8.__sigaction_handler.sa_handler = sigill_sigaction;
  local_a8.sa_flags = 4;  // SA_SIGINFO
  
  // Registra handler para SIGILL (signal 4)
  sigaction(4,&local_a8,(sigaction *)0x0);
  
  // LINHA CRÍTICA: Executa instrução ilegal
  pcVar1 = (code *)invalidInstructionException();
  (*pcVar1)();
}
```

**Análise do Signal Handler**

```c
void sigill_sigaction(undefined8 param_1,undefined8 param_2,long param_3) {
  // Incrementa RIP em 2 bytes para pular instrução UD2
  *(long *)(param_3 + 0xa8) = *(long *)(param_3 + 0xa8) + 2;
  return;
}
```

**🎯 Compreensão da Técnica:**

1. **Setup**: Registra handler customizado para SIGILL
2. **Trigger**: Executa instrução UD2 (illegal instruction)
3. **Bypass**: Handler incrementa RIP+2 para pular UD2
4. **Execution**: Código real executa após a instrução ilegal

#### Procurando a Instrução UD2

```bash
# Buscar bytes UD2 (0x0F 0x0B) no binário
$ hexdump -C behindthescenes | grep "0f 0b"
00001270  48 83 c4 08 c3 66 0f 1f  44 00 00 0f 0b 48 83 ec  |H....f..D...H..|

# Confirmar localização com objdump
$ objdump -d behindthescenes | grep -A5 -B5 "ud2"
0000000000001272:	0f 0b                	ud2    
0000000000001274:	48 83 ec 08          	sub    $0x8,%rsp
# Código continua após UD2!
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

**💡 Insight Crucial:** O código real está localizado **imediatamente após** a instrução UD2 em `0x1274`!

***

### 🎪 ESTRATÉGIA DE BYPASS

Identificamos que é uma técnica anti-debug UD2. Temos várias opções:

#### Opção 1: Bypass Dinâmico (GDB)

#### Opção 2: Patch do Binário

#### Opção 3: Análise Estática dos Dados (ESCOLHIDA!)

**Por que escolhi a Opção 3:**

* Mais elegante e educativa
* Não requer execução do código protegido
* Demonstra poder da análise estática

***

### 🔍 ANÁLISE ESTÁTICA - DATA HUNTING

#### Examinando a Seção .rodata

```bash
# Listar seções do binário
$ readelf -S behindthescenes | grep rodata
  [16] .rodata           PROGBITS         0000000000102000  00002000
       0000000000000036  0000000000000000   A       0     0     1

# Dump da seção .rodata
$ objdump -s -j .rodata behindthescenes
Contents of section .rodata:
 102000 01000200 2e2f6368 616c6c65 6e676520  ..../challenge 
 102010 3c706173 73776f72 643e0049 747a005f  <password>.Itz._
 102020 306e004c 795f0055 44320003 e204854  0n.Ly_.UD2......
 102030 425b2573 7d0a00                      HTB{%s}..

# Converter hex para ASCII para melhor visualização
$ readelf -x .rodata behindthescenes
Hex dump of section '.rodata':
  0x00102000 01000200 2e2f6368 616c6c65 6e676520 ..../challenge 
  0x00102010 3c706173 73776f72 643e0049 747a005f <password>.Itz._
  0x00102020 306e004c 795f0055 44320003 e204854  0n.Ly_.UD2......
  0x00102030 425b2573 7d0a00                      HTB{%s}..
```

#### Análise Detalhada dos Dados

```bash
# Analisar endereços específicos
$ python3 -c "
data = bytes.fromhex('49747a005f306e004c795f0055443200')
strings = []
current = b''
for byte in data:
    if byte == 0:
        if current:
            strings.append(current.decode())
            current = b''
    else:
        current += bytes([byte])
if current:
    strings.append(current.decode())
print('Strings encontradas:', strings)
"
# Resultado: ['Itz', '_0n', 'Ly_', 'UD2']
```

**🔥 DESCOBERTA DA SENHA!**

#### Reconstrução da Senha

```bash
# Concatenar as partes encontradas
$ python3 -c "print('Itz' + '_0n' + 'Ly_' + 'UD2')"
Itz_0nLy_UD2

# Verificar format string HTB
$ python3 -c "print(bytes.fromhex('3e20485442257b73257d0a00').decode())"
> HTB{%s}
```

**💡 Análise da Senha:**

* **"Itz\_0nLy\_UD2"** = "It's only UD2"
* Referência direta à técnica anti-debug usada!
* Format string `"> HTB{%s}"` confirma formato da flag

***

### ⚡ TESTE DA SENHA DESCOBERTA

```bash
# Testar a senha encontrada
$ ./behindthescenes Itz_0nLy_UD2
> HTB{Itz_0nLy_UD2}
```

**🎉 SUCESSO! Flag capturada!**

***

### 🧪 VALIDAÇÃO E TESTES ADICIONAIS

#### Verificação de Case Sensitivity

```bash
# Testar case incorreto
$ ./behindthescenes itz_0nly_ud2
# Resultado: Silêncio (case sensitive confirmado)

# Testar variações
$ ./behindthescenes "Itz_0nLy_UD2 "
# Resultado: Silêncio (espaços extras não aceitos)
```

#### Análise do Comportamento de Falha

```bash
# Testar senhas incorretas
$ ./behindthescenes wrong_password
# Resultado: Saída silenciosa

# Verificar se há diferentes tipos de erro
$ ./behindthescenes ""
# Resultado: Saída silenciosa (provavelmente argc check)
```

#### Trace da Execução Correta

```bash
# Verificar chamadas de sistema com senha correta
$ strace -e trace=write ./behindthescenes Itz_0nLy_UD2 2>&1
write(1, "> HTB{Itz_0nLy_UD2}\n", 18) = 18
# Confirmado: printf() é chamado apenas com senha correta

# Verificar library calls
$ ltrace ./behindthescenes Itz_0nLy_UD2 2>&1
sigaction(SIGILL, { 0x101229, [], SA_SIGINFO }, NULL) = 0
strncmp("Itz_0nLy_UD2", "Itz_0nLy_UD2", 12) = 0  # ← COMPARAÇÃO!
printf("> HTB{%s}\n", "Itz_0nLy_UD2") = 18
```

**✅ Confirmação Final:**

* `strncmp()` compara nossa entrada com senha hardcoded
* `printf()` é chamado apenas quando comparação retorna 0 (igualdade)

***

### 🔧 MÉTODOS ALTERNATIVOS DE SOLUÇÃO

#### Método 1: Bypass com GDB

```bash
# Configurar GDB para ignorar SIGILL
$ gdb behindthescenes
(gdb) handle SIGILL nostop noprint pass
Signal        Stop	Print	Pass to program	Description
SIGILL        No	No	Yes		Illegal instruction

# Colocar breakpoint em strncmp
(gdb) break strncmp
Breakpoint 1 at 0x1050

# Executar com senha teste
(gdb) run test_password
Breakpoint 1, 0x00007ffff7e9f3a0 in strncmp () from /lib/x86_64-linux-gnu/libc.so.6

# Examinar argumentos
(gdb) x/s $rdi
0x7fffffffe123:	"test_password"
(gdb) x/s $rsi  
0x55555555201b:	"Itz_0nLy_UD2"  # ← SENHA REVELADA!
```

#### Método 2: Library Interposition

```bash
# Criar hook para strncmp
$ cat > hook_strncmp.c << 'EOF'
#define _GNU_SOURCE
#include <stdio.h>
#include <string.h>
#include <dlfcn.h>

int strncmp(const char *s1, const char *s2, size_t n) {
    static int (*original_strncmp)(const char*, const char*, size_t) = NULL;
    if (!original_strncmp) {
        original_strncmp = dlsym(RTLD_NEXT, "strncmp");
    }
    
    printf("[HOOK] Comparing: '%s' vs '%s'\n", s1, s2);
    return original_strncmp(s1, s2, n);
}
EOF

# Compilar hook
$ gcc -shared -fPIC -o hook.so hook_strncmp.c -ldl

# Usar hook
$ LD_PRELOAD=./hook.so ./behindthescenes test
[HOOK] Comparing: 'test' vs 'Itz_0nLy_UD2'
# Senha revelada!
```

#### Método 3: Binary Patching

```bash
# Localizar instrução UD2
$ objdump -d behindthescenes | grep -n ud2
    0000000000001272:	0f 0b                	ud2

# Fazer backup
$ cp behindthescenes behindthescenes.backup

# Patch UD2 (0x0F 0x0B) para NOPs (0x90 0x90)
$ printf '\x90\x90' | dd of=behindthescenes bs=1 seek=$((0x1272)) count=2 conv=notrunc

# Testar binário patchado
$ ./behindthescenes test
# Agora deve executar sem anti-debug
```

***

### 📊 ANÁLISE TÉCNICA PROFUNDA

#### Estrutura da Técnica Anti-Debug UD2

**Fluxo de Execução Normal:**

```
1. main() configura sigaction() para SIGILL
2. Handler definido: sigill_sigaction()  
3. Executa invalidInstructionException()
4. CPU encontra UD2 → gera SIGILL
5. Kernel chama sigill_sigaction()
6. Handler incrementa RIP+2 (pula UD2)
7. Execução continua no código oculto
8. Código oculto: strncmp(input, "Itz_0nLy_UD2")
9. Se igual: printf("> HTB{%s}", input)
10. Se diferente: return silencioso
```

**Análise das Estruturas:**

```c
// Estrutura sigaction (152 bytes)
struct sigaction {
    union {
        void (*sa_handler)(int);
        void (*sa_sigaction)(int, siginfo_t*, void*);
    } __sigaction_handler;           // +0x00 (8 bytes)
    sigset_t sa_mask;                // +0x08 (128 bytes)  
    int sa_flags;                    // +0x88 (4 bytes)
    void (*sa_restorer)(void);       // +0x90 (8 bytes)
};

// Context manipulation (offset 0xa8)
// ucontext_t.uc_mcontext.gregs[REG_RIP] = RIP + 2
```

#### Por Que a Técnica é Eficaz

**Vantagens:**

✅ **CPU-level protection**: Funciona abaixo das APIs\
✅ **Minimal overhead**: Apenas 2 bytes (UD2)\
✅ **Legitimate OS feature**: Signal handling é normal\
✅ **Code hiding**: Análise estática para na UD2

**Desvantagens:**

❌ **Static analysis bypass**: Dados visíveis na .rodata\
❌ **Emulation vulnerable**: Unicorn Engine ignora signals\
❌ **Patchable**: UD2 pode ser substituída por NOPs\
❌ **GDB configurable**: `handle SIGILL` bypassa proteção

***

### 💡 LIÇÕES APRENDIDAS

#### Técnicas de Engenharia Reversa

**1. Análise de Imports é Crucial**

```bash
sigaction + strncmp + printf = senha protegida por anti-debug
```

**2. Silent Failure é Red Flag**

Programas normais dão feedback. Silêncio indica proteção.

**3. Data Hunting vs Code Analysis**

Às vezes os dados revelam mais que o código.

**4. Multiple Attack Vectors**

Sempre ter planos B, C, D para diferentes cenários.

#### Conceitos de Anti-Debug

**UD2 Technique Breakdown:**

* **Signal**: SIGILL (Illegal Instruction)
* **Handler**: Custom function to bypass
* **Mechanism**: RIP manipulation (+2 bytes)
* **Hiding**: Code after illegal instruction

**Detection Methods:**

```bash
# Static analysis
objdump -d binary | grep ud2
hexdump -C binary | grep "0f 0b"

# Dynamic analysis  
strace -e trace=signal binary
ltrace binary | grep sigaction
```

#### Red Team Applications

**Offensive Usage:**

```c
// Malware payload protection
void deploy_evil() {
    setup_sigill_handler();
    __asm__("ud2");
    // Hidden payload code here
    inject_shellcode();
}
```

**EDR Evasion:**

* Signal handling is legitimate behavior
* No suspicious API calls
* CPU-level protection vs software detection

***

### 🛠️ FERRAMENTAS UTILIZADAS

#### Análise Estática:

```bash
file            # Identificação de tipo
strings         # Extração de strings
objdump         # Disassembly e análise
readelf         # Análise de ELF headers/sections
hexdump         # Análise hexadecimal
nm              # Símbolos
checksec        # Verificação de proteções
```

#### Análise Dinâmica:

```bash
strace          # System call tracing
ltrace          # Library call tracing  
gdb             # Debugging
```

#### Ferramentas Avançadas:

```bash
ghidra          # Reverse engineering framework
python3         # Scripts de automação
gcc             # Compilação de hooks
```

***

### 📈 TIMELINE DA SOLUÇÃO

| **Tempo** | **Ação**               | **Descoberta**            |
| --------- | ---------------------- | ------------------------- |
| 0-10min   | Reconhecimento inicial | ELF64, imports suspeitos  |
| 10-20min  | Teste dinâmico         | Falha silenciosa, SIGILL  |
| 20-30min  | Análise com Ghidra     | Técnica anti-debug UD2    |
| 30-40min  | Data hunting           | Strings na .rodata        |
| 40-45min  | Reconstituição         | Senha: Itz\_0nLy\_UD2     |
| 45min     | Teste final            | Flag: HTB{Itz\_0nLy\_UD2} |

**Total: \~45 minutos** para solução completa

***

### 🎯 CONCLUSÃO

#### Flag Final

```
HTB{Itz_0nLy_UD2}
```

#### Método de Solução

**Análise estática da seção .rodata** - demonstrando que nem sempre precisamos lutar contra as proteções para vencê-las.

#### Lição Principal

> _"A melhor forma de vencer o anti-debug às vezes é nunca debuggar. Os dados não mentem, mesmo quando o código tenta esconder."_

#### Skills Desenvolvidas

✅ **Anti-debug recognition** (UD2 technique)\
✅ **Static analysis mastery** (data hunting)\
✅ **Signal handling understanding** (Linux internals)\
✅ **Multiple attack vectors** (GDB, patching, hooks)\
✅ **Problem-solving methodology** (lateral thinking)

***

### 🔗 REFERÊNCIAS

#### Documentação Técnica:

* [Intel UD2 Instruction Reference](https://www.intel.com/content/www/us/en/docs/ia-32-ia-64-developer-manual/)
* [Linux Signal Handling](https://man7.org/linux/man-pages/man7/signal.7.html)
* [sigaction(2) Manual](https://man7.org/linux/man-pages/man2/sigaction.2.html)

#### Recursos Educacionais:

* [Anti-Debug Techniques](https://anti-debug.checkpoint.com/)
* [Ghidra Documentation](https://ghidra-sre.org/CheatSheet.html)
* [ELF Format Specification](https://refspecs.linuxfoundation.org/elf/elf.pdf)

***

**Autor**: Ethical Hacker Underground\
**Plataforma**: HackTheBox\
**Desafio**: BehindTheScenes\
**Data**: 2024

_"Todo desafio é uma oportunidade de aprender algo novo sobre a arte da engenharia reversa."_

***

### 📝 COMANDOS RESUMIDOS

```bash
# Reconhecimento
file behindthescenes
strings behindthescenes
objdump -T behindthescenes

# Análise dinâmica
./behindthescenes test
strace -e trace=signal ./behindthescenes test
ltrace ./behindthescenes test

# Análise estática
ghidra  # Importar e analisar
readelf -x .rodata behindthescenes
objdump -s -j .rodata behindthescenes

# Descoberta da senha
python3 -c "print('Itz' + '_0n' + 'Ly_' + 'UD2')"

# Teste final
./behindthescenes Itz_0nLy_UD2
# Output: > HTB{Itz_0nLy_UD2}
```

**Flag: `HTB{Itz_0nLy_UD2}`** 🏆

#### **📊 FASE 1: RECONNAISSANCE - "SNIFFING THE TARGET"**

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>file behindthescenes</p></figcaption></figure>

{% code overflow="wrap" %}
```bash
$ file ./behindthescenes
behindthescenes: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=e60ae4c886619b869178148afd12d0a5428bfe18, for GNU/Linux 3.2.0, not stripped

$ strings behindthescenes | grep -v "lib\|GLIBC"
./challenge <password>
> HTB{%s}
strncmp
sigaction
printf
...

$ ./behindthescenes test
# Silêncio total... CURIOSO, NÃO!?
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>strings behindthescenes</p></figcaption></figure>

**🧠 Red Flag Mental:** _"Programa que não dá erro nem feedback? Isso é alguma proteção, parceiro!"_

#### **🔍 FASE 2: INTELLIGENCE GATHERING - "WHAT'S YOUR GAME?"**

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

**Ghidra time!** Jogando o binário no Ghidra, primeira coisa que vejo:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```c
// Main function decompilada
sigaction(4,&local_a8,(sigaction *)0x0);  // Signal 4 = SIGILL
pcVar1 = (code *)invalidInstructionException(); // ← BINGO!
```

**🔥 EUREKA MOMENT:** _"É UD2 anti-debug! Estão usando instrução ilegal pra esconder código!"_

#### **🧠 FASE 3: PATTERN RECOGNITION - "I SEE YOU"**

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

```c
void sigill_handler(context) {
    context->RIP += 2;  // Pula a instrução UD2
}
```

**Mental Model:**

```
Setup handler → Execute UD2 → SIGILL triggered → 
Handler skips UD2 → HIDDEN CODE executes
```

**🎯 Hacker Logic:** _"Se tem strncmp nos imports, a senha tá em algum lugar. Será?"_

#### **🔍 FASE 4: DATA HUNTING - "FOLLOW THE BREADCRUMBS"**

**Caçando na .rodata section:**

```
0x102004: "./challenge <password>"  // String conhecida
0x10201b: 49 74 7A 00              // "Itz\0" ← SUSPICIOUS!
0x10201f: 5F 30 6E 00              // "_0n\0" ← PATTERN!
0x102023: 4C 79 5F 00              // "Ly_\0" ← CONTINUING!
0x102027: 55 44 32 00              // "UD2\0" ← JACKPOT!
```

**🔥 Mental Explosion:** _"QUATRO STRINGS SEGUIDAS! Isso é a senha em pedaços!"_

#### **💡 FASE 5: PUZZLE SOLVING - "CONNECT THE DOTS"**

```
Part 1: "Itz"
Part 2: "_0n"  
Part 3: "Ly_"
Part 4: "UD2"

Password = "Itz_0nLy_UD2"
```

**🎪 The Beautiful Irony:** _"A senha é uma referência à própria técnica anti-debug! 'It's only UD2'"_

#### **⚡ FASE 6: VALIDATION - "MOMENT OF TRUTH"**

```bash
$ ./behindthescenes Itz_0nLy_UD2
> HTB{Itz_0nLy_UD2}
```

**🎉 BOOM!** _Flag capturada sem nem encostar no anti-debug!_

***

### **🧠 OS PILARES DA MENTALIDADE HACKER**

#### **1. NUNCA ACEITE O ÓBVIO**

* Programa silencioso = programa escondendo algo
* Imports revelam intenções (sigaction + strncmp = senha protegida)
* Uso de Ghidra = bypass automático de proteções

#### **2. PENSE COMO O ADVERSÁRIO**

* _"Se eu fosse esconder uma senha, onde colocaria?"_
* _"Como eu faria pra dificultar a vida do reverser?"_
* _"Que pistas eu deixaria sem querer?"_

#### **3. DADOS > CÓDIGO**

* Código pode mentir, dados não
* .rodata section é ouro puro para senhas
* Padrões sequenciais são sempre suspeitos

#### **4. LATERAL THINKING**

* Não lute contra a proteção, GO AROUND
* Se não pode debuggar, faça static analysis
* Use as ferramentas certas para o job certo

***

### **🎯 TÉCNICAS VISUAIS APLICADAS**

#### **Memory Layout Mental Model:**

```
┌─────────────────┐
│   .text         │ ← UD2 anti-debug aqui
├─────────────────┤
│   .rodata       │ ← SENHA AQUI! 🎯
│   0x102004      │   "./challenge <password>"
│   0x10201b      │   "Itz\0"
│   0x10201f      │   "_0n\0"  
│   0x102023      │   "Ly_\0"
│   0x102027      │   "UD2\0"
│   0x10202b      │   "> HTB{%s}\n\0"
└─────────────────┘
```

#### **Attack Vector Decision Tree:**

```
Binary Analysis
├── Dynamic? → Anti-debug detected → ❌ Hard path
└── Static? → Data analysis → ✅ Easy win!
```

***

### **🔥 LIÇÕES**

#### **🎪 Golden Rules:**

1. **"Imports Don't Lie"** - `sigaction` = anti-debug, `strncmp` = password check
2. **"Data Tells Stories"** - Sequential strings em .rodata = senha fragmentada
3. **"Sometimes the backdoor is the front door"** - Ghidra bypassa anti-debug automaticamente
4. **"Pattern Recognition > Brute Force"** - 4 strings + format string = óbvio demais

#### **🛠️ Ferramentas:**

```bash
# Reconnaissance
file, strings, hexdump, objdump

# Static Analysis  
Ghidra, IDA Pro, radare2, iaito (radare2 GUI)

# Dynamic Analysis (quando necessário)
GDB, strace, ltrace

# Binary Manipulation
hexedit, dd, python struct
```

***

### **📊 IMPACT ASSESSMENT**

#### **Skills Desenvolvidas:**

* ✅ **Anti-debug recognition** (UD2 technique)
* ✅ **Static analysis mastery** (Ghidra power user)
* ✅ **Pattern recognition** (data structure analysis)
* ✅ **Lateral thinking** (bypass vs breakthrough)

#### **Knowledge Gained:**

* ✅ **UD2 instruction behavior** and signal handling
* ✅ **ELF structure** and section analysis
* ✅ **CTF methodology** and writeup documentation
* ✅ **Tool selection** for different scenarios

***

### **🔥 CONCLUSÃO: THE HACKER WAY**

Esse challenge foi uma masterclass em **"work smarter, not harder"**. Enquanto outros hackers estariam quebrando a cabeça tentando bypass o anti-debug, eu fui direto na jugular: **os dados**.

**A real lição:** _"O melhor hack é aquele que nem parece hack. É só olhar no lugar certo."_

**🎯 Flag: `HTB{Itz_0nLy_UD2}`**

***

**Stay hungry, stay foolish, stay hacking! 🔥**
