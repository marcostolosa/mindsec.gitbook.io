# Reversing

1. Strings&#x20;

`-n`: localiza e mostra qualquer sequencia com N caracteres

```bash
strings -n 3 ./binário
```

2. readelf&#x20;

`-a`: equivalent to: -h -l -S -s -r -d -V -A -I

```bash
readelf -a ./binário
```

3. objdump

`-d`: disassemble

`-Mintel`: sintaxe intel -> assembly

```bash
objdump -dM intel ./binário 
```
