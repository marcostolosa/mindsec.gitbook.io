# rax2

### Visão geral

`rax2` é um conversor/avaliador: **bases numéricas**, **expressões**, **bytes⇄hex**, **strings⇄hex**. Excelente para preparar materiais (ex.: gerar IV/Key em hex).

### Conversões base↔base

```bash
rax2 10         # decimal -> hex (0xa)
rax2 0x65       # hex -> decimal (101)
rax2 -b 2 1337  # decimal -> binário
rax2 -b 16 1337 # decimal -> hex
```

### Avaliar expressões

```bash
rax2 "65+35*2*345"
rax2 -b 10 "65+35*2*345"      # força base 10
rax2 -k  "0x65+0x35*2*0x345"  # resolve mantendo formato hex
```

### Bytes ⇄ Hex

```bash
# Bytes -> hexstring (útil p/ IV do AES)
rax2 -S < IV

# Hexstring -> bytes crus
echo -n "414243" | rax2 -s > bytes.bin   # "ABC"
```

### Strings ⇄ Hex

```bash
rax2 -S "mindsecurity"       # string -> hex
echo -n "mindsecurity" | xxd -p
```

### Dicas

* `-S` trata entrada como **string** e cospe **hex**.
* `-s` faz o inverso: **hex** para **bytes**.
