---
description: CTF Try Out - Reversing
---

# FlagCasino

## HTB Casino Challenge - Writeup Completo de Engenharia Reversa

### Visão Geral do Challenge

**Nome do Challenge:** Casino\
**Categoria:** Engenharia Reversa\
**Dificuldade:** very easy\
**Flag:** `HTB{r4*******************bl3}`

### Análise Inicial

#### Informações do Arquivo

```
Arquivo: casino
Tamanho: 16.936 bytes (0x42a8)
Tipo: ELF 64-bit LSB executable
MD5: 0478c3a259655afcd1cc67e808c20861
SHA256: b93bef52fd4178d7914752d87bf0b5ca2b93fcbd9d76a668cd68c8f873169257
```

#### Execução Básica

Quando executado, o binário apresenta uma interface temática de casino e solicita entrada de caracteres em um loop. Entrada incorreta resulta em terminação com uma mensagem de segurança.

### Análise Estática com IDA Pro

#### Identificação de Funções

O binário contém várias funções, com a lógica principal residindo em:

* Função `main` no endereço `0x1185`
* Função placeholder stub em `0x1020`

#### Análise da Função Main

A função main decompilada revela o algoritmo principal:

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char v4; // [rsp+Bh] [rbp-5h] BYREF
  unsigned int i; // [rsp+Ch] [rbp-4h]

  puts("[ ** WELCOME TO ROBO CASINO **]");
  puts(
    "     ,     ,\n"
    "    (\\____/)\n"
    "     (_oo_)\n"
    "       (O)\n"
    "     __||__    \\)\n"
    "  []/______\\[] /\n"
    "  / \\______/ \\/\n"
    " /    /__\\\n"
    "(\\   /____\\\n"
    "---------------------");
  puts("[*** PLEASE PLACE YOUR BETS ***]");
  
  for ( i = 0; i <= 0x1C; ++i )
  {
    printf("> ");
    if ( (unsigned int)__isoc99_scanf(" %c", &v4) != 1 )
      exit(-1);
    srand(v4);
    if ( rand() != check[i] )
    {
      puts("[ * INCORRECT * ]");
      puts("[ *** ACTIVATING SECURITY SYSTEM - PLEASE VACATE *** ]");
      exit(-2);
    }
    puts("[ * CORRECT *]");
  }
  puts("[ ** HOUSE BALANCE $0 - PLEASE COME BACK LATER ** ]");
  return 0;
}
```

#### Análise do Algoritmo

O algoritmo opera da seguinte forma:

1. O loop executa 29 vezes (0 até 0x1C = 28)
2. Para cada iteração:
   * Lê um único caractere da entrada do usuário
   * Usa o valor ASCII do caractere como seed para `srand()`
   * Gera um número aleatório usando `rand()`
   * Compara o resultado com `check[i]`
   * Termina a execução se houver divergência

#### Variável Global Crítica

O array `check` no endereço `0x4080` contém 116 bytes de valores esperados. Como há 29 iterações e cada chamada `rand()` retorna um inteiro de 32 bits, isso se traduz em 29 valores esperados de 4 bytes cada.

#### Extração de Dados Brutos

Os bytes brutos do array `check`:

```
0xbe 0x28 0x4b 0x24 0x5 0x78 0xf7 0xa 0x17 0xfc 0xd 0x11 0xa1 0xc3 0xaf 0x7
0x33 0xc5 0xfe 0x6a 0xa2 0x59 0xd6 0x4e 0xb0 0xd4 0xc5 0x33 0xb8 0x82 0x65 0x28
0x20 0x37 0x38 0x43 0xfc 0x14 0x5a 0x5 0x9f 0x5f 0x19 0x19 0x20 0x37 0x38 0x43
0x80 0x93 0x14 0x63 0x99 0xb2 0x5a 0x61 0x33 0xc5 0xfe 0x6a 0xb8 0xcf 0x6f 0x6c
0x20 0x37 0x38 0x43 0x37 0xa2 0x3d 0xf 0x33 0xc5 0xfe 0x6a 0x99 0xb2 0x5a 0x61
0xb8 0x82 0x65 0x28 0xfc 0x14 0x5a 0x5 0x94 0x49 0xe4 0x3a 0xe9 0xdf 0xd7 0x6
0xa2 0x59 0xd6 0x4e 0xcd 0x4a 0xcd 0xc 0x64 0xed 0xd8 0x57 0x99 0xb2 0x5a 0x61
0x2a 0xbc 0xe9 0x22
```

### Análise de Vulnerabilidade

A vulnerabilidade crítica reside na natureza previsível da função `rand()` da biblioteca padrão C:

1. **Saída Determinística**: Dado o mesmo seed, `rand()` sempre produz a mesma sequência
2. **Chamada Única**: Apenas uma chamada `rand()` por seed significa que cada caractere mapeia para exatamente um valor esperado
3. **Espaço de Chaves Limitado**: Apenas caracteres ASCII imprimíveis (32-126) precisam ser testados

### Desenvolvimento da Solução

#### Abordagem

A solução usa uma abordagem de força bruta que aproveita a natureza determinística de `rand()`:

1. Converter o array de bytes brutos para inteiros de 32 bits (little-endian)
2. Para cada valor esperado, testar todos os caracteres ASCII imprimíveis
3. Usar cada caractere como seed para `srand()`
4. Comparar a saída de `rand()` com o valor esperado
5. Armazenar caracteres correspondentes para construir a flag

#### Implementação

```python
#!/usr/bin/env python3
import ctypes
import os

def get_libc():
    """Carrega a libc apropriada para o sistema"""
    try:
        if os.name == 'posix':
            return ctypes.CDLL("libc.so.6")
        else:
            return ctypes.CDLL("msvcrt.dll")
    except OSError:
        return None

def solve_casino():
    raw_bytes = [
        0xbe, 0x28, 0x4b, 0x24, 0x5, 0x78, 0xf7, 0xa, 0x17, 0xfc, 0xd, 0x11, 0xa1, 0xc3, 0xaf, 0x7,
        0x33, 0xc5, 0xfe, 0x6a, 0xa2, 0x59, 0xd6, 0x4e, 0xb0, 0xd4, 0xc5, 0x33, 0xb8, 0x82, 0x65, 0x28,
        0x20, 0x37, 0x38, 0x43, 0xfc, 0x14, 0x5a, 0x5, 0x9f, 0x5f, 0x19, 0x19, 0x20, 0x37, 0x38, 0x43,
        0x80, 0x93, 0x14, 0x63, 0x99, 0xb2, 0x5a, 0x61, 0x33, 0xc5, 0xfe, 0x6a, 0xb8, 0xcf, 0x6f, 0x6c,
        0x20, 0x37, 0x38, 0x43, 0x37, 0xa2, 0x3d, 0xf, 0x33, 0xc5, 0xfe, 0x6a, 0x99, 0xb2, 0x5a, 0x61,
        0xb8, 0x82, 0x65, 0x28, 0xfc, 0x14, 0x5a, 0x5, 0x94, 0x49, 0xe4, 0x3a, 0xe9, 0xdf, 0xd7, 0x6,
        0xa2, 0x59, 0xd6, 0x4e, 0xcd, 0x4a, 0xcd, 0xc, 0x64, 0xed, 0xd8, 0x57, 0x99, 0xb2, 0x5a, 0x61,
        0x2a, 0xbc, 0xe9, 0x22
    ]
    
    libc = get_libc()
    if not libc:
        return None
    
    # Converte bytes para inteiros de 32 bits (little-endian)
    targets = []
    for i in range(0, len(raw_bytes), 4):
        if i + 3 < len(raw_bytes):
            value = (raw_bytes[i] | 
                    (raw_bytes[i+1] << 8) | 
                    (raw_bytes[i+2] << 16) | 
                    (raw_bytes[i+3] << 24))
            targets.append(value)
    
    flag = []
    
    # Resolve cada posição
    for pos, target in enumerate(targets):
        found_char = None
        
        # Testa caracteres ASCII imprimíveis
        for ascii_val in range(32, 127):
            libc.srand(ascii_val)
            result = ctypes.c_uint(libc.rand()).value
            
            if result == target:
                found_char = chr(ascii_val)
                flag.append(found_char)
                break
        
        if not found_char:
            return None
    
    return ''.join(flag)

if __name__ == "__main__":
    flag = solve_casino()
    print(f"Flag: {flag}")
```

### Execução da Solução

Executar o solver produz a seguinte saída:

```
Posição  1/29: Target = 0x244b28be -> 'H' (ASCII 72)
Posição  2/29: Target = 0x0af77805 -> 'T' (ASCII 84)
Posição  3/29: Target = 0x110dfc17 -> 'B' (ASCII 66)
Posição  4/29: Target = 0x07afc3a1 -> '{' (ASCII 123)
Posição  5/29: Target = 0x6afec533 -> 'r' (ASCII 114)
...
Posição 29/29: Target = 0x22e9bc2a -> '}' (ASCII 125)

Flag: HTB{r4nd_**_****_********bl3}
```

### Verificação

A solução pode ser verificada executando o binário original e inserindo cada caractere da flag sequencialmente. O programa aceitará todos os caracteres e exibirá a mensagem de sucesso.

### Insights Principais e Lições Aprendidas

#### Insights Técnicos

1. **Fraqueza de PRNG Determinístico**: A função `rand()` da biblioteca padrão C é determinística quando semeada, tornando-a inadequada para propósitos criptográficos
2. **Vulnerabilidade de Seed de Uso Único**: Usar cada caractere como um seed fresco cria um mapeamento direto entre entrada e saída
3. **Espaço de Busca Limitado**: Apenas 95 caracteres ASCII imprimíveis reduzem significativamente a complexidade da força bruta

#### Implicações para Red Team

1. **Análise de PRNG**: Sempre analise como geradores de números aleatórios são semeados e usados em aplicações
2. **Exploração de Determinismo**: Procure por padrões onde entrada do usuário influencia diretamente saídas supostamente aleatórias
3. **Predição de Seed**: Em cenários do mundo real, seeds previsíveis podem comprometer sistemas criptográficos inteiros

#### Recomendações Defensivas

1. **PRNGs Criptograficamente Seguros**: Use `/dev/urandom`, `CryptGenRandom`, ou fontes similares para aleatoriedade crítica de segurança
2. **Semeadura Adequada**: Use fontes de alta entropia como tempo do sistema, IDs de processo e geradores de números aleatórios de hardware
3. **Evitar Entrada Direta do Usuário**: Nunca use entrada direta do usuário como seeds para geração de números aleatórios sensíveis à segurança

### Conclusão

Este challenge demonstra uma vulnerabilidade clássica no uso de geradores de números pseudo-aleatórios. A natureza determinística de `rand()` combinada com semeadura previsível cria uma fraqueza trivialmente explorável. A solução exigiu entender o algoritmo através de engenharia reversa, identificar a vulnerabilidade e implementar um ataque de força bruta eficiente que aproveita as propriedades matemáticas do PRNG.

A flag resume perfeitamente a lição central: geradores de números aleatórios são apenas tão imprevisíveis quanto seus padrões de implementação e uso permitem.
