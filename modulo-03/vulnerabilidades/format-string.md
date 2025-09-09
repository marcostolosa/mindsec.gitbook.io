# Format String Bug

## Exploração de Vulnerabilidade de String de Formato

Uma vulnerabilidade de **string de formato** ocorre quando a entrada do usuário é passada diretamente para funções de formatação como `printf`, `fprintf`, `sprintf` e similares, sem especificar corretamente a string de formato.

### Fundamentos

Funções como `printf` esperam uma string de formato que define como os argumentos subsequentes devem ser interpretados e exibidos. Por exemplo:

```c
printf("%s", user_input);
```

é seguro, pois especifica que `user_input` deve ser tratado como string (`%s`).

Já:

```c
printf(user_input);
```

é inseguro, porque se `user_input` contiver especificadores de formato (como `%x`, `%n`), o `printf` tentará ler argumentos inexistentes da pilha, causando vazamento de informações ou corrupção de memória.

### Impacto

* **Vazamento de memória**: `%x`, `%p` e similares permitem ler valores arbitrários da pilha.
* **Escrita arbitrária**: `%n` escreve o número de caracteres impressos até aquele ponto no endereço indicado.
* **Execução arbitrária de código**: combinando escrita controlada em endereços estratégicos.

### Exemplo de vazamento

Se `user_input = "%x %x %x %x"`, o `printf` imprimirá quatro valores da pilha.

### Exemplo de escrita

```c
int target = 0;
printf("%n", &target);
```

Isso escreverá o número de caracteres impressos até o momento no endereço de `target`.

### Exploração típica

1. **Identificar vulnerabilidade**
   * Inserir `%x` repetidamente até ver dados sensíveis na saída.
2. **Descobrir offset**
   * Contar quantos `%x` são necessários até alcançar o endereço/alvo.
3. **Manipular escrita**
   * Usar `%n` para escrever valores específicos em endereços de interesse.
4. **Exploração avançada**
   * Alterar ponteiros de função (GOT/PLT) ou variáveis de controle para redirecionar execução.

### Proteções comuns

* **FORTIFY\_SOURCE**: adiciona verificações extras de segurança.
* **Format String Warnings**: compiladores modernos alertam sobre uso inseguro de `printf`.
* **ASLR/DEP/PIE**: dificultam exploração, mas não previnem completamente.

### Mitigação

* Sempre especificar a string de formato explicitamente.
* Usar funções seguras (`snprintf`, `strncpy`).
* Validar/filtrar entrada do usuário.

***

Essa vulnerabilidade é poderosa porque pode ser explorada tanto para vazamento de informações quanto para execução arbitrária de código, dependendo do contexto e das proteções presentes.
