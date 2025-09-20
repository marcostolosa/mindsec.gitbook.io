---
icon: function
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# C / C++

```c
/*
Aprenda X em Y Minutos
Onde X = C (https://learnxinyminutes.com/pt-br/c/)


Use Clang ou GCC.

Como compilar (recomendado):
  clang -std=c11 -Wall -Wextra -Wpedantic -Wconversion -Wsign-conversion \
        -Wshadow -Wpointer-arith -Wstrict-prototypes -Wundef -Werror \
        -fsanitize=address,undefined -fno-omit-frame-pointer -O2 learnc.c -o learnc

Ou:
  gcc   -std=c11 -Wall -Wextra -Wpedantic -Wconversion -Wsign-conversion \
        -Wshadow -Wpointer-arith -Wstrict-prototypes -Wundef -Werror \
        -fsanitize=address,undefined -fno-omit-frame-pointer -O2 learnc.c -o learnc

Depuração:
  ASAN_OPTIONS=detect_leaks=1 ./learnc
  gdb ./learnc   (ou lldb ./learnc)

Filosofia (K&R aplicada):
  1) Faça o programa funcionar antes de otimizá-lo.
  2) Torne-o claro antes de torná-lo esperto.
  3) Media o que importa (tempo, memória, I/O).
  4) Escreva interfaces pequenas e bem definidas; esconda detalhes em .c.
  5) Teste casos pequenos e extremos; fuja do comportamento indefinido (UB).
*/


// -------------------------------------------------------------
// Comentários e constantes
// -------------------------------------------------------------

// Comentários de uma linha (C99+)
#include <stddef.h> // size_t
/*
Comentários de múltiplas linhas funcionam desde o C89.
*/

// Constantes por macro:
#define DAY_IN_YEAR 365

// Enumerações também criam inteiros-constantes legíveis:
enum day { DOM = 1, SEG, TER, QUA, QUI, SEX, SAB };
// SEG recebe 2 automaticamente, TER 3, etc.

// -------------------------------------------------------------
// Includes usuais da biblioteca padrão
// -------------------------------------------------------------
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <stdint.h>  // tipos de largura fixa (uint32_t etc.)
#include <stdbool.h> // bool, true, false (C99)
#include <limits.h>  // limites de tipos inteiros
#include <errno.h>   // códigos de erro para I/O, strtoul, etc.

// Para cabeçalhos próprios, use aspas:
#include "minha_biblioteca.h"

// -------------------------------------------------------------
// Protótipos (declare antes de usar, ou em um .h)
// -------------------------------------------------------------
void funcao_1(void);
int  funcao_2(void);
int  soma_dois_ints(int x1, int x2); // protótipo de função
void str_reverse(char *str_in);

// -------------------------------------------------------------
// Ponto de entrada
// -------------------------------------------------------------
int main(void) {
    // Saída formatada: %d = inteiro, \n = nova linha
    printf("%d\n", 0); // => 0

    // ---------------------------------------------------------
    // Tipos
    // ---------------------------------------------------------
    int x_int = 0;           // normalmente 4 bytes
    short x_short = 0;       // normalmente 2 bytes
    char x_char = 0;         // sempre 1 byte (8 bits na prática)
    char y_char = 'y';       // literais de caractere usam aspas simples

    long x_long = 0;         // 4 ou 8 bytes
    long long x_long_long = 0; // >= 64 bits

    float  x_float  = 0.0f;  // 32 bits
    double x_double = 0.0;   // 64 bits

    unsigned short ux_short = 0;
    unsigned int   ux_int = 0u;
    unsigned long long ux_long_long = 0ull;

    // Caracteres são inteiros da tabela de caracteres da máquina (ASCII/UTF-8 no byte):
    (void)'0'; // 48 em ASCII
    (void)'A'; // 65 em ASCII

    // sizeof(T) e sizeof(expr) retornam o tamanho em bytes
    printf("sizeof(int) = %zu\n", sizeof(int));

    {
        int a = 1;
        size_t sz = sizeof(a++); // a++ NÃO é avaliada
        printf("sizeof(a++) = %zu; a = %d\n", sz, a);
    }

    // ---------------------------------------------------------
    // Arrays
    // ---------------------------------------------------------
    char meu_char_array[20];   // 20 bytes
    int  meu_int_array[20];    // 20 * sizeof(int)

    // Inicialização com zeros:
    char meu_array[20] = {0};
    meu_array[1] = 2;
    printf("%d\n", meu_array[1]); // => 2

    // VLA (C99; opção em C11): tamanho decidido em tempo de execução
    {
        char buf[0x100];
        fputs("Entre o tamanho do array: ", stdout);
        if (fgets(buf, sizeof buf, stdin)) {
            errno = 0;
            char *end = NULL;
            unsigned long n = strtoul(buf, &end, 10);
            if (errno == 0 && end != buf && n > 0 && n < 100000) {
                size_t size = (size_t)n;
                int var_length_array[size]; // VLA
                printf("sizeof array = %zu\n", sizeof var_length_array);
                // Ex.: entrada 10 -> sizeof = 40 (se int=4 bytes)
                (void)var_length_array; // evitar warning
            } else {
                fputs("tamanho inválido\n", stderr);
            }
        }
    }

    // Strings são arrays de char terminados por '\0'
    char uma_string[20] = "Isto e uma string"; // evite acentos em ASCII puro
    printf("%s\n", uma_string);
    printf("%d\n", uma_string[17]); // => 0 (byte nulo)

    // Literais de char têm tipo 'int' por história:
    int cha = 'a';
    char chb = 'a';
    (void)cha; (void)chb;

    // Arrays multidimensionais
    int multi_array[2][5] = {
        {1, 2, 3, 4, 5},
        {6, 7, 8, 9, 0}
    };
    int array_int = multi_array[0][2]; // 3
    (void)array_int;

    // ---------------------------------------------------------
    // Operadores
    // ---------------------------------------------------------
    {
        int i1 = 1, i2 = 2;
        float f1 = 1.0f, f2 = 2.0f;

        (void)(i1 + i2);
        (void)(i2 - i1);
        (void)(i2 * i1);
        (void)(i1 / i2); // 0

        (void)(f1 / f2); // ~0.5 (ponto flutuante não é exato)

        (void)(11 % 3);  // 2

        // Comparações retornam 0/1; em C99 há _Bool/bool
        (void)(3 == 2);
        (void)(3 != 2);
        (void)(3 >  2);
        (void)(3 <  2);
        (void)(2 <= 2);
        (void)(2 >= 2);

        // Não encadeie comparações como em Python:
        {
            int a = 1;
            int errado = 0 < a < 2;      // errado
            int certo  = 0 < a && a < 2; // certo
            (void)errado; (void)certo;
        }

        // Lógica
        (void)(!3);
        (void)(!0);
        (void)(1 && 1);
        (void)(0 || 1);

        // Ternário
        {
            int a = 5, b = 10;
            int z = (a > b) ? a : b; // 10
            (void)z;
        }

        // Pré/pós-incremento
        {
            const char *s = "iLoveC";
            int j = 0;
            (void)s[j++]; // 'i'
            j = 0;
            (void)s[++j]; // 'L'
        }

        // Bit a bit
        (void)(~0x0F);        // 0xF0
        (void)(0x0F & 0xF0);  // 0x00
        (void)(0x0F | 0xF0);  // 0xFF
        (void)(0x04 ^ 0x0F);  // 0x0B
        (void)(0x01 << 1);    // 0x02
        (void)(0x02 >> 1);    // 0x01

        // Cuidado com shifts em signed e largura do tipo: pode ser UB
    }

    // ---------------------------------------------------------
    // Estruturas de Controle
    // ---------------------------------------------------------
    if (0) {
        puts("Nunca roda");
    } else if (0) {
        puts("Também não");
    } else {
        puts("Eu serei impresso");
    }

    {
        int ii = 0;
        while (ii < 10) {
            printf("%d, ", ii++);
        }
        printf("\n");
    }

    {
        int kk = 0;
        do {
            printf("%d, ", kk);
        } while (++kk < 10);
        printf("\n");
    }

    {
        for (int jj = 0; jj < 10; jj++) {
            printf("%d, ", jj);
        }
        printf("\n");
    }

    // Corpo vazio:
    for (int i = 0; i <= 5; i++) {
        ; // declaração nula
    }
    for (int i = 0; i <= 5; i++); // também compila, mas cuidado: é fácil errar

    // switch
    {
        int alguma_expressao_integral = 1;
        switch (alguma_expressao_integral) {
        case 0:
            puts("zero");
            break;
        case 1:
            puts("um");
            break;
        default:
            fputs("erro!\n", stderr);
            exit(EXIT_FAILURE);
        }
    }

    // ---------------------------------------------------------
    // Casts
    // ---------------------------------------------------------
    {
        int x_hex = 0x01;
        printf("%d %d %d\n", x_hex, (short)x_hex, (char)x_hex);

        // Overflow silencioso:
        printf("%u\n", (unsigned)(unsigned char)257u); // 1 (se char=8 bits)

        printf("%f\n", (float)100);
        printf("%lf\n", (double)100);
        printf("%d\n", (int)(char)100.0);
    }

    // ---------------------------------------------------------
    // Ponteiros
    // ---------------------------------------------------------
    {
        int x = 0;
        printf("%p\n", (void *)&x);

        int *px = &x, nao_eh_um_ponteiro = 0;
        printf("%p\n", (void *)px);
        printf("sizeof(px)=%zu sizeof(nao_eh_um_ponteiro)=%zu\n",
               sizeof px, sizeof nao_eh_um_ponteiro);

        printf("%d\n", *px); // 0
        (*px)++;             // incrementa x via ponteiro
        printf("%d %d\n", *px, x); // 1 1

        // Arrays e aritmética de ponteiros
        int x_array[20];
        for (int xx = 0; xx < 20; xx++) x_array[xx] = 20 - xx;

        int *x_ptr = x_array; // decaimento para ponteiro
        printf("%d %d\n", *(x_ptr + 1), x_array[1]); // 19 19

        // Pega o endereço do array (tipo é "ponteiro para array")
        int arr[10];
        int (*ptr_to_arr)[10] = &arr;
        (void)ptr_to_arr;

        // sizeof em arrays NÃO decai:
        printf("sizeof arr=%zu sizeof ptr=%zu\n", sizeof arr, sizeof x_ptr);

        // Alocação dinâmica (heap)
        int *my_ptr = (int *)malloc(sizeof *my_ptr * 20);
        if (!my_ptr) { perror("malloc"); exit(EXIT_FAILURE); }
        for (int xx = 0; xx < 20; xx++) my_ptr[xx] = 20 - xx;

        // NÃO acesse fora dos limites: comportamento indefinido
        // printf("%d\n", my_ptr[21]); // exemplo propositalmente comentado

        free(my_ptr);

        // Literais de string são imutáveis -> use const char *
        const char *my_str = "Esta e a minha literal string";
        printf("%c\n", *my_str); // 'E'

        // String em memória mutável:
        char foo[] = "foo";
        foo[0] = 'a'; // ok -> "aoo"
    }

    funcao_1(); // demo

    return 0;
}


// -------------------------------------------------------------
// Funções
// -------------------------------------------------------------

int soma_dois_ints(int x1, int x2) {
    return x1 + x2;
}

/*
Funções são chamadas por valor. Array/strings "decaem" para ponteiro no call.
Para modificar o argumento do chamador, receba um ponteiro.
*/

void str_reverse(char *str_in) {
    if (!str_in) return;
    size_t len = strlen(str_in);
    for (size_t i = 0; i < len / 2; i++) {
        char tmp = str_in[i];
        str_in[i] = str_in[len - i - 1];
        str_in[len - i - 1] = tmp;
    }
}

/*
Exemplo de uso:
  char c[] = "Isto e um teste.";
  str_reverse(c);
  printf("%s\n", c); // ".etset mu e otsI"
*/

// Variáveis externas e linkage
int g_contador_publico = 0; // ligação externa por padrão

void testFunc(void) {
    extern int g_contador_publico; // refere-se à global
    g_contador_publico++;
}

// Torne símbolos "privados" ao arquivo com static
static int s_privado = 0; // não visível fora deste .c
static void helper_interno(void) { s_privado++; }


// -------------------------------------------------------------
// Tipos definidos pelo usuário e structs
// -------------------------------------------------------------

typedef int meu_tipo;
meu_tipo var_meu_tipo = 0;

struct retangulo {
    int altura;
    int largura;
};

void funcao_1(void) {
    struct retangulo meu_retan = { .altura = 10, .largura = 20 };

    // Acesso por .
    meu_retan.altura = 30;

    // Ponteiro para struct e ->
    struct retangulo *p = &meu_retan;
    p->largura = 10;

    printf("area=%d\n", p->largura * p->altura);
}

// typedef para struct por conveniência
typedef struct retangulo retan;

// Passagem por valor (copia):
int area_val(retan r) { return r.largura * r.altura; }

// Passagem por ponteiro (sem cópia):
int area_ptr(const retan *r) { return r->largura * r->altura; }


// -------------------------------------------------------------
// Ponteiros para funções
// -------------------------------------------------------------

typedef void (*minha_funcao_type)(char *);

static void minusculo_exemplo(char *s) {
    if (!s) return;
    for (char *p = s; *p; ++p) if (*p >= 'A' && *p <= 'Z') *p += 32;
}

static void reverso_exemplo(char *s) { str_reverse(s); }

static void usar_fp(minha_funcao_type f, char *s) { if (f) f(s); }


// -------------------------------------------------------------
// Caractere especiais e printf
// -------------------------------------------------------------
/*
'\a' '\n' '\t' '\v' '\f' '\r' '\b' '\0' '\\' '\?' '\'' '\"' '\xhh' '\ooo'

Formatos printf:
  %d %u %ld %zu %f %lf %s %c %p %x %o %%
*/


// -------------------------------------------------------------
// Ordem de avaliação e precedência (resumo)
// -------------------------------------------------------------
/*
() [] -> .                         esquerda->direita
! ~ ++ -- + - * (type) sizeof      direita->esquerda
* / %                               esquerda->direita
+ -                                  esquerda->direita
<< >>                                esquerda->direita
< <= > >=                            esquerda->direita
== !=                                esquerda->direita
&                                    esquerda->direita
^                                    esquerda->direita
|                                    esquerda->direita
&&                                   esquerda->direita
||                                   esquerda->direita
?:                                   direita->esquerda
= += -= *= /= %= &= ^= |= <<= >>=    direita->esquerda
,                                    esquerda->direita
*/


// -------------------------------------------------------------
// PRÉ-PROCESSADOR, MACROS e BOAS PRÁTICAS
// -------------------------------------------------------------
/*
Dicas cruciais:
  - Prefira 'const', 'static inline' e funções a macros sempre que possível.
  - Macros precisam de parênteses extras para evitar bugs:
      #define SQ(x) ((x) * (x))
  - Use #if/#ifdef para seleção de plataforma; evite lógica complexa no pré-processador.
  - Use header guards em .h:

      // em arquivo.h
      #ifndef ARQUIVO_H
      #define ARQUIVO_H
      // declarações
      #endif

  - Em headers, exponha apenas o necessário (protótipos e tipos opacos).
*/


// -------------------------------------------------------------
// ARMAZENAMENTO, VIDA ÚTIL e LIGAÇÃO (storage duration & linkage)
// -------------------------------------------------------------
/*
  - auto (padrão): variável local, vida no bloco.
  - static local: vida do programa, visível só no bloco/arquivo.
  - extern: declara símbolo definido em outro lugar.
  - const: somente leitura (o compilador pode otimizar).
  - volatile: diz ao compilador que o valor pode mudar fora do controle
              (memória mapeada, registradores, sinal).
*/


// -------------------------------------------------------------
// MEMÓRIA e ERROS COMUNS (segredos para dominar mais rápido)
// -------------------------------------------------------------
/*
  1) Sempre inicialize ponteiros; cheque retorno de malloc/calloc/realloc.
  2) Emparelhe alocação/liberação no mesmo nível de abstração.
  3) Prefira 'calloc' para zero-inicializar buffers:
       int *v = calloc(n, sizeof *v);
  4) Evite 'realloc' sem temporário:
       int *tmp = realloc(p, novo * sizeof *p);
       if (tmp) p = tmp; else trate erro.
  5) Nunca use memória após 'free' (use-after-free) e nunca leia além dos limites.
  6) Evite UB: ponteiros inválidos, overflow de inteiro assinado, shift inválido,
     desreferenciar NULL, múltiplos frees, violar strict-aliasing.
  7) Ative sanitizers e warnings agressivos (linha de compilação no topo).
  8) Teste com Valgrind (Linux) e com Address/UB Sanitizer.
  9) Em APIs, documente ownership: quem aloca, quem libera.
 10) Regra prática: se a função retorna um ponteiro "novo", o chamador libera;
     se retorna ponteiro para "dentro" de algo recebido, quem recebeu é dono.
*/


// -------------------------------------------------------------
// I/O ROBUSTA e PARSE SEGURO
// -------------------------------------------------------------
/*
  - Use fgets + strtoul/strtol/strtod com checagem de errno para ler números.
  - printf/scanf: prefira scanf com especificadores limitados (%99s) e verifique
    o valor de retorno (quantidade de itens lidos).
  - fprintf(stderr, ...) para erros; retorne códigos consistentes.
*/


// -------------------------------------------------------------
// MÓDULOS, BUILD e ORGANIZAÇÃO
// -------------------------------------------------------------
/*
 Arquitetura básica de projeto C:
   src/
     modulo.c
     outro.c
   include/
     modulo.h
     outro.h
   tests/
     test_modulo.c
   Makefile

 Exemplo mínimo de Makefile:

 CC ?= clang
 CFLAGS := -std=c11 -O2 -Wall -Wextra -Wpedantic
 SAN    := -fsanitize=address,undefined -fno-omit-frame-pointer
 INC    := -Iinclude
 SRC    := $(wildcard src/*.c)
 OBJ    := $(SRC:.c=.o)

 all: app
 app: $(OBJ)
	$(CC) $(CFLAGS) $(SAN) $^ -o $@

 %.o: %.c
	$(CC) $(CFLAGS) $(SAN) $(INC) -c $< -o $@

 clean:
	rm -f src/*.o app

 .PHONY: all clean
*/


// -------------------------------------------------------------
// CONJUNTOS DE BITS, UNIONS, BIT-FIELDS
// -------------------------------------------------------------
/*
  - Use enums + máscaras para 'flags':
      enum perms { P_READ=1<<0, P_WRITE=1<<1, P_EXEC=1<<2 };
      unsigned p = P_READ | P_WRITE;
      if (p & P_WRITE) { ... }

  - unions compartilham a mesma área de memória entre membros; use com cuidado.
  - bit-fields economizam bits, mas têm portabilidade limitada (ordem/empacotamento).
*/


// -------------------------------------------------------------
// CONST-CORRECTNESS, INLINE, RESTRICTION (restrict) e ALIASING
// -------------------------------------------------------------
/*
  - Marque ponteiros que não serão modificados como 'const T *p'.
  - 'restrict' (C99) indica que ponteiros não se sobrepõem -> permite otimização:
      void saxpy(size_t n, float a, const float *restrict x,
                 float *restrict y) { ... }
  - Strict aliasing: compilar com -fno-strict-aliasing evita alguns UB, mas o ideal
    é respeitar as regras (não reinterpretar objetos via ponteiros de tipos
    incompatíveis; use memcpy).
  - 'static inline' em headers para funções pequenas evita macros perigosas.
*/


// -------------------------------------------------------------
// ERROS ESPECÍFICOS DE C QUE UM DEV SÊNIOR EVITA (atalhos de maestria)
// -------------------------------------------------------------
/*
  - Esquecer 'break' em switch (a menos que deseje 'fallthrough' e documente).
  - Comparar ponteiro com inteiro; converter ponteiros sem motivos.
  - Retornar endereço de variável local (dangling pointer).
  - Esquecer 'const' em literais de string e tentar modificá-las.
  - Misturar API que aloca em um módulo e libera em outro sem convenção clara.
  - Não verificar retorno de funções de I/O e conversão (errno).
  - Usar %d para size_t/ptrdiff_t: prefira %zu/%td.
  - Depender do tamanho de tipos (int, long) sem usar <stdint.h>.
*/


// -------------------------------------------------------------
// LEITURA ADICIONAL
// -------------------------------------------------------------
/*
  - K&R: "The C Programming Language" — clássico; contextualize com práticas modernas.
  - comp.lang.c FAQ — histórico de armadilhas e respostas.
  - Linux kernel coding style — clareza e consistência.
  - C11/C17 padrão (n1570 é rascunho público útil para consulta).

Referência sobre padding de struct e sizeof:
  Por que sizeof(struct) != soma dos seus membros?
  Resposta curta: alinhamento e preenchimento (padding) para performance/ABI.
  [Pesquise por "struct padding alignment C" para detalhes.]
*/


// -------------------------------------------------------------
// EXEMPLOS RÁPIDOS DE TESTES (micro-unidade sem framework)
// -------------------------------------------------------------
#ifdef DEMO_TESTS
static int tests_passed = 0;

static void assert_int_eq(int got, int want, const char *msg) {
    if (got != want) {
        fprintf(stderr, "FAIL: %s (got=%d want=%d)\n", msg, got, want);
        exit(EXIT_FAILURE);
    }
    tests_passed++;
}

static void run_tests(void) {
    assert_int_eq(soma_dois_ints(2,3), 5, "soma 2+3");
    char s[] = "abc";
    str_reverse(s);
    assert_int_eq(strcmp(s,"cba")==0, 1, "reverse");
    printf("OK: %d tests\n", tests_passed);
}
#endif

```
