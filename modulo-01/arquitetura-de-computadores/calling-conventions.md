---
icon: phone-arrow-right
---

# Calling Conventions

### 🔁 1. Calling Conventions (Convenções de Chamada) – Como funções se comunicam no x86

As **calling conventions** (ou _convenções de chamada_) definem **como funções recebem parâmetros e retornam valores ao final de sua execução**.

Esse conceito é **fundamental na engenharia reversa e exploração**, porque determina o layout da pilha e o uso dos registradores durante chamadas de função — o que afeta diretamente qualquer análise de controle de fluxo, manipulação de stack frames e crafting de payloads.

#### 🧠 Como funcionam as Calling Conventions?

Na arquitetura **x86**, é possível utilizar diversas convenções de chamada. Elas diferem em vários aspectos, como:

* **Como os parâmetros são passados**
  * Via **registradores** (como `EAX`, `ECX`, `EDX`)
  * Via **pilha** (`stack`) com instruções `PUSH`
  * Ou uma combinação dos dois
* **Ordem dos parâmetros**
  * Por exemplo, da direita para a esquerda (`stdcall`) ou esquerda para a direita (`fastcall`)
* **Quem limpa a pilha após a chamada**
  * Pode ser a **função chamada** (_callee_) ou a **função chamadora** (_caller_)
* **Quais registradores devem ser preservados**
  * Certas convenções exigem que a função chamada preserve valores de registradores como `EBX`, `ESI`, `EDI` — isso é chamado de **preservação do contexto**.

#### 🛠️ Exemplo prático:

| Convenção  | Ordem dos parâmetros | Limpeza da pilha | Uso comum                 |
| ---------- | -------------------- | ---------------- | ------------------------- |
| `cdecl`    | Direita → esquerda   | Caller           | Padrão em C/C++           |
| `stdcall`  | Direita → esquerda   | Callee           | API do Windows            |
| `fastcall` | 2 primeiros via regs | Callee           | Código otimizado, drivers |
| `thiscall` | `this` em ECX        | Callee           | Métodos C++ com `this`    |

#### 🧬 Engenharia Reversa na Prática

Durante análise de binários, é essencial identificar a calling convention correta para:

* Reconstituir a lógica de funções (mesmo sem símbolos);
* Evitar erros de análise no IDA, Ghidra ou radare2;
* Criar **stubs**, **shellcodes**, ou **ROP chains** válidas;
* Evitar corrupção de pilha ao simular chamadas.

Ferramentas como o **IDA Pro** ou **Ghidra** muitas vezes inferem a calling convention automaticamente, mas você pode corrigir manualmente quando identificar inconsistências.

#### 💡 Observação importante

Embora o **compilador geralmente escolha a calling convention padrão**, é possível que o desenvolvedor especifique uma diferente por função, usando diretivas como:

```c
__cdecl int soma(int a, int b);
__stdcall void conectarSocket();
__fastcall float calculaArea(int lado);
```

